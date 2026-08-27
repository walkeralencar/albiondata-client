# Release History Analysis: 0.1.0–0.1.55

## Scope and Method

This analysis covers every repository tag beginning at `0.1.0` and ending at `0.1.55`, inclusive. It was reconstructed from local Git data using:

- tag names sorted with Git's version-aware ordering;
- each tag's peeled commit, commit date, subject, and ancestry;
- non-merge commits between adjacent release boundaries where the graph permits it;
- file-level diffs for vague commit messages;
- the source tree at the end of the requested range.

The resulting [Changelog](CHANGELOG.md) summarizes user-visible and maintainer-relevant behavior. It intentionally omits merge-only noise, formatting-only details, and claims that cannot be supported by the repository.

Dates are commit author dates in ISO `YYYY-MM-DD` format. These can differ from commit dates recorded by the integrator and from GitHub release publication dates. All tags in this range are lightweight tags (`git cat-file -t` reports `commit`), so there are no annotated tag messages to use as release notes.

## Tag Inventory and Anomalies

There are 55 actual tags in scope:

- `0.1.0` through `0.1.15`;
- `0.1.20` through `0.1.46`;
- `0.1.46.1`, `0.1.46.2`, and `0.1.46.3`;
- `0.1.47` through `0.1.55`.

The following irregularities matter when generating comparisons or automating releases:

| Tags | Finding | Consequence |
| --- | --- | --- |
| `0.1.10`, `0.1.11` | Both resolve to commit `9e9b5bf`. | The latter contains no source delta. |
| `0.1.16`–`0.1.19` | No matching tags exist. | Tools must not assume a contiguous patch sequence. |
| `0.1.15`, `0.1.20` | `0.1.20` is not a descendant of `0.1.15`; its commit date is also earlier. | A simple `0.1.15..0.1.20` log does not represent a conventional release delta. Analyze reachable commits and the later merge into `0.1.21`. |
| `0.1.30`, `0.1.31` | Both resolve to commit `fe0cbb1`. | The latter contains no source delta. |
| `0.1.46.1`–`0.1.46.3` | Four-component versions were used for same-day macOS build corrections. | Strict semantic-version parsers may reject these identifiers because SemVer permits only three numeric core components. |

These tags should not be renamed or moved: they are already part of the updater and release history. Future tooling should accommodate them as historical exceptions.

## Development Eras

### 1. Proof-of-work ingest and market reliability (`0.1.0`–`0.1.9`)

The `0.1.x` line began by moving ingestion from direct NATS toward an HTTP proof-of-work gateway and migrating dependencies to Go modules. Subsequent releases followed Albion protocol changes closely: Black Market orders, gold averages, market history, and operation tables all required repeated repairs.

This era also established the current proof-of-work CPU policy. Version `0.1.3` limited CPU use to roughly one quarter of available cores, while `0.1.5` added `-no-limit` as an opt-out. Version `0.1.7` expanded personalized market-mail processing and began centralizing configuration.

### 2. Organization migration and realm awareness (`0.1.10`–`0.1.29`)

Late 2022 releases were dominated by gold, market-offer, and market-history protocol movement. Project update checks migrated from the former repository owner to the `ao-data` organization. GitHub Actions replaced or supplemented older build automation.

The client then became realm-aware. It started reading packet source IPs, attaching a server ID to proof-of-work submissions, and routing uploads to West or East endpoints. macOS version injection and builds received focused repairs. Market-history correlation capacity grew, proof-of-work moved behind a reverse proxy, and trace logging was introduced.

### 3. Large protocol tables and three-realm routing (`0.1.30`–`0.1.39`)

Version `0.1.30` contained a major operation/event table refresh and build-system update. Join/location semantics were corrected in `0.1.32`, while Crystal Raiders and the March/April 2024 game updates caused further mapping changes.

Version `0.1.35` extended routing to the Europe realm, completing the West/East/Europe model visible in current code. The following releases addressed ingest endpoint races, encrypted market-data messaging, and per-upload identifiers. Version `0.1.39` moved identifiers from the serialized payload into proof-of-work form parameters, an externally significant transport change.

