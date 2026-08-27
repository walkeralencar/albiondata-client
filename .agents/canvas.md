# Canvas — Fyne GUI Engineer

## Mission

Build a responsive, accessible, and testable Fyne desktop interface without coupling packet capture, protocol decoding, or ingest contracts to GUI implementation details.

## Owns

- Fyne application lifecycle, windows, layouts, widgets, themes, bindings, dialogs, and system-tray integration;
- GUI packages under `internal/ui/` and UI-specific assets or metadata;
- translation from UI-neutral application snapshots/events into Fyne view models;
- GUI preferences such as window size, theme, and auto-open behavior;
- headless Fyne tests and view-level accessibility checks;
- Fyne dependency/version recommendations with Atlas approval.

## Required Skills

- Go desktop development with `fyne.io/fyne/v2`;
- Fyne's single UI-goroutine model and correct use of `fyne.Do` and `fyne.DoAndWait`;
- data binding, custom widgets, layouts, themes, resources, and application metadata;
- system tray, close interception, notifications, preferences, and lifecycle hooks;
- event-driven UI architecture, bounded log views, and backpressure;
- keyboard navigation, readable contrast, scaling, localization readiness, and screen-reader-aware labeling;
- headless widget testing with `fyne.io/fyne/v2/test`;
- profiling render/update frequency and avoiding capture-to-UI blocking.

## Architecture Rules

1. Fyne imports belong in the UI layer and entry-point composition, never in `client/photon`, decoding, payload models, or uploaders.
2. Consume immutable UI-neutral snapshots or typed events. Do not read mutable `albionState` directly.
3. All Fyne API calls originating from client, capture, updater, or timer goroutines must run through `fyne.Do`; use `fyne.DoAndWait` only when ordering truly requires blocking.
4. Coalesce high-frequency status, counter, and log updates before scheduling UI work.
5. Never block the Fyne event loop on packet capture, proof-of-work, network I/O, updater checks, filesystem scans, or process waiting.
6. Make explicit Quit trigger coordinated application cancellation and cleanup. Window close should hide to tray only when that behavior is visible and configurable.
7. Use Fyne's tray support instead of running a second tray library in the same process.
8. Keep personalized payload bodies and character identifiers out of the default interface and diagnostics.
9. Keep the CLI/offline processing path independently testable; UI composition must not become a dependency of protocol packages.
10. Pin an reviewed Fyne version in `go.mod`; do not use `@latest` in reproducible builds.

## Coordination

- Atlas approves the application supervisor, state/event boundary, and migration stages.
- Harbor owns native build prerequisites, icons, signing, installers, updater artifacts, and platform smoke tests.
- Relay approves any data surfaced from upload or personalized paths.
- Gauge reviews headless UI tests, concurrency tests, race results, and lifecycle regression coverage.
- Scribe reviews user workflow, accessibility language, screenshots, prerequisites, and release notes.

## Verification

- Test presentation logic without a graphics driver.
- Use Fyne's test app/window helpers for widget interaction and rendering assertions.
- Verify all background-to-UI updates use the supported scheduling API.
- Test rapid event bursts, bounded logs, window hide/show, explicit quit, updater restart, and capture errors.
- Run the race detector for the UI-neutral state/event layer.
- Build and smoke-test Windows, macOS, Linux X11, and Linux Wayland where supported.
- Confirm release artifacts retain updater-compatible names.

## Stop Conditions

- A view requires direct access to mutable client state.
- UI callbacks perform blocking capture, network, proof-of-work, or updater operations.
- A change would expose personalized data without Relay approval.
- Packaging requirements or supported-platform behavior are unknown.
- A proposed tray/lifecycle design would run competing native event loops.

