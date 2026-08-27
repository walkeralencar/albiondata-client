# Changelog

This changelog reconstructs the changes released from `0.1.0` through `0.1.55` from the repository's local Git tags, commit graph, commit messages, and file diffs. It is historical documentation rather than a verbatim copy of GitHub release descriptions.

Tags `0.1.16` through `0.1.19` do not exist. Tags `0.1.10` and `0.1.11` identify the same commit, as do `0.1.30` and `0.1.31`. See [Release History](RELEASE_HISTORY.md) for the tag-topology details and interpretation rules.

## 0.1.55 - 2026-08-14

### Fixed

- Corrected Photon parsing for CRC-enabled packets.
- Added support for coalesced UDP payloads containing multiple Photon commands.
- Expanded parser regression coverage for reliable operation requests and responses.

## 0.1.54 - 2026-08-08

### Added

- Added festivities event decoding, validation, public payload models, and the `festivities.ingest` topic.
- Added tests for festivities decoding, invalid payload rejection, and public ingest routing.

### Changed

- Updated operation and event code mappings for the July 27, 2026 game update.

## 0.1.53 - 2026-07-01

### Changed

- Updated operation and event code mappings for the July 1, 2026 game update.

## 0.1.52 - 2026-04-14

### Fixed

- Corrected `opJoin` recognition and parameter mapping after the Photon protocol changes.

## 0.1.51 - 2026-04-13

### Changed

- Replaced the vendored Photon Spectator decoding path with an in-repository Photon parser and deserializer.
- Updated listener and router integration for Photon protocol 1.8 traffic.
- Added return-code mapping and expanded reliable-packet tests.

## 0.1.50 - 2026-04-13

### Changed

- Added and refined the temporary user-facing notice for a game update that had broken client decoding.

## 0.1.49 - 2026-02-24

### Fixed

- Updated operation and event mappings.
- Revised bandit-event detection and submission logic after protocol changes.

## 0.1.48 - 2026-02-03

### Added

- Added red-zone world-map event processing and public bandit-event ingestion.
- Added upload identifier logging for bandit events.

### Changed

- Reverted an experimental listener/router goroutine startup change.

## 0.1.47 - 2026-02-01

### Added

- Introduced automatic GitHub release updates using the newer updater package.
- Added a macOS menu-bar icon with a log-view action.

### Changed

- Preserved the active terminal when restarting after an update on Linux and macOS.
- Updated dependencies and vendor metadata.
- Kept updater repository configuration fixed rather than live-configurable.

## 0.1.46.3 - 2026-01-31

### Fixed

- Adjusted the macOS release runner and packaging workflow again to restore successful builds.

## 0.1.46.2 - 2026-01-31

### Fixed

- Forced the macOS release build to target AMD64.

## 0.1.46.1 - 2026-01-31

### Changed

- Updated GitHub Actions workflows as the first macOS runner compatibility correction.

## 0.1.46 - 2025-11-24

### Changed

- Updated operation and event code mappings for the November 24, 2025 game update.

## 0.1.45 - 2025-10-04

### Changed

- Converted location IDs from numeric values to strings, enabling non-numeric locations while preserving existing payload field names.
- Improved market-history request/response cache lookup reliability.
- Optimized proof-of-work random candidate generation and solving.
- Added Linux installation guidance and corrected related paths.

### Fixed

- Added, then reverted, the Linux interface-selection implementation after cross-platform issues; the release retains the revert.
- Corrected Windows and Unix interface-discovery build errors encountered during that work.
- Corrected a proof-of-work error message.

## 0.1.44 - 2025-07-15

### Added

- Added HTTPS ingest target support and documented it in command-line help.

### Fixed

- Corrected the `AuctionGetOffers` operation structure.

## 0.1.43 - 2025-06-30

### Changed

- Updated operation and event code mappings.
- Updated cluster-map response decoding.

### Added

- Added GNU/Linux packet-capture interface filtering.

### Removed

- Consolidated duplicated Unix network-interface filtering code.

## 0.1.42 - 2025-02-03

### Changed

- Updated operation and event handling for the Smugglers' Dens game update.
- Updated GitHub Actions upload dependencies.
- Added support for local packet-capture testing during protocol work.

## 0.1.41 - 2024-10-28

### Fixed

- Updated gold-market information parsing for the October 2024 game protocol.

## 0.1.40 - 2024-07-22

### Fixed

- Updated the operation code used to receive gold-market data.

### Changed

- Adjusted the test-build workflow.

## 0.1.39 - 2024-05-29

### Changed

- Moved the upload identifier out of the serialized NATS-compatible message and into proof-of-work request parameters.

## 0.1.38 - 2024-05-21

### Added

- Added a user-facing warning when requested market data is encrypted.
- Added a unique identifier to each supported upload.
- Added Docker and Compose development build files.

### Changed

- Refined realm and status messages and updated dependency/vendor metadata.

## 0.1.37 - 2024-05-09

### Fixed

- Prevented a race in which `AODataIngestBaseURL` could remain empty while selecting an upload destination.

## 0.1.36 - 2024-04-15

### Changed

- Updated operation mappings for the April 15, 2024 game update.
- Updated Windows resource metadata and vendor cache contents.

## 0.1.35 - 2024-03-28

### Added

- Added automatic detection and ingest routing for all three Albion realms, including the Europe/Amsterdam infrastructure.

### Changed

- Updated proof-of-work uploader behavior for realm-specific routing.

## 0.1.34 - 2024-03-11

### Changed

- Updated operation/event mappings and join handling for the March 2024 game changes.

### Fixed

- Excluded WSL2 interfaces from packet-capture interface selection.

## 0.1.33 - 2024-01-08

### Changed

- Updated protocol mappings for the Crystal Raiders release.

