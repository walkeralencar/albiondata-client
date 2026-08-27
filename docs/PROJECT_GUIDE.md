# Albion Data Client Project Guide

## Purpose and Boundaries

Albion Data Client is the desktop capture component of the Albion Online Data Project. It observes local TCP and UDP traffic on the Albion game port, parses Photon protocol messages, extracts selected market and world data, and publishes JSON payloads to configured ingest services.

The client is passive: it reads packets but does not modify, inject, delay, or replay traffic into the game. That boundary is both an architectural rule and an important operational safety constraint.

This guide describes the repository as analyzed in August 2026. The game protocol changes over time, so operation and event mappings should always be treated as version-sensitive.

## System Overview

```text
Albion Online traffic / offline capture
                  |
                  v
       libpcap listener(s)
                  |
                  v
       Photon parser/deserializer
                  |
        request / response / event
                  |
                  v
       typed decode and validation
                  |
                  v
        Router + shared Albion state
                  |
                  v
 operation/event Process implementations
                  |
          JSON payload + topic
                  |
       +----------+-----------+-------------+
       |                      |             |
 HTTP / HTTP+PoW             NATS      local WebSocket
 public and private targets             (opt-in, :8099)
```

### Startup lifecycle

1. Package initialization calls `client.ConfigGlobal.SetupFlags()`. Configuration and command-line parsing therefore happen before `main`.
2. `main` prints the version when requested, otherwise starts the GitHub release updater for non-development builds.
3. The systray and client are scheduled differently on macOS because Cocoa requires its event loop on the main thread.
4. `client.Client.Run` configures debug filters, creates the optional WebSocket dispatcher, and selects online capture or offline replay.
5. Online mode discovers physical interfaces, starts a router, and creates one listener per interface for port `5056`.
6. Each listener applies a TCP/UDP BPF filter, extracts transport payloads, and gives them to the Photon parser.

## Main Components

### Entry point and updater

`albiondata-client.go` owns process startup and self-update behavior. Release builds inject `main.version` through linker flags. A non-empty version without `dev` enables hourly GitHub update checks. Successful updates replace the Unix process with `syscall.Exec`; Windows starts a replacement process and exits.

Changes here are cross-platform and can affect terminal ownership, systray behavior, and the updater. Verify at least compilation for each affected operating system.

### Configuration and logging

`client/config.go` defines the global runtime configuration. Most options are command-line flags. `config.yaml` currently controls only the optional WebSocket server and its allowed origins, using Viper and the keys shown in `config.yaml.example`.

Important flags include:

| Flag | Meaning |
| --- | --- |
| `-version` | Print the linked version and exit. |
| `-d` | Disable all remote upload. |
| `-l` | Capture only the listed interfaces. Names are platform-dependent. |
| `-o` | Replay a `.pcap` or recorded `.gob` file instead of capturing live traffic. |
| `-record` | Record raw Photon payloads to a gob file. |
| `-i` | Set comma-separated public ingest targets. |
| `-p` | Set comma-separated private ingest targets. |
| `-debug`, `-trace` | Increase log verbosity. |
| `-events`, `-events-ignore` | Filter event debug output by numeric ID. |
| `-operations`, `-operations-ignore` | Filter operation debug output by numeric ID. |
| `-ignore-decode-errors` | Suppress decoding-error debug output. |
| `-no-limit` | Do not restrict Go CPU use for proof-of-work. |

The process writes colored output to the terminal and uncolored output to `albiondata-client.log`. Startup rotates up to ten numbered backups. These logs may contain gameplay metadata and should not be committed.

### Capture and process watching

`client/albion_watcher.go` discovers capture interfaces through platform-specific implementations and maintains listeners. `client/listener.go` opens libpcap handles and applies `tcp port 5056 || udp port 5056`.

For every IPv4 packet, the listener:

1. updates the detected game-server IP and derives the Albion realm;
2. extracts UDP or TCP application bytes;
3. optionally records the raw Photon payload;
4. passes the bytes to the parser;
5. receives a request, response, or event callback from the parser.

The current realm mapping in `client/albion_state.go` recognizes West, East, and Europe IP prefixes and selects the corresponding proof-of-work ingest endpoint. Changes to game infrastructure can invalidate these mappings without producing a compile-time error.

### Photon parsing

`client/photon/parser.go` and `client/photon/deserializer.go` are the lowest-level protocol layer. They handle Photon framing and convert serialized values into `map[byte]interface{}` parameter maps. The parser supports operation requests, operation responses, and events, including reliable, CRC-enabled, and coalesced payload paths covered by focused tests.

Parser code handles untrusted network bytes. It must check lengths before slicing, accept valid packet variants, and fail without panicking on truncated or unknown data.

### Typed decoding

`client/decode.go` resolves numeric codes, normalizes byte-swapped values seen after protocol changes, selects a concrete Go type, and decodes numeric parameter keys with `mapstructure`.

