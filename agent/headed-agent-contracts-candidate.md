---
title: Headed Agent contracts (candidate)
description: Candidate reusable bitty-ai contracts for a sibling Headed Agent plugin covering sessions streaming routing panels speech avatars budgets and platform fallback
category: specifications
audience: contributor
document_type: specification
status: draft
website_publish: false
sidebar_order: 72
---

# Headed Agent contracts (candidate)

> Status: **draft**, and it remains `draft`. This document records a candidate
> contract set for review. It accepts nothing, authorizes no implementation,
> changes no Wheel behavior, and does not claim that a Headed Agent plugin,
> speech implementation, avatar renderer, provider loader, or panel feature
> exists. Accepted inputs it must not weaken are the IPC and Agent RFC, the
> Panel Runtime RFC, RFC-0006, the scoped provider-routing snapshot binding
> inside the draft R1 disposition, the OQ-058 owner ruling, ADR-0013, ADR-0002,
> and the security corpus. R1's broader ownership and cancellation split, R2,
> R3, the provider boundary, the multimodal speech/job contract, and the AI
> Architecture remain draft inputs. The nine rows in
> [P0 Review Sign-off](#p0-review-sign-off) are the single authoritative review
> gate set for this document. Four independent reviews recorded `NEEDS-FIX`
> (PX-1550 through PX-1553); re-review is pending and no row is approved.

## Purpose and scope

This document defines the reusable `bitty-ai` contracts that a future sibling
Headed Agent plugin would consume. "Headed" describes presentation and human
interaction, never authority, persistence, or lifecycle. The proposed plugin is
a sibling of the Wheel Coding Agent Harness, not a Wheel module and not a Wheel
front end.

The document covers:

- The boundary between Bitty Core, `bitty-ai`, Wheel, the Headed Agent plugin,
  provider plugins, and optional media extensions.
- Independent contract versions for sessions, streaming, routing, snapshots,
  tool approval, panel topics, speech, avatars, budgets, and platform fallback.
- Agent session lifecycle operations (`start`, `attach`, `detach`, `stop`),
  generation fencing, cancellation, event ordering, backpressure, and
  reconnect.
- Provider and model routing, context and session snapshots, and tool-call and
  approval events.
- Panel and Event Bus topics, multi-panel suspend and resume, and the
  Core-versus-plugin ownership split.
- Speech synthesis request, streaming, and cancellation, plus avatar state,
  animation, and viseme events.
- CPU, memory, GPU, audio, token, cost, and storage budgets; Linux, macOS,
  Windows, and FreeBSD fallback; and security, privacy, and consent gates.
- An ownership matrix, open points, cross-repository references, and the
  acceptance conditions for future implementation work.

This document does not define, authorize, or describe as implemented:

- Any Wheel change, Wheel module, `wheel.*` dependency, or headed surface
  inside Wheel.
- Any provider configuration schema, provider loader, credential resolution, or
  vendor adapter.
- Any Core panel implementation, panel runtime code, event-bus implementation,
  renderer, speech engine, audio device layer, avatar rig, 3D engine, or media
  decoder.
- Any UI layout, chat transcript design, character design, voice persona, or
  plugin packaging decision.
- Any claim that the Headed Agent plugin ships, is installable, or has passed
  architecture or security review.

The design remains a candidate input. It references the accepted
[IPC and Agent RFC](../specifications/ipc-agent-rfc.md), the accepted
[Panel Runtime RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/panel-runtime-rfc.md)
with its plugin-facing
[RFC-0006](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/rfcs/RFC-0006-panel-provider-registration-mount.md)
companion, and the scoped accepted provider-routing snapshot binding maintained
inside the draft
[Execution ownership R1](../architecture/execution-ownership-r1.md#providerrouting-snapshot-binding-contract-accepted-2026-10-17)
without restating or modifying them. Panel identity, panel lifecycle, and the
host-mediated Event Bus are accepted by the Panel Runtime RFC and RFC-0006;
those contracts are consumed here, not redefined, and no implementation is
claimed. R1's wider ownership and cancellation material, R2, R3, the provider
boundary, and the multimodal speech contract are draft sources, not accepted
contracts. Linked documents remain authoritative for their own surfaces; this
document only records the additional contract families a headed consumer needs.

## Normative sources this specification must not weaken

This draft is subordinate to the accepted IPC and Agent RFC, the accepted Panel
Runtime RFC with RFC-0006, the scoped accepted provider-routing snapshot
binding, the accepted OQ-058 owner ruling and ADR-0013 identity model, ADR-0002
platform support tiers, and the normative security corpus. It also must not
misstate or weaken the draft dispositions it consumes. It replaces or relaxes
none of them:

- The accepted [IPC and Agent RFC](../specifications/ipc-agent-rfc.md) for
  framing, transport, authentication, scopes, bounded agent messages, consent,
  streaming, versioning, and failure semantics. No credential, transport,
  scope, or method in this document overrides that RFC.
- The accepted
  [Panel Runtime RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/panel-runtime-rfc.md)
  for the generic Panel container, the `Declared -> Created -> Mounted ->
Focused -> Suspended -> Disposed` lifecycle, `(PanelId, Generation)` identity,
  the host-mediated Event Bus, focus routing, bounded resources PR-1 through
  PR-12, and the `panel.*` capability family. Panel identity and the Event Bus
  are accepted contracts; this document claims neither is implemented.
- The accepted
  [RFC-0006](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/rfcs/RFC-0006-panel-provider-registration-mount.md)
  plugin-facing registration and mount contract for the registration triple,
  Panel-to-View placement, and the closed `panel.*` capability mapping. This
  document consumes it and mints no registration entry point.
- The accepted OQ-058 owner ruling (2026-09-23) recorded in the
  [open-question register](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/open-questions.md),
  adopting the SMO-1 through SMO-4 spatial-orchestration topology and keeping
  presentation non-authoritative.
- The accepted
  [ADR-0013](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/adrs/ADR-0013-core-ontology-identity.md)
  core ontology and identity model: `AgentId` is distinct from task, run,
  execution, panel, and workspace ids, `GenerationId` is the fencing generation
  for reloads, leases, and handoffs, and panels never own executions. This
  draft consumes that vocabulary; `GenerationId` fencing mechanics remain a
  follow-up under the ADR.
- The accepted
  [ADR-0002](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/adrs/ADR-0002-platform-support-tiers.md)
  platform support tiers (`Tier 1` supported, `Tier 2` best-effort, `Tier 3`
  community). Presentation classes in this draft are separate and never
  upgrade a platform's support tier.
- The normative
  [P0 acceptance criteria](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/p0-acceptance-criteria.md),
  including `P0-AC-005`, `P0-AC-009`, and `P0-AC-010` for any rich, avatar, or
  media presentation surface.
- The accepted
  [Isolation and Resource RFC](https://github.com/bitty-terminal/bitty-plugins-docs/blob/main/runtime/isolation-resource-rfc.md)
  for the isolation envelope, `RC-1`, `RC-2`, `RC-9`, and `RC-10`; this draft
  adds no weaker bound.
- The scoped accepted provider-routing snapshot binding in
  [Execution ownership R1](../architecture/execution-ownership-r1.md#providerrouting-snapshot-binding-contract-accepted-2026-10-17).
  Only that section is accepted. Core owns publication authority for the
  canonical projection, monotonic `snapshot_version`, and invalidation; the AI
  helper owns the authoritative registry and the opaque proof. R1's cancellation
  contract, execution-ownership disposition, and registry split remain a draft
  disposition; this document consumes that split without promoting it.
- The draft cancellation split in R1: pre-dispatch prevention and post-dispatch
  `Unknown` reconciliation, with no rollback and no exactly-once claim. The
  split is design input, not an accepted cancellation contract.
- The draft unified authorization gate order and fail-closed conditions in
  [Tool transport R2](../architecture/tool-transport-r2.md#unified-authorization-backend).
  A presentation surface never gains tool authority by holding a session, a
  panel, an attachment, or an approval widget.
- The draft consent-bounded retention and deletion rules in
  [Context retention R3](../architecture/context-retention-r3.md#consent-and-recording-contract).
  Audio, visemes, avatar state, and attachment snapshots are not exempt.
- The draft privacy, minimization, redaction, budget, failure-isolation, and
  agent-level requirements in
  [AI Architecture](../architecture/ai-architecture.md#privacy-first),
  including PP-1 through PP-6, FS-AI1 through FS-AI7, AG-1 through AG-5, and
  CP-5 through CP-7.
- The draft Core-owned privacy classes, provider descriptor, secret invariant,
  and project-override prohibition in
  [Provider plugin boundary](../providers/provider-plugin-boundary.md#core-owned-surface)
  and its [secret invariant](../providers/provider-plugin-boundary.md#secret-invariant).
  The provider boundary is not accepted.
- The draft speech task, generation-job lifecycle, `Unknown` reconciliation,
  asset reference, and presentation-handoff rules in
  [Multimodal inference boundary](../providers/multimodal-inference-boundary.md#generationjob-lifecycle).
  This document reuses that draft lifecycle and does not create a second one.
- The normative security corpus in `bitty-docs`. Its entry point is the
  [security index](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/README.md);
  the [threat model](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/threat-model.md)
  is normative. If this draft and a normative requirement disagree, the
  normative requirement wins.

Citing a draft source here records a dependency and a constraint; it does not
accept that source, promote its status, or authorize its implementation.

## Terminology

| Term                  | Meaning in this document                                                                                                                                                                                                                                                                                                        |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Core                  | The terminal platform and its host services. Core is the designated owner of record for panel runtime, IPC transport, consent enforcement, process supervision, and event-bus transport. Panel identity, lifecycle, and the Event Bus are accepted by the Panel Runtime RFC; their implementation is not claimed by this draft. |
| `bittyd`              | A future, undecided Core-side service context in which host-owned lifecycle, panel, and transport authority may run. No v1.0 `bittyd` binary is claimed to exist.                                                                                                                                                               |
| `bitty-ai`            | The independent AI sub-platform that owns provider abstraction, routing, inference jobs, context compilation, and the AI-facing event vocabulary.                                                                                                                                                                               |
| Wheel                 | The out-of-process Coding Agent Harness. It consumes Core and `bitty-ai` and owns coding-workflow policy.                                                                                                                                                                                                                       |
| Headed Agent plugin   | The prospective sibling composition plugin that owns chat composition, presentation policy, speech controls, and avatar appearance mapping. Working name only.                                                                                                                                                                  |
| Presentation surface  | A panel, window, widget tree, or avatar surface that renders state. It is not a session, execution, or consent record.                                                                                                                                                                                                          |
| Media extension       | An optional, separately packaged implementation of audio playback, decoding, 2D rendering, avatar rigging, or 3D presentation. It is never required for text operation.                                                                                                                                                         |
| Media layer           | The combined runtime surface of zero or more media extensions plus Core presentation primitives. It is not an owner, contract, or authority.                                                                                                                                                                                    |
| Principal             | The authenticated `(UID, AgentId)` pair defined by the accepted IPC and Agent RFC. A self-asserted name, plugin identity, or panel label is not a principal.                                                                                                                                                                    |
| Agent session         | The accepted `AgentSession` lifecycle anchor plus the application-level handle and attachments defined here.                                                                                                                                                                                                                    |
| Session handle        | A generation-fenced reference returned by `start`; it is not a grant and not a credential.                                                                                                                                                                                                                                      |
| Attachment            | A presentation or observation subscription created by `attach`. Many attachments may exist for one session.                                                                                                                                                                                                                     |
| Session generation    | The monotonic fence for session-level operations, approvals, and consent/elevation binding, issued by the session authority; the issuing authority is also the generation's sequencer (see [Authority split](#authority-split)). A stale session generation fails closed.                                                       |
| Attachment generation | The monotonic fence for one attachment's local handles and presentation state. It never grants authority or binds consent.                                                                                                                                                                                                      |
| `GenerationId`        | The accepted ADR-0013 fencing identifier for reloads, leases, and handoffs. It composes with the per-contract axes here but does not replace the session or attachment generation.                                                                                                                                              |
| Cursor                | The tuple of session identity, session generation, attachment identity, attachment generation, and last applied sequence per delivery class used for replay or resynchronization.                                                                                                                                               |
| Event Bus             | The host-mediated transport accepted by the Panel Runtime RFC, carrying authorized, bounded, manifest-declared topics between host services, panels, and plugins. It is not implemented by this draft.                                                                                                                          |
| Panel                 | A presentation surface identity accepted by the Panel Runtime RFC, addressed as `(PanelId, Generation)` with the lifecycle `Declared -> Created -> Mounted -> Focused -> Suspended -> Disposed`. A panel projects state; it does not own execution, consent, or the session.                                                    |
| Routing snapshot      | The scoped accepted, read-only, digest-verified provider-routing projection. It is not a context or session snapshot.                                                                                                                                                                                                           |
| Context snapshot      | An immutable, redacted, bounded view of compiled model context for a turn or attachment, identified by version and digest.                                                                                                                                                                                                      |
| Session snapshot      | A bounded, redacted state view used to attach or resynchronize; it is not a transcript and not a replay log.                                                                                                                                                                                                                    |
| Speech job            | A `bitty-ai` generation job for the `Speech` task, reusing the draft multimodal generation-job lifecycle.                                                                                                                                                                                                                       |
| Viseme timeline       | Optional timing metadata that maps synthesized audio intervals to mouth-shape identifiers. It is best-effort presentation data.                                                                                                                                                                                                 |
| Candidate             | Proposed by this document only; no review has accepted it.                                                                                                                                                                                                                                                                      |
| Owner-pending         | Belongs to another repository or team; recorded here as a pointer, never as settled content.                                                                                                                                                                                                                                    |
| Illustrative-only     | A sketch whose names, fields, bounds, or defaults are explicitly undecided and create no schema or compatibility promise.                                                                                                                                                                                                       |

## Boundary and sibling relationship

### Three consumers, one platform

Core is the designated owner of record for the neutral platform surface:
transport, authentication, consent enforcement, process and PTY supervision,
panel identity and lifecycle, and the event-bus transport. The panel identity,
lifecycle, and Event Bus contracts are accepted by the
[Panel Runtime RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/panel-runtime-rfc.md);
their implementation is not claimed here. `bitty-ai` is the designated owner
of reusable AI mechanisms. Wheel and the Headed Agent plugin are independent
consumers of those mechanisms. This is an ownership design target; Core panel,
Event Bus, and future `bittyd` implementation do not exist as a result of this
document.

```text
Bitty Core / future bittyd (implementation not claimed here)
  |-- IPC, consent enforcement, panel runtime (accepted contract),
  |   Event Bus (accepted contract)
  |
  |-- bitty-ai
  |     |-- provider/model routing and inference jobs
  |     |-- context compilation and snapshots
  |     |-- AI event vocabulary and speech job contract
  |     |
  |     |-- Wheel (Coding Agent Harness)
  |     |     `-- coding workflow, task graph, harness policy
  |     |
  |     `-- Headed Agent plugin (presentation composition)
  |           `-- optional media extensions
  |                 |-- audio playback and decoding
  |                 `-- 2D/3D avatar rendering
  |
  `-- Core presentation primitives used by either consumer (contract accepted;
      implementation not claimed)
```

There is deliberately no edge between Wheel and the Headed Agent plugin. They
are siblings, not layers. Shared needs are promoted to Core or `bitty-ai`; a
private cross-consumer dependency is not created.

### Responsibility boundary

The table records design-target ownership. It does not claim that Core panel,
Event Bus, `bittyd`, or media-extension code exists.

| Surface                      | Owns (design target)                                                                                                                           | Consumes                                                | Must not                                                                                                |
| ---------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| Bitty Core / future `bittyd` | Transport, authentication, consent enforcement, scope enforcement, panel identity and lifecycle, Event Bus transport, process/PTY supervision. | No model I/O and no AI policy.                          | Select models, assemble prompts, synthesize speech, animate avatars, or render plugin UI.               |
| `bitty-ai`                   | Provider abstraction, routing, inference jobs, context compilation, snapshots, AI event schemas, speech job contract, viseme vocabulary.       | Core transport, consent verdicts, and provider plugins. | Own a window, panel, audio device, GPU surface, chat layout, or credential store.                       |
| Wheel                        | Coding Agent Harness policy: coding workflow, task graph, tool composition, harness-level consent and audit behavior.                          | Core and `bitty-ai`.                                    | Gain a headed UI, speech, avatar, or 3D responsibility from this design.                                |
| Headed Agent plugin          | Chat and interaction composition, presentation policy, panel layout, speech controls, avatar appearance and animation mapping.                 | Core and `bitty-ai`; optional media extensions.         | Depend on `wheel.*`; hold execution authority; bypass Core authorization; own provider credentials.     |
| Provider plugin              | Vendor protocol adaptation, transport, and provider-specific request mapping behind an opaque credential handle.                               | `bitty-ai` provider contract and Core grants.           | Widen capabilities, raise a privacy class, or expose credentials to the agent, plugin, or presentation. |
| Media extension              | Audio device playback, media decoding, 2D rendering, 3D rigging, and GPU presentation behind a declared capability.                            | `bitty-ai` speech/avatar events and Core grants.        | Become a session authority, read transcripts, capture camera or microphone input, or persist media.     |

### The `wheel.*` prohibition

The Headed Agent plugin must not import, require, call, configure, inspect, or
otherwise depend on any `wheel.*` module, package, service, configuration key,
environment variable, or internal API. It does not load Wheel as a library, act
as a Wheel front end, or reuse Wheel private state. The prohibition also covers
spawning or wrapping the Wheel CLI, executing or reading the executable
`.wheel/init.lua`, reading `$XDG_CONFIG_HOME/wheel/init.lua`, and reading
`$XDG_STATE_HOME/wheel/trusted_projects.json`; those paths are user-owned Wheel
state outside model-visible workspace authority per Wheel's
[SECURITY-MODEL.md](https://github.com/bitty-terminal/wheel/blob/main/SECURITY-MODEL.md).
A negative test for each path belongs to the future implementation task.

If a future headed surface needs a capability that Wheel also needs, the
capability is promoted to `bitty-ai` or Core through a separately reviewed
contract and an owning task. A shared mechanism is never obtained by reaching
into Wheel. "No `wheel.*` dependency" is a review gate for this design and a
prerequisite for any future headed implementation.

### What stays out of Wheel

Wheel is an independent out-of-process harness, not a Bitty plugin: the 0.1.0
position is a single harness artifact that holds execution authority, with
plugin packaging and `compat.plugin-api` dropped, and presentation is never
authority ([Wheel README](https://github.com/bitty-terminal/wheel/blob/main/README.md),
[repo.toml](https://github.com/bitty-terminal/wheel/blob/main/repo.toml),
[SECURITY-MODEL.md](https://github.com/bitty-terminal/wheel/blob/main/SECURITY-MODEL.md)
owner decisions D1, D2, and D6, and
[AGENTS.md](https://github.com/bitty-terminal/wheel/blob/main/AGENTS.md)).
This design assigns no headed UI, chat presentation, speech synthesis or
playback, avatar, viseme, 2D, or 3D work to Wheel. Wheel may continue to evolve
its own coding-workflow surface under its own governance; this document neither
changes that surface nor borrows it.

Wheel's own core/plugin composition is a separate Wheel concern recorded only in
the draft
[Wheel core and plugin boundary](../specifications/wheel-core-plugin-boundary-candidate.md).
That candidate is not an authoritative Wheel contract and does not define this
sibling boundary. The Headed Agent is not a Wheel plugin: it has no Wheel
manifest dependency, no Wheel plugin slot, no `wheel.*` service call, and no
shared UI ownership. A future Wheel plugin facade may return only if a
host-brokered execution service is accepted (owner decision D6); that remains a
separate Wheel decision and does not make the Headed Agent a Wheel UI plugin.

## Contract families and versioning

### Candidate contract families

Each family is versioned independently. The labels below are contract-family
names for review, not wire methods, Lua modules, package names, or accepted API
spellings.

| Family            | Candidate version | Accountable owner(s)                                      | Purpose                                                                                                        | Status |
| ----------------- | ----------------- | --------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- | ------ |
| `headed.session`  | 1                 | Core owner + AI owner                                     | Session handle, `start`/`attach`/`detach`/`stop`, session and attachment generations, cancellation, reconnect. | Draft  |
| `headed.stream`   | 1                 | Core owner + AI owner                                     | Bounded delivery classes, sequencing, backpressure, replay, and resynchronization.                             | Draft  |
| `headed.routing`  | 1                 | AI owner (resolution); Core owner (publication authority) | Model-profile request and routing against the scoped accepted routing snapshot.                                | Draft  |
| `headed.context`  | 1                 | Core owner + AI owner                                     | Context snapshot and session snapshot references, redaction, and viewer filtering.                             | Draft  |
| `headed.tools`    | 1                 | Core owner + AI owner                                     | Tool proposal, approval request, verdict, execution, and result events.                                        | Draft  |
| `headed.panels`   | 1                 | Core owner + Headed Agent owner                           | Reserved topic families, panel attachment, suspend, resume, and audio focus.                                   | Draft  |
| `headed.speech`   | 1                 | AI owner (contract); Media extension owner (playback)     | Speech request, stream, cancel, playback handoff, and asset reference.                                         | Draft  |
| `headed.avatar`   | 1                 | AI owner (events); Headed Agent owner (mapping)           | Presentation state, animation, and viseme events; renderer ownership stays outside `bitty-ai`.                 | Draft  |
| `headed.budget`   | 1                 | Core owner + AI owner                                     | Budget envelope, enforcement attribution, and degradation actions.                                             | Draft  |
| `headed.platform` | 1                 | Core owner + Media extension owner                        | Capability detection and presentation classes on Linux, macOS, Windows, and FreeBSD.                           | Draft  |

### Family boundaries

Several families touch the same mechanic. The authoritative family or accepted
source is named once here so the draft does not define it twice:

| Shared mechanic                                               | Authoritative family or source                                                               | Boundary in this draft                                                                                                                        |
| ------------------------------------------------------------- | -------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| Session handle, session generation, `attach`/`detach`         | `headed.session`                                                                             | `headed.panels` only requests panel identity; panel lifecycle stays with the accepted Panel Runtime RFC.                                      |
| Session snapshot content                                      | `headed.session` defines the bounded attachment view; `headed.context` defines context views | A session snapshot is neither a context snapshot nor a transcript, and it never substitutes for `attach` authorization.                       |
| Context snapshot and CP-5 `current_generation`                | `headed.context` consuming draft CP-5                                                        | CP-5 binds the AG-2 agent generation; `headed.session` does not own that generation.                                                          |
| Event sequencing, backpressure, replay, resynchronization     | `headed.stream` for the session observation stream                                           | The Event Bus owns inter-panel topic delivery and its PR-7/PR-8/PR-9 queues; `headed.stream` never redefines bus queues.                      |
| Topic grammar, manifest produce/consume, per-principal ledger | Accepted Panel Runtime RFC Event Bus                                                         | `headed.panels` consumes the grammar and ledger; it mints neither.                                                                            |
| Panel identity, lifecycle, focus, suspend                     | Accepted Panel Runtime RFC and RFC-0006                                                      | Attachment lifecycle is a separate relation; see [Panel lifecycle versus attachment lifecycle](#panel-lifecycle-versus-attachment-lifecycle). |
| Speech job status and `Unknown`                               | Draft multimodal generation-job lifecycle consumed by `headed.speech`                        | Avatar, animation, and viseme events never carry a terminal job state.                                                                        |
| Rich rendering controls                                       | Normative `P0-AC-005`, `P0-AC-009`, `P0-AC-010`                                              | `headed.avatar` and `headed.panels` inherit them; transcript and chat layout stay out of scope.                                               |

### Version negotiation authority

The **session generation created by `start` is the single authority** for the
contract-version binding of every session-bound family in the table. `start`
returns the selected version for each family it negotiates; a session generation
cannot change that binding in place. `headed.platform` is the one
presentation-scoped family with a standalone pre-session probe path, so `start`
is not its only possible authority.

- `start` negotiates every session-bound family the client declares, including
  the baseline `headed.platform` version even when no presentation attachment
  exists. It fails closed if the client requires an unsupported major.
- `attach` may select only a **compatible subset** of the families and versions
  already negotiated for that session generation. It may choose a lower minor
  only when that lower minor is additive, optional, and non-security-relevant;
  both sides must support it. It cannot introduce a family that `start` did not
  negotiate, select a higher major, lower a version that carries authority,
  ordering, redaction, consent, bounds, or failure semantics, or reinterpret a
  security-relevant semantic. The attach result includes the selected versions
  for that attachment.
- Reconnect reuses the version binding of the original `start` and attachment.
  It does not renegotiate. A client that no longer supports the bound version
  fails closed and must perform a new `start` or a new `attach` as appropriate.
- `headed.platform` is presentation-scoped and is normally bound at `start` and
  selected at `attach`. A pre-session or headless capability probe may negotiate
  `headed.platform` directly with Core in a **standalone handshake** that creates
  no session, grants no consent, and starts no audio or rendering; the probe
  records the selected version, and a later `start` must accept it unchanged or
  fail closed.
- There is no other non-session family in this draft. A future non-session
  family must define its own standalone handshake, owner, and fail-closed
  version rules; it cannot inherit a session generation or bind consent.

### Versioning rules

- A family has a major and minor version. A major change removes or changes
  semantics, ordering, authority, or a required field. A minor change is
  additive and optional. A patch change claims no semantic difference.
- The version binding is fail-closed: an unsupported major, an unnegotiated
  family, or an attachment version outside the session binding is rejected. A
  client never guesses, silently reinterprets, or downgrades a security-relevant
  semantic.
- Live events inherit the versions bound to the session generation. Persisted
  or replayed events record the schema version they were produced under so a
  later reader can interpret them without guessing.
- Unknown major versions fail closed. Unknown optional minor fields are ignored
  only after the payload passes size, type, and depth validation; a required
  field is never inferred from a missing one.
- Capability vocabulary is Core-owned and closed. Adding a capability is a Core
  contract change, not an additive `bitty-ai` minor version.
- Deprecation names the replacement, the affected versions, the transition
  period, and the removal condition. A major version is not removed without a
  documented migration or redirect.
- Contract versions are application-level. They do not replace, overload, or
  reinterpret the IPC wire version `v` defined by the accepted RFC.

## Agent session lifecycle and bounded streaming

### Authority split

The accepted `AgentSession` state machine
(`Created -> Running <-> WaitingToolResult -> Completed/Failed`) remains the
lifecycle anchor. This design adds an application-level handle and attachment
layer around it; it does not replace that state machine.

Core is the designated owner of transport, authentication, consent enforcement,
the consent ledger, and scope enforcement. The accepted `AgentSession` state
machine comes from the IPC and Agent RFC and is carried by the separate
`bitty-agent` state-machine crate, which Core does not link. The AI runtime's
vocabulary dependency is `bitty-agent-api`, not the state-machine crate: at the
pinned `bitty-agent` revision `a8e4112`, `bitty-agent-api` exposes only `AgentId`
and `AgentError`, while `bitty-ai-runtime` defines its own `AgentSession` states
including `Canceled`. The report-only AI-0257 vocabulary-conformance ledger
records that drift; this draft does not rename either vocabulary or claim they
converge. How the AI host consumes the RFC state machine (including how
`Canceled` maps to the accepted terminal set), and whether the durable session
record and generation issuance live in Core or `bitty-ai`, remain open points.
`bitty-ai` owns the model-facing binding: routing snapshot, context snapshot,
turn and generation-bound events, tool proposals, speech jobs, and media event
schemas. The Headed Agent plugin owns presentation, local interaction, and the
mapping of semantic events to widgets, speech controls, and avatars.

A session generation has one issuing authority for its lifetime, and that
authority is also the generation's sequencer: whoever mints
`(session identity, generation)` assigns the per-generation sequence at commit
before any event is admitted. Where issuance and transport sequencing are held
by separate components, the handoff from issuer to sequencer is a single
recorded step that adopts the exact issued generation; no event may be admitted
before adoption, and a mismatch fails closed. Two components may not
independently advance the same session generation. This draft does not decide
whether durable issuance belongs to Core or `bitty-ai`; it fixes the handoff
invariant either placement must satisfy.

This document fixes only the rules that neither side may violate: no
presentation authority, no model I/O in Core, no credential exposure, no
session state owned by a panel, and no split session generation.

### Lifecycle operations

| Operation | Meaning                                                                                                 | Grants authority | Stops work             | Cardinality                  |
| --------- | ------------------------------------------------------------------------------------------------------- | ---------------- | ---------------------- | ---------------------------- |
| `start`   | Create a session handle with identity, session generation, budget envelope, routing profile, and scope. | No               | No                     | One handle per start intent. |
| `attach`  | Add a presentation or observation attachment and return a snapshot plus cursor.                         | No               | No                     | Zero to many per session.    |
| `detach`  | Remove one attachment and release its local resources.                                                  | No               | No                     | One attachment at a time.    |
| `stop`    | Request session termination under the current session generation and cancellation rules.                | No               | Yes, under the policy. | Idempotent per session.      |

**`start`.** The caller supplies an authenticated principal, an idempotency key,
a routing profile, a context scope, a budget envelope, and the contract versions
it supports. The idempotency key scopes one start intent: replaying it before
expiry returns the original handle and never creates a second session; after
expiry it fails closed. The result is a session handle containing the session
identity, session generation, initial cursor, selected contract versions, a
bounded session snapshot, and a budget reference. A session is created without
authority beyond the caller's existing grants. `start` is not a consent prompt,
does not pre-approve a tool, and does not start a provider call by itself.

**`attach`.** The caller supplies a session handle, its session generation, the
requested presentation capabilities, and its supported contract versions. The
result is an attachment identity, its attachment generation, a sanitized session
snapshot, a cursor, the selected versions for that attachment, and the resume
rules. Attaching never moves an execution target, broadens context access,
creates a tool grant, or makes the attachment the session authority. The
accepted transport authenticates the UID, not the `AgentId`: an `AgentId` is a
validated declaration inside that UID's trust domain, so two processes under the
same UID are not isolated from each other and this draft claims no
confidentiality between them. Within that limit, an attachment reads only what
its own principal may read; a different principal, or a different `AgentId` from
the same UID, cannot attach by knowing the session identity and needs an explicit
grant. A same-UID caller that presents a stale or wrong `AgentId` fails closed at
the authorization check, not at transport authentication.

**`detach`.** Detachment removes one attachment and releases its local
presentation resources: render surfaces, audio buffers, animation state, and
device handles. It does not cancel model work, tool work, or the session. It is
idempotent and fails closed when the attachment generation is stale. Detaching
the last visible panel does not imply `stop`; an idle policy may stop a session
later, but UI absence is never lifecycle authority.

**`stop`.** The caller supplies the expected session generation and a bounded
reason. Cancellation follows the draft R1 split: before dispatch, the request is
prevented and no effect occurs; after dispatch, owned execution is asked to
cancel and the recorded outcome is the actual result or `Unknown`. This contract
maps session cancellation to the accepted `Completed` or `Failed` terminal state
with a cancellation attribution; it does not add a third accepted
`AgentSession` state. A speech or media job may separately report `Queued`,
`Preparing`, `Running`, `Streaming`, `Completed`, `Failed`, `Cancelled`, or
`Unknown` in its own draft media-job status. `Completed`, `Failed`, and
`Cancelled` are terminal for that job; `Unknown` is a reconciliation state, never
a terminal success or failure. The two levels are never collapsed: a job
reaching `Cancelled` does not by itself terminate the session, and a session
reaching `Completed` or `Failed` does not rewrite a job's recorded status.
`Unknown` is reconciled by status inspection or user direction before any retry;
no rollback and no exactly-once claim is made. `stop` is idempotent, releases
owned resources at a bounded grace deadline, and reports the retention outcome
for evidence. A stale session generation cannot stop a newer session.

### Outcome levels

Session outcomes and job outcomes are separate records with separate state
sets. A consumer must read the level it is acting on:

| Level      | State set                                                                                     | Terminal states                    | Rule                                                                                                                |
| ---------- | --------------------------------------------------------------------------------------------- | ---------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| Session    | `Created`, `Running`, `WaitingToolResult`, `Completed`, `Failed` (accepted IPC and Agent RFC) | `Completed`, `Failed`              | Cancellation records one of the two terminal states with cancellation attribution; there is no third session state. |
| Media job  | `Queued`, `Preparing`, `Running`, `Streaming`, `Completed`, `Failed`, `Cancelled`, `Unknown`  | `Completed`, `Failed`, `Cancelled` | `Unknown` is reconciled by status inspection before retry and is never recorded as success or failure.              |
| Attachment | `attached`, `suspended`, `closed`, `failed`                                                   | `closed`, `failed`                 | An attachment outcome never terminates the session and never substitutes for a session or job outcome.              |

### Controlled session transitions

`resume`, `takeover`, and `reset` are not additional free-standing operations
alongside `start`, `attach`, `detach`, and `stop`. The first two are controlled
transitions with fail-closed authorization; `reset` is deliberately not defined
by this draft.

| Transition       | Preconditions                                                                                                                                                                                                                            | Generation effect                                                   | Consent and effects                                                                                                                                                                                                                                         |
| ---------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Session `resume` | The principal is authenticated, the session is retained and not live under another writer, and the caller passes the current session generation.                                                                                         | Issues a new session generation.                                    | Prior elevation is invalid. Re-grant requires a fresh prompt. No queued command, tool effect, speech request, or animation is replayed.                                                                                                                     |
| `takeover`       | The current owner or an explicit user-owned host policy authorizes the handoff, the handoff is recorded with both principals and the target session generation, and the target principal's current grants and consent are re-authorized. | Issues a new session generation; the old generation is invalidated. | Fresh consent is required. Pending actions are reconciled, not replayed, and media jobs started under the old generation are reconciled to their recorded status before the new generation is used. A stale owner cannot continue under the old generation. |
| `reset`          | Not defined here. A destructive reset is a new-session operation that requires a separate Core contract.                                                                                                                                 | None.                                                               | This draft provides no reset path and no implicit consent. A caller that needs a clean session performs a new `start`.                                                                                                                                      |

Session `resume` and `takeover` retain the contract-version binding of the
original `start`; they do not renegotiate it. A client that requires a different
major performs a new `start` with a fresh session and fresh consent.

### Generation, version, and consent axes

The word "generation" is not a single universal fence. The following axes are
separate, and a contract family must name the axis it uses:

| Axis                                    | Issued or bound by                                                                                                                                                                                                 | What it fences                                                                                     | Binds consent or elevation?                                                                        |
| --------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| Session generation                      | The session authority at `start`, session `resume`, or `takeover`, which is also that generation's sequencer; the exact Core-versus-`bitty-ai` split for durable issuance remains open.                            | Session-level mutating operations, approval verdicts, session stop, and session-scoped media jobs. | Yes, together with the principal and scope.                                                        |
| Attachment generation                   | The attachment authority at `attach`.                                                                                                                                                                              | One attachment's local handles, cursors, and presentation state.                                   | No. It never grants authority or binds consent.                                                    |
| Routing snapshot version and digest     | Core publication authority under the scoped accepted provider-routing binding, with the AI helper owning the authoritative registry and proof. No separate routing "generation" is defined by this draft.          | The exact provider-routing projection used for a turn or session pin.                              | No. It selects routing data; it never grants a scope.                                              |
| Context snapshot version and generation | `bitty-ai` context compilation. CP-5 `current_generation` binds the AG-2 agent generation, which is separate from the session generation and from the runtime `AgentSession.generation`/`rotate_generation` fence. | The exact compiled context and its freshness.                                                      | No. Expansion re-authorizes the read; the snapshot itself is not a grant.                          |
| Speech job identity and status          | The draft `bitty-ai` media-job lifecycle.                                                                                                                                                                          | Cancellation, status inspection, and `Unknown` reconciliation for one speech job.                  | No. Speech egress consent binds to the session generation, principal, provider, and privacy class. |

- Consent and elevation bind to the principal (UID and AgentId), the session
  generation, and the scope. Approval adds the exact call digest, target,
  schema version, and expiry. An attachment generation, routing version, context
  version, or media-job identity never substitutes for that binding.
- A stale session generation invalidates elevation and pending approvals; a
  stale attachment generation closes only that attachment. A stale routing or
  context version fails the turn closed rather than silently reusing an older
  projection.
- An agent suspend, dispose, or reload invalidates prior elevation even before a
  new session generation is issued. Panel suspension does not invalidate agent
  elevation; it expires only the attachment-local handles.
- A generation or version value is a fence, not a secret and not a capability.
  Holding one grants nothing without the caller's own authorization.

### Event ordering and loss

- The session-generation issuing authority owns its sequence; where that is
  Core, Core is the designated sequencer. One sequencer assigns a monotonically
  increasing sequence number per session generation at commit, before queue
  admission, after the issuer-to-sequencer handoff recorded in
  [Authority split](#authority-split). Every event carries that number; a
  dropped event still consumes a sequence number, so a consumer can distinguish
  a dropped event from an event that was never produced. AI turn sequencing is
  nested inside the session-generation sequence: it can order events within a
  turn, but it cannot advance the session cursor or create a second session
  sequence.
- **Gap detection applies only to non-lossy ordered classes:** session
  lifecycle, stop, approval request and verdict, tool result and error, speech
  job lifecycle and terminal status, and other control events. A gap in one of
  these classes triggers resynchronization and is never treated as benign loss.
- **Lossy classes use lost-count and cumulative-state recovery:** turn deltas,
  progress, routing fallback notices, avatar state, animation steps, and viseme
  markers may be coalesced or dropped. A gap in a lossy class is expected; the
  consumer increments a lost counter and recovers by class rather than forcing a
  session resynchronization:
  - Turn deltas recover from a cumulative turn snapshot or a later cumulative
    delta; the final turn message is a non-lossy control event.
  - Routing notices recover by reading the current routing snapshot and binding;
    a missed notice is not an effect.
  - Avatar state recovers from the latest state revision; animation and viseme
    loss degrades presentation only.
- **Durable media-job events are separate from ephemeral presentation events.**
  Speech job lifecycle and terminal events use the draft multimodal status set
  `Queued`, `Preparing`, `Running`, `Streaming`, `Completed`, `Failed`,
  `Cancelled`, and `Unknown`. That class is control, non-lossy, and
  gap-detected. Avatar state, animation, and viseme events are ephemeral
  presentation, lossy, and latest-wins. A terminal media-job state is never
  carried only by an ephemeral avatar event, and `Unknown` is never treated as a
  terminal success or failure.
- Audio uses its own media-channel chunk sequence and playback buffer. It does
  not use the session sequence for gap detection and never triggers a session
  resynchronization.
- Ordering is guaranteed only where stated: per session generation for control
  and lifecycle events, per turn for model output, and per topic for Event Bus
  topics. There is no global total order across sessions, panels, or plugins.
- A snapshot carries a barrier for the classes it includes. Events at or below
  the barrier are already represented; events above it are applied in order. A
  lossy class may recover from its latest state without a full session snapshot.
- Duplicate event identities are idempotent. A non-lossy gap, a conflicting
  duplicate, or an out-of-order control event beyond the accepted window
  triggers resynchronization rather than silent reordering.
- A reconnect never replays an effect. It replays observations, re-presents
  pending approvals as pending, and re-reads authoritative state.

### Backpressure classes

| Class                   | Examples                                                                                                 | Delivery rule                                                             | Overflow behavior                                                                                 |
| ----------------------- | -------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| Control and authority   | Lifecycle, stop, approval request, approval verdict, tool result, speech job lifecycle/terminal, errors. | Non-lossy, bounded, ordered per session generation.                       | Refuse the newest request and report backpressure; never silently drop.                           |
| Recoverable observation | Turn deltas, progress, context updates, routing fallback notices.                                        | Bounded and sequence-numbered; cumulative resynchronization is available. | Coalesce or drop oldest with counted loss, then resynchronize from a snapshot or cumulative turn. |
| Ephemeral presentation  | Avatar state, animation steps, viseme markers, cursor and focus hints.                                   | Latest-wins or short bounded queue; no user-visible data depends on it.   | Drop stale entries with counters; never queue without bound.                                      |
| Audio                   | Speech chunks and playback buffers.                                                                      | Bounded ring buffer with a real-time budget.                              | Stop or restart playback at a safe boundary; never play out of order or unbounded.                |

- Backpressure never silently escalates into spawning another agent, another
  session, or an unbounded retry loop.
- A dropped lossy event still consumes its sequence number and increments the
  class lost counter. A lossy gap is not by itself a resynchronization trigger;
  the consumer uses the class recovery rule above.
- A slow consumer receives an explicit backpressure or slow-consumer outcome
  and may be detached after the bounded policy is exhausted. Its queue is not
  allowed to grow without limit.
- New numeric caps are named policy constants in the owning contract. This
  document does not set new numeric defaults; inherited RFC caps continue to
  apply.
- Every drop, refusal, coalesce, and shed is attributed to the session,
  session generation, attachment generation, topic, and budget dimension.
  Unattributed enforcement is a conformance defect.

### Reconnect and resynchronization

The client reconnects with its cursor: session identity, session generation,
attachment identity, attachment generation, and the last applied sequence per
delivery class. The server first checks the version binding from the original
`start` and `attach`; reconnect does not renegotiate. It then responds in one of
three ways:

1. **Replay.** The missing non-lossy events remain within the bounded retention
   window. The client applies them in order and continues live.
2. **Snapshot plus delta.** The non-lossy gap exceeds the replay window. The
   server returns a sanitized session snapshot with a barrier sequence, and the
   client resumes from the barrier.
3. **Fresh attachment.** The session generation, attachment generation, or
   version binding changed, the attachment expired, or the caller's authority
   changed. The server refuses the old cursor, requires a new `attach`, and
   re-evaluates the caller's current grants.

Reconnect is a read operation. It does not re-execute a tool, re-send a speech
request, re-play audio, re-run an animation backlog, or re-grant consent. It
revalidates the caller's current consent and grants before replay; a revoked
grant fails closed and requires a fresh `attach`. Pending approvals reappear as
pending with their remaining lifetime; an expired
approval becomes denied. Lossy-class gaps recover by their class rule and do not
force a full session snapshot. Audio does not auto-play after reconnect; a fresh
user gesture is required unless an explicit, user-owned autoplay policy already
covers this attachment.

## Provider routing, snapshots, and tool approval

### Model-profile routing

The Headed Agent plugin asks for a model profile or role together with
constraints. It does not select a provider, name an endpoint, supply a
credential, read a provider registry, or construct an alias route.

An illustrative request carries:

- the model profile or role;
- required capabilities, such as text, streaming, tool use, vision, or speech;
- the minimum acceptable privacy class;
- the token, cost, latency, and locality constraints;
- the budget reference and the caller's routing snapshot version and digest.

`bitty-ai` resolves the request against the scoped accepted, read-only,
digest-verified provider-routing snapshot. The accepted
[provider/routing snapshot binding](../architecture/execution-ownership-r1.md#providerrouting-snapshot-binding-contract-accepted-2026-10-17)
remains authoritative for the snapshot domain, canonical bytes, digest,
ownership proof, and lifecycle. That binding splits authority: Core owns
publication authority for the canonical projection, the monotonic
`snapshot_version`, the never-reused publication identity, and invalidation; the
AI helper owns the authoritative populated registry, validates the publication,
and issues the opaque proof only after exact acknowledgement and registry
equality. This document adds only the headed consumer's rule: the plugin treats
the projection as opaque and never reconstructs routing from local state. No
live publication or strict enablement is claimed.

Routing rules:

- The privacy class is a hard constraint, not a preference. A `local-only`
  request never falls back to a remote provider. A `network-minimized` request
  does not silently become `upload-notice`.
- A model alias is outside the accepted strict path. The accepted binding
  excludes the alias table from the initial projection until a later accepted
  alias projection decision, and this draft defines no alias routing. A client
  that needs an alias resolves it to a model profile before `start` or fails
  closed; the routing layer never silently interprets an alias, and an alias
  request is never counted as strict provider-list coverage. Any alias-table
  mutation invalidates the current binding before strict use.
- A provider that resolves to a stored credential requires the matching
  `ai.provider` consent for that principal, provider, and privacy class. A
  standing tool grant or session-wide approval never pre-authorizes provider
  credential use; consent is re-checked when the provider or privacy class
  changes.
- Ordered fallback stays inside the same privacy and capability constraints and
  inside the budget envelope. A fallback that would exceed the cost, token, or
  locality bound fails closed.
- A fallback is visible as a bounded routing event with the reason and the
  selected model identity. Silent degradation of privacy, capability, or cost
  is forbidden.
- A routing snapshot version and digest are pinned for the turn and associated
  with the session generation. A refresh is explicit, attributed, and
  session-generation-bound; it is not triggered by a failed request as an
  unreviewed retry.
- Routing failure is a typed outcome. The UI may explain it and offer a consent
  flow, but the agent never asks the user to paste a credential into chat.
- No provider loader, configuration schema, credential resolution, or vendor
  adapter is defined here. Those remain owner-pending.

### Snapshot distinctions

| Snapshot         | Content                                                                                                                         | Authority                                                                             | Not                                                                                       |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| Routing snapshot | Provider-routing registry projection, domain, version, digest, and ownership proof.                                             | Scoped accepted R1 binding: Core publication authority, AI helper registry and proof. | Not a context snapshot, session snapshot, tool-list snapshot, or completion request.      |
| Context snapshot | Compiled model-facing context, version and generation, token and byte counts, redaction state, provenance references.           | `bitty-ai` context contract.                                                          | Not the transcript, journal, or evidence store; not permission to re-execute an effect.   |
| Session snapshot | Sanitized session state, accepted lifecycle state, session generation, cursor barrier, active turn and jobs, pending approvals. | Core owner + AI owner.                                                                | Not a replay log, not a full transcript, and not a substitute for `attach` authorization. |

Snapshot rules:

- A snapshot is immutable for a version. A change produces a new version and a
  new digest; readers never observe a half-updated snapshot.
- A snapshot is bounded, redacted before it enters a queue, and filtered to the
  requesting principal's current grants. Presence markers replace secret
  values.
- A context snapshot is not a durable transcript. It is a compiled view, and
  the journal or evidence store remains authoritative for history under R3.
- Expanding a snapshot reference re-authorizes the read under the same consent,
  budget, attribution, and untrusted-surface rules. Expansion is not ambient
  context growth.
- No snapshot is written to disk by default. Recording requires explicit
  consent and still passes typed redaction. Retention, deletion, and recovery
  limits follow R3; this document adds no retention rule.

### Tool-call and approval events

The tool event sequence is:

```text
tool.proposed              bitty-ai relays the model's proposal as data
tool.approval.requested    Core opens an approval request
tool.approved | denied     Core records the verdict
tool.started               Core starts an authorized execution
tool.result | tool.failed  Core reports the outcome as untrusted observation
```

These labels are illustrative event-family names, not wire methods.

Rules:

- A proposal is not an execution and carries no authority. The model cannot
  approve its own proposal, and the Headed Agent plugin cannot approve on the
  model's behalf.
- The approval request binds the principal, the session identity, the session
  generation, the tool name and tool version (not the name alone), a canonical
  digest of the validated arguments, the schema version, the required scope, the
  target, and the expiry. The prompt text is authored canonically by Core from
  those bound fields; a presentation surface renders it without rewriting the
  decision semantics, and a prompt that does not match the bound digest fails
  closed.
- An approval applies to exactly that call and digest by default. A changed
  argument, a changed target, a changed session generation, or a changed scope
  requires a new approval. Standing or session-wide approval is a separate
  explicit grant, never an implicit default.
- A stale session generation, an expired approval, a replayed verdict, or a
  verdict from a different principal fails closed.
- A reconnect re-presents a pending approval as pending. It never converts a
  pending request into an approval.
- Voice, avatar gesture, animation state, and model output cannot grant an
  approval. Approval requires an explicit user action through the Core
  authorization path; a future voice-authentication contract would need its own
  accepted security review.
- Tool results are untrusted observation. They never become instructions,
  policy, or automatic follow-up tool calls without fresh authorization.
- The approval ledger, the verdict, and the execution record are Core-owned.
  The Headed Agent plugin owns only the presentation of the request and the
  collection of the user's explicit decision.

## Panel and Event Bus ownership

### Topic families

Topic identifiers follow the accepted Panel Runtime RFC grammar
`owner.name:topic`, matching
`^[a-z][a-z0-9_-]*\.[a-z][a-z0-9_-]*:[a-z][a-z0-9_.-]*$` and bounded to 64
bytes. The rows below are candidate family allocations in that grammar, not
accepted wire strings, subscription methods, or plugin API names. `panel.*` in
this document is the accepted capability family (`panel.provider`,
`panel.create`, `panel.focus`, `panel.overlay`), not a topic namespace;
Core-owned panel topics use qualified names such as `bitty.panel:mounted`.

| Family                   | Producer           | Consumer                   | Accountable owner  | Delivery class         |
| ------------------------ | ------------------ | -------------------------- | ------------------ | ---------------------- |
| `session.lifecycle:*`    | Core               | Attachments, host services | Core owner         | Control, non-lossy     |
| `ai.turn:*`              | AI owner           | Attachments                | AI owner           | Lossy observation      |
| `ai.context:*`           | AI owner           | Attachments                | AI owner           | Lossy observation      |
| `ai.tool-proposal:*`     | AI owner           | Attachments, approval UI   | AI owner           | Control, non-lossy     |
| `tool.approval:*`        | Core owner         | Attachments, approval UI   | Core owner         | Control, non-lossy     |
| `tool.result:*`          | Core owner         | Attachments                | Core owner         | Control, non-lossy     |
| `ai.speech-job:*`        | AI owner           | Headed Agent owner         | AI owner           | Control, non-lossy     |
| `ai.avatar:*`            | AI owner           | Headed Agent owner         | AI owner           | Ephemeral presentation |
| `bitty.panel:*`          | Core owner         | Attachments, host services | Core owner         | Control                |
| `example.headed-agent:*` | Headed Agent owner | Its own subscribers        | Headed Agent owner | Plugin-defined         |

A row's accountable owner is the repository accountable for the topic schema,
producer identity, and authorization; a consumer is never the owner of a topic
it only reads. Audio chunks are not an Event Bus topic: they travel on the media
channel with their own chunk sequence.

Topic rules:

- The accepted RFC enforces topic grammar at declaration and publish, rejects a
  bare name without an owner prefix with `UnknownTopic`, and fails a non-Core
  topic that impersonates `bitty.*`. The Core owner reserves `bitty.*` for Core
  producers and, as a design target, reserves `ai.*` for the AI owner. The
  `panel.*` capability family grants `panel.provider`, `panel.create`,
  `panel.focus`, and `panel.overlay`; it is not a publishable topic namespace.
  A plugin publishes only inside its own owner-qualified namespace.
- A provider, panel, or plugin declares the topics it produces and consumes in
  its manifest. Undeclared produce or consume is denied with `UndisclosedTopic`;
  subscribing to a topic is never a way to discover it. `PR-3` caps distinct
  topics at 256 per process and `PR-4` caps subscriptions at 32 per panel or
  plugin.
- Subscription requires authorization for the topic scope, evaluated per
  client principal: `(authenticated UID, AgentId)`, `(PluginId, generation)`, or
  `(PanelId, generation)` has its own ledger. Knowing a topic name is not
  permission to read it, and a grant to one principal never augments another.
  Cross-plugin subscription requires an explicit grant; there is no ambient
  wildcard bus.
- Payloads are bounded `BoundedText` JSON-compatible values of at most 8 KiB
  (`PR-5`) and at most 32 events or 8 KiB per wakeup (`PR-6`), schema-versioned,
  redacted, and free of credentials, raw environment values, and unbounded
  transcript text.
- Delivery uses the accepted three-level envelope: 64 events FIFO per
  subscription (`PR-7`), 1024 events or 256 KiB per panel or plugin (`PR-8`),
  and 8192 events or 2 MiB globally (`PR-9`); `RC-1`, `RC-2`, `RC-9`, and
  `RC-10` continue to apply. `DropOldest` is the accepted v1 default, with
  `DropNewest` as the documented alternative; both count drops per queue and
  attribute them to the owning panel or plugin.
- This draft's control class is deliberately stricter than the v1 default:
  approval, stop, tool-proposal, speech-job lifecycle, and tool-result events
  are bounded non-lossy and refuse the newest event instead of dropping an
  older one. That tightening must be accepted in the owning bus contract and is
  not claimed as accepted here.
- Ordering is FIFO within one queue and per session generation where the topic
  carries session events. There is no ordering across panels or topics and no
  global total order.
- Retention is per topic class. Control topics retain enough to support the
  documented reconnect window; ephemeral presentation topics may retain only
  the latest state.
- A plugin-private topic is not a covert cross-plugin channel. It is still
  subject to payload bounds, audit, and the platform's plugin isolation rules.

### Core-versus-plugin ownership

The table records design-target ownership only; it claims no implementation.

| Concern                       | Core / future `bittyd`                                                                                                  | `bitty-ai`                                                                                         | Headed Agent plugin                        |
| ----------------------------- | ----------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- | ------------------------------------------ |
| Panel identity and lifecycle  | Owner under the accepted Panel Runtime RFC: identity, create, mount, suspend, dispose; implementation not claimed here. | Consumes panel references only; no ownership.                                                      | Requests a panel; never invents identity.  |
| Layout, focus, and visibility | Owner of accepted host focus and visibility routing.                                                                    | No ownership.                                                                                      | Owns its internal composition and widgets. |
| Rendering and animation       | Owner of presentation primitives under accepted rich-presentation contracts; implementation not claimed here.           | Owns the semantic state and viseme event schemas.                                                  | Maps semantics to 2D/3D appearance.        |
| Agent and turn state          | Host-side transport; durable session-record and state-machine placement open.                                           | Owns model-facing turn and job state; consumes the RFC state names, not a shared crate vocabulary. | Projects the sanitized view.               |
| Topic schema                  | Owner of accepted reserved namespaces (`bitty.*`) and transport; `ai.*` reserved for the AI owner by this draft.        | Owns AI-domain topic schemas it produces.                                                          | Owns only its private topics.              |
| Consent and approval          | Ledger, prompt, and verdict owner; no implementation claimed.                                                           | Relays proposals and results; grants nothing.                                                      | Presents the request; cannot grant.        |
| Retention and deletion        | Host evidence policy owner.                                                                                             | Owns AI record semantics under draft R3.                                                           | Keeps local media ephemeral by default.    |
| Audio focus                   | Host device and window focus owner; one device plays one stream at a time.                                              | Owns the speech job lifecycle.                                                                     | Requests playback; does not steal focus.   |

A panel is a presentation surface, not a session, execution, or consent record;
ADR-0013 keeps panel provenance non-authoritative and separates `PanelId` from
`ExecutionContextId` and `AgentId`. The draft
[execution-projection binding](../specifications/execution-projection-binding-candidate.md#the-binding-relation)
is a candidate design reference for panel-to-execution projection; it is not
accepted and governs nothing yet. A session attachment is a different relation:
it binds a presentation client to a session's observation stream and does not
replace or duplicate an execution binding.

The accepted OQ-058 ruling's SMO-1 through SMO-4 mandates are consumed by
reference: bounded typed events on the accepted bus rather than shared memory;
presentation is not authority; routing fails closed; and the substrate is the
existing IPC, attribution, and Panel Runtime identity rather than a second
registry or daemon. This draft adds no competing orchestration topology.

## Multi-panel suspend and resume

### Panel lifecycle versus attachment lifecycle

The accepted Panel Runtime lifecycle and this draft's attachment lifecycle are
related but not identical. A visible headed surface needs both relations, and
neither one implies the other:

| Panel Runtime lifecycle (accepted) | Attachment state (this draft) | Relationship                                                                                                                                                                            |
| ---------------------------------- | ----------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Declared`                         | none                          | A panel type is admitted by the provider manifest; no session attachment exists.                                                                                                        |
| `Created`                          | none                          | `(PanelId, Generation)` is allocated and not mounted; the panel has no attachment.                                                                                                      |
| `Mounted`                          | `attached`                    | Mount binds `PanelId` to a `ViewId`; `attach` binds a presentation client to a session stream. A mounted panel can show a non-session surface, and an attachment can outlive one mount. |
| `Focused`                          | `attached`                    | Focus is accepted host routing; an attachment never gains authority from focus.                                                                                                         |
| `Suspended`                        | `suspended`                   | Accepted panel suspend makes the panel invisible without destroying its attachment; attachment suspend pauses presentation and releases devices without destroying the attachment.      |
| `Disposed`                         | `closed`                      | Disposing the panel removes its attachment; the session continues unless an explicit session lifecycle operation stops it.                                                              |
| failed panel operation             | `failed`                      | A panel-generation mismatch or unrecoverable panel error fails the attachment closed; it does not terminate the session.                                                                |

- The panel generation fences panel identity and stale-handle rejection. The
  attachment generation fences one attachment's local handles. The session
  generation fences session operations. None substitutes for another.
- Panel suspend and resume change visibility, not panel generation and not
  attachment generation. A resumed panel therefore reuses its attachment unless
  authorization or a version binding changed and the attachment must fail
  closed.
- Panel creation is not `attach`, and `attach` is not a panel mount. A headless
  session has zero attachments while still existing; a panel with no session
  attachment is still a valid panel surface.

### Attachment states

| State       | Meaning                                                                                                       |
| ----------- | ------------------------------------------------------------------------------------------------------------- |
| `attached`  | Live, authorized, and receiving the session stream within its budget.                                         |
| `suspended` | Presentation is paused; cursor and snapshot reference are retained; local devices are released.               |
| `closed`    | Attachment is removed; no further events are delivered; local resources are released.                         |
| `failed`    | Attachment cannot continue because of authority loss, attachment-generation mismatch, or unrecoverable error. |

### Suspend rules

- Suspend is a host operation on one attachment. It is not session
  cancellation, consent revocation, or a change to the execution target.
- Suspend stops rendering, animation, and audio playback for that attachment,
  releases GPU surfaces and audio device handles, and stops consuming
  presentation budget. The session, model turn, and tool work continue unless
  separately stopped.
- The attachment retains its cursor and last snapshot reference within the
  retention window. It does not retain a live GPU texture, audio device, or
  panel-owned execution handle.
- A suspended attachment does not receive unbounded queued updates. The
  producer coalesces to state or drops ephemeral events with counters; control
  events remain recorded for replay within their window.
- Suspending a panel does not invalidate the agent's elevation. That rule
  applies to agent suspend, dispose, or reload, not to presentation suspension.
  Suspension does not destroy the attachment and does not advance its
  generation, mirroring the accepted panel rule that suspend makes a panel
  invisible without destroying the attachment. Device handles are released and
  reacquired on resume; the caller's authorization and the retained attachment
  generation are revalidated then.

### Resume rules

- Resume revalidates the caller's current authorization, consent state, the
  session generation, the attachment generation, and the version binding from
  the original `start` and `attach`. It does not renegotiate versions. A version
  or attachment-generation mismatch fails closed and requires a fresh `attach`;
  a session-generation mismatch requires the controlled session `resume` or
  `takeover` path. A revoked grant is never restored by resume.
- If the cursor is still within the replay window, the attachment replays
  missing non-lossy control events and applies the latest presentation state. If
  not, it loads a sanitized session snapshot and resumes from the barrier.
- Animation state is rehydrated, not replayed as a backlog. Viseme events are
  not replayed for audio that already finished. Audio does not auto-play on
  resume; a fresh user gesture is required unless an explicit, user-owned
  autoplay policy already covers this attachment.
- If the session ended while the attachment was suspended, resume returns the
  terminal session outcome and the retention result; it does not recreate the
  session.

### Multi-panel cardinality

- One session may have many attachments across panels and windows.
- One panel may host several session attachments in separate surfaces, but one
  surface has one primary attachment at a time. The existing execution-binding
  cardinality still applies to execution projections.
- One audio device plays at most one stream at a time; that per-device
  exclusivity is a physical invariant. Whether playback focus is scoped per
  panel, per session, or globally is an open policy (see
  [Open points](#open-points)); this draft does not fix that scope. Focus
  changes are user or host decisions; an agent may request attention, never
  preempt focus or start playback.
- Panel closure removes that attachment. The session may continue headless;
  stopping it requires an explicit lifecycle action or a reviewed idle policy.
- A panel that becomes invisible is suspended or detached according to host
  policy. Invisibility never broadens context access, manufactures consent, or
  moves the execution target.

### Lifecycle distinctions

| Action          | Session continues | Attachment continues | Local devices | Elevation               |
| --------------- | ----------------- | -------------------- | ------------- | ----------------------- |
| Panel hidden    | Yes               | Yes                  | May release   | Unchanged               |
| Panel suspended | Yes               | Paused               | Released      | Unchanged               |
| Panel closed    | Yes               | No                   | Released      | Unchanged               |
| `detach`        | Yes               | No                   | Released      | Unchanged               |
| `stop`          | No                | No                   | Released      | Ends with the session   |
| Agent suspend   | Policy-dependent  | May pause            | Released      | Invalidated; re-consent |
| Runtime restart | Reconcile         | No                   | Released      | Fresh consent required  |

## Speech synthesis contract

### Reuse, not a second job model

Speech synthesis reuses the draft `Speech` task, generation-job lifecycle,
status set, `Unknown` reconciliation, and asset-reference rules of the draft
[Multimodal inference boundary](../providers/multimodal-inference-boundary.md#generationjob-lifecycle).
This contract adds only the headed consumer's request, streaming, playback, and
cancellation rules. It creates no competing inference job, no second audio
provider registry, and no inline audio payload in model context. The multimodal
boundary remains draft; reuse does not promote it.

### Flow

```text
explicit user or policy request
  -> consent and privacy-class check
  -> text validation and redaction
  -> budget check (characters, duration, cost, audio memory)
  -> Core one-shot speech authorization (text digest, provider, privacy class,
     session generation, principal, expiry)
  -> speech job (Queued -> Preparing -> Running -> Streaming ->
     Completed | Failed | Cancelled; Unknown reconciles)
  -> bounded audio chunks and optional viseme timeline
  -> optional media extension playback
  -> terminal status, or Unknown reconciliation before any retry
```

### Request rules

An illustrative request carries the text to synthesize, a voice profile
identifier, a locale, an output format, a streaming preference, a minimum
privacy class, a budget reference, and a consent reference. It does not carry a
provider identity, endpoint, credential, or raw vendor parameter.

- Before any provider I/O, the Core authorization path issues a one-shot speech
  authorization bound to the text digest of the exact post-validation,
  post-redaction bytes to be sent, the provider identity, the privacy class, the
  session generation, the principal, and an expiry. The speech host consumes it
  once and records a per-use audit entry. Reuse, a text digest mismatch, a
  provider or privacy-class change, a stale session generation, a different
  principal, or an expired authorization fails closed before egress. A
  compromised plugin cannot mint, widen, or replay the authorization.
- Text is bounded before it leaves the plugin, and the speech host revalidates
  length, redaction, consent, and privacy class before any provider I/O. A
  client-side precheck is advisory only and never authoritative. The bound is a
  named policy constant in the future accepted speech contract, not an implicit
  function of the provider.
- Text is redacted before synthesis. Secret-shaped values, raw environment
  values, credential references, and unapproved context are removed.
- Only text the user is allowed to see and explicitly chose to speak is
  eligible. Hidden context, sibling-panel content, and full transcripts are
  never synthesized by default.
- The request cannot contain external entity references, file references,
  network references, or unbounded markup that a speech engine might resolve.
  Markup, if supported at all, is parsed with the same bounded, fail-closed
  discipline as any other untrusted input.
- Voice cloning, speaker identification, biometric voiceprints, and persistent
  voice profiles are out of scope. They require separate accepted contracts and
  security review.
- Derived audio and viseme artifacts follow draft R3 deletion propagation:
  deleting the source history deletes them, and the only exception is an
  explicit, consented retention exception that names the derived artifact.
  Partial propagation is non-conformance.

### Stream and delivery

- Audio is delivered as bounded chunks with sequence, end-of-stream, and
  format metadata. The terminal chunk is non-lossy; intermediate chunks follow
  the audio backpressure class.
- A streaming request has a first-chunk deadline and a total duration bound.
  Missing the deadline produces a typed timeout, not an unbounded wait.
- A non-streaming request returns an asset reference and metadata. Audio bytes
  are not placed in context, events, or snapshots. The reference carries an
  expiry and a version; expanding an expired reference fails closed and is
  re-requested, never replayed. Job submission carries an idempotency key: a
  duplicate submission before expiry returns the original job, and after expiry
  it fails closed (the store-expiry facet recorded under AIQ-03).
- A player may start before the full asset exists, but it never plays
  out-of-order chunks. On underflow it pauses or restarts at a safe boundary
  and reports the degradation.
- Viseme timing metadata is optional and best-effort. Its absence or loss
  degrades animation only; audio and captions continue.

### Cancellation

- Cancellation is idempotent and may be requested at any chunk boundary, from
  the user, the owning panel, or the session stop path.
- Local playback stops first, queued audio is discarded, and the device buffer
  is cleared and overwritten before release. Provider-side cancellation is
  best-effort.
- The job terminal outcome is `Cancelled` when cancellation is confirmed, or
  `Unknown` when the remote job may have completed and billed. `Completed`,
  `Failed`, and `Cancelled` are the job terminal states; `Unknown` is a
  reconciliation state, not a terminal outcome. `Cancelled` here is the draft
  speech-job status, not an accepted `AgentSession` state; the session records
  `Completed` or `Failed` with cancellation attribution at the session level.
  `Unknown` is reconciled by status inspection before any retry; a retry is a
  new submission, not a replay, and carries a new idempotency key.
- Cancellation never leaves a partial asset silently presented as complete.
  Partial audio is discarded or explicitly marked incomplete.
- Detaching the only playback attachment stops local playback but does not
  cancel the speech job unless the user or policy asks for cancellation. A
  detached job may complete to an asset reference that no panel is currently
  playing.

### Playback ownership

Audio device selection, decoding, buffering, volume, mute, and device-loss
recovery belong to a media extension. Core may provide a neutral presentation
primitive, but no Core audio implementation is claimed here. `bitty-ai` owns the
speech job and the semantic event; it does not open an audio device. The Headed
Agent plugin owns when speech is offered, which messages are eligible, and the
user controls.

Playback is off by default. Autoplay may be enabled only by an explicit,
user-owned host policy that names the applicable session or attachment and
remains revocable; absent that policy, a fresh user gesture is required. A host
policy, plugin default, agent request, or model output cannot silently start
audio. Captions or a text transcript remain available whenever speech is
available. A host without an audio device degrades to text; it does not fail the
session.

## Avatar, animation, and viseme contract

### Presentation state, not identity

An avatar is a presentation of session state. It is never the session
identity, authorization signal, or source of truth. The semantic state
vocabulary is intentionally small and non-anthropomorphic:

| State       | Meaning                                                         |
| ----------- | --------------------------------------------------------------- |
| `idle`      | No active turn, speech, or approval is being presented.         |
| `thinking`  | A turn or job is in progress.                                   |
| `speaking`  | Speech audio or its viseme timeline is being presented.         |
| `waiting`   | An approval, user decision, or resynchronization is pending.    |
| `error`     | A typed failure is being presented; it is not an emotion.       |
| `suspended` | The attachment is suspended and local presentation is released. |
| `detached`  | No attachment is presenting this session.                       |

State is derived from session, turn, approval, and media events. The model does
not author avatar state directly, and a state transition never grants
authority. A smiling or attentive avatar cannot approve a tool call.

### Animation rules

- Animation is a bounded, interruptible timeline. A state change supersedes
  stale animation steps; the queue never grows without bound.
- Reduced-motion, static, and text-only modes are first-class. A user can
  disable animation or the avatar entirely without losing session function.
- Animation never blocks the event loop, the terminal hot path, or audio
  playback. Rendering work runs off the hot path under the presentation budget.
- A device loss, GPU context loss, or missing renderer falls back to the next
  lower presentation class. It does not crash the session or silently continue
  with a corrupted surface.

### Viseme events

- A viseme event maps a bounded time interval in the speech audio to a
  renderer-neutral mouth-shape identifier. The identifier vocabulary is
  versioned and owned by `bitty-ai`; a renderer maps it to its own rig,
  sprites, or blendshapes.
- The audio clock is authoritative. Viseme timing is best-effort and may be
  corrected by the renderer within a declared tolerance.
- Viseme events are ephemeral presentation data. They may be coalesced or
  dropped under backpressure; audio continues and captions remain accurate.
- A viseme timeline may come from provider metadata or a local analysis step.
  Local analysis is an optional media-extension capability and is not defined
  here.
- This contract makes no lip-sync accuracy claim. Accuracy evidence belongs to
  the renderer or media extension that implements it.

### 2D and 3D ownership

The same semantic state, animation, and viseme events drive both 2D and 3D
presentations. A 2D implementation may use sprites, layered images, or a small
state machine; a 3D implementation may use a rig, blendshapes, or a scene graph.
Neither implementation lives in `bitty-ai`, Core, or Wheel.

The Headed Agent plugin owns appearance, character composition, and the mapping
from semantic events to its chosen presentation. A 3D renderer is an optional,
separately packaged media extension with its own capability declaration,
resource budget, and security review. No 3D code, GPU surface, or avatar asset
pipeline is authorized by this document.

### Accessibility and privacy

- Captions or a text transcript accompany speech; audio is never the only
  channel for required information.
- Flashing, rapid motion, and high-contrast effects follow the host's
  accessibility settings and can be disabled.
- The avatar does not read the transcript, capture camera or microphone input,
  perform face or voice recognition, or persist biometric data.
- Viseme timelines and avatar state are derived from speech text. They inherit
  the speech privacy class, are not egressed to a model or provider by default,
  and are excluded from logs and traces unless a recording consent names them.
- Audio buffers, decoded PCM, and viseme payloads are cleared before release on
  cancel, detach, suspend, takeover, or session end.
- Avatar assets are untrusted input. Decoding, texture upload, and shader
  composition follow the media extension's bounded, isolated implementation and
  the `P0-AC-005` deny-by-default loader policy.
- GPU memory and textures are released on detach, suspend, or session end and
  are not reused across principals without clearing.

## Resource budgets

### Budget envelope

`start` accepts a budget envelope. The envelope is owned by user or host
policy, requested by the plugin, and enforced at the owning layer. A plugin
cannot raise its own envelope, and a project or model cannot widen it.

| Dimension       | Examples                                                                 | Accountable enforcement owner(s)                                               | Exhaustion action                                    |
| --------------- | ------------------------------------------------------------------------ | ------------------------------------------------------------------------------ | ---------------------------------------------------- |
| Tokens and cost | Context tokens, output tokens, model cost, speech characters or seconds. | Core owner + AI owner                                                          | Truncate under CP-5, refuse a new job, or stop.      |
| CPU             | Turn processing, rendering, animation, decoding threads.                 | Core owner (host); Headed Agent owner (render); Media extension owner (decode) | Coalesce, lower quality, or suspend presentation.    |
| Memory          | Event queues, history, snapshots, decoded audio, textures, meshes.       | Owner of the allocation: Core, AI, Headed Agent, or Media extension            | Evict oldest, drop ephemeral state, or stop the job. |
| GPU             | Texture memory, render targets, mesh and animation buffers.              | Media extension owner                                                          | Release surfaces, fall back to 2D, or text-only.     |
| Audio           | Buffer duration, decoded bytes, concurrent playback, device handles.     | Media extension owner                                                          | Stop or restart at a safe boundary; degrade to text. |
| Network         | Provider calls, retries, concurrent streams, media downloads.            | Core owner + AI owner                                                          | Refuse, back off, or fail closed.                    |
| Wall clock      | Turn duration, speech duration, animation duration, reconnect window.    | Core owner + AI owner + Headed Agent owner                                     | Typed timeout or terminal state; no unbounded wait.  |
| Storage         | Snapshots, cached audio, logs, avatar assets.                            | Core owner + AI owner                                                          | Ephemeral default; consent-gated retention only.     |

### Enforcement rules

- Every budget has a hard cap and, where useful, a soft target. A soft target
  produces a degradation or warning; a hard cap refuses or stops. Neither is
  silently exceeded.
- Enforcement emits the attribution set required by `FS-AI4`: authenticated
  client id, `AgentId`, generation, `StreamHandle` when applicable, the Stable Id
  set, the budget dimension, the observed value, the limit, and the action taken.
  This draft adds the session generation and attachment generation where they
  exist. Unattributed enforcement is a conformance defect.
- Network and media bounds inherit `RC-9` (IPC request rate and payload) and
  `RC-10` (snapshot-bounded responses, 256 KiB stream chunks) from the accepted
  Isolation and Resource RFC. Presentation queues inherit the Panel Runtime
  `PR-7`, `PR-8`, and `PR-9` ceilings rather than inventing a parallel budget
  family.
- A suspended or hidden attachment receives no rendering or audio budget. A
  detached attachment receives no presentation budget at all.
- Queue bounds, audio buffers, texture budgets, and media-job counts are named
  policy constants in their owning contracts. This document does not invent
  numeric defaults; inherited RFC caps continue to apply.
- Budget exhaustion is a typed outcome, not a panic, a crash, or an unbounded
  retry. The UI may explain the degradation and offer an explicit change of
  policy, but the agent cannot grant itself more.
- CPU, memory, GPU, audio, and network accounting are per session and per
  attachment where the host supports it. A fault or exhaustion in one session
  does not degrade a sibling session beyond the shared hard limits.

## Platform fallback

### Capability detection and classes

The platform layer detects capabilities; it does not assume them from an
operating-system name. Presentation classes are separate from the accepted
support tiers in
[ADR-0002](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/adrs/ADR-0002-platform-support-tiers.md):
a presentation class never upgrades a platform's support tier.

| Presentation class | Capability                                           | Required behavior                                                      |
| ------------------ | ---------------------------------------------------- | ---------------------------------------------------------------------- |
| `P-A`              | Audio playback plus GPU-accelerated 2D or 3D avatar. | Full presentation within the declared budgets.                         |
| `P-B`              | Audio playback plus 2D or software-rendered avatar.  | Reduced animation and resolution; no 3D requirement.                   |
| `P-C`              | Text and captions only; no audio device or no GPU.   | Full session function; speech may produce an asset for later playback. |
| `P-D`              | Headless or no presentation attachment.              | Session and tools continue; no rendering, audio, or avatar resources.  |

Class selection is explicit, visible, and reversible on device change. A lower
class never weakens consent, redaction, approval, or retention rules. It only
reduces presentation.

### Platform matrix

The table records the fallback expectation, not a support claim. Platform
support commitments are the three tiers accepted by
[ADR-0002](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/adrs/ADR-0002-platform-support-tiers.md)
(`Tier 1` supported, `Tier 2` best-effort, `Tier 3` community); that ADR is the
authoritative support matrix, and cross-platform claims require CI or reviewed
test evidence.

| Platform | Expected baseline                                                                                                                                   | Audio                                                    | GPU / avatar                                    | Notes                                                                            |
| -------- | --------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------- | ----------------------------------------------- | -------------------------------------------------------------------------------- |
| Linux    | ADR-0002 `Tier 1` on x86_64; presentation class `P-A` or `P-B` where devices exist.                                                                 | Native audio stack through Core or a media extension.    | Native GPU surface through the media extension. | Device loss and compositor restart must degrade without killing the session.     |
| macOS    | ADR-0002 `Tier 1` on ARM64, `Tier 2` on x86_64; presentation class `P-A` or `P-B` where devices exist.                                              | Native platform audio through Core or a media extension. | Native GPU surface through the media extension. | App suspension and audio device changes must be handled explicitly.              |
| Windows  | ADR-0002 `Tier 1` on x86_64 with ConPTY; presentation class `P-A` or `P-B` where devices exist.                                                     | Native platform audio through Core or a media extension. | Native GPU surface through the media extension. | Audio session and device-change behavior require platform evidence.              |
| FreeBSD  | ADR-0002 `Tier 2` best-effort on x86_64; presentation class `P-C` or `P-D` by default, `P-B` only with detected audio and software or 2D rendering. | Only when a supported audio path is detected.            | Not assumed; software or 2D fallback.           | Missing audio or GPU must fall back to text or headless, never fail the session. |

Platform rules:

- No operating-system API, device path, driver, or GPU backend is named or
  embedded by `bitty-ai`. The Core owner owns host integration where a host
  adapter exists; the Media extension owner owns device-specific audio, GPU, and
  decoder adapters.
- Missing audio, missing GPU, a lost device, a context loss, an unsupported
  codec, or unavailable decoder isolation falls back to the next lower
  presentation class with a user-visible reason. A class that requires isolated
  decoding is unavailable when that isolation cannot be verified.
- A platform that cannot enforce a security requirement for a class does not
  enter that class. It fails closed to a lower presentation class rather than
  serving unbounded or unredacted media.
- Suspend and resume, session lock, display change, and audio-route change are
  treated as device events. They do not change session authority or consent.
- The standalone `headed.platform` capability probe authenticates the caller
  under the accepted IPC transport, is rate-limited under `RC-9`, and emits an
  audit record. It creates no session, grant, device claim, or rendering work,
  and a probe result is never authority for a later operation.
- The media and rich-presentation surfaces in this draft inherit the security
  controls mapped to `T-03`/`R-003` (`P0-AC-005`), `T-05`/`R-005`
  (`P0-AC-009`), and the rich-rendering control under `R-021`
  (`P0-AC-010`); a platform claim that cannot cite that mapping stays
  unverified.
- Local developer-host behavior proves nothing about another platform. Any
  platform row remains unverified until the owning repository records evidence.

## Security, privacy, and consent

### Consent categories

| Category                | Examples                                                  | Default     | Revocation behavior                                                        |
| ----------------------- | --------------------------------------------------------- | ----------- | -------------------------------------------------------------------------- |
| Model egress            | Sending context to a remote provider.                     | Denied      | Cancel in-flight streams at the next chunk boundary; no new calls.         |
| Tool execution          | Running a proposed tool with a specific target and scope. | Denied      | Deny pending calls; do not roll back completed effects; reconcile Unknown. |
| Speech egress           | Sending text to a remote speech provider.                 | Denied      | Stop streaming, discard queued audio, reconcile the job.                   |
| Playback and autoplay   | Producing sound without a fresh user action.              | Off         | Stop playback and release the device.                                      |
| Recording and retention | Writing audio, visemes, snapshots, or traces to disk.     | Off         | Stop new writes; apply deletion under R3.                                  |
| Avatar media            | Camera, microphone, biometric, or voice-profile use.      | Not granted | Out of scope; requires a separate accepted contract.                       |

Consent rules:

- Consent is per authenticated principal, per session generation and, for
  provider egress, per provider and privacy class, per scope, time bound,
  visible, and revocable. A plugin install, a model choice, a project file, a
  panel attachment, or an avatar state never implies consent.
- No bundled admin grant, wildcard consent, or "allow all" control. A new
  provider or a raised privacy class requires a fresh decision.
- Revocation takes effect at the next dispatch or chunk boundary. It detaches
  affected handlers, cancels in-flight media, invalidates outstanding one-shot
  speech and playback authorizations, and records an auditable revocation event.
- A re-grant after revocation requires a fresh prompt. A reconnect or a
  restored snapshot never restores a revoked grant.
- Approval for a tool, speech egress, or playback cannot be granted by model
  output, voice, avatar gesture, or animation. Only an explicit user action
  through the Core authorization path counts.
- Autoplay is governed by the rule in [Playback ownership](#playback-ownership):
  off by default, enabled only by an explicit, user-owned, revocable host
  policy, otherwise requiring a fresh user gesture. No model, agent, plugin
  default, or unattended host policy may start audio on its own.

### Data rules

- Minimize before egress: send only the text or context the user approved, at
  the smallest scope that satisfies the task.
- Redact before queueing and before writing. Secret-shaped values, raw
  environment values, credential handles, and unapproved context never enter
  events, snapshots, speech text, avatar state, or traces.
- Environment access follows the sanitized Agent View: names and presence
  markers, never secret values. Execution may use an environment without
  disclosing it to the model.
- Audio, viseme timelines, animation state, and attachment snapshots are
  ephemeral by default. Recording requires explicit retention consent and still
  passes redaction.
- Deleting source history propagates deletion to derived audio, visemes, and
  derived avatar state under draft R3. The only exception is an explicit,
  consented retention exception that names the derived artifact. Partial
  propagation is non-conformance, not a background cleanup opportunity.
  Deleting an audio artifact alone does not delete the source text history.
- No hidden context in speech. A user who can hear the output can inspect the
  text that produced it.

### Untrusted surfaces

Model output, terminal text, tool results, provider metadata, speech markup,
audio containers, animation files, avatar assets, and plugin-private events are
untrusted. They are data, never instructions or authority.

- A model that writes "speak this secret" or "approve this tool" gains nothing.
  Speech text is filtered by the same policy as any other egress, and approval
  is a separate explicit action.
- Media decoding is mandatory-isolated: audio containers, avatar assets, and
  animation inputs are decoded in a separate process or an equivalent verified
  sandbox, with bounded input, depth limits, and fuzzed parsing. A decoder fault
  is contained to its media job only when that isolation is present.
- If the required isolation is unavailable or cannot be verified, the media
  extension fails closed to the next lower presentation class or text-only. It
  does not decode untrusted media in-process and does not claim containment.
- No shell command, file path, URL, or environment value from an untrusted
  media payload is executed or resolved by `bitty-ai`.
- A malicious or spoofed panel cannot publish into a reserved topic, forge a
  producer identity, or read another plugin's stream.

### Rich rendering controls

A headed surface that renders Markdown, rich blocks, images, or scene content
inherits the normative rich-rendering controls; a presentation surface never
becomes a rendering sandbox exception. The accepted
[Rich Presentation RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/rich-presentation-rfc.md)
defines the presentation contract, and the P0 criteria define the security
floor:

- `P0-AC-005`: local resources load only through the shared deny-by-default
  policy-controlled loader. Device nodes, sockets, procfs/sysfs/devfs entries,
  symlink escapes, and non-regular files are denied, including avatar sprites,
  fonts, and audio assets referenced from rich content.
- `P0-AC-009`: a hyperlink uses an approved-scheme allowlist, launches through
  the platform API only after an explicit user gesture, and never constructs or
  interpolates a shell command.
- `P0-AC-010`: hostile Markdown or rich input becomes a constrained AST or scene
  representation with no WebView script execution; resources load only through
  the `P0-AC-005` policy.
- The transcript and chat layout are out of scope for this document. The
  rendering and security controls above are still inherited by any headed
  surface that displays them.
- Adversarial tests must cover script-bearing Markdown/HTML, local-resource
  references, dangerous and encoded URI schemes, shell metacharacters, and
  oversized or deeply nested scenes before any rich surface ships.

## Ownership matrix

Owner labels are repository roles, not team names: **Core owner** is
`bitty`/future `bittyd`; **AI owner** is `bitty-ai`; **Wheel owner** is `wheel`;
**Headed Agent owner** is the future sibling plugin repository; **Media
extension owner** is the future audio/avatar/3D extension repository; **Provider
plugin owner** is the provider adapter repository. Each row is a review item for
the named owner.

Review roles are cross-cutting, not additional implementation repositories:
**Security owner** is the security reviewer role, **Product owner** is the
product/docs review role, **Accessibility owner** is the accessibility review
role, **terminal docs owner** is `bitty-terminal-docs`, and **Plugin ecosystem
owner** is `bitty-plugins-docs` plus the plugin repositories. **Architecture
owner** and **Documentation curator** are the existing architecture-review and
docs-curation roles. A review role can own a review row without owning
implementation code; the matrix columns still name the repository accountable
for the code.

Wheel 0.1.0 is a standalone harness with no host IPC client, no plugin
packaging, and no credential-handle consumption (Wheel owner decisions D1, D2,
D6). Every Wheel cell below is a post-0.1.0 sibling design target, not a
current role, and the column header says so.

| Concern                                     | Core owner (`bitty`; future `bittyd`)                                                 | AI owner (`bitty-ai`)                                                                                                                                   | Wheel owner (post-0.1.0 target)       | Headed Agent owner                     | Media extension owner             | Provider plugin owner                     |
| ------------------------------------------- | ------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------- | -------------------------------------- | --------------------------------- | ----------------------------------------- |
| IPC framing, auth, instance, rate limits    | Owner                                                                                 | Consumer                                                                                                                                                | None in 0.1.0; post-0.1.0 target      | Consumer                               | Consumer through Core             | Consumer through AI and Core              |
| Consent ledger and revocation               | Owner                                                                                 | Requests verdicts; never stores grants                                                                                                                  | None in 0.1.0; post-0.1.0 target      | Presents prompts; never grants         | Consumes verdicts                 | Consumes verdicts                         |
| Agent session state machine                 | Host-side transport; state-machine placement open                                     | Consumes the RFC state names; `bitty-agent-api` supplies only `AgentId`/`AgentError`; runtime has its own states including `Canceled`; consumption open | None in 0.1.0; post-0.1.0 target      | Projects sanitized view                | None                              | None                                      |
| Session handle and generation               | Owner of issuance placement target; sequencer under the issuer-as-sequencer invariant | Binds model-facing state to the session generation                                                                                                      | None in 0.1.0; post-0.1.0 target      | Consumer                               | None                              | None                                      |
| Event sequencing and replay window          | Issuer-as-sequencer for the session stream                                            | Owns AI event schemas and nested turn sequencing                                                                                                        | None in 0.1.0; post-0.1.0 target      | Consumer                               | Consumes media events             | None                                      |
| Provider registry and routing               | Owns publication authority, scopes, and transport                                     | Owns the authoritative registry, resolution, and opaque proof                                                                                           | None in 0.1.0; post-0.1.0 target      | Consumer; no provider selection        | None                              | Registers; never widens                   |
| Credentials and secret storage              | Host-owned storage and grants                                                         | Uses opaque handles; never reads values                                                                                                                 | None in 0.1.0; post-0.1.0 target only | Never receives values                  | Never receives values             | Resolves at the adapter edge only         |
| Context compilation and token budget        | Enforces quotas                                                                       | Owner                                                                                                                                                   | None in 0.1.0; post-0.1.0 target      | Consumer                               | None                              | None                                      |
| Context and session snapshots               | Owns host record and retention policy                                                 | Owns compilation, redaction, and snapshot view                                                                                                          | None in 0.1.0; post-0.1.0 target      | Reads sanitized view                   | None                              | None                                      |
| Tool proposal                               | Receives and validates                                                                | Owner of model proposal relay                                                                                                                           | None in 0.1.0; post-0.1.0 target      | Presents                               | None                              | None                                      |
| Tool approval, execution, result            | Owner of ledger and execution                                                         | Consumes result as untrusted observation                                                                                                                | None in 0.1.0; post-0.1.0 target      | Presents request and collects decision | None                              | None; tool providers are separate plugins |
| Panel identity, lifecycle, focus            | Owner under the accepted Panel Runtime RFC                                            | Reads references only                                                                                                                                   | None in 0.1.0; post-0.1.0 target      | Requests panel and composes UI         | Renderer consumes a panel surface | None                                      |
| Event Bus transport and reserved topics     | Owner under the accepted Panel Runtime RFC                                            | Publishes AI-domain topics                                                                                                                              | None in 0.1.0; post-0.1.0 target      | Publishes only its private topics      | Publishes declared media topics   | None                                      |
| Speech job lifecycle                        | Enforces consent and transport                                                        | Owner of request, stream, cancel, asset ref                                                                                                             | None                                  | Owns eligibility and controls          | Owns playback                     | Synthesizes under the speech contract     |
| Audio device and playback                   | May provide a presentation primitive; none claimed                                    | No ownership                                                                                                                                            | None                                  | Requests playback                      | Owner of the device               | None                                      |
| Avatar semantic state and viseme vocabulary | No ownership                                                                          | Owner of neutral event vocabulary                                                                                                                       | None                                  | Maps to appearance and animation       | Renders 2D/3D                     | May emit timing metadata                  |
| 3D rig, shaders, and GPU surfaces           | No ownership                                                                          | No ownership                                                                                                                                            | None                                  | May request                            | Owner                             | None                                      |
| CPU, memory, GPU, audio budgets             | Host and transport budgets                                                            | AI and media-job budgets                                                                                                                                | None in 0.1.0; post-0.1.0 target      | Presentation budgets                   | Device and decoder budgets        | Provider quota reporting                  |
| Platform audio/GPU adapters                 | Host integration where a host adapter exists                                          | No platform API                                                                                                                                         | None in 0.1.0; post-0.1.0 target      | No platform API                        | Device-specific adapter owner     | No platform API                           |
| Coding workflow and task graph              | No ownership                                                                          | No ownership                                                                                                                                            | Owner                                 | No `wheel.*` dependency                | None                              | None                                      |
| Retention and deletion                      | Host evidence and consent                                                             | AI record semantics under draft R3                                                                                                                      | Its own records                       | Ephemeral local media by default       | No persistent media by default    | Not owned here; provider policy governs   |

## Security review

No security review has been completed for this draft. This section records the
threat model and the gates that a later review must close; it is a review input,
not a sign-off. The draft must remain `draft` until every row in
[P0 Review Sign-off](#p0-review-sign-off) records a disposition. That nine-row
table is the single authoritative sign-off set; this section does not create a
smaller or alternate role list.

### Threat analysis

| Threat                                      | Impact                                                                                  | Required gate                                                                                                                                  | Accountable owner(s)                          |
| ------------------------------------------- | --------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------- |
| Presentation becomes authority              | A panel, avatar, or attachment can approve or execute without consent.                  | Presentation-never-authority: every effect re-checks the caller's grants.                                                                      | Core owner + Headed Agent owner               |
| Stale session or attachment generation      | Old approval, stop, or attachment revives a newer session.                              | Session and attachment generation fencing on every mutating operation and event.                                                               | Core owner + AI owner                         |
| Approval substitution                       | A user approves one call and a different call executes.                                 | Tool identity and version, canonical argument digest, Core-authored prompt, target, scope, expiry, and session-generation binding.             | Core owner                                    |
| Same-UID principal confusion                | A process under the same UID presents another `AgentId` or takes over a session.        | `AgentId` is a validated declaration, not transport authentication; attach and takeover re-authorize grants and consent; cross-principal test. | Core owner + Security owner                   |
| Speech egress substitution or replay        | Text, provider, or privacy class changes after approval, or an authorization is reused. | Core-issued one-shot authorization bound to text digest, provider, privacy class, session generation, principal, and expiry; per-use audit.    | Core owner + AI owner                         |
| Approval prompt spoofing                    | A presentation surface rewrites the prompt while the bound call differs.                | Core-authored canonical prompt bound to the digest; a mismatch fails closed before the verdict is recorded.                                    | Core owner + Headed Agent owner               |
| Rich rendering escape                       | Hostile Markdown, URI, or local resource reaches a desktop capability.                  | `P0-AC-005`, `P0-AC-009`, and `P0-AC-010` controls plus an adversarial corpus.                                                                 | Headed Agent owner + security reviewer        |
| Secret leakage through snapshots or events  | Credentials or environment values reach a model, log, or UI.                            | Typed redaction before queueing and writing; presence markers only.                                                                            | Core owner + AI owner                         |
| Unbounded media parsing                     | A malicious audio or avatar asset exhausts memory or executes code.                     | Mandatory isolated decode, depth limits, and fuzzed decoders; fail closed when isolation is unavailable.                                       | Media extension owner                         |
| Prompt injection into speech or approval    | Model output causes an egress or an approval.                                           | Egress filtering; approval requires explicit user action, never model text.                                                                    | AI owner + Core owner + Headed Agent owner    |
| Cross-panel eavesdropping or topic spoofing | One plugin reads or forges another plugin's stream.                                     | Reserved namespaces, per-topic authorization, producer identity checks.                                                                        | Core owner                                    |
| Privacy downgrade on fallback               | A local-only request silently goes remote.                                              | Privacy class is a hard constraint; fallback stays within it and never bypasses Core publication authority.                                    | AI owner                                      |
| Resource exhaustion or denial of service    | One session starves audio, GPU, or the host.                                            | Hard caps, per-session attribution, containment, and slow-consumer shed.                                                                       | Core owner + AI owner + Media extension owner |
| GPU or audio device data retention          | One principal's media remains in device memory for another.                             | Release and clear on detach, suspend, and session end.                                                                                         | Media extension owner                         |
| `wheel.*` coupling                          | The Headed Agent plugin reaches into Wheel private state.                               | Dependency review; no `wheel.*` import, call, config, or environment key.                                                                      | Headed Agent owner + security reviewer        |

### Security gates

| Gate                             | Requirement                                                                                                                                                                                         | Evidence required before implementation                                                                                                           |
| -------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| Presentation-never-authority     | A panel, attachment, avatar state, animation, or speech control cannot approve, execute, or broaden a scope.                                                                                        | Adversarial tests that attempt approval and execution from every presentation surface.                                                            |
| Generation fencing               | Stale session, attachment, approval binding, and media-job identities fail closed before effects.                                                                                                   | Property tests and replay corpus across suspend, reconnect, takeover, and restart.                                                                |
| Approval binding                 | Approval binds the exact tool identity and version, validated call digest, Core-authored prompt, target, scope, expiry, session generation, and principal.                                          | Mutation, prompt-rewrite, replay, expiry, and cross-principal test matrix.                                                                        |
| Speech authorization             | Every speech egress consumes one Core-issued authorization bound to the text digest, provider, privacy class, session generation, principal, and expiry.                                            | Reuse, digest-mismatch, provider-change, privacy-change, stale-generation, cross-principal, and compromised-plugin tests; per-use audit evidence. |
| Same-UID boundary                | `AgentId` is a validated declaration inside the UID trust domain; attach and takeover re-authorize grants, and no confidentiality between same-UID processes is claimed.                            | Cross-principal attach and takeover tests, plus a documented same-UID confidentiality boundary.                                                   |
| Secret-free surfaces             | Snapshots, events, speech text, avatar state, and traces contain no raw secret or unapproved environment value.                                                                                     | Seeded-secret negative tests on every producer and every write path.                                                                              |
| Bounded media                    | Audio, viseme, animation, and avatar inputs are size-, depth-, and time-bounded; decode is mandatorily isolated or fails closed.                                                                    | Fuzz corpus, malformed asset suite, isolation proof, and fail-closed presentation-class tests.                                                    |
| Consent before egress            | Model, speech, and tool egress require the matching grant; autoplay is off unless an explicit user-owned policy enables it; revocation stops the next chunk.                                        | Consent matrix and revocation lifecycle tests.                                                                                                    |
| Retention and erasure            | Media and snapshots are ephemeral by default; recording is consented and redacted; source-history deletion propagates to derived media.                                                             | Retention, export-preview, and deletion-propagation tests.                                                                                        |
| Rich rendering controls          | Any Markdown, rich-block, image, or scene surface uses the `P0-AC-005` loader, the `P0-AC-009` URI and launch policy, and the `P0-AC-010` constrained AST/scene with no WebView script.             | Script, local-resource, hostile-URI, and constrained-scene corpus on every rich surface.                                                          |
| Capability fallback fails closed | A missing or unverified platform capability lowers presentation only; it never weakens security, consent, or redaction.                                                                             | Platform matrix tests and capability-detection negative tests.                                                                                    |
| No Wheel coupling                | The Headed Agent plugin has no `wheel.*` import, service call, CLI spawn/wrapper, `.wheel/init.lua` or Wheel config/state read, configuration dependency, environment key, or private state access. | Dependency graph review, path-access review, and a negative integration test per forbidden path.                                                  |
| Independent review               | Every row in [P0 Review Sign-off](#p0-review-sign-off) differs from the implementer and records a disposition before implementation tasks open.                                                     | Nine review records linked from the owning task.                                                                                                  |

### Residual risks

- No implementation exists, so every gate above is unverified.
- Media sandboxing and GPU memory isolation are platform-dependent. A
  presentation class that cannot verify the required isolation is disabled
  rather than treated as contained, so the residual risk is reduced
  presentation, not unaudited media parsing.
- Voice provenance, watermarking, and synthetic-speech disclosure are not
  defined; a future speech contract must decide them.
- Multi-user, multi-device, and remote headed clients are deferred. They would
  need their own authentication and consent ledger review.
- The exact Core-versus-`bitty-ai` split for session identity and durable
  records remains open; the issuer-as-sequencer invariant prevents two
  components from advancing one generation, but a wrong durable placement could
  still duplicate records.
- The accepted transport authenticates the UID, not the `AgentId`. Processes
  under the same UID are not isolated from each other, and this draft claims no
  confidentiality between them; that boundary needs its own accepted decision
  if same-UID isolation is required.
- The plugin packaging model for media extensions is not decided, so supply
  chain and permission review for those extensions remain future work.

## Verification plan

This repository is documentation-only. The current evidence for this document is
the local documentation gate (`just check`) plus independent review of the
diff. No implementation, performance, platform, or security evidence exists.

Future implementation tasks must add, per contract family once it is accepted:

- Schema and version-negotiation tests: session-bound selection, attach
  compatible subset, standalone `headed.platform` probe, unknown major
  rejection, additive minor tolerance, persisted-event readability, and
  rejection of a lower minor that changes authority, ordering, redaction,
  consent, bounds, or failure semantics.
- Lifecycle property tests for `start`, `attach`, `detach`, `stop`, session
  `resume`, `takeover`, generation mismatch, idempotency-key replay and expiry,
  issuer-to-sequencer handoff, panel/attachment lifecycle mapping, and
  cancellation before and after dispatch.
- Event-ordering and resynchronization tests: non-lossy gaps, lossy lost-count
  recovery, duplicates, snapshot barriers, replay-window boundaries, and
  reconnect during a turn.
- Event Bus tests: accepted `owner.name:topic` grammar, manifest-declared
  produce/consume, `UndisclosedTopic`, PR-3 through PR-9 bounds, `DropOldest`
  counting and attribution, per-principal ledgers, and the stricter control
  class refusal.
- Backpressure and soak tests: slow consumer, bounded queues, counted drops,
  audio underflow, and no unbounded memory growth.
- Approval tests: exact tool identity and version, digest binding, Core-authored
  prompt integrity, argument mutation, target change, expiry, cross-principal
  replay, and reconnect with a pending approval.
- Snapshot tests: bounded size, redaction, viewer filtering, immutable
  versions, separation of routing/context/session snapshots, CP-5/AG-2
  generation binding, and expansion re-authorization.
- Speech tests: one-shot authorization bound to text digest, provider, privacy
  class, session generation, principal, and expiry; reuse, digest mismatch,
  provider or privacy change, cross-principal, and compromised-plugin failures;
  per-use audit; redaction; first-chunk timeout; cancellation at every boundary;
  exact eight-state status set; `Unknown` reconciliation; asset expiry and
  idempotency; and no auto-play after reconnect without a user-owned policy.
- Takeover tests: target grants and consent re-authorization, old-generation
  media-job reconciliation, and a cross-principal attempt from the same UID and
  from a different UID.
- Avatar and viseme tests: latest-wins state, bounded animation queue, clock
  tolerance, loss degradation, reduced motion, egress prohibition, buffer
  clearing, and release on suspend.
- Budget tests: per-dimension hard and soft limits, `FS-AI4` attribution
  fields, `RC-9`/`RC-10` and PR-7 through PR-9 inheritance, degradation, and
  isolation between sessions.
- Rich rendering tests: the `P0-AC-005`, `P0-AC-009`, and `P0-AC-010`
  adversarial corpus on every Markdown, rich-block, image, and scene surface.
- Platform matrix tests: Linux, macOS, Windows, and FreeBSD capability
  detection, ADR-0002 support-tier alignment, missing device, device loss,
  decode-isolation failure, and fallback to text or headless.
- Wheel boundary tests: a negative integration test for `wheel.*` import, CLI
  spawn or wrapper, `.wheel/init.lua`, `$XDG_CONFIG_HOME/wheel/init.lua`, and
  `$XDG_STATE_HOME/wheel/trusted_projects.json`.
- Security adversarial tests for every gate in the security review table.

A future contract is not accepted because this document exists. Each family
needs its own accepted specification, owner, implementation task, evidence, and
independent review.

## Alternatives considered

| Alternative                                                    | Why it is not selected                                                                                                                                                            |
| -------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Put headed UI, speech, avatars, or 3D in Wheel                 | Couples high-churn presentation and media to the coding harness, expands Wheel's product scope, and contradicts the requirement that Wheel remain a focused Coding Agent Harness. |
| Let the Headed Agent depend on Wheel as a library or front end | Creates a private cross-consumer dependency, makes Wheel's internal API a de facto contract, and prevents independent evolution. Shared needs belong in Core or `bitty-ai`.       |
| Put the Headed Agent inside Core                               | Makes the terminal platform own AI composition, chat policy, and media presentation, violating the Core-versus-AI boundary and the no-model-I/O rule for Core.                    |
| One global event stream with a total order                     | Creates head-of-line blocking, scales poorly across sessions and panels, and turns unrelated plugins into a shared failure domain. Per-session and per-topic ordering suffice.    |
| Drop-oldest delivery for approvals and tool results            | Can silently lose an approval, a denial, or a result, which is an authority and safety defect. Control events must be non-lossy.                                                  |
| Let the plugin resolve providers and hold credentials          | Fragments routing and consent, exposes secrets to presentation code, and bypasses the draft provider boundary and secret invariant.                                               |
| Let Core render avatars and play speech directly               | Makes platform code own media policy, GPU lifetime, and accessibility behavior that belong to media extensions and the Headed Agent owner.                                        |
| Inline audio bytes in model context                            | Bypasses context budgets and redaction, bloats snapshots and events, and contradicts the asset-reference rule in the multimodal boundary.                                         |
| Require 3D or audio for a headed session                       | Excludes headless, low-resource, accessibility, and FreeBSD-style environments. Text, captions, and 2D/software fallbacks are mandatory.                                          |
| Treat panel closure as session stop                            | Conflates presentation with lifecycle and lets UI absence destroy work. Panel closure detaches; stopping is explicit.                                                             |

## Affected contracts

No contract is changed by this draft. The table records the expected
relationship for the owning documents and repositories.

| Contract or surface                                                                                                                          | Owner                            | Effect of this draft                                                                                                     | Status                                          |
| -------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------- | ------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------- |
| [IPC and Agent RFC](../specifications/ipc-agent-rfc.md)                                                                                      | Core owner                       | Consumed unchanged for framing, auth, scopes, bounded messages, consent, streaming, and versioning.                      | Accepted; no change                             |
| [Panel Runtime RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/panel-runtime-rfc.md)                     | Core owner                       | Consumed unchanged for panel identity, lifecycle, focus, Event Bus grammar, and PR-1 through PR-12.                      | Accepted; no change                             |
| [RFC-0006](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/rfcs/RFC-0006-panel-provider-registration-mount.md)         | Core owner                       | Consumed unchanged for plugin-facing registration, mount placement, and the closed `panel.*` capability mapping.         | Accepted; no change                             |
| [Provider-routing snapshot binding](../architecture/execution-ownership-r1.md#providerrouting-snapshot-binding-contract-accepted-2026-10-17) | Core owner + AI owner            | Consumed unchanged for routing snapshot domain, bytes, digest, publication authority, and ownership proof.               | Accepted section; no change                     |
| [ADR-0013](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/adrs/ADR-0013-core-ontology-identity.md)                    | bitty-docs decision owner        | Consumed for ontology, `AgentId`, `GenerationId`, and the Panel/Execution separation.                                    | Accepted; no change                             |
| [ADR-0002](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/adrs/ADR-0002-platform-support-tiers.md)                    | bitty-docs decision owner        | Consumed for support tiers; presentation classes never upgrade a support tier.                                           | Accepted; no change                             |
| [P0 acceptance criteria](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/p0-acceptance-criteria.md)                     | Security owner                   | Rich, avatar, and media surfaces inherit `P0-AC-005`, `P0-AC-009`, and `P0-AC-010`.                                      | Normative; no change                            |
| [Isolation and Resource RFC](https://github.com/bitty-terminal/bitty-plugins-docs/blob/main/runtime/isolation-resource-rfc.md)               | Security owner                   | Inherits `RC-1`, `RC-2`, `RC-9`, and `RC-10`; this draft adds no weaker bound.                                           | Accepted; no change                             |
| [Unified authorization backend](../architecture/tool-transport-r2.md#unified-authorization-backend)                                          | Core owner                       | Tool approval events must map to the existing gate order; no new authorization path.                                     | Draft; no change                                |
| [Context retention R3](../architecture/context-retention-r3.md)                                                                              | Core owner                       | Speech, viseme, avatar, and attachment data follow the draft consent, retention, and deletion rules.                     | Draft; no change                                |
| [Provider plugin boundary](../providers/provider-plugin-boundary.md)                                                                         | AI owner + Provider plugin owner | Consumed unchanged for privacy classes, capabilities, secret invariant, and project overrides.                           | Draft; no change                                |
| [Multimodal inference boundary](../providers/multimodal-inference-boundary.md)                                                               | AI owner                         | Draft speech and generation-job lifecycle reused; this draft adds only headed request, playback, and cancellation rules. | Draft; review follow-up                         |
| [Agent coordination architecture](agent-coordination.md)                                                                                     | AI owner + Core owner            | Aligns panel attachment, suspend, and lifecycle outcomes with the existing reconciliation.                               | Draft; candidate alignment                      |
| [Panel environment awareness](../interfaces/panel-environment-awareness.md)                                                                  | AI owner + Core owner            | Sanitized Agent View and no-persistence defaults continue to apply to headed presentation.                               | Draft; no change                                |
| [Execution-projection binding](../specifications/execution-projection-binding-candidate.md)                                                  | Core owner + AI owner            | Candidate design reference only; session attachment is distinguished from execution projection.                          | Draft; no change                                |
| [Wheel core and plugin boundary](../specifications/wheel-core-plugin-boundary-candidate.md)                                                  | Wheel owner + AI owner           | Wheel's internal composition candidate, not this sibling boundary; no Wheel work is assigned.                            | Draft; no change to Wheel                       |
| Core panel and Event Bus implementation                                                                                                      | Core owner                       | The panel and Event Bus contracts are accepted by the Panel Runtime RFC and RFC-0006; implementation is not claimed.     | Accepted contract; implementation owner-pending |
| Plugin packaging for media extensions                                                                                                        | Plugin ecosystem owner           | Would need registry, manifest, capability, and supply-chain review; not defined here.                                    | Owner-pending                                   |
| Speech and avatar event schemas                                                                                                              | AI owner                         | Candidate families only; no wire schema, field list, or compatibility promise is accepted.                               | Draft                                           |

### Cross-repository references

The owning repositories are linked once, here. The [References](#references)
section carries the local canonical documents and the security entry point.

- [bitty-docs](https://github.com/bitty-terminal/bitty-docs) owns shared
  governance, the decision register, the open-question register, and the
  normative security corpus. This document links to those owners instead of
  copying or reinterpreting them.
- [bitty-terminal-docs](https://github.com/bitty-terminal/bitty-terminal-docs)
  owns the terminal platform's accepted panel runtime, panel presentation,
  rich presentation, and platform-support contracts. Panel identity, lifecycle,
  and Event Bus references here point at that accepted contract; audio/GPU
  mechanism references remain owner-pending pointers to that repository.
- [bitty-plugins-docs](https://github.com/bitty-terminal/bitty-plugins-docs)
  owns the plugin platform, manifest, capability, and packaging contracts.
  Media-extension packaging and permission review belong there. The accepted
  isolation envelope lives at
  [Isolation and Resource RFC](https://github.com/bitty-terminal/bitty-plugins-docs/blob/main/runtime/isolation-resource-rfc.md).
- [bitty-ai](https://github.com/bitty-terminal/bitty-ai) owns implementation
  evidence for the AI runtime and providers. This document claims no
  implementation status from that repository.
- [wheel](https://github.com/bitty-terminal/wheel) owns the Coding Agent
  Harness. Its
  [README](https://github.com/bitty-terminal/wheel/blob/main/README.md),
  [repo.toml](https://github.com/bitty-terminal/wheel/blob/main/repo.toml),
  [SECURITY-MODEL.md](https://github.com/bitty-terminal/wheel/blob/main/SECURITY-MODEL.md),
  and [AGENTS.md](https://github.com/bitty-terminal/wheel/blob/main/AGENTS.md)
  are the authoritative sources for its 0.1.0 position; this document records
  a sibling boundary only and assigns no Wheel implementation work.
- [bitty](https://github.com/bitty-terminal/bitty) owns Core implementation
  evidence. Future `bittyd`, Core panel, Event Bus, and platform-adapter work
  require their own accepted contracts; no v1.0 `bittyd` binary is claimed.

## Open points

None of these is resolved by this draft. Each needs an owner, a reviewed
decision, and evidence before the related implementation task opens. This table
is a review index: the `Blocks` column names the contract families whose
acceptance depends on the entry, but the table is not itself a blocker register
and carries no authority. Every row names a register, a tracked issue, a review
route, or an explicit deferral; promotion runs through AI-0347
([bitty-ai#553](https://github.com/bitty-terminal/bitty-ai/issues/553)) and the
named owner.

| Open point                                     | Question                                                                                                                                                                                                       | Likely owner                                 | Disposition or deferral                                                                                                                                                          | Blocks                                                              |
| ---------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------- |
| Session authority split                        | Does Core or `bitty-ai` own the durable session handle, generation issuance, the reconnect record, and consumption of the accepted state machine?                                                              | Core owner + AI owner                        | [bitty#2546](https://github.com/bitty-terminal/bitty/issues/2546) (bittyd session-host contract); OQ-061 covers identity/projection only; deferred until #2546 decides placement | `headed.session`, `headed.stream`, `headed.context`, `headed.tools` |
| Headed plugin identity and packaging           | Is the Headed Agent a Lua plugin, an out-of-process component, or a composed distribution? Who signs and updates it?                                                                                           | Plugin ecosystem owner                       | `AI-0347/#553` then plugin-ecosystem review in `bitty-plugins-docs`; deferred until a plugin packaging contract exists                                                           | Plugin packaging, media extension packaging                         |
| Coding capability access                       | If the Headed Agent needs coding intelligence, does it use only public Core/`bitty-ai` contracts, or must the needed mechanism be promoted there? A `wheel.*` runtime dependency is not authorized either way. | AI owner + Core owner; Wheel consulted       | Promote to a public Core/`bitty-ai` contract; Wheel decisions D4/D6 record no host-brokered execution service in 0.1.0; `AI-0347/#553`; deferred                                 | `wheel.*` prohibition, product scope                                |
| Replay retention window                        | How long are control events retained for reconnect, and what storage or memory budget applies?                                                                                                                 | Core owner + AI owner                        | `AI-0347/#553` for promotion to a Core/AI retention task; AIQ-04/55/5A are adjacent closed(partial) facets; deferred                                                             | `headed.stream`, Event Bus control retention                        |
| Multi-user and remote headed clients           | Are remote, multi-device, or shared-panel sessions in scope? What authentication and consent model applies?                                                                                                    | Core owner + Security owner                  | ADR-0008 (`OQ-020` closed, prior decision); out of scope for 0.1.0; promotion tracked by `AI-0347/#553`                                                                          | Identity, consent, transport                                        |
| Speech privacy classes and provider consent    | Which speech providers are local-only, which require egress consent, and what is the minimum baseline?                                                                                                         | AI owner + Security owner                    | `AI-0347/#553` plus AI/security review; deferred until an accepted speech contract defines the classes                                                                           | `headed.speech`                                                     |
| Viseme source and accuracy                     | Is the viseme timeline provider metadata, local analysis, or both? What accuracy and tolerance claims are allowed?                                                                                             | AI owner + Media extension owner             | `AI-0347/#553` plus media/architecture review; deferred                                                                                                                          | `headed.avatar`, `headed.speech`                                    |
| 3D renderer placement                          | Is 3D an in-process extension, a separate process, or a remote surface? How is GPU memory isolated between principals?                                                                                         | Media extension owner + Architecture owner   | `AI-0347/#553` plus plugin-ecosystem packaging review; deferred                                                                                                                  | `headed.avatar`, `headed.platform`                                  |
| Audio focus policy                             | Is audio focus per panel, per session, or global? What happens when two sessions speak at once?                                                                                                                | Core owner + Headed Agent owner              | `AI-0347/#553` for promotion to Core host policy; per-device exclusivity is fixed, scope is deferred                                                                             | `headed.panels`, `headed.speech`                                    |
| Platform support evidence                      | Which Linux, macOS, Windows, and FreeBSD configurations are actually supported, and who records the evidence?                                                                                                  | Core owner + terminal docs owner             | ADR-0002 accepted tiers; media-specific evidence routes to `bitty-terminal-docs`; `AI-0347/#553`; deferred                                                                       | `headed.platform`                                                   |
| Budget defaults                                | What are the numeric CPU, memory, GPU, audio, token, and cost defaults, and who may change them?                                                                                                               | Core owner + AI owner + Security owner       | `OQ-066` for the context token profile; `AI-0347/#553` for media and host budgets; deferred                                                                                      | `headed.budget`, `headed.stream`, Event Bus queues                  |
| Approval lifetime and standing grants          | How long does a one-shot approval live, and are session-wide or tool-wide grants allowed?                                                                                                                      | Core owner + Security owner                  | `AI-0347/#553` plus security review; OQ-018 is closed for consent scope, not approval lifetime; deferred                                                                         | `headed.tools`, `headed.session`                                    |
| Synthetic speech disclosure                    | Is watermarking, provenance metadata, or a spoken disclosure required for synthesized audio?                                                                                                                   | Security owner + Product owner               | `AI-0347/#553` plus security/product review; deferred                                                                                                                            | `headed.speech`                                                     |
| Recording and export of speech or avatar state | Can users record, export, or delete synthesized audio and viseme data, and under which retention class?                                                                                                        | Core owner + Security owner                  | `AI-0347/#553`; AIQ-55/5A adjacent closed(partial) facets; deferred until draft R3 is accepted                                                                                   | `headed.speech`, `headed.avatar`                                    |
| Shared chat or transcript contract             | Do Wheel and the Headed Agent share a neutral presentation contract in Core or the plugin ecosystem, or only `bitty-ai` events plus Core panels? A `wheel.*` dependency is excluded.                           | Wheel owner + Plugin ecosystem owner         | `OQ-056` (presentation projection); otherwise `AI-0347/#553`; deferred                                                                                                           | UI composition, duplication risk                                    |
| Accessibility baseline                         | Which reduced-motion, caption, contrast, and screen-reader behaviors are mandatory versus optional?                                                                                                            | Accessibility owner + Plugin ecosystem owner | `AI-0347/#553` plus accessibility review (`bitty-a11y`); deferred                                                                                                                | `headed.avatar`, `headed.speech`                                    |

The existing panel-identity and projection questions in the
[AI Unresolved Questions](../product/ai-unresolved-questions.md) register remain
open; this document creates no register identifier and closes none of them. The
register admits no task-level identifier minting, so an entry without a register
row carries an explicit deferral above and is promoted through AI-0347 rather
than asserted here as an authoritative decision.

### Review follow-ups

The following items need an owner decision or implementation evidence that this
docs-only correction cannot supply. They remain open and are not silently
closed:

- The [AI Architecture](../architecture/ai-architecture.md) still labels
  SMO-1 through SMO-6 as candidate while the OQ-058 owner ruling accepted the
  SMO-1 through SMO-4 topology; the two repositories need a status
  reconciliation under `AI-0347/#553`.
- A forward alias projection remains undecided. Until Core and the AI owner
  accept one, aliases stay outside the strict path and any alias-table mutation
  invalidates the binding.
- Same-UID confidentiality is not provided by the accepted transport. If it is
  required, it needs its own accepted authentication contract.
- The standalone `headed.platform` probe needs an accepted authentication,
  rate-limit, and audit shape in the owning Core contract.
- Wheel's post-0.1.0 host IPC client, consumer roles, and credential-handle use
  are design targets only; they need Wheel owner decisions before any consumer
  cell becomes current.
- These tests cannot exist before implementation and are owned by future
  implementation tasks: the `wheel.*` CLI/config negative test, cross-principal
  attach/takeover, compromised-plugin speech authorization, rich-rendering
  adversarial corpus, and the nine P0 review records.

## Acceptance criteria

This design is acceptable as a draft when all of the following hold:

- The document lives in the `agent/` tree, follows the specification spine, and
  carries the required flat frontmatter and status qualifiers.
- The boundary is explicit: Wheel 0.1.0 is an independent Coding Agent Harness
  and not a Bitty plugin; the Headed Agent is a sibling plugin consuming Core
  and `bitty-ai`; no `wheel.*` dependency is defined; no headed UI, speech,
  avatar, or 3D work is assigned to Wheel.
- Each contract family has an owner, a candidate version, versioning rules, and
  an explicit statement that it is not accepted.
- The accepted IPC and Agent RFC, Panel Runtime RFC, RFC-0006, the scoped
  provider-routing binding, OQ-058 SMO-1 through SMO-4, ADR-0013, ADR-0002, the
  P0 criteria, and the Isolation and Resource RFC are consumed by reference and
  not restated or weakened.
- Session lifecycle, session and attachment generation, the issuer-as-sequencer
  handoff, two-level session/job outcomes, cancellation, event ordering,
  backpressure, and reconnect are covered without redefining the accepted
  `AgentSession` or IPC RFC.
- Provider routing, context/session snapshots, and tool-call/approval events are
  covered without adding a provider loader, credential path, or second
  generation-job lifecycle.
- Panel and Event Bus topics, panel-versus-attachment lifecycle, multi-panel
  suspend/resume, and the Core-versus-plugin split are covered by consuming the
  accepted contracts without asserting implementation.
- Speech and avatar contracts cover request, stream, cancel, state, animation,
  and visemes without implementing TTS, rendering, or 3D.
- Resource budgets, platform fallback, security, privacy, and consent are
  covered, with security gates and residual risks explicit.
- The ownership matrix and cross-repository references are complete enough for
  the owning repositories to review their own rows.
- Open points are explicit, and none is silently decided across a public
  contract boundary. Each entry names a register, a tracked issue, a review
  route, or an explicit deferral, and the review follow-ups are listed.
- The local documentation gate passes, and every row in
  [P0 Review Sign-off](#p0-review-sign-off) records a disposition before any
  implementation task is opened.

Acceptance of this draft does not accept any contract family for
implementation. Each family needs its own accepted specification and evidence.

## P0 Review Sign-off

This nine-row table is the **single authoritative sign-off set** for this
document. Earlier and later summaries that mention approval, review, or a gate
refer to these rows and do not define a smaller role list.

| Role                         | Reviewer | Status                        | Evidence                                                                                                                                               |
| ---------------------------- | -------- | ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Architecture owner           | Pending  | Pending — NEEDS-FIX (PX-1552) | Independent architecture review of accepted-source citations, R1 routing split, CP-5/AG-2 binding, session outcomes, and Event Bus. Re-review pending. |
| Security reviewer            | Pending  | Pending — NEEDS-FIX (PX-1553) | Threat model, gates, alias routing, speech authorization, attach/takeover, and rich-rendering controls. Re-review pending.                             |
| Core / future `bittyd` owner | Pending  | Pending                       | Panel, Event Bus, consent, transport, and session issuance/sequencer rows.                                                                             |
| `bitty-ai` owner             | Pending  | Pending                       | Routing publication/proof split, context, speech, avatar, and budget rows.                                                                             |
| Wheel owner                  | Pending  | Pending — NEEDS-FIX (PX-1550) | Sibling boundary, authoritative Wheel citations, and the `wheel.*` prohibition including CLI and config/state paths. Re-review pending.                |
| Plugin ecosystem owner       | Pending  | Pending                       | Packaging, media extension, and capability rows.                                                                                                       |
| Product owner                | Pending  | Pending                       | Product scope, open-point dispositions, and the no-acceptance claim.                                                                                   |
| Accessibility owner          | Pending  | Pending                       | Captions, reduced motion, contrast, and screen-reader baseline.                                                                                        |
| Documentation curator        | Pending  | Pending — NEEDS-FIX (PX-1551) | Structure, metadata, links, self-containment, terminology, taxonomy, navigation, and work-register sync. Re-review pending.                            |

No P0 sign-off has been recorded. The four `NEEDS-FIX` entries are review
evidence only, not approvals; re-review is pending. The document remains
`draft` until all nine rows record dispositions and any defects are resolved or
tracked.

## References

Accepted and normative sources outside this repository:

- [Panel Runtime RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/panel-runtime-rfc.md)
  — accepted (2026-09-14).
- [RFC-0006 Panel Provider Registration Mount](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/rfcs/RFC-0006-panel-provider-registration-mount.md)
  — accepted (2026-10-07).
- [OQ-058 in the open-question register](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/open-questions.md)
  — accepted owner ruling (2026-09-23), SMO-1 through SMO-4 topology.
- [ADR-0013 Core Ontology and Identity Model](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/adrs/ADR-0013-core-ontology-identity.md)
  — accepted (2026-09-23).
- [ADR-0002 Platform Support Tiers](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/adrs/ADR-0002-platform-support-tiers.md)
  — accepted (2026-08-26).
- [P0 Security Acceptance Criteria](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/p0-acceptance-criteria.md)
  — normative.
- [Isolation and Resource RFC](https://github.com/bitty-terminal/bitty-plugins-docs/blob/main/runtime/isolation-resource-rfc.md)
  — accepted (2026-08-28).
- [Rich Presentation RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/rich-presentation-rfc.md)
  — accepted.
- [Wheel README](https://github.com/bitty-terminal/wheel/blob/main/README.md),
  [repo.toml](https://github.com/bitty-terminal/wheel/blob/main/repo.toml),
  [SECURITY-MODEL.md](https://github.com/bitty-terminal/wheel/blob/main/SECURITY-MODEL.md),
  and [AGENTS.md](https://github.com/bitty-terminal/wheel/blob/main/AGENTS.md)
  — Wheel 0.1.0 policy record and repository guidance.

Local canonical documents:

- [IPC and Agent RFC](../specifications/ipc-agent-rfc.md) — accepted.
- [Execution ownership R1](../architecture/execution-ownership-r1.md) — draft,
  with the scoped accepted provider-routing snapshot binding.
- [Tool transport R2](../architecture/tool-transport-r2.md) — draft.
- [Context retention R3](../architecture/context-retention-r3.md) — draft.
- [AI Architecture](../architecture/ai-architecture.md) — draft.
- [Provider plugin boundary](../providers/provider-plugin-boundary.md) — draft.
- [Multimodal inference boundary](../providers/multimodal-inference-boundary.md)
  — draft.
- [Agent coordination architecture](agent-coordination.md) — draft.
- [Panel environment awareness](../interfaces/panel-environment-awareness.md) —
  draft.
- [Session model (candidate)](../specifications/session-model-candidate.md) —
  draft.
- [Execution-projection binding (candidate)](../specifications/execution-projection-binding-candidate.md)
  — draft.
- [Wheel scope and framework illustration (candidate)](../specifications/wheel-scope-and-framework-candidate.md)
  — draft.
- [Wheel core and plugin boundary (candidate)](../specifications/wheel-core-plugin-boundary-candidate.md)
  — draft.
- [Wheel-to-runtime coupling (candidate)](../specifications/wheel-to-runtime-coupling-candidate.md)
  — draft.
- [Context management architecture](../context/context-management.md) — draft.
- [Prefix-cache-friendly context design](../context/prefix-cache-context-design.md)
  — draft.
- [v0.1 Implementation Profile](../product/implementation-profile-v0.1.md) —
  draft.
- [AI Unresolved Questions](../product/ai-unresolved-questions.md) — draft.
- [Documentation workflow](../docs/development/documentation-workflow.md) —
  normative.

Cross-repository owners are listed once under
[Cross-repository references](#cross-repository-references).

### Security entry point

- [Security corpus index](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/README.md)
- [Threat model](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/threat-model.md)
  is normative.
