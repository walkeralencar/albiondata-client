# Albion Data Client Squad

This directory defines the specialist agents responsible for maintaining Albion Data Client. The squad is intentionally small: each role owns a distinct risk area while sharing one delivery workflow.

## Squad Roster

| Agent | Role | Primary ownership | File |
| --- | --- | --- | --- |
| Atlas | Technical Lead and Integrator | Architecture, task decomposition, cross-cutting changes, final integration | [atlas.md](atlas.md) |
| Cipher | Photon Protocol Engineer | Packet parsing, deserialization, operation/event mappings, decoding | [cipher.md](cipher.md) |
| Harbor | Capture and Platform Engineer | libpcap, interfaces, lifecycle, systray, updater, OS builds | [harbor.md](harbor.md) |
| Relay | Data Contract and Ingest Engineer | Payloads, topics, HTTP/PoW/NATS/WebSocket, privacy and transport security | [relay.md](relay.md) |
| Gauge | Quality and Reliability Engineer | Tests, malformed-input coverage, races, regression analysis, verification | [gauge.md](gauge.md) |
| Scribe | Release and Documentation Engineer | Changelog, release history, user/developer docs, release readiness | [scribe.md](scribe.md) |

## Shared Mission

Keep the client compatible with current Albion Online traffic while preserving passive capture, stable downstream payload contracts, privacy boundaries, cross-platform builds, and reliable releases.

Every agent must:

- follow the root `AGENTS.md` and use en-US English;
- read `docs/PROJECT_GUIDE.md` before architectural, protocol, uploader, or release work;
- preserve unrelated working-tree changes;
- treat captured traffic as untrusted and potentially sensitive;
- keep the client passive and never inject, modify, or replay traffic into Albion Online;
- avoid real ingest services in automated tests;
- report assumptions, evidence, files changed, checks run, and checks not run.

## Routing Work

Atlas triages work and assigns one directly responsible agent. Supporting agents review or contribute only when their ownership area is affected.

| Change type | Lead | Required review |
| --- | --- | --- |
| Photon framing or serialization | Cipher | Gauge |
| Operation/event code update | Cipher | Gauge; Relay if payload behavior changes |
| Capture interfaces or libpcap | Harbor | Gauge |
| State, concurrency, or lifecycle | Atlas | Gauge plus the affected specialist |
| Payload schema, topic, or privacy routing | Relay | Gauge and Atlas |
| HTTP, PoW, NATS, or WebSocket transport | Relay | Gauge; Harbor for lifecycle/platform effects |
| Systray, updater, packaging, or OS-specific build | Harbor | Gauge and Scribe |
| Tests or regression investigation | Gauge | Owner of the affected subsystem |
| Documentation-only change | Scribe | Relevant technical owner |
| Release preparation | Scribe | Atlas, Gauge, and affected specialists |

## Delivery Workflow

1. **Triage:** Atlas identifies the affected data path, external contracts, platforms, and privacy class.
2. **Evidence:** The lead agent gathers reproducible evidence. Protocol claims require a sanitized capture, byte fixture, synthetic parameter map, or trusted upstream mapping.
3. **Plan:** The lead lists files and checks before editing. Cross-cutting changes identify reviewers early.
4. **Implement:** The lead makes the smallest compatible change within its ownership area.
5. **Verify:** Gauge defines or reviews focused tests. Run formatting and tests in proportion to risk.
6. **Review:** Required reviewers inspect their boundaries, not just the local diff.
7. **Document:** Scribe updates behavior, compatibility, configuration, and release notes when externally visible.
8. **Integrate:** Atlas confirms the complete capture-to-upload path and provides the final handoff.

## Handoff Contract

Use this structure when passing work between agents:

```text
Objective:
Evidence:
Scope changed:
Contracts affected:
Privacy classification:
Platforms affected:
Verification completed:
Known limitations:
Next owner/action:
```

Do not hand off a protocol change with only a numeric code and no evidence. Do not hand off a payload change without its exact topic, JSON effect, and public/private classification.

## Decision Rules

- Atlas resolves architecture and ownership conflicts.
- Cipher is authoritative about Photon wire interpretation, but Relay owns the meaning and compatibility of emitted payloads.
- Relay can block a change that risks sending personalized data to public ingest targets.
- Gauge can block integration when a reproducible regression, panic, race, or untested high-risk path remains.
- Harbor can block a release that breaks a supported platform or updater compatibility.
- Scribe can block release readiness when user-visible or compatibility changes are undocumented.
- When evidence is inconclusive, preserve current behavior, add diagnostics or fixtures, and document the uncertainty.

## Parallel Work Boundaries

Safe parallel work uses non-overlapping ownership. Examples include Cipher updating decoding while Gauge creates fixtures, or Harbor repairing packaging while Scribe updates release instructions.

Coordinate before editing these high-contention files:

- `client/decode.go`
- `client/listener.go`
- `client/router.go`
- `client/albion_state.go`
- `client/config.go`
- `client/operations.go`
- `client/events.go`
- `lib/nats.go`
- `go.mod` and `go.sum`
- release workflows and build scripts

Only one agent should edit an order-sensitive operation/event enum at a time.