The main dispatch functions are:

- `decodeRequest` for client-to-server operations;
- `decodeResponse` for server-to-client operation responses;
- `decodeEvent` for server-pushed events.

Concrete structs use numeric `mapstructure` tags because the Photon payload is keyed by byte values. Some request data is cached in `albionState` and combined with a later response; market history is the clearest example.

`client/operations.go` and `client/events.go` are long `iota` enumerations mirroring Albion's numeric protocol tables. Their position is their value. Reordering one line can corrupt every mapping after it. Generated `String` methods live in `operationtype_string.go` and `eventtype_string.go`.

### Router and state

`client/router.go` owns:

- a shared `albionState`;
- a buffered typed-operation channel;
- a buffered recording channel;
- lifecycle signaling.

Each decoded operation or event implements:

```go
type operation interface {
    Process(state *albionState)
}
```

The router launches `Process` in a new goroutine. This keeps parsing responsive but means all processors can access shared state concurrently. Existing state includes location and character identity, realm selection, request/response correlation data, and timestamps used to throttle some event uploads. New shared state should be synchronized or redesigned to avoid races.

### Payload contracts

`lib/` contains exported payload types and NATS-compatible topic constants. The public topics currently include market orders, market histories, gold prices, map data, bandit events, and festivities. Private topics include skill data and market notifications.

JSON tags deliberately use established property casing such as `ItemTypeId`, `LocationId`, and `UnitPriceSilver`. These are external contracts, not local style choices. A schema change can break the deduper, API, website, and third-party subscribers.

Public handlers call `sendMsgToPublicUploaders`; personalized handlers call `sendMsgToPrivateUploaders`. Public payloads go to both configured public and private destinations. Private payloads go only to private destinations. When WebSockets are enabled, either class can also be emitted locally, so origin restrictions in `config.yaml` matter.

### Upload transports

Target URL prefixes select an uploader at runtime:

| Prefix | Behavior |
| --- | --- |
| `http://`, `https://` | JSON POST to `baseURL/topic`. |
| `http+pow://`, `https+pow://` | Fetch a proof-of-work challenge, solve it, and submit the payload and realm ID. |
| `nats://` | Publish the JSON body on the topic. |

The default public target is a special `https+pow://albion-online-data.com` placeholder. Before upload it is replaced with a realm-specific endpoint from `albionState`.

Proof-of-work limits `GOMAXPROCS` to approximately 25 percent of detected CPU cores unless `-no-limit` is set. Transport changes should include timeouts, response-body cleanup, status handling, and tests with local test servers rather than real infrastructure.

### Optional WebSocket server

When `EnableWebsockets` is true in `config.yaml`, the client starts an HTTP server on port `8099` and serves `/ws`. Messages have a `topic` and `data` envelope. `AllowedWebsocketHosts` controls accepted origins.

This server is disabled when the configuration file cannot be read. Because it may expose personalized data to local browser clients, do not weaken origin checks or enable it by default without an explicit security review.

### Offline replay

Offline mode is the preferred protocol-development loop:

- `.pcap` files are read through libpcap and exercise capture-to-parser behavior;
- `.gob` files contain recorded `photon.RawPacket` values and bypass link/IP extraction.

Supplying `-o` enables offline mode and normally disables upload. Use `-d` explicitly when investigating unfamiliar captures. Offline processing currently waits ten seconds for asynchronous work and then exits.

Capture files can contain sensitive network and character data. Keep them outside version control unless they are intentionally minimized, anonymized, and approved as test fixtures.

## Building and Testing

### Prerequisites

- Go compatible with the module declaration; CI currently uses Go 1.25.
- A C toolchain because packet capture uses CGO.
- libpcap development headers on Linux (`libpcap-dev` on Debian-derived systems).
- `goimports` for repository formatting commands.
- Platform packaging tools only when producing release assets: NSIS and `go-winres` for Windows, and the relevant macOS toolchain for Darwin.

The checked-in `vendor/` directory may not contain all packages referenced by `vendor/modules.txt`. Go automatically selects vendor mode for this module, so a clean developer command can fail with missing vendored packages. Until vendoring is deliberately repaired, run module-backed checks with:

```bash
GOFLAGS=-mod=mod go test ./...
```

Do not commit a wholesale vendor refresh as a side effect of unrelated work.

### Common workflow

```bash
# Run a development build.
make run

# Format all non-vendor Go packages.
make fmt

# Verify formatting.
make validate-fmt

# Run all tests (libpcap headers required).
GOFLAGS=-mod=mod go test ./...

# Exercise decoding without remote upload.
go run -ldflags="-w -s -X main.version=dev" \
  albiondata-client.go -d -debug -o path/to/capture.pcap
```

Live Linux capture generally requires root or capabilities such as `cap_net_raw` and `cap_net_admin`. Avoid elevating the whole development environment when offline replay is sufficient.

### Release builds