### 4. Protocol upkeep and location generalization (`0.1.40`–`0.1.46`)

Gold data, Smugglers' Dens, cluster-map responses, market offers, and operation/event tables continued to track game updates. Linux interface filtering and HTTPS ingest support were added.

The largest data-model change in this era was `0.1.45`, which converted location IDs to strings. This allowed special and non-numeric location identifiers but widened an external JSON contract. The same release collected market-history cache work, proof-of-work optimization, installation documentation, and cross-platform interface experiments.

### 5. Updater, new events, and parser ownership (`0.1.46.1`–`0.1.55`)

Three immediate follow-up tags repaired macOS CI and packaging after `0.1.46`. Version `0.1.47` modernized automatic updates and added macOS menu-bar behavior. Versions `0.1.48` and `0.1.49` introduced and stabilized bandit-event ingestion.

The most important architectural transition arrived in `0.1.51`: Photon parsing moved into `client/photon`, replacing the vendored Photon Spectator path. Join mapping needed a follow-up repair in `0.1.52`. Two later protocol table updates, festivities ingestion, and the CRC/coalesced-packet fixes in `0.1.55` completed the requested range.

## Compatibility Timeline

| Concern | Important releases |
| --- | --- |
| HTTP proof-of-work ingestion | Introduced `0.1.0`; CPU limit `0.1.3`; opt-out `0.1.5`; reverse proxy `0.1.28`; optimized `0.1.45`. |
| Realm routing | Source-IP groundwork `0.1.24`; server ID `0.1.25`; East `0.1.29`; Europe/all three realms `0.1.35`; race fix `0.1.37`. |
| Market data | Frequent fixes in `0.1.1`–`0.1.14`; correlation cache `0.1.27`; large-response fix `0.1.33`; string locations and lookup reliability `0.1.45`. |
| Protocol parsing | Repeated enum updates throughout; in-repository parser `0.1.51`; join follow-up `0.1.52`; CRC/coalesced UDP support `0.1.55`. |
| Event coverage | Bandit events `0.1.48`–`0.1.49`; festivities `0.1.54`. |
| Build and distribution | Go modules `0.1.0`; macOS ZIP `0.1.1`; GitHub Actions around `0.1.20`; macOS fixes `0.1.24`, `0.1.26`, and `0.1.46.1`–`.3`. |
| Automatic updates | Repository migration `0.1.15`/`0.1.20`/`0.1.21`; macOS version fix `0.1.24`; updater overhaul `0.1.47`. |

## Lessons for Future Releases

1. **Generate notes from the graph, not tag names alone.** Duplicate tags, missing numbers, and the `0.1.15`/`0.1.20` divergence make numeric assumptions unsafe.
2. **Use three-component semantic versions going forward.** Represent emergency follow-ups as the next patch version instead of a fourth numeric component.
3. **Prefer annotated, signed tags.** They can carry release metadata and provide stronger provenance than the lightweight tags used in this range.
4. **Record protocol evidence.** Most releases repair numeric mappings after game updates. A sanitized fixture and mapping source should accompany each protocol release.
5. **Call out payload and routing compatibility.** Location type changes, identifier placement, privacy classification, realm IDs, and topic changes affect downstream consumers even when the desktop UI does not change.
6. **Separate protocol releases from build-only releases when practical.** This makes rollback and diagnosis easier across platforms.
7. **Run tests before tagging.** Release CI in this range primarily builds artifacts; parser, handler, formatting, and race checks should become explicit gates.

## Recommended Changelog Process

For a future release `X.Y.Z`:

1. Confirm the previous release is an ancestor with `git merge-base --is-ancestor`.
2. Review both merge and non-merge commits in the range.
3. Inspect diffs for commits whose subjects only say “update” or similar.
4. Categorize notes as Added, Changed, Fixed, Removed, Security, or Deprecated.
5. Highlight protocol versions, operation/event mappings, public/private payload contracts, endpoints, flags, and platform requirements.
6. Add the release to `docs/CHANGELOG.md` before creating an annotated tag.
7. Verify release asset names remain compatible with the updater.
