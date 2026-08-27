# Repository Guidelines

## Language

- Use en-US English for source code, comments, identifiers, commit messages, documentation, logs, and user-facing text, even when a request is written in another language.
- Preserve protocol field names, external API names, and proper nouns exactly when compatibility requires it.

## Project Summary

This repository contains the Go client for the Albion Online Data Project. It passively captures Albion Online traffic, decodes Photon operations and events, builds typed payloads, and forwards public or personalized data to HTTP, proof-of-work HTTP, NATS, and optionally local WebSocket consumers.

The project uses Go modules (`go 1.24` in `go.mod`), CGO, and libpcap. Release CI currently builds with Go 1.25. Do not modify or inject game traffic; this client is intentionally read-only at the network boundary.

Read `docs/PROJECT_GUIDE.md` before making architectural, protocol, uploader, or release changes.

## Repository Map

- `albiondata-client.go`: process entry point, background updater, platform-specific systray scheduling.
- `client/`: capture, Photon decoding, state, routing, operations/events, uploaders, offline replay, and WebSocket support.
- `client/photon/`: low-level Photon packet parser and deserializer.
- `lib/`: JSON payload models and public/private topic constants. Treat exported types and JSON tags as downstream contracts.
- `log/`, `notification/`, `systray/`, `icon/`: platform integration.
- `scripts/`, `Makefile`, `pkg/nsis/`, `winres/`: local and release builds.
- `.github/workflows/`: release and cross-platform build automation.
- `thirdparty/`: checked-in Windows packet-capture build assets; avoid editing these files.

## Development Workflow

1. Inspect the complete data path affected by a change. Protocol work commonly spans a code constant, decoder dispatch, an operation/event struct, `Process`, a `lib` payload, and a topic.
2. Make the smallest compatible change. Albion operation and event IDs are order-sensitive `iota` enumerations; inserting, deleting, or reordering entries shifts every following value.
3. Add focused table-driven tests where practical. Use captured byte fixtures or synthetic parameter maps for protocol changes; never require live game traffic or a live ingest service in unit tests.
4. Format with `make fmt` (requires `goimports`).
5. Run `go test ./...`. On Linux, install `libpcap-dev` first. If the checked-in `vendor/` tree is incomplete, use `GOFLAGS=-mod=mod`; do not silently regenerate or commit vendor contents unless the task explicitly includes dependency vendoring.
6. Run `make validate-fmt` before handing off changes. For platform-specific work, build or test the affected target and state any target that could not be verified locally.

Useful commands:

```bash
make run
make fmt
make validate-fmt
go test ./...
go test ./client -run TestName
go run -ldflags="-w -s -X main.version=dev" albiondata-client.go -h
```

Live capture requires libpcap and elevated packet-capture capability. Prefer offline replay with `-o sample.pcap` or `-o sample.gob` during development. Pass `-d` to disable uploads and `-record path.gob` to record decoded raw Photon payloads. Packet captures, recordings, logs, generated binaries, and local `config.yaml` files must remain uncommitted.

## Code Conventions

- Follow standard Go style and existing package naming. Keep platform files behind the established `_win.go`, `_nix.go`, and `_darw.go` patterns.
- Keep capture and parsing separate from business processing. Decode wire parameters into typed operation/event structs, then perform validation, state updates, and publishing in `Process`.
- Access runtime behavior through `ConfigGlobal` unless a deliberate configuration refactor is in scope.
- Preserve wire compatibility: numeric `mapstructure` tags, JSON property casing, topic strings, identifiers, and public/private routing are contracts.
- Validate parameter lengths and types before indexing or converting. Packet formats change frequently and malformed traffic must not panic the client.
- Be concurrency-aware. `Router.run` invokes each `Process` in its own goroutine while operations share `albionState`; avoid introducing unsynchronized shared mutation. Do not block packet capture on network I/O.
- Close response bodies and capture handles, handle uploader connection errors, and avoid logging private payload contents or character identifiers unnecessarily.
- Do not introduce network writes outside configured ingest targets, the updater, or the opt-in local WebSocket server.

## Protocol Change Checklist

- Confirm the current operation/event number from a reproducible capture or trusted upstream evidence.
- Update `operations.go` or `events.go` without accidentally shifting unrelated constants.
- Update generated string mappings (`operationtype_string.go` or `eventtype_string.go`) when constants change.
- Register the type in `decodeRequest`, `decodeResponse`, or `decodeEvent`.
- Add or update the typed handler and its `Process` implementation.
- Verify payload validation, location/server state, identifier generation, privacy classification, topic, and JSON schema.
- Test decoding plus publishing behavior, including malformed or incomplete input.
- Document externally visible payload or configuration changes.

## Scope and Safety

- Preserve unrelated working-tree changes and generated/local artifacts.
- Do not edit generated string files by guesswork; use the generation workflow or make a mechanically verified update.
- Do not change release asset names, updater prefixes, GitHub owner/repository defaults, ingest endpoints, server-IP mappings, or the default port (`5056`) without tracing all consumers.
- Do not make public data private or private data public accidentally. Personalized uploads belong only on configured private targets and the opt-in local WebSocket feed.
- Keep documentation in `docs/` except for conventional root files such as `README.md`, `AGENTS.md`, `LICENSE`, and contribution metadata.