The Makefile delegates to scripts for Linux, Windows, and Darwin. Release workflows trigger when a GitHub release is created and upload platform-specific compressed updater files and installers. Asset naming is coupled to the updater's `update-` prefix and must remain stable unless the updater and release consumers are changed together.

The release scripts perform destructive cleanup of known generated artifacts. Review them before local use, especially on a working tree with uncommitted build outputs.

## How to Add Protocol Support

### Add an operation or event

1. Establish the numeric ID and packet direction from a current, reproducible capture.
2. Update the correct position in `operations.go` or `events.go`. Review the resulting numeric values around the change.
3. Regenerate or mechanically update the corresponding string mapping.
4. Define a typed request, response, or event struct with numeric `mapstructure` tags.
5. Register it in `decodeRequest`, `decodeResponse`, or `decodeEvent`.
6. Implement `Process`, separating state updates, validation, payload creation, and publishing.
7. Define or reuse a payload in `lib/`; preserve existing topic and JSON conventions.
8. Decide explicitly whether the payload is public or personalized.
9. Generate a stable identifier where required by proof-of-work ingest deduplication.
10. Add decode tests, processing/upload tests, and malformed-payload tests.

When an operation request supplies context absent from its response, cache only the minimum correlation data and document how response lookup works. Account for wraparound and concurrency.

### Add an uploader

1. Implement the small `uploader` interface.
2. Add an unambiguous target prefix in `createUploaders`.
3. Make construction failures explicit; avoid storing an unusable connection.
4. Bound network calls with timeouts and release all resources.
5. Preserve topic, realm ID, identifier, and privacy semantics.
6. Test with an in-process fake or test server.

### Change a payload

Before changing a `lib` type, identify every downstream consumer. Prefer additive fields over renaming or changing types. Verify exact serialized JSON, including capitalization and zero-value behavior. If compatibility must break, document the migration and coordinate it across the related deduper, models, API, and website projects.

## Testing Strategy

The current automated tests are concentrated in Photon parsing, debug formatting, and festivities decoding/publishing. This leaves capture lifecycle, state races, most operation handlers, upload failures, configuration, and WebSocket security lightly covered.

For new work, prioritize:

- byte-level parser tests for framing changes;
- table-driven decoder tests for type variants and byte order;
- handler tests for valid, empty, truncated, and mismatched arrays;
- exact JSON and topic assertions;
- `httptest.Server` coverage for HTTP and proof-of-work behavior;
- race-detector runs for shared-state changes (`go test -race ./client`);
- compile checks for affected platform build tags.

Tests should be deterministic and must not depend on Albion Online, public ingest servers, GitHub releases, desktop notifications, or root access.

## Maintenance Risks and Improvement Areas

The following observations are useful when planning work; they are not instructions to refactor unrelated code:

1. **Protocol drift is the primary operational risk.** Long positional enums and heuristic byte-order normalization make updates fragile. Captured regression fixtures and explicit numeric mapping generation would reduce mistakes.
2. **Shared state is processed concurrently.** `Router.run` launches a goroutine per operation against one `albionState`. The current design can race under load; run the race detector when touching state and consider serialized state mutation in a dedicated change.
3. **Network clients lack consistent timeouts.** Several uploader paths can wait indefinitely, and NATS connection errors are discarded during construction. Reliability work should address these deliberately with tests.
4. **Uploader instances are recreated for each public message.** This prevents effective connection reuse and repeatedly establishes NATS connections. A lifecycle-managed uploader registry would be more efficient but requires careful configuration and shutdown design.
5. **Offline completion uses a fixed delay.** The ten-second sleep is not proof that all asynchronous processing finished. A wait group or drain protocol would make offline tests faster and deterministic.
6. **The vendor tree is inconsistent with module metadata.** Default `go test ./...` can fail before compilation. Dependency policy should eventually choose and enforce either a complete vendor tree or module mode.
7. **Build versions differ.** `go.mod` declares Go 1.24 while CI installs Go 1.25, and the general Dockerfile still uses an obsolete Go 1.16 image. Aligning these reduces local/CI surprises.
8. **Platform and release paths dominate CI.** Existing workflows build packages but do not run the unit test suite or formatting validation explicitly. Adding those checks would catch protocol regressions earlier.
9. **Configuration is split between flags and a small YAML surface.** Viper currently reads WebSocket settings only. Document and test any expansion rather than assuming all struct fields can be set in YAML.
10. **Some process exits bypass cleanup.** Offline mode calls `os.Exit`, and panic/fatal paths can skip graceful shutdown. Lifecycle refactoring should close listeners, files, WebSocket clients, and uploader connections explicitly.

## Documentation Rules

Use en-US English for all repository documentation and user-visible strings. Put detailed technical documentation in `docs/`; keep the root focused on conventional entry points. When behavior, flags, payload schemas, ports, release artifacts, or prerequisites change, update this guide and the user-facing `README.md` as appropriate.
