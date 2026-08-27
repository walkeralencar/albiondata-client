# Fyne GUI Analysis and Adoption Plan

## Executive Decision

Fyne is a good architectural fit for Albion Data Client if it is introduced as a presentation layer over a new UI-neutral application state boundary. It keeps the desktop stack in Go, provides native desktop windows and system-tray support, avoids a Node/WebView frontend toolchain, and offers headless UI testing utilities.

Do not transplant the Wails dashboard from release `0.1.56`, and do not reproduce its direct dashboard calls throughout capture and upload code. Build a Fyne-specific interface on the current `0.1.55`-based branch, selectively adopting only the non-GUI backend improvements described in [0.1.56 Analysis](0.1.56_ANALYSIS.md).

The recommended target is Fyne `v2.8.0`, pinned exactly after a short proof of concept. As of August 27, 2026, `v2.8.0` is the current Fyne feature release. It includes full Wayland support and follows the single UI-goroutine model introduced in `v2.6`. Pinning avoids unreproducible `@latest` builds. Verify patch releases again immediately before implementation because desktop libraries change quickly. [Fyne releases](https://github.com/fyne-io/fyne/releases)

## Why Fyne Fits This Repository

### Advantages

- **One primary language:** the GUI, client, updater, and business logic remain Go code.
- **No embedded browser frontend:** no Node, npm, Svelte, WebView2, WebKitGTK, or generated JavaScript bindings are required.
- **Built-in desktop lifecycle:** Fyne provides application windows, preferences, notifications, lifecycle callbacks, and a blocking application event loop. [Fyne App API](https://docs.fyne.io/api/v2/fyne/app/)
- **Built-in system tray:** Fyne supports desktop tray menus and the hide-on-close pattern, allowing replacement of the separate `getlantern/systray` loop. [Fyne system tray guide](https://docs.fyne.io/explore/systray/)
- **Headless tests:** `fyne.io/fyne/v2/test` provides in-memory apps, windows, canvases, input simulation, and render assertions. [Fyne test package](https://docs.fyne.io/api/v2/test/package/)
- **Existing CGO context:** the project already requires CGO and libpcap, so adding native graphics compilation is an extension of the build environment rather than introducing CGO for the first time.
- **Cross-platform packaging tools:** Fyne's tooling can create Windows resources, macOS bundles, and Linux packages from metadata and a PNG icon. [Fyne desktop packaging](https://docs.fyne.io/started/packaging/)

### Costs and Constraints

- Linux development and CI require graphics headers in addition to libpcap, including Mesa/X11 and, for current Fyne, Wayland dependencies. [Fyne prerequisites](https://docs.fyne.io/started/quick/)
- Cross-compiling native graphics code requires target C toolchains. Fyne recommends `fyne-cross` as one option, but the existing release scripts and updater asset contracts still need repository-specific integration. [Fyne cross-compilation](https://docs.fyne.io/started/cross-compiling/)
- The application currently has multiple lifecycle shortcuts: `os.Exit`, a blocking client run loop, platform-specific systray loops, and updater process replacement. These must be coordinated before the GUI becomes the process owner.
- The current router processes operations concurrently against shared state. A GUI must not observe `albionState` directly or amplify its existing race risk.
- Rendering every log line or upload synchronously would overwhelm the UI goroutine during bursts.
- Linux tray availability depends on the desktop environment. Tray behavior must degrade gracefully to a normal window.

## Current Architecture Gaps

The present entry point starts the client and systray differently by operating system. Fyne instead expects one application event loop called near the end of `main`, where it blocks until quit. The first/last-window behavior and explicit `Quit` semantics are part of its application contract. [Fyne App API](https://docs.fyne.io/api/v2/fyne/app/)

Before building views, the project needs:

1. a cancellable application supervisor;
2. a UI-neutral state and event boundary;
3. explicit capture lifecycle and error reporting;
4. a bounded log/event stream;
5. graceful updater and quit coordination;
6. a decision to replace, rather than run alongside, the current systray implementation.

These changes improve the CLI client as well and should not import Fyne.

## Recommended Architecture

```text
                    +--------------------------+
                    |      Fyne main loop      |
                    | window / tray / dialogs  |
                    +------------+-------------+
                                 |
                     snapshots + typed commands
                                 |
                    +------------v-------------+
                    |    Application service   |
                    | supervisor / UI state /  |
                    | bounded event broker     |
                    +-----+-------------+------+
                          |             |
                 lifecycle commands    observations
                          |             |
               +----------v--+     +----v----------------+
               | Client/capture|   | log/status adapters |
               +------+-------+   +---------------------+
                      |
       Photon -> decode -> router -> Process -> ingest
```

### Package boundaries

Recommended packages:

| Package | Purpose | May import Fyne? |
| --- | --- | --- |
| `internal/appstate` | Immutable status snapshots, typed events, bounded subscriptions | No |
| `internal/supervisor` | Client, updater, quit, restart, and capture lifecycle | No |
| `internal/ui` | Fyne application composition and windows | Yes |
| `internal/ui/viewmodel` | Coalescing and formatting UI-neutral state for views | Prefer no |
| `internal/ui/theme` | Fyne colors, icons, sizes, and theme variants | Yes |
| `internal/ui/views` | Dashboard, logs, settings, about/help | Yes |

Do not place Fyne imports in `client/`, `client/photon/`, `lib/`, or uploader implementations. Existing packages should publish small typed events through interfaces defined outside the GUI.

### UI-neutral snapshot

A first snapshot can contain:

```go
type Snapshot struct {
    Phase             Phase
    CaptureStatus     CaptureStatus
    ActiveInterfaces  []InterfaceStatus
    Realm             Realm
    MarketEncrypted   bool
    UploadEnabled     bool
    PublicTargetCount int
    PrivateTargetSet  bool
    Counters          Counters
    LastError         string
    Version           string
}
```

Use immutable copies and replace the complete snapshot atomically or behind one owner goroutine. Avoid pointers to `albionState`, listener handles, uploader instances, or payload bodies.

### Commands from the GUI

Keep the initial command surface deliberately small:

- show/hide the main window;
- open the log file or documentation URL;
- start a controlled retry after a recoverable capture failure;
- request application quit;
- request an updater check if the updater supports it safely.

Do not let the GUI mutate protocol mappings, inject packets, resend captures to the game, or edit ingest targets until configuration validation and secret handling are designed.

## Threading and Backpressure

Since Fyne `v2.6`, callbacks run on one UI goroutine. Code running in application-created goroutines must schedule Fyne API calls with `fyne.Do` or `fyne.DoAndWait`. The client is heavily concurrent, so this rule is central to the design. [Fyne goroutine guide](https://docs.fyne.io/started/goroutines/) [Fyne v2.6 upgrade guide](https://docs.fyne.io/api/v2.6/upgrading.html)

Use these rules:

- capture, parsing, proof-of-work, uploads, updates, and file I/O remain background work;
- background code publishes UI-neutral events without importing Fyne;
- one GUI adapter coalesces events and calls `fyne.Do` at a bounded refresh rate, initially 4–10 updates per second;
- counters update from deltas or snapshots, not one scheduled callback per message;
- logs use a bounded ring buffer and batch append/refresh operations;
- never hold an application-state lock while waiting in `fyne.DoAndWait`;
- UI callbacks enqueue commands and return quickly.

This architecture prevents packet capture from waiting for rendering and prevents an event burst from starving user input.

## Window, Tray, and Quit Lifecycle

Create the app with a stable reverse-domain ID so Fyne preferences and metadata have a stable namespace. Fyne preferences require a unique application ID. [Fyne preferences guide](https://docs.fyne.io/explore/preferences/)

Recommended behavior:

1. Start the Fyne application on the process main goroutine on every desktop platform.
2. Start the client supervisor in a background goroutine after UI state subscribers are ready.
3. Create one long-lived main window.
4. Use `SetCloseIntercept` to hide the window when tray mode is available; expose a clear “Quit Albion Data Client” action.
5. Use Fyne's `desktop.App` tray capability and remove the competing `getlantern/systray` runtime after feature parity.
6. On explicit quit, cancel the supervisor, stop listeners, drain/close resources with a deadline, then call `App.Quit`.
7. Before updater process replacement, request GUI shutdown and release platform resources. Preserve existing updater asset names.
8. If tray initialization is unavailable on Linux, keep the window openable and make closing behavior unambiguous.

Do not use `os.Exit` for normal GUI shutdown because it bypasses deferred cleanup and Fyne lifecycle callbacks.

## Proposed User Experience

### Minimum viable GUI

The first releasable interface should answer four questions without exposing private data:

1. Is the client running?
2. Is packet capture active, and on which interfaces?
3. Which Albion realm was detected, and is market data encrypted?
4. Are uploads enabled, succeeding, or failing?

Recommended first views:

- **Overview:** capture status, realm, encryption warning, upload status, version, and last error.
- **Activity:** bounded counters by public topic; no raw personalized payloads.
- **Logs:** bounded recent log lines, filter by level, copy, and open full log file.
- **Settings:** only safe local UI preferences initially—theme, open on launch, minimize to tray, and log verbosity.
- **Help/About:** project disclaimer, version, official documentation links, capture-driver guidance, and update status.

### Accessibility and privacy

- Use text labels with icons instead of color-only status.
- Preserve keyboard traversal and meaningful focus order.
- Test light/dark themes and system scaling.
- Keep warnings actionable and concise.
- Redact character identifiers and private endpoints from default logs and screenshots.
- Do not show personalized payload contents unless a future explicit diagnostic mode receives Relay review.

## Packaging and Build Impact

Fyne currently requires Go, a C compiler, and graphics development headers for development. End users generally do not need those development packages. [Fyne quick start](https://docs.fyne.io/started/quick/)

### Linux

- Add Mesa, X11, Wayland, and keyboard/common headers to CI builders in addition to `libpcap-dev`.
- Test X11 and Wayland sessions; current Fyne `v2.8` targets full Wayland support.
- Decide whether the updater continues shipping a raw compressed binary or moves to a desktop package. Preserve the existing raw updater artifact until migration is proven.

### Windows

- Use MinGW/CGO settings compatible with both Fyne graphics and gopacket/libpcap.
- Reconcile Fyne metadata/icon embedding with the existing `go-winres` and NSIS pipeline; avoid embedding competing resources twice.
- Combine the work with the Npcap migration only after each can be tested independently.
- Preserve `update-windows-amd64.exe.gz` and installer naming expected by the updater.

### macOS

- Build the Fyne event loop on the main thread; this replaces the current special-case systray scheduling.
- Produce a proper `.app` bundle and plan signing/notarization separately from basic compilation.
- Verify Intel AMD64 compatibility because historical release assets target AMD64.
- Preserve the raw updater executable inside the bundle only after defining how self-update safely replaces bundled content.

Fyne's `fyne package` can create desktop bundles and embed metadata, but repository release scripts remain the authority for asset naming and updater compatibility. [Fyne desktop packaging](https://docs.fyne.io/started/packaging/)

## Testing Strategy

### Headless tests

Use Fyne's test package for:

- view construction without a graphics display;
- button, menu, keyboard, and close-intercept behavior;
- binding and snapshot rendering;
- theme/layout markup checks where stable;
- preference-backed behavior with temporary apps;
- accessibility labels and enabled/disabled state.

Keep domain presentation logic in plain Go view models so most formatting, filtering, redaction, and state-transition tests do not need Fyne.

### Concurrency tests

- publish status and log bursts while windows show/hide;
- verify bounded queues drop or coalesce predictably;
- run `go test -race` across `internal/appstate`, `internal/supervisor`, and the client integration;
- ensure no client goroutine calls Fyne APIs directly;
- test quit during capture startup, proof-of-work, updater checks, and interface reconciliation.

### Platform smoke tests

- launch, show, hide, tray restore, and quit on Windows, macOS, Linux X11, and Linux Wayland;
- verify icons, scaling, window persistence, notifications, and external links;
- verify capture permissions and missing-driver messages;
- inspect release artifacts and updater restarts;
- run offline replay with uploads disabled while observing UI counters and logs.

## Squad Changes

A dedicated [Canvas agent](../.agents/canvas.md) now owns Fyne lifecycle, views, bindings, tray behavior, accessibility, and headless UI tests. This responsibility is distinct from Harbor's native platform/build ownership.

| Work | Lead | Required review |
| --- | --- | --- |
| Application supervisor and state/event boundary | Atlas | Canvas, Gauge, Harbor |
| Fyne proof of concept and view architecture | Canvas | Atlas, Gauge |
| Payload/status privacy classification | Relay | Canvas, Gauge |
| Fyne windows, widgets, theme, tray | Canvas | Gauge, Scribe |
| Native build, icons, packaging, signing | Harbor | Canvas, Gauge, Scribe |
| Headless and concurrency tests | Gauge | Canvas and affected owner |
| User documentation and release notes | Scribe | Canvas and Harbor |

Cipher does not need Fyne-specific responsibilities. Protocol code must remain GUI-independent.

## Phased Adoption Plan

### Phase 0 — Prove toolchain viability

- Pin Fyne `v2.8.0` on a short-lived proof-of-concept change.
- Build a window, tray menu, hide/show behavior, and headless test.
- Compile on all three desktop targets without modifying capture code.
- Compare artifact sizes and startup behavior.
- Decide the stable application ID and metadata source.

**Exit:** reproducible Windows, macOS, and Linux builds plus a headless CI test.

### Phase 1 — Build the UI-neutral foundation

- Introduce the supervisor, immutable snapshots, typed commands, bounded events, and log hook.
- Make listener shutdown and explicit quit deterministic.
- Implement selected P0 capture fixes from the `0.1.56` analysis.
- Test the foundation without Fyne.

**Exit:** CLI behavior remains functional; status and lifecycle tests pass under the race detector.

### Phase 2 — Deliver the read-only MVP

- Implement overview, bounded logs, activity counters, help/about, tray, and explicit quit.
- Add coalesced `fyne.Do` updates and headless view tests.
- Keep configuration editing and private payload inspection out of scope.

**Exit:** the GUI observes the client without changing capture, protocol, or ingest behavior.

### Phase 3 — Package and harden

- Update platform builders, icons, bundles, NSIS, signing/notarization plans, and release workflows.
- Run platform smoke tests and updater compatibility tests.
- Add Npcap guidance and dynamic interface reconciliation as separately reviewed backend work.

**Exit:** release artifacts install, capture, update, and quit correctly on supported platforms.

### Phase 4 — Optional controlled settings

- Add validated settings only after configuration ownership is redesigned.
- Separate Fyne UI preferences from client configuration.
- Require Relay review for ingest-target UI and secret handling.

**Exit:** settings are validated, persisted deliberately, and covered by migration tests.

## Risks and Mitigations

| Risk | Mitigation |
| --- | --- |
| UI calls from client goroutines | One GUI adapter; enforce `fyne.Do`; concurrency review and tests. |
| Shared-state races | Immutable snapshots and one state owner; no direct `albionState` access. |
| Event/log flood | Bounded buffers, batching, coalescing, and fixed refresh rate. |
| Competing tray/event loops | Replace the old systray after feature parity; never run both. |
| Shutdown skips cleanup | Supervisor cancellation and deadline; reserve `os.Exit` for unrecoverable paths. |
| Build complexity | Phase 0 cross-platform proof before application integration. |
| Updater breaks packaged apps | Preserve artifact names and test bundle-aware replacement before release. |
| Linux desktop variation | Test X11 and Wayland; degrade gracefully without tray. |
| Private data exposure | UI-neutral redacted state; Relay approval for any personalized display. |
| Dependency churn | Pin a reviewed Fyne version and update intentionally with release notes/tests. |

## Definition of Done

Fyne adoption is ready for release only when:

- the client remains passive and its payload/topic contracts are unchanged;
- Fyne is isolated to the UI and composition layers;
- all background UI changes use the supported scheduling model;
- capture, uploads, proof-of-work, and updates never block the UI goroutine;
- explicit quit closes listeners and resources cleanly;
- the old tray integration is removed or inactive;
- headless UI, lifecycle, concurrency, and existing protocol tests pass;
- Windows, macOS, Linux X11, and Linux Wayland smoke tests are recorded;
- installer and updater artifact compatibility is verified;
- user-facing behavior and prerequisites are documented.

## Final Recommendation

Proceed with Fyne, but start with Phase 0 rather than changing production code immediately. The key design decision is not the widget toolkit; it is the UI-neutral supervisor and immutable state boundary. Once that foundation exists, Fyne can provide the desired dashboard with substantially less build and architectural overhead than the Wails approach while keeping capture, decoding, and ingest logic independent and testable.
