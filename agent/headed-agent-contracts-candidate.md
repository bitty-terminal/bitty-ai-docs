---
title: Headed Agent contracts (candidate)
description: Candidate reusable bitty-ai contracts for a sibling headed Agent plugin covering sessions streaming routing panels speech avatars budgets and platform fallback
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
> changes no Wheel behavior, and does not claim that a headed Agent plugin,
> speech implementation, avatar renderer, provider loader, or Core panel
> feature exists. Only the accepted IPC and Agent RFC and the scoped accepted
> provider-routing snapshot binding inside the draft R1 disposition are cited
> as accepted contracts. R1's broader ownership and cancellation split, R2,
> R3, the provider boundary, the multimodal speech/job contract, and the AI
> Architecture remain draft inputs. The seven rows in
> [P0 Review Sign-off](#p0-review-sign-off) are the single authoritative review
> gate set for this document.

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
[IPC and Agent RFC](../specifications/ipc-agent-rfc.md) and the scoped accepted
provider-routing snapshot binding maintained inside the draft
[Execution ownership R1](../architecture/execution-ownership-r1.md#providerrouting-snapshot-binding-contract-accepted-2026-10-17)
without restating or modifying either. R1's wider ownership and cancellation
material, R2, R3, the provider boundary, and the multimodal speech contract are
draft sources, not accepted contracts. Linked documents remain authoritative
for their own surfaces; this document only records the additional contract
families a headed consumer needs.

## Normative sources this specification must not weaken

This draft is subordinate to the accepted IPC and Agent RFC and to the scoped
accepted provider-routing snapshot binding. It also must not misstate or weaken
the draft dispositions it consumes. It replaces or relaxes none of them:

- The accepted [IPC and Agent RFC](../specifications/ipc-agent-rfc.md) for
  framing, transport, authentication, scopes, bounded agent messages, consent,
  streaming, versioning, and failure semantics. No credential, transport,
  scope, or method in this document overrides that RFC.
- The scoped accepted provider-routing snapshot binding in
  [Execution ownership R1](../architecture/execution-ownership-r1.md#providerrouting-snapshot-binding-contract-accepted-2026-10-17).
  Only that section is accepted. R1's cancellation contract, execution-ownership
  disposition, and registry split remain a draft disposition; this document
  consumes that split without promoting it.
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

| Term                  | Meaning in this document                                                                                                                                                                                                                                                                       |
| --------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Core                  | The terminal platform and its host services. Core is the designated owner of record (design target) for panel runtime, IPC transport, consent enforcement, process supervision, and event-bus transport; those panel and Event Bus mechanisms are not implemented or authorized by this draft. |
| `bittyd`              | A future, undecided Core-side service context in which host-owned lifecycle, panel, and transport authority may run. No v1.0 `bittyd` binary is claimed to exist.                                                                                                                              |
| `bitty-ai`            | The independent AI sub-platform that owns provider abstraction, routing, inference jobs, context compilation, and the AI-facing event vocabulary.                                                                                                                                              |
| Wheel                 | The out-of-process Coding Agent Harness. It consumes Core and `bitty-ai` and owns coding-workflow policy.                                                                                                                                                                                      |
| Headed Agent plugin   | The prospective sibling composition plugin that owns chat composition, presentation policy, speech controls, and avatar appearance mapping. Working name only.                                                                                                                                 |
| Presentation surface  | A panel, window, widget tree, or avatar surface that renders state. It is not a session, execution, or consent record.                                                                                                                                                                         |
| Media extension       | An optional, separately packaged implementation of audio playback, decoding, 2D rendering, avatar rigging, or 3D presentation. It is never required for text operation.                                                                                                                        |
| Media layer           | The combined runtime surface of zero or more media extensions plus Core presentation primitives. It is not an owner, contract, or authority.                                                                                                                                                   |
| Principal             | The authenticated `(UID, AgentId)` pair defined by the accepted IPC and Agent RFC. A self-asserted name, plugin identity, or panel label is not a principal.                                                                                                                                   |
| Agent session         | The accepted `AgentSession` lifecycle anchor plus the application-level handle and attachments defined here.                                                                                                                                                                                   |
| Session handle        | A generation-fenced reference returned by `start`; it is not a grant and not a credential.                                                                                                                                                                                                     |
| Attachment            | A presentation or observation subscription created by `attach`. Many attachments may exist for one session.                                                                                                                                                                                    |
| Session generation    | The monotonic fence for session-level operations, approvals, and consent/elevation binding. A stale session generation fails closed.                                                                                                                                                           |
| Attachment generation | The monotonic fence for one attachment's local handles and presentation state. It never grants authority or binds consent.                                                                                                                                                                     |
| Cursor                | The tuple of session identity, session generation, attachment identity, attachment generation, and last applied sequence per delivery class used for replay or resynchronization.                                                                                                              |
| Event Bus             | The Core-designated transport (planned; not implemented here) that carries authorized, bounded topics between host services and plugins.                                                                                                                                                       |
| Panel                 | A Core-designated presentation surface identity (planned). A panel projects state; it does not own execution, consent, or the session.                                                                                                                                                         |
| Routing snapshot      | The scoped accepted, read-only, digest-verified provider-routing projection. It is not a context or session snapshot.                                                                                                                                                                          |
| Context snapshot      | An immutable, redacted, bounded view of compiled model context for a turn or attachment, identified by version and digest.                                                                                                                                                                     |
| Session snapshot      | A bounded, redacted state view used to attach or resynchronize; it is not a transcript and not a replay log.                                                                                                                                                                                   |
| Speech job            | A `bitty-ai` generation job for the `Speech` task, reusing the draft multimodal generation-job lifecycle.                                                                                                                                                                                      |
| Viseme timeline       | Optional timing metadata that maps synthesized audio intervals to mouth-shape identifiers. It is best-effort presentation data.                                                                                                                                                                |
| Candidate             | Proposed by this document only; no review has accepted it.                                                                                                                                                                                                                                     |
| Owner-pending         | Belongs to another repository or team; recorded here as a pointer, never as settled content.                                                                                                                                                                                                   |
| Illustrative-only     | A sketch whose names, fields, bounds, or defaults are explicitly undecided and create no schema or compatibility promise.                                                                                                                                                                      |

## Boundary and sibling relationship

### Three consumers, one platform

Core is the designated owner of record for the neutral platform surface:
transport, authentication, consent enforcement, process and PTY supervision,
panel identity and lifecycle, and the event-bus transport. `bitty-ai` is the
designated owner of reusable AI mechanisms. Wheel and the Headed Agent plugin
are independent consumers of those mechanisms. This is an ownership design
target; Core panel, Event Bus, and future `bittyd` implementation do not exist
as a result of this document.

```text
Bitty Core / future bittyd (not implemented here)
  |-- IPC, consent enforcement, panel runtime (planned), Event Bus (planned)
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
  `-- Core presentation primitives used by either consumer (planned)
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
as a Wheel front end, or reuse Wheel private state.

If a future headed surface needs a capability that Wheel also needs, the
capability is promoted to `bitty-ai` or Core through a separately reviewed
contract and an owning task. A shared mechanism is never obtained by reaching
into Wheel. "No `wheel.*` dependency" is a review gate for this design and a
prerequisite for any future headed implementation.

### What stays out of Wheel

Wheel remains a focused Coding Agent Harness. This design assigns no headed UI,
chat presentation, speech synthesis or playback, avatar, viseme, 2D, or 3D work
to Wheel. Wheel may continue to evolve its own coding-workflow surface under its
own governance; this document neither changes that surface nor borrows it.

Wheel's candidate plugin boundary permits UI plugins in its own composition.
The Headed Agent is not one of them: it is a sibling plugin with no Wheel
manifest dependency, no Wheel plugin slot, no `wheel.*` service call, and no
shared UI ownership. If Wheel later hosts a coding-oriented UI, that remains a
separate Wheel decision and does not make the Headed Agent a Wheel UI plugin.

## Contract families and versioning

### Candidate contract families

Each family is versioned independently. The labels below are contract-family
names for review, not wire methods, Lua modules, package names, or accepted API
spellings.

| Family            | Candidate version | Accountable owner(s)                                  | Purpose                                                                                                        | Status |
| ----------------- | ----------------- | ----------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- | ------ |
| `headed.session`  | 1                 | Core owner + AI owner                                 | Session handle, `start`/`attach`/`detach`/`stop`, session and attachment generations, cancellation, reconnect. | Draft  |
| `headed.stream`   | 1                 | Core owner + AI owner                                 | Bounded delivery classes, sequencing, backpressure, replay, and resynchronization.                             | Draft  |
| `headed.routing`  | 1                 | AI owner                                              | Model-profile request and routing against the scoped accepted routing snapshot.                                | Draft  |
| `headed.context`  | 1                 | Core owner + AI owner                                 | Context snapshot and session snapshot references, redaction, and viewer filtering.                             | Draft  |
| `headed.tools`    | 1                 | Core owner + AI owner                                 | Tool proposal, approval request, verdict, execution, and result events.                                        | Draft  |
| `headed.panels`   | 1                 | Core owner + Headed Agent owner                       | Reserved topic families, panel attachment, suspend, resume, and audio focus.                                   | Draft  |
| `headed.speech`   | 1                 | AI owner (contract); Media extension owner (playback) | Speech request, stream, cancel, playback handoff, and asset reference.                                         | Draft  |
| `headed.avatar`   | 1                 | AI owner (events); Headed Agent owner (mapping)       | Presentation state, animation, and viseme events; renderer ownership stays outside `bitty-ai`.                 | Draft  |
| `headed.budget`   | 1                 | Core owner + AI owner                                 | Budget envelope, enforcement attribution, and degradation actions.                                             | Draft  |
| `headed.platform` | 1                 | Core owner + Media extension owner                    | Capability detection and fallback tiers on Linux, macOS, Windows, and FreeBSD.                                 | Draft  |

### Version negotiation authority

The **session generation created by `start` is the single authority** for the
contract-version binding of every family in the table. `start` returns the
selected version for each family it negotiates; a session generation cannot
change that binding in place.

- `start` negotiates every family the client declares, including the baseline
  `headed.platform` version even when no presentation attachment exists. It
  fails closed if the client requires an unsupported major.
- `attach` may select only a **compatible subset** of the families and versions
  already negotiated for that session generation. It may choose a lower minor
  when both sides support it, but it cannot introduce a family that `start` did
  not negotiate, select a higher major, or reinterpret a security-relevant
  semantic. The attach result includes the selected versions for that
  attachment.
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
vocabulary dependency is `bitty-agent-api`, not the state-machine crate. How the
AI host consumes the state machine, and whether the durable session record and
generation issuance live in Core or `bitty-ai`, remain open points. `bitty-ai`
owns the model-facing binding: routing snapshot, context snapshot, turn and
generation-bound events, tool proposals, speech jobs, and media event schemas. The Headed Agent plugin owns presentation, local
interaction, and the mapping of semantic events to widgets, speech controls, and
avatars.

This document fixes only the rules that neither side may violate: no
presentation authority, no model I/O in Core, no credential exposure, and no
session state owned by a panel.

### Lifecycle operations

| Operation | Meaning                                                                                                 | Grants authority | Stops work             | Cardinality                  |
| --------- | ------------------------------------------------------------------------------------------------------- | ---------------- | ---------------------- | ---------------------------- |
| `start`   | Create a session handle with identity, session generation, budget envelope, routing profile, and scope. | No               | No                     | One handle per start intent. |
| `attach`  | Add a presentation or observation attachment and return a snapshot plus cursor.                         | No               | No                     | Zero to many per session.    |
| `detach`  | Remove one attachment and release its local resources.                                                  | No               | No                     | One attachment at a time.    |
| `stop`    | Request session termination under the current session generation and cancellation rules.                | No               | Yes, under the policy. | Idempotent per session.      |

**`start`.** The caller supplies an authenticated principal, an idempotency key,
a routing profile or model alias, a context scope, a budget envelope, and the
contract versions it supports. The result is a session handle containing the
session identity, session generation, initial cursor, selected contract
versions, a bounded session snapshot, and a budget reference. A session is
created without authority beyond the caller's existing grants. `start` is not a
consent prompt, does not pre-approve a tool, and does not start a provider call
by itself.

**`attach`.** The caller supplies a session handle, its session generation, the
requested presentation capabilities, and its supported contract versions. The
result is an attachment identity, its attachment generation, a sanitized session
snapshot, a cursor, the selected versions for that attachment, and the resume
rules. Attaching never moves an execution target, broadens context access,
creates a tool grant, or makes the attachment the session authority. An
attachment reads only what its own principal may read; a different principal or
AgentId cannot attach by knowing the session identity.

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
`AgentSession` state. A speech job may separately report `Cancelled` or
`Unknown` in its own draft media-job status. `Unknown` is reconciled by status
inspection or user direction before any retry; no rollback and no exactly-once
claim is made. `stop` is idempotent, releases owned resources at a bounded grace
deadline, and reports the retention outcome for evidence. A stale session
generation cannot stop a newer session.

### Controlled session transitions

`resume`, `takeover`, and `reset` are not additional free-standing operations
alongside `start`, `attach`, `detach`, and `stop`. The first two are controlled
transitions with fail-closed authorization; `reset` is deliberately not defined
by this draft.

| Transition       | Preconditions                                                                                                                                                       | Generation effect                                                   | Consent and effects                                                                                                                     |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| Session `resume` | The principal is authenticated, the session is retained and not live under another writer, and the caller passes the current session generation.                    | Issues a new session generation.                                    | Prior elevation is invalid. Re-grant requires a fresh prompt. No queued command, tool effect, speech request, or animation is replayed. |
| `takeover`       | The current owner or an explicit user-owned host policy authorizes the handoff, and the handoff is recorded with both principals and the target session generation. | Issues a new session generation; the old generation is invalidated. | Fresh consent is required. Pending actions are reconciled, not replayed. A stale owner cannot continue under the old generation.        |
| `reset`          | Not defined here. A destructive reset is a new-session operation that requires a separate Core contract.                                                            | None.                                                               | This draft provides no reset path and no implicit consent. A caller that needs a clean session performs a new `start`.                  |

Session `resume` and `takeover` retain the contract-version binding of the
original `start`; they do not renegotiate it. A client that requires a different
major performs a new `start` with a fresh session and fresh consent.

### Generation, version, and consent axes

The word "generation" is not a single universal fence. The following axes are
separate, and a contract family must name the axis it uses:

| Axis                                    | Issued or bound by                                                                                                      | What it fences                                                                                     | Binds consent or elevation?                                                                        |
| --------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| Session generation                      | The session authority at `start`, session `resume`, or `takeover`; the exact Core-versus-`bitty-ai` split remains open. | Session-level mutating operations, approval verdicts, session stop, and session-scoped media jobs. | Yes, together with the principal and scope.                                                        |
| Attachment generation                   | The attachment authority at `attach`.                                                                                   | One attachment's local handles, cursors, and presentation state.                                   | No. It never grants authority or binds consent.                                                    |
| Routing snapshot version and digest     | The scoped accepted provider-routing publication. No separate routing "generation" is defined by this draft.            | The exact provider-routing projection used for a turn or session pin.                              | No. It selects routing data; it never grants a scope.                                              |
| Context snapshot version and generation | `bitty-ai` context compilation, with `current_generation` bound to the session generation under CP-5.                   | The exact compiled context and its freshness.                                                      | No. Expansion re-authorizes the read; the snapshot itself is not a grant.                          |
| Speech job identity and status          | The draft `bitty-ai` media-job lifecycle.                                                                               | Cancellation, status inspection, and `Unknown` reconciliation for one speech job.                  | No. Speech egress consent binds to the session generation, principal, provider, and privacy class. |

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

- The Core owner owns the session-generation sequence. One Core-designated
  sequencer assigns a monotonically increasing sequence number per session
  generation at commit, before queue admission. Every event carries that number;
  a dropped event still consumes a sequence number, so a consumer can distinguish
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
  Speech job lifecycle and terminal events (`queued`, `running`, `completed`,
  `cancelled`, `unknown`) are control, non-lossy, and gap-detected. Avatar state,
  animation, and viseme events are ephemeral presentation, lossy, and
  latest-wins. A terminal media-job state is never carried only by an ephemeral
  avatar event.
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
request, re-play audio, re-run an animation backlog, or re-grant consent.
Pending approvals reappear as pending with their remaining lifetime; an expired
approval becomes denied. Lossy-class gaps recover by their class rule and do not
force a full session snapshot. Audio does not auto-play after reconnect; a fresh
user gesture is required unless an explicit, user-owned autoplay policy already
covers this attachment.

## Provider routing, snapshots, and tool approval

### Model-profile routing

The Headed Agent plugin asks for a model profile, role, or alias together with
constraints. It does not select a provider, name an endpoint, supply a
credential, or read a provider registry.

An illustrative request carries:

- the model profile, role, or alias;
- required capabilities, such as text, streaming, tool use, vision, or speech;
- the minimum acceptable privacy class;
- the token, cost, latency, and locality constraints;
- the budget reference and the caller's routing snapshot version and digest.

`bitty-ai` resolves the request against the scoped accepted, read-only,
digest-verified provider-routing snapshot. The accepted
[provider/routing snapshot binding](../architecture/execution-ownership-r1.md#providerrouting-snapshot-binding-contract-accepted-2026-10-17)
remains authoritative for the snapshot domain, canonical bytes, digest,
ownership proof, and lifecycle. This document adds only the headed consumer's
rule: the plugin treats the snapshot as opaque and never reconstructs routing
from local state.

Routing rules:

- The privacy class is a hard constraint, not a preference. A `local-only`
  request never falls back to a remote provider. A `network-minimized` request
  does not silently become `upload-notice`.
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

| Snapshot         | Content                                                                                                                         | Authority                    | Not                                                                                       |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------- | ---------------------------- | ----------------------------------------------------------------------------------------- |
| Routing snapshot | Provider-routing registry projection, domain, version, digest, and ownership proof.                                             | Scoped accepted R1 binding.  | Not a context snapshot, session snapshot, tool-list snapshot, or completion request.      |
| Context snapshot | Compiled model-facing context, version and generation, token and byte counts, redaction state, provenance references.           | `bitty-ai` context contract. | Not the transcript, journal, or evidence store; not permission to re-execute an effect.   |
| Session snapshot | Sanitized session state, accepted lifecycle state, session generation, cursor barrier, active turn and jobs, pending approvals. | Core owner + AI owner.       | Not a replay log, not a full transcript, and not a substitute for `attach` authorization. |

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
  generation, the tool name, a canonical digest of the validated arguments, the
  schema version, the required scope, the target, and the expiry.
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

Topic names below are family labels for review. They are not accepted wire
strings, subscription methods, or plugin API names.

| Family                       | Producer           | Consumer                   | Accountable owner  | Delivery class         |
| ---------------------------- | ------------------ | -------------------------- | ------------------ | ---------------------- |
| `session.lifecycle.*`        | Core               | Attachments, host services | Core owner         | Control, non-lossy     |
| `ai.turn.*`                  | AI owner           | Attachments                | AI owner           | Lossy observation      |
| `ai.context.*`               | AI owner           | Attachments                | AI owner           | Lossy observation      |
| `ai.tool.proposal.*`         | AI owner           | Attachments, approval UI   | AI owner           | Control, non-lossy     |
| `tool.approval.*`            | Core owner         | Attachments, approval UI   | Core owner         | Control, non-lossy     |
| `tool.result.*`              | Core owner         | Attachments                | Core owner         | Control, non-lossy     |
| `ai.speech.job.*`            | AI owner           | Headed Agent owner         | AI owner           | Control, non-lossy     |
| `ai.avatar.*`                | AI owner           | Headed Agent owner         | AI owner           | Ephemeral presentation |
| `panel.*`                    | Core owner         | Attachments, host services | Core owner         | Control                |
| `plugin.<headed-agent-id>.*` | Headed Agent owner | Its own subscribers        | Headed Agent owner | Plugin-defined         |

A row's accountable owner is the repository accountable for the topic schema,
producer identity, and authorization; a consumer is never the owner of a topic
it only reads. Audio chunks are not an Event Bus topic: they travel on the media
channel with their own chunk sequence.

Topic rules:

- The Core owner reserves, as a design target, `session.*`, `panel.*`,
  `tool.approval.*`, `tool.result.*`, and any host-service namespace. A plugin
  cannot publish into a reserved namespace or spoof a producer identity.
- Subscription requires authorization for the topic scope. Knowing a topic
  name is not permission to read it. Cross-plugin subscription requires an
  explicit grant; there is no ambient wildcard bus.
- Payloads are bounded, schema-versioned, redacted, and free of credentials,
  raw environment values, and unbounded transcript text.
- Ordering is per topic and per session generation where the topic carries
  session events. A global total order is not promised.
- Approval, stop, proposal, speech-job, and tool-result events are non-lossy.
  Turn deltas, context notices, and avatar/viseme events may coalesce or drop
  with counters. Durable speech-job lifecycle is never routed only through the
  ephemeral avatar topic.
- Retention is per topic class. Control topics retain enough to support the
  documented reconnect window; ephemeral presentation topics may retain only
  the latest state.
- A plugin-private topic is not a covert cross-plugin channel. It is still
  subject to payload bounds, audit, and the platform's plugin isolation rules.

### Core-versus-plugin ownership

The table records design-target ownership only; it claims no implementation.

| Concern                       | Core / future `bittyd`                                                              | `bitty-ai`                                        | Headed Agent plugin                        |
| ----------------------------- | ----------------------------------------------------------------------------------- | ------------------------------------------------- | ------------------------------------------ |
| Panel identity and lifecycle  | Design-target owner of identity, create, close, and suspend; none implemented here. | Consumes panel references only; no ownership.     | Requests a panel; never invents identity.  |
| Layout, focus, and visibility | Design-target owner of host focus and visibility rules.                             | No ownership.                                     | Owns its internal composition and widgets. |
| Rendering and animation       | Design-target owner of presentation primitives; none implemented here.              | Owns the semantic state and viseme event schemas. | Maps semantics to 2D/3D appearance.        |
| Agent and turn state          | Host-side transport; session-record placement open.                                 | Owns model-facing turn and job state.             | Projects the sanitized view.               |
| Topic schema                  | Design-target owner of reserved namespaces and transport.                           | Owns AI-domain topic schemas it produces.         | Owns only its private topics.              |
| Consent and approval          | Ledger, prompt, and verdict owner; no implementation claimed.                       | Relays proposals and results; grants nothing.     | Presents the request; cannot grant.        |
| Retention and deletion        | Host evidence policy owner.                                                         | Owns AI record semantics under draft R3.          | Keeps local media ephemeral by default.    |
| Audio focus                   | Host window/panel focus owner.                                                      | Owns the speech job lifecycle.                    | Requests playback; does not steal focus.   |

A panel is a presentation surface, not a session, execution, or consent record.
The existing
[execution-projection binding](../specifications/execution-projection-binding-candidate.md#the-binding-relation)
continues to govern panel-to-execution projection. A session attachment is a
different relation: it binds a presentation client to a session's observation
stream and does not replace or duplicate an execution binding.

## Multi-panel suspend and resume

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
  Local attachment handles and their generations do expire and must be
  revalidated on resume.

### Resume rules

- Resume revalidates the caller's current authorization, the session
  generation, the attachment generation, and the version binding from the
  original `start` and `attach`. It does not renegotiate versions. A binding
  mismatch fails closed and requires a fresh `attach`; a session-generation
  mismatch requires the controlled session `resume` or `takeover` path.
- If the cursor is still within the replay window, the attachment replays
  missing non-lossy control events and applies the latest presentation state. If
  not, it loads a sanitized session snapshot and resumes from the barrier.
- A session-generation or attachment-generation mismatch fails closed and
  requires a fresh `attach` or the controlled session `resume`/`takeover` path.
  It never resurrects an old attachment or restores old elevation.
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
- At most one attachment holds audio playback focus at a time. Focus changes
  are user or host decisions; an agent may request attention, never preempt
  focus or start playback.
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
  -> speech job (queued, preparing, running, streaming, terminal)
  -> bounded audio chunks and optional viseme timeline
  -> optional media extension playback
  -> terminal status and reconciliation if Unknown
```

### Request rules

An illustrative request carries the text to synthesize, a voice profile alias,
a locale, an output format, a streaming preference, a minimum privacy class, a
budget reference, and a consent reference. It does not carry a provider
identity, endpoint, credential, or raw vendor parameter.

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
  are not placed in context, events, or snapshots.
- A player may start before the full asset exists, but it never plays
  out-of-order chunks. On underflow it pauses or restarts at a safe boundary
  and reports the degradation.
- Viseme timing metadata is optional and best-effort. Its absence or loss
  degrades animation only; audio and captions continue.

### Cancellation

- Cancellation is idempotent and may be requested at any chunk boundary, from
  the user, the owning panel, or the session stop path.
- Local playback stops first, queued audio is discarded, and the device buffer
  is cleared. Provider-side cancellation is best-effort.
- The terminal outcome is `Cancelled` when cancellation is confirmed, or
  `Unknown` when the remote job may have completed and billed. `Cancelled` here
  is the draft speech-job status, not an accepted `AgentSession` state; the
  session records `Completed` or `Failed` with cancellation attribution.
  `Unknown` is reconciled by status inspection before any retry; a retry is a
  new submission, not a replay.
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
  lower presentation tier. It does not crash the session or silently continue
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
- Avatar assets are untrusted input. Decoding, texture upload, and shader
  composition follow the media extension's bounded, isolated implementation.
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
- Enforcement emits an attributed record with the session identity, session
  generation, attachment generation, budget dimension, observed value, limit,
  and action taken. Unattributed enforcement is a conformance defect.
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

### Capability detection and tiers

The platform layer detects capabilities; it does not assume them from an
operating-system name. The presentation tiers are:

| Tier | Presentation capability                              | Required behavior                                                      |
| ---- | ---------------------------------------------------- | ---------------------------------------------------------------------- |
| A    | Audio playback plus GPU-accelerated 2D or 3D avatar. | Full presentation within the declared budgets.                         |
| B    | Audio playback plus 2D or software-rendered avatar.  | Reduced animation and resolution; no 3D requirement.                   |
| C    | Text and captions only; no audio device or no GPU.   | Full session function; speech may produce an asset for later playback. |
| D    | Headless or no presentation attachment.              | Session and tools continue; no rendering, audio, or avatar resources.  |

Tier selection is explicit, visible, and reversible on device change. A lower
tier never weakens consent, redaction, approval, or retention rules. It only
reduces presentation.

### Platform matrix

The table records the fallback expectation, not a support claim. Core's
published platform support matrix remains authoritative, and cross-platform
claims require CI or reviewed test evidence.

| Platform | Expected baseline                                                             | Audio                                                    | GPU / avatar                                    | Notes                                                                            |
| -------- | ----------------------------------------------------------------------------- | -------------------------------------------------------- | ----------------------------------------------- | -------------------------------------------------------------------------------- |
| Linux    | Tier A or B where devices exist.                                              | Native audio stack through Core or a media extension.    | Native GPU surface through the media extension. | Device loss and compositor restart must degrade without killing the session.     |
| macOS    | Tier A or B where devices exist.                                              | Native platform audio through Core or a media extension. | Native GPU surface through the media extension. | App suspension and audio device changes must be handled explicitly.              |
| Windows  | Tier A or B where devices exist.                                              | Native platform audio through Core or a media extension. | Native GPU surface through the media extension. | Audio session and device-change behavior require platform evidence.              |
| FreeBSD  | Tier C or D by default; B only with detected audio and software/2D rendering. | Only when a supported audio path is detected.            | Not assumed; software or 2D fallback.           | Missing audio or GPU must fall back to text or headless, never fail the session. |

Platform rules:

- No operating-system API, device path, driver, or GPU backend is named or
  embedded by `bitty-ai`. The Core owner owns host integration where a host
  adapter exists; the Media extension owner owns device-specific audio, GPU, and
  decoder adapters.
- Missing audio, missing GPU, a lost device, a context loss, an unsupported
  codec, or unavailable decoder isolation falls back to the next tier with a
  user-visible reason. A tier that requires isolated decoding is unavailable
  when that isolation cannot be verified.
- A platform that cannot enforce a security requirement for a tier does not
  enter that tier. It fails closed to a lower presentation tier rather than
  serving unbounded or unredacted media.
- Suspend and resume, session lock, display change, and audio-route change are
  treated as device events. They do not change session authority or consent.
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
  affected handlers, cancels in-flight media, and records an auditable
  revocation event.
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
  extension fails closed to the next lower presentation tier or text-only. It
  does not decode untrusted media in-process and does not claim containment.
- No shell command, file path, URL, or environment value from an untrusted
  media payload is executed or resolved by `bitty-ai`.
- A malicious or spoofed panel cannot publish into a reserved topic, forge a
  producer identity, or read another plugin's stream.

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

| Concern                                     | Core owner (`bitty`; future `bittyd`)              | AI owner (`bitty-ai`)                                                   | Wheel owner       | Headed Agent owner                     | Media extension owner             | Provider plugin owner                     |
| ------------------------------------------- | -------------------------------------------------- | ----------------------------------------------------------------------- | ----------------- | -------------------------------------- | --------------------------------- | ----------------------------------------- |
| IPC framing, auth, instance, rate limits    | Owner                                              | Consumer                                                                | Consumer          | Consumer                               | Consumer through Core             | Consumer through AI and Core              |
| Consent ledger and revocation               | Owner                                              | Requests verdicts; never stores grants                                  | Requests verdicts | Presents prompts; never grants         | Consumes verdicts                 | Consumes verdicts                         |
| Agent session state machine                 | Host-side transport; state-machine placement open  | Uses `bitty-agent-api`; owns AI records; state-machine consumption open | Consumer          | Projects sanitized view                | None                              | None                                      |
| Session handle and generation               | Transport only; durable identity placement open    | Binds model-facing state to the session generation                      | Consumer          | Consumer                               | None                              | None                                      |
| Event sequencing and replay window          | Owner of session-generation sequencing             | Owns AI event schemas and nested turn sequencing                        | Consumer          | Consumer                               | Consumes media events             | None                                      |
| Provider registry and routing               | Enforces scopes and transport                      | Owner of resolution and routing snapshot                                | Consumer          | Consumer; no provider selection        | None                              | Registers; never widens                   |
| Credentials and secret storage              | Host-owned storage and grants                      | Uses opaque handles; never reads values                                 | Uses handles      | Never receives values                  | Never receives values             | Resolves at the adapter edge only         |
| Context compilation and token budget        | Enforces quotas                                    | Owner                                                                   | Consumer          | Consumer                               | None                              | None                                      |
| Context and session snapshots               | Owns host record and retention policy              | Owns compilation, redaction, and snapshot view                          | Consumer          | Reads sanitized view                   | None                              | None                                      |
| Tool proposal                               | Receives and validates                             | Owner of model proposal relay                                           | Consumer          | Presents                               | None                              | None                                      |
| Tool approval, execution, result            | Owner of ledger and execution                      | Consumes result as untrusted observation                                | Consumer          | Presents request and collects decision | None                              | None; tool providers are separate plugins |
| Panel identity, lifecycle, focus            | Owner                                              | Reads references only                                                   | Consumer          | Requests panel and composes UI         | Renderer consumes a panel surface | None                                      |
| Event Bus transport and reserved topics     | Owner                                              | Publishes AI-domain topics                                              | Consumer          | Publishes only its private topics      | Publishes declared media topics   | None                                      |
| Speech job lifecycle                        | Enforces consent and transport                     | Owner of request, stream, cancel, asset ref                             | None              | Owns eligibility and controls          | Owns playback                     | Synthesizes under the speech contract     |
| Audio device and playback                   | May provide a presentation primitive; none claimed | No ownership                                                            | None              | Requests playback                      | Owner of the device               | None                                      |
| Avatar semantic state and viseme vocabulary | No ownership                                       | Owner of neutral event vocabulary                                       | None              | Maps to appearance and animation       | Renders 2D/3D                     | May emit timing metadata                  |
| 3D rig, shaders, and GPU surfaces           | No ownership                                       | No ownership                                                            | None              | May request                            | Owner                             | None                                      |
| CPU, memory, GPU, audio budgets             | Host and transport budgets                         | AI and media-job budgets                                                | Harness work      | Presentation budgets                   | Device and decoder budgets        | Provider quota reporting                  |
| Platform audio/GPU adapters                 | Host integration where a host adapter exists       | No platform API                                                         | No platform API   | No platform API                        | Device-specific adapter owner     | No platform API                           |
| Coding workflow and task graph              | No ownership                                       | No ownership                                                            | Owner             | No `wheel.*` dependency                | None                              | None                                      |
| Retention and deletion                      | Host evidence and consent                          | AI record semantics under draft R3                                      | Its own records   | Ephemeral local media by default       | No persistent media by default    | Not owned here; provider policy governs   |

## Security review

No security review has been completed for this draft. This section records the
threat model and the gates that a later review must close; it is a review input,
not a sign-off. The draft must remain `draft` until every row in
[P0 Review Sign-off](#p0-review-sign-off) records a disposition. That seven-row
table is the single authoritative sign-off set; this section does not create a
smaller or alternate role list.

### Threat analysis

| Threat                                      | Impact                                                                 | Required gate                                                                                            | Accountable owner(s)                          |
| ------------------------------------------- | ---------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------- | --------------------------------------------- |
| Presentation becomes authority              | A panel, avatar, or attachment can approve or execute without consent. | Presentation-never-authority: every effect re-checks the caller's grants.                                | Core owner + Headed Agent owner               |
| Stale session or attachment generation      | Old approval, stop, or attachment revives a newer session.             | Session and attachment generation fencing on every mutating operation and event.                         | Core owner + AI owner                         |
| Approval substitution                       | A user approves one call and a different call executes.                | Canonical argument digest, target, scope, expiry, and session-generation binding.                        | Core owner                                    |
| Secret leakage through snapshots or events  | Credentials or environment values reach a model, log, or UI.           | Typed redaction before queueing and writing; presence markers only.                                      | Core owner + AI owner                         |
| Unbounded media parsing                     | A malicious audio or avatar asset exhausts memory or executes code.    | Mandatory isolated decode, depth limits, and fuzzed decoders; fail closed when isolation is unavailable. | Media extension owner                         |
| Prompt injection into speech or approval    | Model output causes an egress or an approval.                          | Egress filtering; approval requires explicit user action, never model text.                              | AI owner + Core owner + Headed Agent owner    |
| Cross-panel eavesdropping or topic spoofing | One plugin reads or forges another plugin's stream.                    | Reserved namespaces, per-topic authorization, producer identity checks.                                  | Core owner                                    |
| Privacy downgrade on fallback               | A local-only request silently goes remote.                             | Privacy class is a hard constraint; fallback stays within it.                                            | AI owner                                      |
| Resource exhaustion or denial of service    | One session starves audio, GPU, or the host.                           | Hard caps, per-session attribution, containment, and slow-consumer shed.                                 | Core owner + AI owner + Media extension owner |
| GPU or audio device data retention          | One principal's media remains in device memory for another.            | Release and clear on detach, suspend, and session end.                                                   | Media extension owner                         |
| `wheel.*` coupling                          | The headed plugin reaches into Wheel private state.                    | Dependency review; no `wheel.*` import, call, config, or environment key.                                | Headed Agent owner + security reviewer        |

### Security gates

| Gate                             | Requirement                                                                                                                                                  | Evidence required before implementation                                                |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------- |
| Presentation-never-authority     | A panel, attachment, avatar state, animation, or speech control cannot approve, execute, or broaden a scope.                                                 | Adversarial tests that attempt approval and execution from every presentation surface. |
| Generation fencing               | Stale session, attachment, approval binding, and media-job identities fail closed before effects.                                                            | Property tests and replay corpus across suspend, reconnect, takeover, and restart.     |
| Approval binding                 | Approval binds the exact validated call digest, target, scope, expiry, session generation, and principal.                                                    | Mutation, replay, expiry, and cross-principal test matrix.                             |
| Secret-free surfaces             | Snapshots, events, speech text, avatar state, and traces contain no raw secret or unapproved environment value.                                              | Seeded-secret negative tests on every producer and every write path.                   |
| Bounded media                    | Audio, viseme, animation, and avatar inputs are size-, depth-, and time-bounded; decode is mandatorily isolated or fails closed.                             | Fuzz corpus, malformed asset suite, isolation proof, and fail-closed tier tests.       |
| Consent before egress            | Model, speech, and tool egress require the matching grant; autoplay is off unless an explicit user-owned policy enables it; revocation stops the next chunk. | Consent matrix and revocation lifecycle tests.                                         |
| Retention and erasure            | Media and snapshots are ephemeral by default; recording is consented and redacted; source-history deletion propagates to derived media.                      | Retention, export-preview, and deletion-propagation tests.                             |
| Capability fallback fails closed | A missing or unverified platform capability lowers presentation only; it never weakens security, consent, or redaction.                                      | Platform matrix tests and capability-detection negative tests.                         |
| No Wheel coupling                | The headed plugin has no `wheel.*` import, service call, configuration dependency, environment key, or private state access.                                 | Dependency graph review and a negative integration test.                               |
| Independent review               | Every row in [P0 Review Sign-off](#p0-review-sign-off) differs from the implementer and records a disposition before implementation tasks open.              | Seven review records linked from the owning task.                                      |

### Residual risks

- No implementation exists, so every gate above is unverified.
- Media sandboxing and GPU memory isolation are platform-dependent. A tier that
  cannot verify the required isolation is disabled rather than treated as
  contained, so the residual risk is reduced presentation, not unaudited media
  parsing.
- Voice provenance, watermarking, and synthetic-speech disclosure are not
  defined; a future speech contract must decide them.
- Multi-user, multi-device, and remote headed clients are deferred. They would
  need their own authentication and consent ledger review.
- The exact Core-versus-`bitty-ai` split for session identity and durable
  records remains open; a wrong split could duplicate authority.
- The plugin packaging model for media extensions is not decided, so supply
  chain and permission review for those extensions remain future work.

## Verification plan

This repository is documentation-only. The current evidence for this document is
the local documentation gate (`just check`) plus independent review of the
diff. No implementation, performance, platform, or security evidence exists.

Future implementation tasks must add, per contract family once it is accepted:

- Schema and version-negotiation tests: session-bound selection, attach
  compatible subset, standalone `headed.platform` probe, unknown major
  rejection, additive minor tolerance, and persisted-event readability.
- Lifecycle property tests for `start`, `attach`, `detach`, `stop`, session
  `resume`, `takeover`, generation mismatch, idempotency, and cancellation
  before and after dispatch.
- Event-ordering and resynchronization tests: non-lossy gaps, lossy lost-count
  recovery, duplicates, snapshot barriers, replay-window boundaries, and
  reconnect during a turn.
- Backpressure and soak tests: slow consumer, bounded queues, counted drops,
  audio underflow, and no unbounded memory growth.
- Approval tests: exact digest binding, argument mutation, target change,
  expiry, cross-principal replay, and reconnect with a pending approval.
- Snapshot tests: bounded size, redaction, viewer filtering, immutable
  versions, separation of routing/context/session snapshots, and expansion
  re-authorization.
- Speech tests: client and host text-bound revalidation, redaction, first-chunk
  timeout, cancellation at every boundary, `Unknown` reconciliation, and no
  auto-play after reconnect without a user-owned policy.
- Avatar and viseme tests: latest-wins state, bounded animation queue, clock
  tolerance, loss degradation, reduced motion, and release on suspend.
- Budget tests: per-dimension hard and soft limits, attribution, degradation,
  and isolation between sessions.
- Platform matrix tests: Linux, macOS, Windows, and FreeBSD capability
  detection, missing device, device loss, decode-isolation failure, and
  fallback to text or headless.
- Security adversarial tests for every gate in the security review table.

A future contract is not accepted because this document exists. Each family
needs its own accepted specification, owner, implementation task, evidence, and
independent review.

## Alternatives considered

| Alternative                                                    | Why it is not selected                                                                                                                                                            |
| -------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Put headed UI, speech, avatars, or 3D in Wheel                 | Couples high-churn presentation and media to the coding harness, expands Wheel's product scope, and contradicts the requirement that Wheel remain a focused Coding Agent Harness. |
| Let the Headed Agent depend on Wheel as a library or front end | Creates a private cross-consumer dependency, makes Wheel's internal API a de facto contract, and prevents independent evolution. Shared needs belong in Core or `bitty-ai`.       |
| Put the headed agent inside Core                               | Makes the terminal platform own AI composition, chat policy, and media presentation, violating the Core-versus-AI boundary and the no-model-I/O rule for Core.                    |
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

| Contract or surface                                                                                                                          | Owner                            | Effect of this draft                                                                                                     | Status                      |
| -------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------- | ------------------------------------------------------------------------------------------------------------------------ | --------------------------- |
| [IPC and Agent RFC](../specifications/ipc-agent-rfc.md)                                                                                      | Core owner                       | Consumed unchanged for framing, auth, scopes, bounded messages, consent, streaming, and versioning.                      | No change                   |
| [Provider-routing snapshot binding](../architecture/execution-ownership-r1.md#providerrouting-snapshot-binding-contract-accepted-2026-10-17) | Core owner + AI owner            | Consumed unchanged for routing snapshot domain, bytes, digest, and ownership proof.                                      | No change                   |
| [Unified authorization backend](../architecture/tool-transport-r2.md#unified-authorization-backend)                                          | Core owner                       | Tool approval events must map to the existing gate order; no new authorization path.                                     | No change                   |
| [Context retention R3](../architecture/context-retention-r3.md)                                                                              | Core owner                       | Speech, viseme, avatar, and attachment data follow the draft consent, retention, and deletion rules.                     | No change                   |
| [Provider plugin boundary](../providers/provider-plugin-boundary.md)                                                                         | AI owner + Provider plugin owner | Consumed unchanged for privacy classes, capabilities, secret invariant, and project overrides.                           | No change                   |
| [Multimodal inference boundary](../providers/multimodal-inference-boundary.md)                                                               | AI owner                         | Draft speech and generation-job lifecycle reused; this draft adds only headed request, playback, and cancellation rules. | No change; follow-up review |
| [Agent coordination architecture](agent-coordination.md)                                                                                     | AI owner + Core owner            | Aligns panel attachment, suspend, and lifecycle outcomes with the existing reconciliation.                               | Candidate alignment         |
| [Panel environment awareness](../interfaces/panel-environment-awareness.md)                                                                  | AI owner + Core owner            | Sanitized Agent View and no-persistence defaults continue to apply to headed presentation.                               | No change                   |
| [Execution-projection binding](../specifications/execution-projection-binding-candidate.md)                                                  | Core owner + AI owner            | Session attachment is distinguished from execution projection; no binding semantics are redefined.                       | No change                   |
| [Wheel core and plugin boundary](../specifications/wheel-core-plugin-boundary-candidate.md)                                                  | Wheel owner + AI owner           | Records the sibling boundary and the `wheel.*` prohibition; no Wheel work is assigned.                                   | No change to Wheel          |
| Core panel and Event Bus implementation                                                                                                      | Core owner                       | Would need its own accepted contract before implementation; this draft authorizes none.                                  | Owner-pending               |
| Plugin packaging for media extensions                                                                                                        | Plugin ecosystem owner           | Would need registry, manifest, capability, and supply-chain review; not defined here.                                    | Owner-pending               |
| Speech and avatar event schemas                                                                                                              | AI owner                         | Candidate families only; no wire schema, field list, or compatibility promise is accepted.                               | Draft                       |

### Cross-repository references

The owning repositories are linked once, here. The [References](#references)
section carries the local canonical documents and the security entry point.

- [bitty-docs](https://github.com/bitty-terminal/bitty-docs) owns shared
  governance, the decision register, the open-question register, and the
  normative security corpus. This document links to those owners instead of
  copying or reinterpreting them.
- [bitty-terminal-docs](https://github.com/bitty-terminal/bitty-terminal-docs)
  owns the terminal platform's panel runtime, panel presentation, rich
  presentation, and platform-support contracts. The panel and audio/GPU
  mechanism references here are owner-pending pointers to that repository.
- [bitty-plugins-docs](https://github.com/bitty-terminal/bitty-plugins-docs)
  owns the plugin platform, manifest, capability, and packaging contracts.
  Media-extension packaging and permission review belong there.
- [bitty-ai](https://github.com/bitty-terminal/bitty-ai) owns implementation
  evidence for the AI runtime and providers. This document claims no
  implementation status from that repository.
- [wheel](https://github.com/bitty-terminal/wheel) owns the Coding Agent
  Harness. This document records a sibling boundary only and assigns no Wheel
  implementation work.
- [bitty](https://github.com/bitty-terminal/bitty) owns Core implementation
  evidence. Future `bittyd`, Core panel, Event Bus, and platform-adapter work
  require their own accepted contracts; no v1.0 `bittyd` binary is claimed.

## Open points

None of these is resolved by this draft. Each needs an owner, a reviewed
decision, and evidence before the related implementation task opens. This table
is a review index, not a register and not a local blocker set: entries route to
the named register or are explicitly marked as follow-up work in `TODO.md`
until the owning commander files the tracked issue.

| Open point                                     | Question                                                                                                                                                                                                       | Likely owner                                 | Candidate register routing                                                   | Blocks                                      |
| ---------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------- | ---------------------------------------------------------------------------- | ------------------------------------------- |
| Session authority split                        | Does Core or `bitty-ai` own the durable session handle, generation issuance, and reconnect record?                                                                                                             | Core owner + AI owner                        | `OQ-061` (primary)                                                           | `headed.session`, `headed.stream`           |
| Headed plugin identity and packaging           | Is the Headed Agent a Lua plugin, an out-of-process component, or a composed distribution? Who signs and updates it?                                                                                           | Plugin ecosystem owner                       | Follow-up issue                                                              | Plugin packaging, media extension packaging |
| Coding capability access                       | If the Headed Agent needs coding intelligence, does it use only public Core/`bitty-ai` contracts, or must the needed mechanism be promoted there? A `wheel.*` runtime dependency is not authorized either way. | Wheel owner + Plugin ecosystem owner         | Wheel scope decision; Follow-up issue                                        | `wheel.*` prohibition, product scope        |
| Replay retention window                        | How long are control events retained for reconnect, and what storage or memory budget applies?                                                                                                                 | Core owner + AI owner                        | Follow-up issue (AIQ-04/55/5A are adjacent closed(partial) facets)           | `headed.stream`                             |
| Multi-user and remote headed clients           | Are remote, multi-device, or shared-panel sessions in scope? What authentication and consent model applies?                                                                                                    | Core owner + Security owner                  | ADR-0008 (OQ-020 closed; prior decision); Follow-up issue                    | Identity, consent, transport                |
| Speech privacy classes and provider consent    | Which speech providers are local-only, which require egress consent, and what is the minimum baseline?                                                                                                         | AI owner + Security owner                    | Follow-up issue                                                              | `headed.speech`                             |
| Viseme source and accuracy                     | Is the viseme timeline provider metadata, local analysis, or both? What accuracy and tolerance claims are allowed?                                                                                             | AI owner + Media extension owner             | Follow-up issue                                                              | `headed.avatar`                             |
| 3D renderer placement                          | Is 3D an in-process extension, a separate process, or a remote surface? How is GPU memory isolated between principals?                                                                                         | Plugin ecosystem owner + Core owner          | Follow-up issue                                                              | `headed.avatar`, `headed.platform`          |
| Audio focus policy                             | Is audio focus per panel, per session, or global? What happens when two sessions speak at once?                                                                                                                | Core owner + Headed Agent owner              | Follow-up issue                                                              | `headed.panels`, `headed.speech`            |
| Platform support evidence                      | Which Linux, macOS, Windows, and FreeBSD configurations are actually supported, and who records the evidence?                                                                                                  | Core owner + terminal docs owner             | OQ-003 / ADR-0002 (accepted tiers); Follow-up issue for media specifics      | `headed.platform`                           |
| Budget defaults                                | What are the numeric CPU, memory, GPU, audio, token, and cost defaults, and who may change them?                                                                                                               | Core owner + AI owner + Security owner       | `OQ-066` (context token profile); Follow-up issue for media and host budgets | `headed.budget`                             |
| Approval lifetime and standing grants          | How long does a one-shot approval live, and are session-wide or tool-wide grants allowed?                                                                                                                      | Core owner + Security owner                  | Follow-up issue (OQ-018 is closed)                                           | `headed.tools`                              |
| Synthetic speech disclosure                    | Is watermarking, provenance metadata, or a spoken disclosure required for synthesized audio?                                                                                                                   | Security owner + Product owner               | Follow-up issue                                                              | `headed.speech`                             |
| Recording and export of speech or avatar state | Can users record, export, or delete synthesized audio and viseme data, and under which retention class?                                                                                                        | Core owner + Security owner                  | Follow-up issue (AIQ-55/5A are adjacent closed(partial) facets)              | `headed.speech`, `headed.avatar`            |
| Shared chat or transcript contract             | Do Wheel and the Headed Agent share a neutral presentation contract in Core or the plugin ecosystem, or only `bitty-ai` events plus Core panels? A `wheel.*` dependency is excluded.                           | Wheel owner + Plugin ecosystem owner         | OQ-056 (presentation projection); otherwise Follow-up issue                  | UI composition, duplication risk            |
| Accessibility baseline                         | Which reduced-motion, caption, contrast, and screen-reader behaviors are mandatory versus optional?                                                                                                            | Plugin ecosystem owner + Accessibility owner | Follow-up issue                                                              | `headed.avatar`, `headed.speech`            |

The existing panel-identity and projection questions in the
[AI Unresolved Questions](../product/ai-unresolved-questions.md) register remain
open; this document creates no register identifier and closes none of them. The
register admits no task-level identifier minting, so the unregistered entries
above are recorded in [TODO.md](../TODO.md) as promotion work for the owning
commander rather than asserted here as an authoritative blocker set.

## Acceptance criteria

This design is acceptable as a draft when all of the following hold:

- The document lives in the `agent/` tree, follows the specification spine, and
  carries the required flat frontmatter and status qualifiers.
- The boundary is explicit: Wheel is a focused Coding Agent Harness; the Headed
  Agent is a sibling plugin consuming Core and `bitty-ai`; no `wheel.*`
  dependency is defined; no headed UI, speech, avatar, or 3D work is assigned
  to Wheel.
- Each contract family has an owner, a candidate version, versioning rules, and
  an explicit statement that it is not accepted.
- Session lifecycle, session and attachment generation, cancellation, event ordering, backpressure,
  and reconnect are covered without redefining the accepted `AgentSession` or
  IPC RFC.
- Provider routing, context/session snapshots, and tool-call/approval events are
  covered without adding a provider loader, credential path, or second
  generation-job lifecycle.
- Panel and Event Bus topics, multi-panel suspend/resume, and the Core-versus-
  plugin split are covered without authorizing Core panel code.
- Speech and avatar contracts cover request, stream, cancel, state, animation,
  and visemes without implementing TTS, rendering, or 3D.
- Resource budgets, platform fallback, security, privacy, and consent are
  covered, with security gates and residual risks explicit.
- The ownership matrix and cross-repository references are complete enough for
  the owning repositories to review their own rows.
- Open points are explicit, and none is silently decided across a public
  contract boundary. The table is a review index, not a substitute register;
  each entry names its routing to the owning register or follow-up work.
- The local documentation gate passes, and every row in
  [P0 Review Sign-off](#p0-review-sign-off) records a disposition before any
  implementation task is opened.

Acceptance of this draft does not accept any contract family for
implementation. Each family needs its own accepted specification and evidence.

## P0 Review Sign-off

This seven-row table is the **single authoritative sign-off set** for this
document. Earlier and later summaries that mention approval, review, or a gate
refer to these rows and do not define a smaller role list.

| Role                         | Reviewer | Status  | Evidence                                           |
| ---------------------------- | -------- | ------- | -------------------------------------------------- |
| Architecture owner           | Pending  | Pending | Independent architecture review of this diff.      |
| Security reviewer            | Pending  | Pending | Threat model, gates, and residual-risk review.     |
| Core / future `bittyd` owner | Pending  | Pending | Panel, Event Bus, consent, and transport rows.     |
| `bitty-ai` owner             | Pending  | Pending | Routing, context, speech, avatar, and budget rows. |
| Wheel owner                  | Pending  | Pending | Sibling boundary and `wheel.*` prohibition.        |
| Plugin ecosystem owner       | Pending  | Pending | Packaging, media extension, and capability rows.   |
| Documentation curator        | Pending  | Pending | Structure, metadata, links, and self-containment.  |

No P0 sign-off has been recorded. The document remains `draft` until all seven
rows record dispositions and any defects are resolved or tracked.

## References

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
- [Agent coordination architecture](agent-coordination.md)
- [Panel environment awareness](../interfaces/panel-environment-awareness.md)
- [Session model (candidate)](../specifications/session-model-candidate.md)
- [Execution-projection binding (candidate)](../specifications/execution-projection-binding-candidate.md)
- [Wheel scope and framework illustration (candidate)](../specifications/wheel-scope-and-framework-candidate.md)
- [Wheel core and plugin boundary (candidate)](../specifications/wheel-core-plugin-boundary-candidate.md)
- [Wheel-to-runtime coupling (candidate)](../specifications/wheel-to-runtime-coupling-candidate.md)
- [Context management architecture](../context/context-management.md)
- [Prefix-cache-friendly context design](../context/prefix-cache-context-design.md)
- [v0.1 Implementation Profile](../product/implementation-profile-v0.1.md)
- [AI Unresolved Questions](../product/ai-unresolved-questions.md)
- [Documentation workflow](../docs/development/documentation-workflow.md)

Cross-repository owners are listed once under
[Cross-repository references](#cross-repository-references).

### Security entry point

- [Security corpus index](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/README.md)
- [Threat model](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/threat-model.md)
  is normative.