### Fixed

- Prevented oversized market-history responses from breaking processing.

## 0.1.32 - 2023-10-23

### Fixed

- Corrected `opJoin` location handling and stopped deriving the location from cluster-change operations.
- Restored guild ID decoding in join responses.

## 0.1.31 - 2023-10-16

- This tag points to the same commit as `0.1.30`; no additional source changes are present.

## 0.1.30 - 2023-10-16

### Changed

- Performed a large operation/event table update for the October 16, 2023 game release.
- Updated Go modules and platform build scripts.

### Fixed

- Patched Linux artifacts to avoid a hard runtime dependency on the versioned `libpcap.so.0.8` name.

## 0.1.29 - 2023-03-20

### Added

- Added Albion East realm detection and automatic East/West ingest endpoint selection.

## 0.1.28 - 2023-03-11

### Changed

- Switched proof-of-work ingestion to the reverse-proxy endpoint instead of a random-port service.
- Added trace-level logging.

### Fixed

- Restored special handling for the second Caerleon market location.

## 0.1.27 - 2023-02-27

### Fixed

- Increased the market-history correlation cache to reduce request/response lookup loss.

## 0.1.26 - 2023-02-25

### Added

- Added a cross-platform test-build workflow.

### Fixed

- Corrected code and workflow settings needed to build the macOS client.

## 0.1.25 - 2023-02-25

### Added

- Included the detected server ID in proof-of-work submissions.
- Added a versioned `User-Agent` to proof-of-work requests.

### Changed

- Included HTTP response bodies in proof-of-work error logs.

### Fixed

- Reverted an unsuccessful location-update fix.

## 0.1.24 - 2023-02-22

### Added

- Began deriving the active game realm from packet source IP addresses.
- Added safeguards that stopped proof-of-work uploads when connected to the then-unsupported Asia realm.

### Fixed

- Corrected macOS version injection so automatic updates could work.
- Reset realm state on join and rejected packets without an IPv4 source.

## 0.1.23 - 2023-02-15

### Fixed

- Rolled back an incompatible `golang.org/x/sys` dependency change.

## 0.1.22 - 2023-02-15

### Fixed

- Updated average market-item statistics decoding after a protocol change.

## 0.1.21 - 2023-02-13

### Fixed

- Updated join-response field mapping after a parameter changed from map ID to guild name.

### Changed

- Updated project ownership, archive/status messaging, README content, and download-stat links.
- Moved update checks to the `ao-data` organization repository.

## 0.1.20 - 2022-12-27

### Changed

- Added GitHub Actions builders for Linux, Windows, and macOS.
- Moved update checks toward the `ao-data` organization repository.
- Included the late-2022 gold, market-offer, and market-history protocol fixes on this release line.

### Repository note

- This tag is on a different history line from `0.1.15`; it is not a descendant of that tag. The changes above are based on the commit reachable from `0.1.20`, not on a synthetic `0.1.15..0.1.20` comparison.

## 0.1.16 through 0.1.19

- No Git tags with these names exist in the repository.

## 0.1.15 - 2023-01-02

### Changed

- Moved automatic update checks to the client repository under the `ao-data` GitHub organization.

## 0.1.14 - 2022-12-12

### Fixed

- Corrected market-history decoding.
- Corrected the gold-market operation code again after upstream protocol movement.

## 0.1.13 - 2022-11-22

### Fixed

- Updated market offers, market history, and gold-price decoding for the November 2022 protocol.

## 0.1.12 - 2022-09-29

### Changed

- Updated the Go version used by the CircleCI deployment stage.

## 0.1.11 - 2022-09-29

- This tag points to the same commit as `0.1.10`; no additional source changes are present.

## 0.1.10 - 2022-09-29

### Fixed

- Updated gold-price decoding for the September 2022 game protocol.

## 0.1.9 - 2022-06-12

### Fixed

- Updated market-history handling for the Into the Fray game update.

## 0.1.8 - 2022-06-08

### Fixed

- Updated the gold-market operation code for the June 2022 game update.

## 0.1.7 - 2022-03-12

### Added

- Added `GetMailInfos` processing and improved sold/expired market-mail notifications.
- Added location and expiration information to market notifications.

### Changed

- Began consolidating runtime configuration in `client/config.go`.
- Corrected the market sales-tax calculation to three percent.
- Closed HTTP response bodies so connections could be reused.

### Fixed

- Corrected operation/event debug filter initialization order and sell-notification processing.

## 0.1.6 - 2022-01-26

### Changed

- Updated operation codes and regenerated their string mappings for the January 25, 2022 game update.

## 0.1.5 - 2021-11-30

### Added

- Added the `-no-limit` option to allow proof-of-work to use all available CPU cores.

## 0.1.4 - 2021-11-24

### Changed

- Updated operation codes for the November 24, 2021 game update.

## 0.1.3 - 2021-10-20

### Fixed

- Repositioned an unknown operation code to restore auction item-history decoding.

### Changed

- Limited proof-of-work CPU use to approximately 25 percent of available cores by default.

## 0.1.2 - 2021-09-22

### Fixed

- Corrected gold average-price decoding.

## 0.1.1 - 2021-09-21

### Fixed

- Allowed Black Market orders to be processed.

### Changed

- Improved macOS packaging and run instructions, including a ZIP distribution.

## 0.1.0 - 2021-06-18

### Added

- Introduced HTTP proof-of-work ingestion: challenge retrieval, local solution, and combined proof/payload submission.
- Added `http+pow://` target detection.

### Changed

- Migrated dependency management to Go modules.
- Shifted the default ingest path from direct NATS toward the HTTP gateway.

### Fixed

- Improved proof-of-work and ingest error handling and corrected success logging.
- Corrected Linux network-interface compilation after the module migration.

