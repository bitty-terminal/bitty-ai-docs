---
title: Headed Agent contracts (candidate)
description: Candidate reusable bitty-ai contracts for an optional Headed Agent surface covering sessions streaming routing panels speech avatars accessibility budgets and platform fallback
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
> changes no Wheel behavior, and does not claim that a Headed Agent surface,
> speech implementation, avatar renderer, provider loader, or panel feature
> exists. Accepted inputs it must not weaken are the IPC and Agent RFC, the
> Panel Runtime RFC, RFC-0006, ADR-0019, ADR-0016 Boundaries 1 and 3, ADR-0013,
> ADR-0002, the accepted accessibility extraction contract `W-134`, RFC-0001,
> RFC-0002, the scoped provider-routing snapshot binding inside the draft R1
> disposition, the OQ-058 and OQ-083 owner rulings, and the security corpus.
> R1's broader ownership and cancellation split, R2, R3, the provider boundary,
> the multimodal speech/job contract, and the AI Architecture remain draft
> inputs. The nine rows in [P0 Review Sign-off](#p0-review-sign-off) are the
> single authoritative review gate set for this document. The third-pass
> verification at `3091977` recorded design-review approvals for Architecture
> (`PX-1580`), Documentation (`PX-1579`), bitty-ai (`PX-1582`), Plugin
> (`PX-1583`), Wheel (`PX-1585`), and Product (`PX-1586`). Security (`PX-1581`)
> and Accessibility (`PX-1587`) remain `NEEDS-FIX` pending the AT-approval trust
> boundary corrected in this pass; Core (`PX-1584`) remains `NEEDS-FIX` pending
> re-review and accepted `bitty#2546`. This correction pass fixes the AT path and
> the Core text findings; re-review is pending for those three rows. No other
> approval is claimed.

## Purpose and scope

This document defines the reusable `bitty-ai` contracts that an optional Headed
Agent surface would consume. "Headed" describes presentation and human
interaction, never authority, persistence, or lifecycle. The proposed surface is
an optional `bitty-ai` extension and the successor candidate to the removed
`ai-panel` agent surface, not a Wheel module, not a Wheel front end, and not a
new Core feature. It consumes Core and `bitty-ai` mechanisms as a sibling
consumer relative to Wheel; it does not share Wheel's code, state, or packaging.

The document covers:

- The product thesis, candidate v1 scope, and placement of the surface,
  including the `ai-panel` predecessor and the product decisions that remain
  open.
- The boundary between Bitty Core, `bittyd`, `bitty-execution`, `bitty-ai`,
  Wheel, the Headed Agent surface, provider plugins, and optional media
  presentation.
- Independent contract versions for sessions, streaming, routing, snapshots,
  tool approval, panel topics, speech, avatars, accessibility, budgets, and
  platform fallback.
- Agent session lifecycle operations (`start`, `attach`, `detach`, `stop`),
  generation fencing, cancellation, event ordering, backpressure, and
  reconnect.
- Provider and model routing, context and session snapshots, and tool-call and
  approval events.
- Panel and Event Bus topics, multi-panel suspend and resume, and the
  Core-versus-plugin ownership split.
- Speech synthesis request, streaming, and cancellation, plus avatar state,
  animation, viseme events, and the accessibility baseline they must satisfy.
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
- Any claim that the Headed Agent surface ships, is installable, or has passed an
  implementation-level architecture or security review. The design security
  review recorded under `PX-1575` covers design content only.

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
binding, the accepted OQ-058 and OQ-083 owner rulings and ADR-0013 identity
model, ADR-0019 `bittyd` staging, ADR-0016 execution and accessibility
boundaries, ADR-0002 platform support tiers, the accepted accessibility
extraction contract `W-134`, RFC-0001 and RFC-0002, and the normative security
corpus. It also must not misstate or weaken the draft dispositions it consumes.
It replaces or relaxes none of them:

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
- The accepted OQ-083 owner ruling (2026-09-23): the panel-lease kernel
  direction (`Idle`/`Occupied`, acquire, release, and handoff) composing with
  the Stable Id hierarchy and the event bus. This draft consumes it as a
  design constraint and defines no competing panel lease; OQ-083 remains the
  owner of lease semantics, and takeover here is a session relation, not a
  panel lease.
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
- The accepted
  [ADR-0019](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/adrs/ADR-0019-bittyd-staging-advance.md)
  `bittyd` staging decision (2026-10-22): the local, per-user, single-instance
  host process enters the v0.2.0 slice with session-grained attach/detach, a
  per-domain single writer, service API versioning, and no persistence across a
  daemon restart. This draft consumes that staging and claims no daemon binary.
- The accepted
  [ADR-0016](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/adrs/ADR-0016-execution-graphics-accessibility-storage-platform-boundaries.md)
  Boundaries 1 and 3: the AI-agnostic execution supervisor moves to
  `bitty-execution` under focused contract `W-132` while Core retains PTY,
  permission and resource enforcement, host-side generation fencing, and the
  AI-agnostic execution bridge; the platform accessibility adapter moves to
  `bitty-a11y` under `W-134` while Core retains the accessibility baseline,
  projection-not-authority, and focus/terminal-state association. Both are
  accepted directions; neither implementation is claimed.
- The accepted
  [Accessibility Extraction Contract](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/accessibility-extraction-contract.md)
  (`W-134`) for the derived semantic snapshot, focus synchronization, the
  controlled action interface, the platform AT backends (AT-SPI/D-Bus, Windows
  UI Automation, macOS AX), and the retained Core baseline. It fixes that an
  adapter is never a principal, never mutates terminal state directly, and
  never publishes the projection on the Event Bus.
- The accepted
  [RFC-0001 appearance configuration](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/rfcs/RFC-0001-appearance-configuration.md)
  for the minimum-contrast rule and fail-closed configuration resolution, and
  the accepted
  [RFC-0002 panel animations](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/rfcs/RFC-0002-panel-animations.md)
  for the bounded reduced-motion and safe-mode contract. The high-contrast
  posture and reduced motion are Core-retained, non-plugin-overridable
  baselines.
- The open question OQ-078 as scoped by ADR-0016 Boundary 3: it remains open
  on the accessibility API/tree shape and its exact v1 versus post-v1 scope.
  The accepted direction and `W-134` obligations are not reopened by this
  draft; only the still-open adapter and surface details are referenced.
- The accepted
  [Plugin Platform RFC](https://github.com/bitty-terminal/bitty-plugins-docs/blob/main/specifications/plugin-platform-rfc.md)
  for the v1 Lua package boundary, the closed capability grammar, declarative
  UI limits, generation lifecycle, and the prohibition on native in-process
  plugins (`R-017`). This draft does not define a plugin category, package
  class, or capability identifier.
- The capability-catalog dependency rule: `panel.*` identifiers come from the
  accepted Panel Runtime RFC and RFC-0006 and are the only accepted capability
  family cited here. `ai.*`, `agent.*`, and `mcp.*` identifiers are
  **owner-pending**: a `bitty-ai` extension capability catalog exists only as a
  draft and `bitty-plugins` has not accepted extension-contributed manifest
  grammar. This draft defines none of those identifiers and adds no `media.*`,
  `audio.*`, or `gpu.*` family. Presentation capabilities in `headed.platform`
  are presentation classes, not plugin grants.
- The accepted
  [Package Lifecycle RFC](https://github.com/bitty-terminal/bitty-plugins-docs/blob/main/packaging/package-lifecycle-rfc.md)
  and [Package Follow-up RFC](https://github.com/bitty-terminal/bitty-plugins-docs/blob/main/packaging/package-followup-rfc.md)
  for staged activation, integrity, rollback, the resolver, registry, and key
  management. Those contracts are accepted; their implementation is missing.
  Whether the Headed Agent appears in a distribution at all remains `P-3`.
- The accepted
  [Product Vision](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/product/vision.md#agent-friendly-not-agent-centric)
  constraint that Bitty is agent-friendly, not agent-centric, and the accepted
  [AI Surface Accepted Record](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/ai-surface-accepted-record.md)
  recording the removal of the `ai-panel` Core surface and the move of AI
  surfaces to the optional `bitty-ai` extension.
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
- The draft `bittyd` session-host consumer requirements in
  [bittyd session host requirements](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/bittyd-session-host-requirements-candidate.md),
  especially `R-B09` (exactly one writer per fact: `bittyd` is the only writer
  of registry, lifecycle, generation, attach-count, lease, and last-seen state;
  the worker owns run state; one controller lease unless Core adopts a
  multi-controller model; clients mutate only through generation-checked
  methods) and the companion
  [bitty#2546](https://github.com/bitty-terminal/bitty/issues/2546) contract
  scope. That document is consumer design input, not an accepted contract, and
  this draft does not treat its candidate field names as settled.
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
- The draft `bitty-ai` Core-owned privacy classes, provider descriptor, secret
  invariant, and project-override prohibition in
  [Provider plugin boundary](../providers/provider-plugin-boundary.md#core-owned-surface)
  and its [secret invariant](../providers/provider-plugin-boundary.md#secret-invariant).
  "Core" in that document means the `bitty-ai` Core surface, not the terminal
  Core; the provider boundary is not accepted.
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

| Term                               | Meaning in this document                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| ---------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Core                               | The terminal platform and its host services. Core authorizes and fences execution and retains PTY ownership, permission and resource enforcement, host-side generation fencing, and the AI-agnostic execution bridge; the `bitty-execution` supervisor performs AI-agnostic supervision and execution (ADR-0016 Boundary 1, focused contract `W-132`). Core is also the owner of record for panel runtime, IPC transport, consent enforcement, and event-bus transport. Panel identity, lifecycle, and the Event Bus are accepted by the Panel Runtime RFC; their implementation is not claimed by this draft.                                                                                                                                                                                                                                                                                                |
| `bittyd`                           | The local, per-user, single-instance Core host process accepted into the v0.2.0 slice by ADR-0019 (2026-10-22). It owns the session registry, lifecycle, generation, attach-count, lease, and last-seen state as sole writer (`R-B09`), with session-grained attach/detach and no persistence across a daemon restart. No `bittyd` binary ships today.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `bitty-execution`                  | The independent AI-agnostic execution supervisor direction accepted by ADR-0016 Boundary 1 and parked to focused contract `W-132`. It supervises and executes; Core authorizes each operation and fences stale handles. No implementation is claimed.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| `bitty-ai`                         | The independent AI sub-platform that owns provider abstraction, routing, inference jobs, context compilation, and the AI-facing event vocabulary. "AI Core" in provider documents means this surface, never the terminal Core.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| Wheel                              | The out-of-process Coding Agent Harness. It consumes Core and `bitty-ai` and owns coding-workflow policy.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| Headed Agent surface               | The prospective optional `bitty-ai` extension that owns chat composition, presentation policy, speech controls, and avatar appearance mapping. It is the successor candidate to the removed `ai-panel` surface. Working name only.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| Presentation surface               | A panel, window, widget tree, or avatar surface that renders state. It is not a session, execution, or consent record.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| Media presentation (owner-pending) | An owner-pending presentation category for audio playback or decoding, GPU-backed 2D or 3D avatar rendering, avatar rigging, or 3D presentation. Declarative text-and-2D composition through Core presentation primitives is **not** this category: it owns no GPU surface, audio device, or decoder. It is **not** a v1 Lua plugin or package: the v1 plugin boundary forbids native in-process code, native module loading, and direct GPU, window, or PTY access (`R-017`). Its trusted form is a Core extension component that follows the `bitty-graphics`/`bitty-a11y` component pattern (`bitty-component.toml`, per-target digest, descriptor validation) and reaches GPU, window, and audio resources only through Core-owned interfaces. Its untrusted form, post-1.0, is a declared native helper that receives no GPU, window, PTY, or host-Rust handle. It is never required for text operation. |
| Media layer                        | The combined runtime surface of zero or more media presentations plus Core presentation primitives. It is not an owner, contract, or authority.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| Principal                          | The authenticated `(UID, AgentId)` pair defined by the accepted IPC and Agent RFC. A self-asserted name, plugin identity, or panel label is not a principal.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| Agent session                      | The accepted IPC and Agent RFC `AgentSession` contract (`Created`, `Running`, `WaitingToolResult`, `Completed`, `Failed`) as the lifecycle anchor, plus the `bitty-ai` runtime's own vocabulary (`Active`, `Completed`, `Failed`, `Canceled`) and the application-level handle and attachments defined here. It is not a `bittyd` session and not a panel.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| Session handle                     | A generation-fenced reference returned by `start`; it is not a grant and not a credential.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| Attachment                         | A presentation or observation subscription created by `attach`. Many attachments may exist for one session.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| Session generation                 | The application-level fencing value for session-level operations, approvals, and consent/elevation binding, returned by `start`. It is **not** the `bittyd` worker-binding generation of `R-S16`, which increments only on a worker bind or rebind and never on attach or detach. Its issuing authority and durable record are the subject of `bitty#2546`; if `bittyd` owns that record, `bitty#2546` must add a distinct session-generation field and `bittyd` is its sole writer under `R-B09`, while whether it also sequences the control stream remains part of `bitty#2546`. No second component may advance the same generation. A stale session generation fails closed.                                                                                                                                                                                                                             |
| Attachment generation              | The monotonic fence for one attachment's local handles and presentation state. It never grants authority or binds consent.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `GenerationId`                     | The accepted ADR-0013 fencing identifier for reloads, leases, and handoffs. It composes with the per-contract axes here but does not replace the session or attachment generation.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| `bittyd` session                   | One unit registered and supervised by `bittyd`: a client-visible record with lifecycle, a worker-binding generation, a state revision, attach-count, leases, and last-seen state. The accepted v0.2.0 slice scopes `bittyd` to terminal sessions; no `bittyd` binary ships today. Whether it also hosts a non-terminal agent worker — including a headed agent worker — is undecided (`bitty#2546`, `D-01` in the consumer requirements).                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| Application handle                 | The `start` result the Headed Agent holds: session identity, generation, initial cursor, selected versions, snapshot, and budget reference. It is the same returned object the [Session handle](#terminology) names, viewed with its full field set; the two terms do not denote different records. It references an AgentSession and, only when the host is supervised by `bittyd`, a `bittyd` session. It is not a grant, credential, or `bittyd` lease.                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| Agent generation (`AG-2`)          | The `(AgentId, generation)` or `(PluginId, generation)` binding defined by AG-2 and used by CP-5 `current_generation`. It is separate from the session generation and from the AI runtime's `AgentSession.generation`/rotation fence.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| Panel generation                   | The `(PanelId, generation)` identity accepted by the Panel Runtime RFC. A stale panel handle is rejected by the host; an attachment may not outlive its panel generation.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| AT adapter                         | The `bitty-a11y` platform accessibility backend (AT-SPI/D-Bus, Windows UI Automation, macOS AX). It materializes a projection and forwards controlled actions; it is never a principal and never records a verdict on its own.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| Caption                            | A synchronized text rendering of the currently presented speech audio, tied to the audio clock with a stated timing tolerance and bounded latency.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| Transcript                         | An inspectable, scrollable text record of spoken content. It is bounded, redacted with the same rules as the speech text, locale-tagged, and retained under the declared retention class.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| Cursor                             | The tuple of session identity, session generation, attachment identity, attachment generation, and last applied sequence per delivery class used for replay or resynchronization.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| Panel Event Bus                    | The host-mediated, in-process, single-window transport accepted by the Panel Runtime RFC, carrying authorized, bounded, manifest-declared topics between host services, panels, and plugins. It is **not** the IPC/SMO event path and **not** the session control stream. It is not implemented by this draft.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| SMO / IPC event path               | The accepted OQ-058 SMO-1 through SMO-4 direction: typed events ride the accepted IPC transport with `(AgentId, generation, kind)` identity and the accepted framing, quotas, and `DropOldest` observation queues. Its durable sequenced control form is a candidate (`bitty-ai` AI Architecture) and is not an accepted wire.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| Session control stream             | The non-lossy, per-session-generation ordered delivery class defined by `headed.stream`. Where the accepted IPC covers it, it rides accepted request/response and chunked `chunk` frames with `seq`/`total`; a durable sequenced stream across reconnects would need a new accepted IPC/topic transport amendment.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| Panel                              | A presentation surface identity accepted by the Panel Runtime RFC, addressed as `(PanelId, Generation)` with the lifecycle `Declared -> Created -> Mounted -> Focused -> Suspended -> Disposed`. A panel projects state; it does not own execution, consent, or the session.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| Routing snapshot                   | The scoped accepted, read-only, digest-verified provider-routing projection. It is not a context or session snapshot.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| Context snapshot                   | An immutable, redacted, bounded view of compiled model context for a turn or attachment, identified by version and digest.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| Session snapshot                   | A bounded, redacted state view used to attach or resynchronize; it is not a transcript and not a replay log.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| Speech job                         | A `bitty-ai` generation job for the `Speech` task, reusing the draft multimodal generation-job lifecycle.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| `AssetRef`                         | The artifact-reference contract **proposed** as a shared Core contract. Its only source today is the draft multimodal boundary, and ADR-0016 Boundary 1 assigns output retention and artifact-reference mechanics to `bitty-execution`/`W-132`; no accepted owner has published the contract, so this draft labels it proposed and consumes it by reference. `bitty-ai` owns only the speech-job instance (job identity, media type, expiry, version, retention class); this draft does not define its fields.                                                                                                                                                                                                                                                                                                                                                                                                |
| Viseme timeline                    | Optional timing metadata that maps synthesized audio intervals to mouth-shape identifiers. It is best-effort presentation data.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| Candidate                          | Proposed by this document only; no review has accepted it.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| Owner-pending                      | Belongs to another repository or team; recorded here as a pointer, never as settled content.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| Illustrative-only                  | A sketch whose names, fields, bounds, or defaults are explicitly undecided and create no schema or compatibility promise.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |

## Product thesis and v1 placement

### Thesis

Bitty is an agent-friendly, not agent-centric terminal: AI and Agent
capabilities extend Bitty through plugins rather than becoming the fixed center
of the core product, and the core binds to no model vendor, prompt system, or AI
workflow
([Product Vision](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/product/vision.md#agent-friendly-not-agent-centric)).
The Headed Agent gives an existing Bitty user a live, human-usable agent
workspace inside that architecture: a panel composition that presents an agent
session, its streaming output, approvals, speech, and avatar state, while Core
keeps terminal truth and authority and `bitty-ai` keeps model I/O and AI policy.

The product thesis is therefore narrower than "a chat UI". The surface exists to
make agent work legible and controllable without leaving the terminal host, and
to do so without granting presentation any authority, without binding Core to a
provider, and without making Wheel the product center. It is not a replacement
terminal, not the Wheel TUI, not a general chat client, and not a model vendor's
front end. A terminal agent pane is a neighboring presentation of the same
session data, not this product: whether the Headed Agent composes with or
substitutes for such a pane is product decision `D1` and is not settled here.

### Candidate v1 scope

The following is a product-review candidate, not a commitment. It states the
minimum surface this contract set is written to support; the final scope is
product decision `D1` in [Review follow-ups](#review-follow-ups).

| Area         | Candidate v1 scope                                                                                                                                                                                           | Explicitly not v1                                                                                                                                           |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Presentation | Text-first panel composition: streaming turn text, tool proposal/approval cards, status, errors, and session attach/detach.                                                                                  | Chat transcript persistence, custom chat transport, shared Wheel chat contract.                                                                             |
| Interaction  | Keyboard-complete operation, explicit approval decisions, stop/cancel, reconnect and resync.                                                                                                                 | Voice approval, gesture approval, ambience-driven control.                                                                                                  |
| Speech       | Optional speech synthesis producing an `AssetRef` and text, with synchronized captions and an inspectable transcript as **candidate** v1 requirements under `D4`; playback off by default and owner-pending. | Accepted baseline captions/transcript (none exists), voice cloning, speaker identification, always-on audio, autoplay defaults, v1 playback implementation. |
| Avatar       | Optional 2D or software-rendered state indicator with text-equivalent announcements.                                                                                                                         | 3D rigs, GPU scene graphs, photoreal characters, biometric sensing.                                                                                         |
| Media        | Core presentation primitives only; no v1 media package.                                                                                                                                                      | Lua media plugin, in-process decoder, GPU/window/PTY access from a plugin or helper.                                                                        |
| Recording    | Ephemeral by default; recording and export only under explicit retention consent if the product offers them at all (`D3`).                                                                                   | Default recording, silent export, indefinite retention.                                                                                                     |
| Platforms    | ADR-0002 support tiers unchanged; text works at every class, and captions exist only in the `P-C` text class, not `P-D`.                                                                                     | Any presentation class upgrading a platform's support tier.                                                                                                 |

### Placement and the `ai-panel` predecessor

The recorded predecessor is `bitty-terminal.ai-panel`, the bundled agent panel
surface. `bitty` CTX-0886 removed `ai-panel` (with `browser-panel`,
`mail-panel`, and `project`) from Core on 2026-09-30; AI surfaces move to the
separate optional `bitty-ai` extension, and the accepted subset of the former
surface is retained as a dated record in
[AI Surface Accepted Record](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/ai-surface-accepted-record.md).
The split decision recorded `ai-panel` as a **split-later hybrid**: Core-owned
bounded snapshots, semantic-zone context, and MCP transport feed a Lua chat UI,
history, and commands
([Bundled-Plugin Split Decision](https://github.com/bitty-terminal/bitty-plugins-docs/blob/main/product/bundled-plugin-split-decision.md#ai-panel--split-later-hybrid)).
That split-later posture is **superseded** for `ai-panel`: the 2026-09-30 removal
settled that the surface does not remain a bundled Core plugin to be split later,
and the predecessor record is retained as history, not as a pending split.

This document treats the Headed Agent as that predecessor's successor candidate,
not as a generic "future sibling repository". Repository placement is **fixed**
by the accepted AI Surface record: AI surfaces move to the separate optional
`bitty-ai` extension, so this draft consumes that placement instead of asserting
it as an assumption. What remains open is product decision `D5`, limited to the
user-facing name and identity of the surface, recorded as packaging decision
`P-1`: a renamed `ai-panel` successor versus a new plugin identity. It is not a
Core feature, not a Wheel module, and not a bundled plugin; whether a distribution
carries it is `P-3` and no distribution entry is claimed. The panel-provider
registration and mount contract it needs is accepted by RFC-0006; the remaining
`bitty-ai` surface and distribution questions stay with the packaging decisions.

The predecessor's Lua chat UI, history, and commands map to the candidate v1 scope
as follows: chat presentation maps to the text-first panel composition; history
maps only to the ephemeral session snapshot and is **not** a persistent chat
transcript (excluded in v1); commands map to the approved command/approval path,
not to a private chat command set. None of the three is carried over as an
accepted requirement.

### Open product decisions versus settled contract inputs

The contract inputs below are already accepted and are not reopened here. The
product decisions below are genuinely open and are recorded in
[Review follow-ups](#review-follow-ups) rather than silently answered.

Settled contract inputs consumed by this draft:

- Panel identity, lifecycle, focus routing, and Event Bus grammar from the
  accepted Panel Runtime RFC and RFC-0006.
- `AgentId`, `PanelId`, `ExecutionContextId`, and `GenerationId` separation from
  ADR-0013, with panels never owning executions.
- Platform support tiers from ADR-0002; presentation classes never upgrade a
  tier.
- The accessibility baseline obligations retained by Core under ADR-0016
  Boundary 3 and elaborated by accepted `W-134`: projection-not-authority,
  focus association, tab order, announcements, reduced motion, and contrast.
- The provider-routing publication/proof split in the scoped accepted R1
  binding.
- Consent, approval, redaction, and retention floors in the normative security
  corpus and the accepted IPC and Agent RFC.
- The `bittyd` v0.2.0 staging and session-grained attach/detach in ADR-0019.

Open product decisions, detailed in
[Review follow-ups](#review-follow-ups): `D1` product thesis and v1 scope, `D2`
synthetic-speech disclosure, `D3` recording/export/delete rights and retention
class, `D4` accessibility floor beyond the accepted baseline, `D5` identity and
user-facing name, `D6` consent surface, and `D7` gate correction.
`D7` is discharged by the nine-row table in this revision; the other six remain
open owner decisions.

## Boundary and sibling relationship

### Three consumers, one platform

Core is the designated owner of record for the neutral platform surface:
transport, authentication, consent enforcement, PTY ownership and process
permission enforcement, panel identity and lifecycle, and the event-bus
transport. Core authorizes and fences execution; the accepted ADR-0016
Boundary 1 direction moves AI-agnostic supervision and execution to
`bitty-execution` under `W-132`, not into Core and not into `bitty-ai`. The
panel identity, lifecycle, and Event Bus contracts are accepted by the
[Panel Runtime RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/panel-runtime-rfc.md);
their implementation is not claimed here. `bitty-ai` is the designated owner
of reusable AI mechanisms. Wheel and the Headed Agent surface are independent
consumers of those mechanisms, and the Headed Agent is an optional `bitty-ai`
extension rather than a Core or Wheel component. This is an ownership design
target; Core panel, Event Bus, `bitty-execution`, and `bittyd` implementation do
not exist as a result of this document.

```text
Bitty Core / bittyd (ADR-0019 v0.2.0 staging; no binary claimed)
  |-- IPC, consent enforcement, panel runtime (accepted contract),
  |   Event Bus (accepted contract), PTY ownership and permission enforcement
  |
  |-- bitty-execution (W-132 direction: AI-agnostic supervisor)
  |-- bitty-graphics (W-133), bitty-a11y (W-134), platform services (W-136)
  |
  `-- bitty-ai
        |-- provider/model routing and inference jobs
        |-- context compilation and snapshots
        |-- AI event vocabulary and speech job contract
        |
        |-- Wheel (Coding Agent Harness)
        |     `-- coding workflow, task graph, harness policy
        |
        `-- Headed Agent surface (optional bitty-ai extension)
              `-- media presentation
                    |-- Core extension component (bitty-graphics-style), or
                    `-- post-1.0 native helper (no GPU/window/PTY handle)
```

There is deliberately no edge between Wheel and the Headed Agent surface. They
are sibling consumers, not layers. Shared needs are promoted to Core or
`bitty-ai`; a private cross-consumer dependency is not created.

### Responsibility boundary

The table records design-target ownership. It does not claim that Core panel,
Event Bus, `bittyd`, `bitty-execution`, or media presentation code exists.

| Surface                            | Owns (design target)                                                                                                                                                                                                                                                                                                                           | Consumes                                                            | Must not                                                                                                                                                                                                    |
| ---------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Bitty Core / `bittyd`              | Transport, authentication, consent enforcement, scope enforcement, panel identity and lifecycle, Event Bus transport, PTY ownership, permission/resource enforcement, host-side generation fencing.                                                                                                                                            | No model I/O and no AI policy.                                      | Select models, assemble prompts, synthesize speech, animate avatars, render plugin UI, or supervise AI-agnostic execution itself.                                                                           |
| `bitty-execution` (`W-132`)        | AI-agnostic execution supervision: spawn-time lifetime, cancellation execution, PTY/process resource mechanics, structured outcome, bounded output, recovery.                                                                                                                                                                                  | Core authorizations, generation fences, and the AI-agnostic bridge. | Learn agent or task ontology, grant consent, or own `bitty-ai` semantic state.                                                                                                                              |
| `bitty-ai`                         | Provider abstraction, routing, inference jobs, context compilation, snapshots, AI event schemas, speech job contract, viseme vocabulary.                                                                                                                                                                                                       | Core transport, consent verdicts, and provider plugins.             | Own a window, panel, audio device, GPU surface, chat layout, or credential store.                                                                                                                           |
| Wheel                              | Coding Agent Harness policy: coding workflow, task graph, tool composition, harness-level consent and audit behavior.                                                                                                                                                                                                                          | Core and `bitty-ai`.                                                | Gain a headed UI, speech, avatar, or 3D responsibility from this design.                                                                                                                                    |
| Headed Agent surface               | Chat and interaction composition, presentation policy, panel layout, speech controls, avatar appearance and animation mapping.                                                                                                                                                                                                                 | Core and `bitty-ai`; presentation primitives.                       | Depend on `wheel.*`; hold execution authority; bypass Core authorization; own provider credentials; define a plugin category or capability family.                                                          |
| Provider plugin                    | Vendor protocol adaptation, transport, and provider-specific request mapping behind an opaque credential handle.                                                                                                                                                                                                                               | `bitty-ai` provider contract and Core grants.                       | Widen capabilities, raise a privacy class, or expose credentials to the agent, plugin, or presentation.                                                                                                     |
| Media presentation (owner-pending) | Presentation of audio, GPU-backed 2D/3D, or avatar content through a trusted Core extension component (reaching GPU, window, and audio resources only through Core-owned interfaces) or, post-1.0, an untrusted native helper, with a declared capability and budget. Declarative text/2D composition stays with Core presentation primitives. | `bitty-ai` speech/avatar events and Core grants.                    | Be a v1 Lua plugin/package; hold raw GPU, window, PTY, or host-Rust handles outside Core-owned interfaces; become a session authority; read transcripts; capture camera or microphone input; persist media. |

### The `wheel.*` prohibition

The Headed Agent surface must not import, require, call, configure, inspect, or
otherwise depend on any `wheel.*` module, package, service, configuration key,
environment variable, or internal API. It does not load Wheel as a library, act
as a Wheel front end, or reuse Wheel private state. The prohibition also covers
spawning or wrapping the Wheel CLI, parsing Wheel CLI output, executing or
reading the executable `.wheel/init.lua`, reading
`$XDG_CONFIG_HOME/wheel/init.lua`, and reading
`$XDG_STATE_HOME/wheel/trusted_projects.json`. The paths are not one class and
are split deliberately:

- `.wheel/init.lua` is **workspace project configuration**, not user state
  outside authority. It sits inside the model-visible workspace, it is
  untrusted before consent and executable after consent, a tool may write it,
  and the next load re-enters Wheel's per-project trust gate, content-hash
  pinning, and revocation checks (owner decision D5). Reading or executing it
  from this surface would **bypass that trust gate**, which is a distinct risk
  from reading host state.
- `$XDG_CONFIG_HOME/wheel/init.lua` and
  `$XDG_STATE_HOME/wheel/trusted_projects.json` are user-owned Wheel
  configuration and state **outside** model-visible workspace authority, per
  Wheel's
  [SECURITY-MODEL.md](https://github.com/bitty-terminal/wheel/blob/main/SECURITY-MODEL.md)
  and [AGENTS.md](https://github.com/bitty-terminal/wheel/blob/main/AGENTS.md).

A negative test for each path belongs to the future implementation task. The
test set names the three cases separately and adds two behavioral cases: a
`.wheel/init.lua` write attempt that must not become an execution path, and a
trust-gate bypass attempt that must fail closed before evaluation. Parsing Wheel
CLI output is also a dependency on Wheel's presentation contract and belongs to
the same negative set.

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
owner decisions D1, D2, D4, and D6, and
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
host-brokered execution service is accepted (owner decisions D4 and D6); that
remains a separate Wheel decision and does not make the Headed Agent a Wheel UI
plugin.

## Contract families and versioning

### Candidate contract families

Each family is versioned independently. The labels below are contract-family
names for review, not wire methods, Lua modules, package names, or accepted API
spellings.

| Family            | Candidate version | Accountable owner(s)                                                                               | Purpose                                                                                                                                                                             | Status |
| ----------------- | ----------------- | -------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------ |
| `headed.session`  | 1                 | Core owner (host authority, pending `bitty#2546`) + AI owner                                       | Session handle, `start`/`attach`/`detach`/`stop`, session and attachment generations, cancellation, reconnect.                                                                      | Draft  |
| `headed.stream`   | 1                 | Core owner + AI owner                                                                              | Bounded delivery classes, sequencing, backpressure, replay, and resynchronization.                                                                                                  | Draft  |
| `headed.routing`  | 1                 | AI owner (resolution); Core owner (publication authority)                                          | Model-profile request and routing against the scoped accepted routing snapshot.                                                                                                     | Draft  |
| `headed.context`  | 1                 | Core owner + AI owner                                                                              | Context snapshot and session snapshot references, redaction, and viewer filtering.                                                                                                  | Draft  |
| `headed.tools`    | 1                 | Core owner (authorization); `bitty-execution` (execution under `W-132`); AI owner (proposal relay) | Tool proposal, approval request, verdict, execution, and result events.                                                                                                             | Draft  |
| `headed.panels`   | 1                 | Core owner + Headed Agent owner (presentation only)                                                | Panel-scoped presentation policy: suspend, resume, and audio focus. Attachment creation and removal stay in `headed.session`; panel identity and lifecycle are consumed, not owned. | Draft  |
| `headed.speech`   | 1                 | AI owner (contract); owner-pending media presentation                                              | Speech request, stream, cancel, playback handoff, and asset reference.                                                                                                              | Draft  |
| `headed.avatar`   | 1                 | AI owner (events); Headed Agent owner (mapping)                                                    | Presentation state, animation, and viseme events; renderer ownership stays outside `bitty-ai`.                                                                                      | Draft  |
| `headed.budget`   | 1                 | Core owner + AI owner                                                                              | Budget envelope, enforcement attribution, and degradation actions.                                                                                                                  | Draft  |
| `headed.platform` | 1                 | Core owner + accessibility owner (`bitty-a11y` adapter)                                            | Capability detection, presentation classes, and AT adapter availability on Linux, macOS, Windows, and FreeBSD.                                                                      | Draft  |

### Family boundaries

Several families touch the same mechanic. The authoritative family or accepted
source is named once here so the draft does not define it twice:

| Shared mechanic                                               | Authoritative family or source                                                               | Boundary in this draft                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| ------------------------------------------------------------- | -------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Session handle, session generation, `attach`/`detach`         | `headed.session`                                                                             | `headed.panels` only requests panel identity; panel lifecycle stays with the accepted Panel Runtime RFC.                                                                                                                                                                                                                                                                                                                                                             |
| Session snapshot content                                      | `headed.session` defines the bounded attachment view; `headed.context` defines context views | A session snapshot is neither a context snapshot nor a transcript, and it never substitutes for `attach` authorization.                                                                                                                                                                                                                                                                                                                                              |
| Context snapshot and CP-5 `current_generation`                | `headed.context` consuming draft CP-5                                                        | CP-5 binds the AG-2 agent generation; `headed.session` does not own that generation.                                                                                                                                                                                                                                                                                                                                                                                 |
| Event sequencing, backpressure, replay, resynchronization     | `headed.stream` for the non-lossy session control stream                                     | The Panel Event Bus carries authorized observation and lifecycle topics with its PR-7/PR-8/PR-9 `DropOldest` queues; approvals, stop, tool results, and speech-job control never travel on it. The control stream is not an accepted wire: where accepted IPC covers it, it rides request/response plus chunked `chunk` frames, and a durable sequenced form needs a new accepted transport amendment. `headed.stream` redefines neither bus queues nor IPC framing. |
| Session authority, generation, attach leases                  | `bitty#2546` (Core contract) with ADR-0019 staging and `R-B09` single-writer rule            | `headed.session` records consumer requirements only. A `bittyd` session, an AgentSession, and the application handle are distinct; `start` does not create all three.                                                                                                                                                                                                                                                                                                |
| Plugin identity, package category, distribution, signing      | `bitty-plugins-docs` plugin-platform and packaging contracts                                 | The Headed Agent product owner (`bitty-ai`) chooses whether to publish a package; it does not define the platform category or signing rules.                                                                                                                                                                                                                                                                                                                         |
| Topic grammar, manifest produce/consume, per-principal ledger | Accepted Panel Runtime RFC Event Bus                                                         | `headed.panels` consumes the grammar and ledger; it mints neither.                                                                                                                                                                                                                                                                                                                                                                                                   |
| Panel identity, lifecycle, focus, suspend                     | Accepted Panel Runtime RFC and RFC-0006                                                      | Attachment lifecycle is a separate relation; see [Panel lifecycle versus attachment lifecycle](#panel-lifecycle-versus-attachment-lifecycle).                                                                                                                                                                                                                                                                                                                        |
| Speech job status and `Unknown`                               | Draft multimodal generation-job lifecycle consumed by `headed.speech`                        | Avatar, animation, and viseme events never carry a terminal job state.                                                                                                                                                                                                                                                                                                                                                                                               |
| Rich rendering controls                                       | Normative `P0-AC-005`, `P0-AC-009`, `P0-AC-010`                                              | `headed.avatar` and `headed.panels` inherit them; transcript and chat layout stay out of scope.                                                                                                                                                                                                                                                                                                                                                                      |

### Version negotiation authority

The **application session generation returned by `start` is the single
authority** for the contract-version binding of every session-bound family in
the table. `start` returns the selected version for each family it negotiates; a
session generation cannot change that binding in place. `headed.platform` is the one
presentation-scoped family with a standalone pre-session probe path, so `start`
is not its only possible authority. `start` here is the application-level
operation; whether it also creates a `bittyd` session remains `bitty#2546` and
does not change this version binding.

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
- Plugin capability vocabulary is host/plugin-platform-owned and closed: the accepted Plugin
  Platform RFC fixes the families and manifest grammar, the Panel Runtime RFC
  fixes the `panel.*` family, and adding a plugin capability is a contract
  change, not an additive `bitty-ai` minor version. Two separate vocabularies are
  not plugin grants: `headed.*` presentation families are contract families in
  this draft, and `ai.*`/`agent.*`/`mcp.*` extension-contributed identifiers are
  owner-pending (`bitty-plugins` has not accepted that manifest extension).
- Deprecation names the replacement, the affected versions, the transition
  period, and the removal condition. A major version is not removed without a
  documented migration or redirect.
- Contract versions are application-level. They do not replace, overload, or
  reinterpret the IPC wire version `v` defined by the accepted RFC.

## Agent session lifecycle and bounded streaming

### Authority split

Three session concepts are routinely conflated and are kept separate here:

| Level              | What it is                                                                                                                                                         | Identity and generation                                                                                                                                                   | Authority and sole writer                                                                                           | Status                                                          |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------- |
| `bittyd` session   | One unit registered and supervised by the local daemon: a terminal session and, only if `bitty#2546` permits non-terminal hosting, an agent worker (open; `D-01`). | Server-assigned session id, a worker-binding generation that increments only on worker bind or rebind (`R-S16`), and a state revision that covers attach/detach changes.  | `bittyd` is the only writer of registry, lifecycle, generation, attach-count, lease, and last-seen state (`R-B09`). | ADR-0019 staging accepted; the contract is `bitty#2546`, open.  |
| AgentSession       | The accepted IPC and Agent RFC lifecycle anchor: `Created`, `Running`, `WaitingToolResult`, `Completed`, `Failed`.                                                 | Its own RFC lifecycle state; the application-level session identity and generation used by `start`/`attach` belong to the handle, with placement open under `bitty#2546`. | Core authorizes and holds the consent ledger; the AI side owns model-facing turn and job state.                     | RFC contract accepted at design level; no implementation claim. |
| Application handle | The object the Headed Agent holds after `start`: identity, generation, cursor, selected versions, snapshot, budget reference, attachments.                         | References the application session generation; carries its own attachment generations.                                                                                    | Owned by the Headed Agent as a consumer object; it grants nothing.                                                  | Candidate (this draft).                                         |

`start` does not necessarily create all three. Whether a headed `start` creates
a `bittyd` session, only an AgentSession, or only an application handle that
references an already-hosted session is exactly what `bitty#2546` must settle;
this draft does not decide it. A Headed Agent that runs without `bittyd` (the
supported daemon-absent state under ADR-0019 D5) cannot assume a `bittyd`
session, a lease, or restart persistence. When `bittyd` does host the session,
the worker or agent runtime remains the only writer of the run's task, context,
tool, and audit state; `bittyd` never stores or mutates that state (`R-B09`).

The **application session generation and the `bittyd` worker-binding generation
are two different facts**. The `bittyd` generation of `R-S16` is a
worker-binding counter: it increments on a worker bind or rebind, never on
attach or detach, and it cannot fence an approval or a consent elevation. The
application session generation is the fence this draft uses for session
mutations, approvals, and consent/elevation binding. Its issuer, durable record,
and sequencer are exactly what `bitty#2546` must define. If `bitty#2546` places
the session record in `bittyd`, that contract must add a **separate**
application-session-generation field and name `bittyd` as its sole writer; the
existing worker-binding generation is not that field and is not reused for it.
Until `bitty#2546` decides, this draft names no issuer and no record, and a
consumer that has only a `bittyd` worker generation cannot claim a session
generation.

The accepted RFC `AgentSession` state names remain the lifecycle anchor at the
contract level. The `bitty-agent` crate's session lifecycle is draft and not
normative, and the AI-0257 vocabulary-conformance ledger records implementation
drift without resolving it. At the pinned `bitty-agent` revision `a8e4112`,
`bitty-agent-api` re-exports `AgentId`, `AgentError`, and `ErrorClass` (plus the
`AgentId` bound checks) and is a **test-only dev-dependency** used for
report-only conformance evidence; the AI runtime does not depend on
`bitty-agent` or on its state machine in production. `Created`, `Running`, and
`WaitingToolResult` are the accepted RFC and draft `bitty-agent` phases, not
`bitty-ai` runtime states. The `bitty-ai` runtime records `Active`, `Completed`,
`Failed`, and `Canceled`, mapped explicitly in
[Outcome levels](#outcome-levels). Whether the vocabularies converge or the
crate is archived remains open; this draft renames neither and claims no
convergence.

Execution authority follows ADR-0016 Boundary 1. Core authorizes every
`observe`, `read_output`, `write_input`, `signal`, `cancel`, `attach`, and
`transfer` operation, retains PTY ownership and permission/resource
enforcement, rejects stale host handles itself, and keeps the AI-agnostic typed
execution bridge. The AI-agnostic supervisor `bitty-execution` (`W-132`
direction, no implementation claimed) supervises and executes. Core does not
supervise execution itself, and `bitty-ai` does not own host process or PTY
lifecycle. The execution record's durable placement remains part of the draft
R1 disposition and is not assigned here.

Generation sequencing is a single-writer problem, not a declared Core role. The
accepted `R-B09` rule requires exactly one writer per fact. If `bitty#2546`
places the session record in `bittyd`, `bittyd` issues the application session
generation and is its only writer; whether it also sequences the session control
stream is part of `bitty#2546`, and any separation of issuance from sequencing
uses the handoff invariant below. If placement is elsewhere, the same invariant
applies to that authority. The handoff adopts the exact issued generation in one
recorded step; no event is admitted before adoption, a mismatch fails closed, and
two components never advance one generation. Issuance and sequencing are
separate roles in that case: the issuer records the generation, the sequencer
adopts it before emitting any event, and neither may advance the other's value.
The earlier draft sentence naming Core as the designated sequencer is withdrawn:
it decided an open placement question in the body while the open-point table
called it undecided.

`bitty-ai` owns the model-facing binding: routing snapshot consumption, context
snapshot, turn and generation-bound events, tool proposals, speech jobs, and
media event schemas. The Headed Agent surface owns presentation, local
interaction, and the mapping of semantic events to widgets, speech controls, and
avatars.

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
cancel and the recorded outcome is the actual result or `Unknown`. The accepted
RFC `AgentSession` terminal set stays `Completed`/`Failed`; a cancellation
recorded in that vocabulary is one of those two with a cancellation
attribution. The `bitty-ai` runtime instead records `Active` while accepting
turns and a terminal `Canceled` on cancellation; this contract maps to it rather
than rewriting it into `Completed` or `Failed`, and the mapping is recorded in
[Outcome levels](#outcome-levels).
This draft adds no third state to the accepted RFC set and claims no vocabulary
convergence. A speech or media job may separately report `Queued`, `Preparing`,
`Running`, `Streaming`, `Completed`, `Failed`, `Cancelled`, or `Unknown` in its
own draft media-job status. `Completed`, `Failed`, and `Cancelled` are terminal
for that job; `Unknown` is a reconciliation state, never a terminal success or
failure. The levels are never collapsed: a job reaching `Cancelled` does not by
itself terminate the session, and a session reaching `Completed`, `Failed`, or
`Canceled` does not rewrite a job's recorded status. `Unknown` is reconciled by
status inspection or user direction before any retry; no rollback and no
exactly-once claim is made. `stop` is idempotent, releases owned resources at a
bounded grace deadline, and reports the retention outcome for evidence. A stale
session generation cannot stop a newer session.

### Outcome levels

Session outcomes and job outcomes are separate records with separate state
sets. A consumer must read the level it is acting on:

| Level                            | State set                                                                                    | Terminal states                    | Rule                                                                                                                                                                                                                                                                                                                                |
| -------------------------------- | -------------------------------------------------------------------------------------------- | ---------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Session, accepted RFC vocabulary | `Created`, `Running`, `WaitingToolResult`, `Completed`, `Failed`                             | `Completed`, `Failed`              | Cancellation records one of the two terminal states with cancellation attribution. The RFC vocabulary is not extended.                                                                                                                                                                                                              |
| Session, `bitty-ai` runtime      | `Active`, `Completed`, `Failed`, `Canceled`                                                  | `Completed`, `Failed`, `Canceled`  | The accepted RFC `Created`, `Running`, and `WaitingToolResult` phases all map to `Active`; `Completed` and `Failed` map unchanged; a cancellation is recorded as `Canceled`. `Canceled` is terminal and is not rewritten. A UI maps either vocabulary to one user-visible cancellation outcome; the records are mapped, not merged. |
| Media job                        | `Queued`, `Preparing`, `Running`, `Streaming`, `Completed`, `Failed`, `Cancelled`, `Unknown` | `Completed`, `Failed`, `Cancelled` | `Unknown` is reconciled by status inspection before retry and is never recorded as success or failure.                                                                                                                                                                                                                              |
| Attachment                       | `attached`, `suspended`, `closed`, `failed`                                                  | `closed`, `failed`                 | An attachment outcome never terminates the session and never substitutes for a session or job outcome.                                                                                                                                                                                                                              |

The spellings differ deliberately: the `bitty-ai` session state is `Canceled`
(one `l`, matching that vocabulary) and the media-job state is `Cancelled` (two
`l`). They are separate records; a UI may present both as "cancelled", but the
stored states are not merged.

### Controlled session transitions

`resume`, `takeover`, and `reset` are not additional free-standing operations
alongside `start`, `attach`, `detach`, and `stop`. The first two are controlled
transitions with fail-closed authorization; `reset` is deliberately not defined
by this draft.

| Transition       | Preconditions                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             | Generation effect                                                                                                                                           | Consent and effects                                                                                                                                                                                                                                         |
| ---------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Session `resume` | The principal is authenticated, the session is retained and not live under another writer, and the caller passes the current application session generation.                                                                                                                                                                                                                                                                                                                                              | Issues a new application session generation through the authority `bitty#2546` designates; never reuses or advances the `bittyd` worker-binding generation. | Prior elevation is invalid. Re-grant requires a fresh prompt. No queued command, tool effect, speech request, or animation is replayed.                                                                                                                     |
| `takeover`       | The target holds session-management authority (or an explicit user-owned host policy names it), the handoff is recorded with both principals and the target session generation, and the target principal's current grants and consent are re-authorized. **The current owner may request a handoff but cannot grant control**: detaching another client's lease requires session-management authority (`R-A03`), and one controller lease exists unless Core adopts a multi-controller model (`R-B09.4`). | Issues a new application session generation through the authority `bitty#2546` designates; the old generation is invalidated.                               | Fresh consent is required. Pending actions are reconciled, not replayed, and media jobs started under the old generation are reconciled to their recorded status before the new generation is used. A stale owner cannot continue under the old generation. |
| `reset`          | Not defined here. A destructive reset is a new-session operation that requires a separate Core contract.                                                                                                                                                                                                                                                                                                                                                                                                  | None.                                                                                                                                                       | This draft provides no reset path and no implicit consent. A caller that needs a clean session performs a new `start`.                                                                                                                                      |

Session `resume` and `takeover` retain the contract-version binding of the
original `start`; they do not renegotiate it. A client that requires a different
major performs a new `start` with a fresh session and fresh consent.

Session `resume` is **not restart persistence**. ADR-0019 excludes sessions that
survive a daemon restart; a session retained across a detach is still live in the
same daemon instance. After a daemon restart the old session and generation are
gone, and the client performs a new `start` rather than claiming recovery.

`takeover` transfers the session-controller relation. It is not the OQ-083 panel
lease or handoff: a panel lease change follows the accepted OQ-083 kernel and
does not by itself move session control, and a session takeover does not grant a
panel lease. Authority to move control comes from `R-A03`
session-management authority plus the target principal's own re-authorization;
holding the current session lease is a precondition for requesting a handoff,
never a source of authority to grant one.

### Generation, version, and consent axes

The word "generation" is not a single universal fence. The following axes are
separate, and a contract family must name the axis it uses:

| Axis                                    | Issued or bound by                                                                                                                                                                                                                                                                                                                                                                | What it fences                                                                                     | Binds consent or elevation?                                                                        |
| --------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| Session generation                      | The application-level authority designated by `bitty#2546`, returned by `start`. If `bittyd` owns the session record, `bitty#2546` must add a distinct field and `bittyd` is its sole writer (`R-B09`); the `R-S16` worker-binding generation is not this axis. Whether `bittyd` also sequences the control stream is part of `bitty#2546`. This draft does not decide placement. | Session-level mutating operations, approval verdicts, session stop, and session-scoped media jobs. | Yes, together with the principal and scope.                                                        |
| Attachment generation                   | The attachment authority at `attach`.                                                                                                                                                                                                                                                                                                                                             | One attachment's local handles, cursors, and presentation state.                                   | No. It never grants authority or binds consent.                                                    |
| Panel generation                        | The host under the accepted Panel Runtime RFC, as `(PanelId, generation)`.                                                                                                                                                                                                                                                                                                        | Panel identity, mount, focus, suspend, and stale panel-handle rejection.                           | No.                                                                                                |
| Agent generation (`AG-2`)               | The `bitty-ai` agent/plugin generation, as `(AgentId, generation)` or `(PluginId, generation)`; CP-5 `current_generation` binds this axis.                                                                                                                                                                                                                                        | Agent-level elevation, context assembly freshness, and turn-scoped records.                        | Yes for agent elevation. CP-5 itself grants nothing.                                               |
| `GenerationId` (ADR-0013)               | The accepted core ontology, for reloads, leases, and handoffs.                                                                                                                                                                                                                                                                                                                    | The accepted fencing identity that composes with the narrower axes above.                          | Indirectly, where a lease or handoff carries authority; it is not itself consent.                  |
| Execution projection                    | The draft [execution-projection binding](../specifications/execution-projection-binding-candidate.md#the-binding-relation); candidate only.                                                                                                                                                                                                                                       | The candidate panel-to-execution projection relation.                                              | No. The projection is not an authorization.                                                        |
| Routing snapshot version and digest     | Core publication authority under the scoped accepted provider-routing binding, with the AI helper owning the authoritative registry and proof. No separate routing "generation" is defined by this draft.                                                                                                                                                                         | The exact provider-routing projection validated for one turn.                                      | No. It selects routing data; it never grants a scope.                                              |
| Context snapshot version and generation | `bitty-ai` context compilation. CP-5 `current_generation` is the AG-2 agent generation, separate from the session generation and from the runtime `AgentSession.generation`/`rotate_generation` fence.                                                                                                                                                                            | The exact compiled context and its freshness.                                                      | No. Expansion re-authorizes the read; the snapshot itself is not a grant.                          |
| Speech job identity and status          | The draft `bitty-ai` media-job lifecycle.                                                                                                                                                                                                                                                                                                                                         | Cancellation, status inspection, and `Unknown` reconciliation for one speech job.                  | No. Speech egress consent binds to the session generation, principal, provider, and privacy class. |

The combined elevation binding is layered, and each layer fences a different
fact:

| Layer                       | What it binds                                                                                                              | What invalidates it                                                                                     |
| --------------------------- | -------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| Grant (accepted IPC ledger) | `(UID, AgentId)` pair, scope, and time bound; instance-scoped, revocation survives reconnect and restart.                  | Revocation, expiry, or a changed scope. A session-generation change does **not** revoke the grant.      |
| Session elevation           | The current application session generation plus the principal and scope; a per-session narrowing of what the grant allows. | A new session generation, a takeover, or an expired elevation. It can never widen the underlying grant. |
| Agent elevation (`AG-2`)    | `(AgentId, generation)` / `(PluginId, generation)` and CP-5 `current_generation`.                                          | Agent suspend, dispose, reload, or generation advance.                                                  |
| Approval                    | The exact call digest, target, schema version, expiry, and the session generation that bound the decision.                 | Argument/target mutation, expiry, stale session generation, replay, or a different principal.           |

- A stale session generation invalidates elevation and pending approvals; a
  stale attachment generation closes only that attachment; a stale panel
  generation rejects the panel handle; a stale AG-2 generation fails context
  assembly with `StaleGeneration`. None of these is interchangeable.
- An execution projection is a candidate relation, not a fourth authority. It
  binds a `PanelId` and generation to an `ExecutionContextId` for presentation
  and never lets a panel own an execution. It fails closed when any bound
  generation is stale.
- Consent is granted per principal (UID and AgentId) and scope; session
  elevation narrows it under the current session generation, and approval adds
  the exact call digest, target, schema version, and expiry. An attachment
  generation, panel generation, routing version, context version, agent
  generation, or media-job identity never substitutes for that binding, and a
  session generation never replaces the grant.
- A stale routing or context version fails the turn closed rather than silently
  reusing an older projection. A routing snapshot pin is not a durable lease:
  any provider, model, capability, context-window, cost, or alias mutation
  invalidates the proof before strict use, and the turn fails closed rather
  than continuing on the old pin.
- An agent suspend, dispose, or reload invalidates prior elevation even before a
  new session generation is issued. Panel suspension does not invalidate agent
  elevation; it expires only the attachment-local handles.
- A generation or version value is a fence, not a secret and not a capability.
  Holding one grants nothing without the caller's own authorization.

### Event ordering and loss

- **Three transports, three names.** The **Panel Event Bus** is the accepted
  host-mediated, in-process, single-window bus of the Panel Runtime RFC. The
  **SMO/IPC event path** is the accepted OQ-058 direction, where typed events
  ride the accepted IPC transport with `(AgentId, generation, kind)` identity,
  the accepted framing, quotas, and attributed `DropOldest` observation queues.
  The **session control stream** is this draft's non-lossy ordered class on
  `headed.stream`. Session lifecycle, `attach`/`detach`, stop, approval requests
  and verdicts, tool results, final turn messages, and speech-job lifecycle
  travel on the session control stream, never on either bus. Treating a control
  event as a bus topic would put an approval or a stop behind a lossy queue and
  is forbidden.
- **The control stream has no accepted durable wire.** The accepted IPC contract
  provides request/response and chunked `chunk` frames with `seq`/`total`,
  deterministic timeouts, and bounded side queues; it explicitly leaves an event
  subscription model as a candidate. Where those accepted mechanisms suffice
  (per-request correlation, chunk ordering, cancellation at a chunk boundary),
  the control stream consumes them unchanged. A durable, sequenced control
  stream that survives reconnects needs a **new accepted IPC transport or topic
  amendment**; this draft specifies the required semantics and claims no such
  wire. The SMO-4 delivery semantics are inherited by reference: envelope
  identity, sender/recipient attribution, deadline, priority, `message_id`
  deduplication, cancellation, and expiry. The Panel Event Bus is neither
  transport: it has no cross-process routing and must not be described as the
  SMO bus.
- **Cross-window delivery has no v1 path.** The accepted bus is
  single-process and single-window, and a topic never escapes a window without
  an explicit cross-window or cross-process transport contract (`RFC-OQ-4`).
  Cross-window session attachment therefore requires that accepted contract; it
  is not provided by the Panel Event Bus, and this draft does not assume it.
- The session-generation issuing authority owns its sequence. The sequencer is
  whichever authority `bitty#2546` designates; this draft designates none and
  does not place a sequencer in Core by assertion. One sequencer assigns a
  monotonically increasing sequence number per session generation at commit,
  before queue admission, after the issuer-to-sequencer handoff recorded in
  [Authority split](#authority-split). Every event carries that number; a
  dropped event still consumes a sequence number, so a consumer can distinguish
  a dropped event from an event that was never produced. AI turn sequencing is
  nested inside the session-generation sequence: it can order events within a
  turn, but it cannot advance the session cursor or create a second session
  sequence.
- **Gap detection applies only to non-lossy ordered classes:** session
  lifecycle, stop, approval request and verdict, tool result and error, final
  turn message, speech job lifecycle and terminal status, and other control
  events. A gap in one of these classes triggers resynchronization and is never
  treated as benign loss.
- **Lossy classes use lost-count and cumulative-state recovery:** turn deltas,
  progress, routing fallback notices, avatar state, animation steps, and viseme
  markers may be coalesced or dropped. A gap in a lossy class is expected; the
  consumer increments a lost counter and recovers by class rather than forcing a
  session resynchronization:
  - Turn deltas recover from a cumulative turn snapshot or a later cumulative
    delta; the final turn message is a non-lossy control event on the session
    control stream, not a lossy delta.
  - Routing notices recover by reading the current routing snapshot and binding;
    a missed notice is not an effect.
  - Avatar state recovers from the latest state revision; animation and viseme
    loss degrades presentation only.
- **Durable media-job events are separate from ephemeral presentation events.**
  Speech job lifecycle and terminal events use the draft multimodal status set
  `Queued`, `Preparing`, `Running`, `Streaming`, `Completed`, `Failed`,
  `Cancelled`, and `Unknown`. That class travels on the non-lossy session
  control stream and is gap-detected; it is not a Panel Event Bus topic. Avatar
  state, animation, and viseme events are ephemeral presentation, lossy, and
  latest-wins. A terminal media-job state is never carried only by an ephemeral
  avatar event, and `Unknown` is never treated as a terminal success or failure.
- Audio uses its own media-channel chunk sequence and playback buffer. It does
  not use the session sequence for gap detection and never triggers a session
  resynchronization.
- Ordering is guaranteed only where stated: per session generation for control
  and lifecycle events, per turn for model output, and FIFO within one Panel
  Event Bus queue. There is no ordering across panels, across bus topics, or
  across sessions, and no global total order.
- A snapshot carries a barrier for the classes it includes. Events at or below
  the barrier are already represented; events above it are applied in order. A
  lossy class may recover from its latest state without a full session snapshot.
- Duplicate event identities are idempotent. A non-lossy gap, a conflicting
  duplicate, or an out-of-order control event beyond the accepted window
  triggers resynchronization rather than silent reordering.
- A reconnect never replays an effect. It replays observations, re-presents
  pending approvals as pending, and re-reads authoritative state.

### Backpressure classes

| Class                   | Examples                                                                                                                     | Delivery rule                                                             | Overflow behavior                                                                                 |
| ----------------------- | ---------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| Control and authority   | Lifecycle, stop, approval request, approval verdict, tool result, final turn message, speech job lifecycle/terminal, errors. | Non-lossy, bounded, ordered per session generation.                       | Refuse the newest request and report backpressure; never silently drop.                           |
| Recoverable observation | Turn deltas, progress, context updates, routing fallback notices.                                                            | Bounded and sequence-numbered; cumulative resynchronization is available. | Coalesce or drop oldest with counted loss, then resynchronize from a snapshot or cumulative turn. |
| Ephemeral presentation  | Avatar state, animation steps, viseme markers, cursor and focus hints.                                                       | Latest-wins or short bounded queue; no user-visible data depends on it.   | Drop stale entries with counters; never queue without bound.                                      |
| Audio                   | Speech chunks and playback buffers.                                                                                          | Bounded ring buffer with a real-time budget.                              | Stop or restart playback at a safe boundary; never play out of order or unbounded.                |

The control row describes the **session control stream**, not either bus. The
Panel Event Bus carries observation and lifecycle classes with `DropOldest` for
its queues, and the SMO/IPC event path carries typed observation events with
attributed drop-oldest side queues; the control class's non-lossy rule is neither
queue policy.

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

The Headed Agent surface asks for a model profile or role together with
constraints. It does not select a provider, name an endpoint, supply a
credential, read a provider registry, or construct an alias route.

An illustrative request carries:

- the model profile or role;
- required capabilities, such as text, streaming, tool use, image input, audio
  input, or audio output;
- the minimum acceptable privacy class;
- the token, cost, latency, and locality constraints;
- the budget reference and the caller's **believed** routing snapshot version
  and digest, carried as an assertion only.

A caller-supplied version or digest is an assertion, never authority. `bitty-ai`
validates it against the Core-published projection and the AI helper's ownership
proof before strict use; a mismatch, an unknown version, or a proof that cannot be
verified fails the turn closed rather than resolving against the caller's value.
The accepted binding's publication authority, not the request, decides which
projection is current.

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
  alias projection decision, and this draft defines no alias routing. A strict
  request carries a model profile, never an alias; an alias identifier is
  rejected before routing and is never interpreted as a profile. A client that
  has only an alias fails closed until an accepted alias projection exists; it
  never falls back to the provider list or to a local alias table. Any
  registry, capability, context-window, cost, or alias-table mutation
  invalidates the proof before strict use, and an alias request is never
  counted as strict provider-list coverage.
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
- A routing snapshot version and digest are validated for **one turn**; they are
  not a durable session pin. Any provider, model, capability, context-window,
  cost, or alias mutation invalidates the proof before strict use, and the turn
  fails closed rather than continuing on the previous projection. A later turn
  binds a new version only after a fresh publication and proof. A refresh is
  explicit, attributed, and never triggered by a failed request as an
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
tool.started               Core authorizes; bitty-execution starts the execution
tool.result | tool.failed  bitty-execution reports the outcome; presentation shows it as untrusted observation
```

These labels are illustrative event-family names, not wire methods.

Rules:

- A proposal is not an execution and carries no authority. The model cannot
  approve its own proposal, and the Headed Agent surface cannot approve on the
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
  requires a new approval. Whether a standing or session-wide approval is
  allowed at all is open ([Approval lifetime and standing grants](#open-points));
  if a future accepted contract allows one, it is a separate explicit grant,
  never an implicit default, and never a credential-use grant.
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
- The approval ledger and the verdict are Core-owned. The execution record's
  durable placement is part of the draft R1 disposition, and the structured
  execution outcome is owned by `bitty-execution` under `W-132`; neither is
  assigned to a panel or to `bitty-ai` by this draft. The Headed Agent surface
  owns only the presentation of the request and the collection of the user's
  explicit decision.

## Panel and Event Bus ownership

### Topic families

Topic identifiers follow the accepted Panel Runtime RFC grammar
`owner.name:topic`, matching
`^[a-z][a-z0-9_-]*\.[a-z][a-z0-9_-]*:[a-z][a-z0-9_.-]*$` and bounded to 64
bytes. The rows below are illustrative bus topics in that grammar, not accepted
wire strings, subscription methods, or plugin API names. `panel.*` in this
document is the accepted capability family (`panel.provider`,
`panel.create`, `panel.focus`, `panel.overlay`), not a topic namespace;
Core-owned panel topics use qualified names such as `bitty.panel:mounted`.

The **Panel Event Bus** does not carry approval, stop, tool-result, speech-job,
or final-turn control events; those travel on the session control stream (below)
and are not governed by `DropOldest`. The bus itself is broader than observation:
it also carries non-coalescable lifecycle topics such as `panel.created`,
`panel.closed`, and `bell`, and high-risk topics that inherit their capability's
consent surface. Calling it "observation only" would be inaccurate; calling it a
control transport would be wrong for this draft. The `ai.*` prefix in the table
is an illustrative owner prefix for a topic, not a capability identifier and not
a new catalog. `ai.*`, `agent.*`, and `mcp.*` capability identifiers are
**owner-pending**: the `bitty-ai` extension catalog exists only as a draft, and
`bitty-plugins` has not accepted extension-contributed manifest grammar, so only
`panel.*` is an accepted capability family today.

| Illustrative bus topic         | Producer           | Consumer                   | Accountable owner  | Delivery class          |
| ------------------------------ | ------------------ | -------------------------- | ------------------ | ----------------------- |
| `bitty.panel:mounted`          | Core               | Attachments, host services | Core owner         | Observation, DropOldest |
| `ai.turn:delta`                | AI owner           | Attachments                | AI owner           | Lossy observation       |
| `ai.context:updated`           | AI owner           | Attachments                | AI owner           | Lossy observation       |
| `ai.avatar:state`              | AI owner           | Headed Agent owner         | AI owner           | Ephemeral presentation  |
| `example.headed-agent:private` | Headed Agent owner | Its own subscribers        | Headed Agent owner | Plugin-defined          |

Control families and their actual transport:

| Control family                                  | Transport                                                                                     | Producer                 | Rule                                                       |
| ----------------------------------------------- | --------------------------------------------------------------------------------------------- | ------------------------ | ---------------------------------------------------------- |
| Session lifecycle, `attach`, `detach`, `stop`   | Session control stream (accepted IPC request/response plus `chunk` framing where it suffices) | Session authority        | Non-lossy, ordered per session generation, gap-detected    |
| Approval request and verdict, tool result/error | Session control stream                                                                        | Core / `bitty-execution` | Non-lossy; a bus or durable-stream form needs an amendment |
| Final turn message                              | Session control stream                                                                        | AI owner                 | Non-lossy control event; never a lossy turn delta          |
| Speech job lifecycle and terminal status        | Session control stream                                                                        | `bitty-ai`               | Non-lossy, gap-detected                                    |
| Audio chunks                                    | Media channel                                                                                 | Media presentation       | Own chunk sequence and playback buffer                     |

A row's accountable owner is the repository accountable for the topic schema,
producer identity, and authorization; a consumer is never the owner of a topic
it only reads. Audio chunks are not an Event Bus topic: they travel on the media
channel with their own chunk sequence. The **exact v1 bus topic taxonomy**,
including whether AI and cross-window topics ride the Panel Event Bus or the IPC
path, remains `RFC-OQ-4`; the per-panel capability mapping remains `RFC-OQ-5`.
This draft reserves no namespace and freezes no taxonomy.

Topic rules:

- The accepted RFC enforces topic grammar at declaration and publish, rejects a
  bare name without an owner prefix with `UnknownTopic`, and fails a non-Core
  topic that impersonates `bitty.*`. The Core owner reserves `bitty.*` for Core
  producers. An `ai.*` reservation is a design target only: no accepted bus
  taxonomy exists yet, and `RFC-OQ-4` owns the v1 topic taxonomy, including
  whether AI topics ride the bus at all. The `panel.*` capability family grants
  `panel.provider`, `panel.create`, `panel.focus`, and `panel.overlay`; it is not
  a publishable topic namespace. A plugin publishes only inside its own
  owner-qualified namespace.
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
- Control events are not made reliable by relabeling them as bus topics. The
  non-lossy session control stream is the transport; a bus-based control queue
  would need its own accepted amendment and is not claimed here.
- Ordering is FIFO within one bus queue and, separately, per session generation
  on the control stream. There is no ordering across panels or bus topics and no
  global total order.
- Retention is per class. Control events retain enough to support the documented
  reconnect window on the session control stream; ephemeral presentation topics
  on the bus may retain only the latest state.
- A plugin-private topic is not a covert cross-plugin channel. It is still
  subject to payload bounds, audit, and the platform's plugin isolation rules.

### Core-versus-plugin ownership

The table records design-target ownership only; it claims no implementation.

| Concern                                    | Core / `bittyd` / `bitty-execution`                                                                                                                                   | `bitty-ai`                                                                                                           | Headed Agent surface                                               |
| ------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------ |
| Panel identity and lifecycle               | Core owner under the accepted Panel Runtime RFC: identity, create, mount, suspend, dispose; implementation not claimed.                                               | Consumes panel references only; no ownership.                                                                        | Requests a panel; never invents identity.                          |
| Layout, focus, and visibility              | Core owner of accepted host focus and visibility routing.                                                                                                             | No ownership.                                                                                                        | Owns its internal composition and widgets.                         |
| Execution                                  | Core authorizes and fences every operation and retains PTY, permission, and resource enforcement; `bitty-execution` (`W-132`) supervises and executes.                | Owns semantic task, binding, retry policy, and model-facing execution intent.                                        | Requests work through the session; never executes or approves.     |
| Rendering and animation                    | Core owner of presentation primitives under accepted rich-presentation contracts; implementation not claimed.                                                         | Owns the candidate semantic state and viseme event vocabulary.                                                       | Maps semantics to 2D/3D appearance.                                |
| Agent and turn state                       | Host-side transport; `bittyd` is the sole session-registry writer if `bitty#2546` places the record there; placement open.                                            | Owns model-facing turn and job state; consumes the RFC state names, not a shared crate vocabulary.                   | Projects the sanitized view.                                       |
| Accessibility                              | Core retains the baseline, projection mechanism, focus-order rule, announcements, reduced motion, and contrast; `bitty-a11y` owns platform AT adapters under `W-134`. | Supplies semantic state for its own content; no AT authority.                                                        | Conforms to the baseline; owns no AT backend or terminal mutation. |
| Topic schema                               | Core owner of accepted reserved namespaces (`bitty.*`) and transport; `ai.*` is a design-target namespace under `RFC-OQ-4`, not an accepted reservation.              | Owns AI-domain observation topic schemas it produces.                                                                | Owns only its private observation topics.                          |
| Consent and approval                       | Core ledger, prompt, and verdict owner; no implementation claimed.                                                                                                    | Relays proposals and results; grants nothing.                                                                        | Presents the request; cannot grant.                                |
| Provider descriptor and privacy vocabulary | Consumes the vocabulary; does not define it.                                                                                                                          | Owns the provider descriptor, capability labels, and privacy classes as the `bitty-ai` Core surface.                 | Reads bounded results only; never selects a provider.              |
| Retention and deletion                     | Terminal Core owns host evidence, consent, and storage; R3 owns the context retention policy.                                                                         | Owns AI record semantics and the speech-job asset-reference instance.                                                | Its own records (current 0.1.0 role)                               |
| Plugin packaging                           | No ownership.                                                                                                                                                         | Product owner chooses whether to publish the Headed Agent package; does not define the platform category or signing. | Consumes the accepted plugin-platform contract and manifest rules. |
| Audio focus                                | Host device and window focus owner; one device plays one stream at a time.                                                                                            | Owns the speech job lifecycle.                                                                                       | Requests playback; does not steal focus.                           |

A panel is a presentation surface, not a session, execution, or consent record;
ADR-0013 keeps panel provenance non-authoritative and separates `PanelId` from
`ExecutionContextId` and `AgentId`. The draft
[execution-projection binding](../specifications/execution-projection-binding-candidate.md#the-binding-relation)
is a candidate design reference for panel-to-execution projection; it is not
accepted and governs nothing yet. A session attachment is a different relation:
it binds a presentation client to a session's observation stream and does not
replace or duplicate an execution binding.

The accepted OQ-058 ruling's SMO-1 through SMO-4 mandates are consumed by
reference, including its delivery semantics: bounded typed events with envelope
identity, sender/recipient attribution, deadline, priority, deduplication,
cancellation, and expiry; presentation is not authority; routing fails closed;
and the substrate is the existing IPC, attribution, and Panel Runtime identity
rather than a second registry or daemon. Two different mechanisms are still kept
apart by name. The SMO direction rides the **accepted IPC transport**. The
**Panel Event Bus** is the accepted in-process, single-window observation and
lifecycle bus; it is not the IPC transport and carries no cross-window topic
without the `RFC-OQ-4` transport contract. This draft adds no competing
orchestration topology and claims neither bus is implemented.

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
- An attachment is bound to the `(PanelId, generation)` that mounted it. A panel
  dispose, rebuild, or generation bump closes or fails that attachment before
  any further event is delivered; a stale attachment cannot keep receiving
  session events into a destroyed panel surface.
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

- One session may have many attachments across panels **within one window**. The
  accepted Panel Event Bus is single-process and single-window, so an attachment
  in another window needs the explicit cross-window transport contract tracked as
  `RFC-OQ-4`; this draft does not assume such a path exists.
- One panel hosts at most one session attachment: the accepted Panel Runtime
  contract has at most one panel host per panel, and a mounted panel binds
  `PanelId` to at most one `ViewId`. Attachments are session subscriptions, not
  extra panel mounts, so "several attachments in one panel" is not a supported
  shape. One surface has one primary attachment at a time. The existing
  execution-binding cardinality still applies to execution projections, and it is
  a draft candidate rather than an accepted rule.
- One audio device plays at most one stream at a time; that per-device
  exclusivity is a physical invariant. Whether playback focus is scoped per
  panel, per session, or globally is an open policy (see
  [Open points](#open-points)); this draft does not fix that scope. Focus
  changes are user or host decisions; an agent may request attention, never
  preempt focus or start playback.
- Panel closure removes that attachment. The session may continue headless;
  stopping it requires an explicit lifecycle action or a reviewed idle policy.
- A panel that becomes hidden keeps its attachment and may keep receiving, as the
  accepted suspend rule makes a panel invisible without destroying its
  attachment; a host may separately suspend presentation or detach, and those are
  distinct operations. Invisibility never broadens context access, manufactures
  consent, or moves the execution target.

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
`AssetRef` is consumed here as a **proposed** shared Core artifact-reference
contract, not an accepted one: its only source is the draft multimodal boundary,
no accepted owner has published it, and ADR-0016 Boundary 1 assigns output
retention and artifact-reference mechanics to `bitty-execution`/`W-132`.
`bitty-ai` owns only the speech-job asset-reference instance (job identity, media
type, expiry, and version). This draft consumes `AssetRef` by reference, defines
none of its fields, and does not claim it is accepted.

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
  -> presentation playback (Core extension component or post-1.0 helper)
  -> synchronized captions and inspectable transcript
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
- A non-streaming request returns an `AssetRef` and metadata. Audio bytes are
  not placed in context, events, or snapshots. The reference carries an expiry,
  a version, and its declared retention class; expanding an expired reference
  fails closed and is re-requested, never replayed. Job submission carries an
  idempotency key: a duplicate submission before expiry returns the original
  job, and after expiry it fails closed (the store-expiry facet recorded under
  AIQ-03).
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
  speech-job status, not an `AgentSession` state. At the session level the
  cancellation is recorded in whichever session vocabulary applies: the
  `bitty-ai` runtime records terminal `Canceled`, while the accepted RFC
  vocabulary records `Completed` or `Failed` with cancellation attribution (see
  [Outcome levels](#outcome-levels)). `Unknown` is reconciled by status
  inspection before any retry; a retry is a new submission, not a replay, and
  carries a new idempotency key.
- Cancellation never leaves a partial asset silently presented as complete.
  Partial audio is discarded or explicitly marked incomplete.
- Detaching the only playback attachment stops local playback but does not
  cancel the speech job unless the user or policy asks for cancellation. A
  detached job may complete to an asset reference that no panel is currently
  playing.

### Playback ownership

Audio device selection, decoding, buffering, volume, mute, and device-loss
recovery belong to a media presentation implementation. That implementation is
an **owner-pending category**: a trusted Core extension component following the
`bitty-graphics`/`bitty-a11y` component pattern (`bitty-component.toml`,
per-target digest, descriptor validation), which reaches GPU, window, and audio
resources only through Core-owned interfaces, or a post-1.0 declared native
helper, which receives no GPU, window, PTY, or host-Rust handle. The two forms
are not the same trust story, and neither is a v1 Lua plugin or package: no
native module loads into the host. (The `bitty-platform-services` repository is
an L1 extension crate for platform adapters under ADR-0016 Boundary 5 and `W-136`,
not a component-pattern example.) Core may provide a neutral presentation
primitive, but no Core audio implementation is claimed here. `bitty-ai` owns the
speech job and the semantic event; it does not open an audio device. The Headed
Agent surface owns when speech is offered, which messages are eligible, the user
controls, and the caption/transcript presentation. The playback implementation
receives bounded audio chunks only and is transcript-free: it neither reads nor
renders the transcript.

Playback is off by default. Autoplay may be enabled only by an explicit,
user-owned host policy that names the applicable session or attachment and
remains revocable; absent that policy, a fresh user gesture is required. A host
policy, plugin default, agent request, or model output cannot silently start
audio.

Whenever speech is presented, **synchronized captions and an inspectable
transcript are candidate v1 requirements**, and audio is never the only channel
for required information. They are **not** accepted baseline obligations: no
accepted source requires them, and the v1 commitment is product decision `D4`.
The rules below describe the candidate obligation a future accepted contract
would have to carry:

- Captions follow the audio clock within a declared timing tolerance and a
  bounded added latency. The tolerance is a named policy constant in the future
  accepted speech contract; this draft fixes only that one exists and that
  captions never run ahead of the audio.
- Captions and the transcript render the same post-validation, post-redaction
  text that was authorized for synthesis. They never reveal hidden context or
  sibling-panel content that the speech request excluded.
- The transcript is bounded, locale-tagged, scrollable or inspectable, and
  toggleable. Turning captions off does not remove transcript access.
- Both are exposed to assistive technology as text with a stable accessible
  name and state, and caption changes announce only at a sentence or declarative
  boundary, not per chunk.
- Retention follows the declared class: ephemeral by default, deleted with the
  source history under R3, and retained only under explicit, consented
  recording. The transcript's retention class is named independently of the
  audio `AssetRef`.
- Synthesized audio is disclosed as synthetic at the point of playback. Provenance
  metadata on an exported asset applies **only if** `D3` offers export at all and
  is then part of the `D2` minimum; this draft does not settle the form.
- A host without an audio device degrades to text and captions; it does not fail
  the session or drop the transcript.

## Avatar, animation, and viseme contract

### Presentation state, not identity

An avatar is a presentation of session state. It is never the session
identity, authorization signal, or source of truth. The semantic state
vocabulary below is a **new candidate vocabulary**; it is not an accepted
`bitty-ai` contract and creates no compatibility promise:

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
  Reduced motion is the accepted RFC-0002 / `W-134` obligation: it collapses
  animation to the final committed state and is **not plugin-overridable**.
  Motion is decoration; the final state is exposed immediately.
- Flashing and rapid motion respect the host flash threshold. No avatar or
  media surface may flash above that threshold, and no plugin setting may
  re-enable a disallowed flash rate. The threshold value and its owning contract
  are **unassigned**: no accepted source defines a numeric flash-rate limit, so
  the owning task must name the threshold, its measurement method, and its
  reviewer before implementation (tracked in
  [Remaining review follow-ups](#remaining-review-follow-ups)).
- Animation never blocks the event loop, the terminal hot path, or audio
  playback. Rendering work runs off the hot path under the presentation budget.
- A device loss, GPU context loss, or missing renderer falls back to the next
  lower presentation class. It does not crash the session or silently continue
  with a corrupted surface.

### Viseme events

- A viseme event maps a bounded time interval in the speech audio to a
  renderer-neutral mouth-shape identifier. The identifier vocabulary is a new
  candidate vocabulary owned by `bitty-ai`; it is not accepted. A renderer maps
  it to its own rig, sprites, or blendshapes.
- The audio clock is authoritative. Viseme timing is best-effort and may be
  corrected by the renderer within a declared tolerance.
- Viseme events are ephemeral presentation data. They may be coalesced or
  dropped under backpressure; audio continues and captions remain accurate.
- A viseme timeline may come from provider metadata or a local analysis step.
  Local analysis is an optional media-presentation capability and is not
  defined here.
- This contract makes no lip-sync accuracy claim. Accuracy evidence belongs to
  the renderer or media presentation that implements it.

### 2D and 3D ownership

The same semantic state, animation, and viseme events drive both 2D and 3D
presentations. Two different 2D paths must not be conflated:

- **Declarative 2D composition** uses Core presentation primitives. It owns no
  GPU surface, decoder, or audio device, and it is the only 2D form available in
  the candidate v1 scope.
- **GPU-backed 2D or 3D rendering** is the owner-pending media presentation: a
  sprite batch, layered compositor, rig, blendshapes, shader, or scene graph
  belongs there, not to Core primitives and not to `bitty-ai`.

Neither implementation lives in `bitty-ai`, Core, or Wheel as application code.
The Headed Agent surface owns appearance, character composition, and the mapping
from semantic events to its chosen presentation. A GPU-backed 2D or 3D renderer
is an optional media presentation in the owner-pending category: a trusted Core
extension component with its own capability declaration, resource budget, and
security review, or a post-1.0 native helper that receives no GPU, window, PTY,
or host-Rust handle. No 3D code, GPU surface, shader pipeline, or avatar asset
pipeline is authorized by this document, and a v1 Lua plugin is excluded.

### Accessibility and privacy

The full headed accessibility baseline is defined in
[Accessibility baseline](#accessibility-baseline). For the avatar specifically:

- **Alternative content and state equivalence.** An informative avatar exposes
  an accessible name and description plus a text state equivalent for
  `idle`, `thinking`, `speaking`, `waiting`, `error`, `suspended`, and
  `detached`. A decorative avatar is hidden from assistive technology and adds
  no announcement. No session state, approval need, or error is conveyed by
  animation alone; the same information is always available as text.
- **State announcements.** A state transition that changes what the user must
  know announces once through the bounded announcement path; animation frames
  and viseme changes never announce.
- The avatar does not read the transcript, capture camera or microphone input,
  perform face or voice recognition, or persist biometric data.
- Viseme timelines and avatar state are derived from speech text. They inherit
  the speech privacy class, are not egressed to a model or provider by default,
  and are excluded from logs and traces unless a recording consent names them.
- Audio buffers, decoded PCM, and viseme payloads are cleared before release on
  cancel, detach, suspend, takeover, or session end.
- Avatar assets are untrusted input. Decoding, texture upload, and shader
  composition follow the media presentation's bounded, isolated implementation
  and the `P0-AC-005` deny-by-default loader policy.
- GPU memory and textures are released on detach, suspend, or session end and
  are not reused across principals without clearing.

## Accessibility baseline

### Consumed accepted baseline

Accessibility is a Core-retained normative obligation, not an optional plugin
effect. The headed surface consumes the accepted baseline unchanged and does not
reopen it. `OQ-078` remains open on the accessibility API/tree shape and its exact
v1 versus post-v1 scope, and it is not the only open accessibility question:
`OQ-037` owns the minimum content-contrast contract, and the flash threshold has
no accepted owner at all. Neither open item reopens the obligations below.

| Source                                               | Obligation consumed here                                                                                                                                                                                                                      |
| ---------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ADR-0016 Boundary 3 (accepted 2026-10-02)            | Core retains the accessibility baseline, the projection mechanism, focus/terminal-state association, the focus-order rule, bounded announcements, reduced-motion and contrast respect, and the privacy rule that the projection is host-side. |
| `W-134` accessibility extraction contract (accepted) | The derived semantic snapshot, stable element identity and generation, controlled action interface, AT-SPI/UIA/AX adapter responsibilities, absent/denied-adapter behavior, and the rule that an adapter is not trusted by location.          |
| RFC-0001 (accepted)                                  | The minimum-contrast **resolution** rule for the appearance token pair: a high-contrast posture selects a compliant pair and is not plugin-overridable. The minimum content-contrast floor for text and rich content is `OQ-037`, still open. |
| RFC-0002 (accepted)                                  | `appearance.animations.reduced_motion` (`auto` default, `always`, `never`), instant final-state rendering under `always`, reduced motion, or `--safe`, and the rule that motion never blocks input.                                           |
| OQ-078 (open)                                        | The remaining API/tree choice and surface scope. No accepted obligation above is downgraded while it is open.                                                                                                                                 |
| P0 acceptance criteria                               | `P0-AC-005` local-resource loading, `P0-AC-009` URI/launch policy, `P0-AC-010` constrained scene, and `P0-AC-015` hot-path exclusion.                                                                                                         |

### Screen-reader semantics, names, and states

Every presented element that a user must understand or operate exposes a role,
an accessible name, and its current state. The projection is derived from the
surface's own truth, is read-only, is never persisted, and is never consulted for
routing or dispatch.

| Surface element           | Required exposure                                                                                                                                                                                              |
| ------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Session panel / root      | Follows `W-134`: the accessibility root is role `group` with the name from the window or icon title. A headed region may map to `group` with an additional named subgroup, never to a guessed `region`.        |
| Terminal text             | Role `text` with an accessible name; row content is text content, not an accessible name, per `W-134`.                                                                                                         |
| Streaming turn text       | Role `text` with an accessible name; live updates announce once per state transition, never per token and never on a repeating cadence.                                                                        |
| Tool proposal / approval  | Role `button` or `dialog` per presentation, name naming the tool and target, state `pending`/`approved`/`denied`, description carrying the bound digest summary.                                               |
| Status and error          | Role `status` or `alert`; typed failure announced once per transition, not on render.                                                                                                                          |
| Code, diff, table, list   | Role from the declared scene kind (`code`, `group`, `table`, `list`, `separator`), never a generic text role that flattens structure.                                                                          |
| Image or scene content    | Role `image` for a declared image kind; a scene kind with no mapped role is non-conformant and fails the whole snapshot closed.                                                                                |
| Speech controls           | Role `button`/`slider` with name, state (`playing`, `paused`, `stopped`), and value for volume/rate.                                                                                                           |
| Captions / transcript     | Role `text` region with a stable name; caption changes exposed as text; transcript inspectable and scrollable. Caption size, position, contrast, and non-speech cues are candidate v1 requirements under `D4`. |
| Avatar                    | Decorative: hidden. Informative: role `img` or `status` with name, description, and the text state equivalent.                                                                                                 |
| Modal approval or consent | Role `dialog`, modal state, labelled title and description, focus confined to the dialog.                                                                                                                      |

Rules:

- A state is never conveyed by color, animation, avatar posture, or audio alone.
  Every such state has a text or accessible-name equivalent.
- The accessible name is stable across projection generations for the same
  element identity; a renamed element is a new identity, not a silent rename.
- An unmapped element kind or scene kind fails the **whole snapshot** closed
  before anything is exposed; no partial tree is published and no node is
  silently omitted (`W-134`).

### Focus order, announcements, and modal confinement

- **Focus order.** Assistive focus follows the visible reading and interaction
  order: session root, turn text, pending approval or status, speech controls,
  captions/transcript, then avatar only when it is interactive. Chrome is
  excluded from the tab order by default under `W-134`. Focus moves between
  panels, views, or overlays report the destination that actually holds focus.
- **Announcements.** One coalesced, rate-limited announcement per focus or async
  state transition, never on render and never on a repeating cadence. A burst of
  streaming updates produces one announcement at the transition boundary, not a
  timed cadence. An approval request and an error announce once and remain
  inspectable.
- **Modal confinement.** A modal approval or consent dialog confines focus to
  the dialog, excludes the underlying session content while modal, and restores
  focus to the invoking control on close or cancellation. Escape or the
  documented cancel path closes it without approving. A modal never traps focus
  after the dialog is gone.
- **Focus restoration.** Detach, suspend, panel rebuild, or failed follow-up
  restores focus to a defined safe target (the session root or the last valid
  control), never to a destroyed element or a stale projection generation.

### Keyboard equivalence

Every function has a keyboard path with no pointer-only action. The table is the
minimum v1 equivalence set; the exact bindings are product/UI decisions, not
contract spellings.

| Function                      | Keyboard-equivalent requirement                                                                   |
| ----------------------------- | ------------------------------------------------------------------------------------------------- |
| Read streaming output         | Focus the stream, move by line/paragraph, reach the latest output without losing position.        |
| Approve, deny, or defer       | Reach the pending approval, read the bound prompt, choose each verdict, confirm, cancel.          |
| Stop or cancel                | A documented stop path that works from the session and from the active control.                   |
| Attach, detach, reconnect     | Reachable from the session surface; reconnect reports the outcome and does not auto-approve.      |
| Speech control                | Play, pause, stop, mute, volume, rate, and caption/transcript toggles, each with a visible state. |
| Caption and transcript access | Toggle captions, open and scroll the transcript, return focus to the prior control.               |
| Avatar state                  | Read the text state equivalent; no action requires interpreting the avatar.                       |
| Modal dismissal               | Cancel/close without granting, with focus restored.                                               |

### Motion, contrast, and flashing

- **Reduced motion.** `reduced_motion` collapses animation to the final
  committed state. The default `auto` follows the platform signal; `always`
  forces instant transitions; `never` keeps bounded animation when the host
  allows it; `--safe` behaves as reduced motion regardless of the configured
  value. `never` is a user-visible choice, not a plugin escape from bounds, and
  no value bypasses the flash threshold. This setting is Core-owned and **cannot
  be overridden by a plugin, theme, avatar, or media presentation**.
- **Contrast.** The high-contrast posture resolves the RFC-0001 appearance token
  pair and fails closed on a violating configuration; a plugin cannot lower that
  resolution. The **minimum content-contrast floor** for text and rich content is
  owned by `OQ-037` and the theme contract, which remain open. This draft asserts
  no numeric floor and defers the acceptance criterion to that owner; until it
  closes, a headed surface must not claim a tested contrast floor.
- **Flashing.** Flashing and rapid motion respect the host flash threshold; no
  surface exceeds it, and no plugin setting may re-enable a disallowed rate. The
  threshold value, measurement method, and owning contract are **unassigned** and
  tracked in [Remaining review follow-ups](#remaining-review-follow-ups).
  Animation never delays exposure of the final state.
- **Safe mode.** `bitty --safe` starts with zero AT adapters and no animation;
  the Core baseline, text, and captions still function.

### Captions and transcript

Synchronized captions plus an inspectable transcript are a **candidate v1
obligation** whenever speech is presented, not an accepted baseline obligation
and not an optional alternative. The accepted baseline (`ADR-0016`, `W-134`,
RFC-0001, RFC-0002) does not require captions or a transcript; the commitment is
product decision `D4` and the candidate obligation is recorded here so a future
accepted speech contract can carry it.

| Property    | Requirement                                                                                                                       |
| ----------- | --------------------------------------------------------------------------------------------------------------------------------- |
| Text source | The same post-validation, post-redaction text authorized for synthesis; no hidden or sibling-panel content.                       |
| Timing      | Captions follow the audio clock within a declared tolerance and bounded added latency; captions never lead the audio.             |
| Locale      | The transcript records the synthesis locale; a locale change starts a new transcript segment rather than relabeling old text.     |
| Bounds      | Caption lines and transcript length are bounded by named policy constants; overflow is visible and never silently truncated.      |
| Redaction   | Same redaction policy as speech text; presence markers for removed secrets; no raw environment or credential values.              |
| Retention   | Ephemeral by default; deleted with the source history under R3; retained only under explicit, consented recording.                |
| AT exposure | Captions and transcript are text regions with stable names and states; changes announce at sentence/declarative boundaries.       |
| Toggle      | Captions can be turned off without removing transcript access; both can be reached by keyboard.                                   |
| Ownership   | The Headed Agent surface presents them; the playback implementation is transcript-free and never renders or reads the transcript. |

### Avatar alternative content and state equivalence

An informative avatar has an accessible name and description and a text state
equivalent for every semantic state. A decorative avatar is hidden from
assistive technology and adds no announcement. No approval need, error, busy
state, or waiting state is conveyed only by animation or posture. The avatar
state is derived from session events and never from model-authored text.

### Accessible media controls

Any speech or avatar playback control exposes a label, role, state, and value,
is keyboard reachable, and reports changes to assistive technology:

- Play, pause, stop, replay-at-safe-boundary, mute, volume, and rate.
- Caption toggle and transcript open/close, with state and focus restoration.
- A stop that takes effect at the next safe boundary and never leaves partial
  audio presented as complete.
- A visible and announced degraded state when the device is lost or playback
  falls back to text.

### Platform AT ownership and absent-adapter behavior

| Platform | Adapter owner and mechanism                                                                                                     | Absent, denied, or crashed adapter                                                                                                                  |
| -------- | ------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| Linux    | `bitty-a11y` backend over AT-SPI/D-Bus, capability-gated by Core.                                                               | Removes platform exposure only; the Core baseline, text, keyboard operation, and captions continue; safe mode has zero adapters.                    |
| Windows  | `bitty-a11y` backend over Windows UI Automation, capability-gated by Core.                                                      | Same: no exposure, no terminal degradation, no security downgrade, and the missing adapter is reported honestly.                                    |
| macOS    | `bitty-a11y` backend over the macOS Accessibility (AX) bridge, capability-gated by Core.                                        | Same.                                                                                                                                               |
| FreeBSD  | **No AT backend is claimed.** ADR-0002 keeps FreeBSD at best-effort, and this draft asserts no adapter or platform model there. | AT is unavailable by statement, not silently absent: the surface reports no AT support, and text, keyboard operation, and the Core baseline remain. |
| Other    | A platform model admitted by a future focused contract; no generic fallback is claimed.                                         | Text and keyboard paths remain; the surface does not claim AT support it does not have.                                                             |

- Core owns the baseline and the projection; `bitty-a11y` owns the platform
  adapters and their permission state. The Headed Agent surface owns conformance
  for its own content and owns no AT backend.
- The platform's own accessibility permission is surfaced and consent-gated. A
  denied permission is reported, not bypassed.
- An absent backend is never described as working AT support. No AT backend
  exists today beyond a headless test backend; the honest status is "not
  implemented".

### AT-action authority

The presentation-never-authority rule extends to assistive technology:

- An AT adapter is **not a principal**. It has no `(UID, AgentId)` authority of
  its own and is not trusted by location or by the fact that it is first-party.
- An AT action is a typed, bounded request through the **Core controlled action
  interface**, resolving through the accepted command registry under the target
  owner's capability grants. The host revalidates the element identity and
  projection generation before dispatch; an unknown, expired, or stale element
  fails closed.
- An AT action cannot write the PTY, mutate grid/cursor/modes/scrollback, move
  focus outside the accepted routing path, or call an internal renderer or
  compositor API.
- An AT adapter cannot grant or approve consent, a tool call, a speech egress,
  or a panel lease **on its own**. AT activation of a decision control is
  **non-approving**: it may reveal, focus, announce, scroll, or select a
  proposed verdict, but it cannot record the verdict. The verdict is recorded
  only by a Core-observed keyboard or pointer activation of the focused bound
  decision control through Core's trusted input path. The adapter cannot mint,
  forward, or assert that activation; adapter-supplied activation flags,
  confirmation strings, and action provenance are ignored. The dialog announces
  the keyboard confirmation path. If the platform cannot deliver a
  Core-observable activation, AT remains non-approving and the keyboard path is
  the documented fallback. Merely exposing, focusing, or programmatically
  invoking the control is not user input and cannot approve.
- The projection, element identities, focus associations, and action outcomes
  are host-side and are **never published on the Panel Event Bus**. Exposing one
  surface to assistive technology must not become a cross-panel read path for
  plugins.
- A compromised adapter gains only the bounded action interface; it cannot mint
  a generation, impersonate a principal, publish a reserved topic, or record an
  approval verdict.

### Accessibility verification and acceptance criteria

| Criterion                   | Acceptance evidence                                                                                                                                                                                                                                                                                                                                             |
| --------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Complete semantics map      | Every presented element kind and scene kind has a role, name, and state mapping; an unmapped kind fails the whole snapshot closed with no partial exposure.                                                                                                                                                                                                     |
| Focus order and restoration | Keyboard/AT traversal matches the defined order; chrome is excluded; modal confinement and focus restoration pass.                                                                                                                                                                                                                                              |
| Announcement discipline     | Exactly one coalesced announcement per focus or async state transition; no render, per-token, per-frame, per-viseme, or timed-cadence announcement; duplicate coalescing passes.                                                                                                                                                                                |
| Keyboard equivalence        | The full function table is reachable and operable without a pointer.                                                                                                                                                                                                                                                                                            |
| Reduced motion              | `always` and `--safe` render the final state instantly; `never` keeps only bounded animation; a plugin override attempt fails.                                                                                                                                                                                                                                  |
| Contrast                    | High contrast resolves a compliant appearance pair or fails closed; no numeric content floor is claimed until `OQ-037` closes.                                                                                                                                                                                                                                  |
| Flash threshold             | A flash-rate corpus confirms no surface exceeds the threshold once an owning contract names it; no plugin setting re-enables a rate.                                                                                                                                                                                                                            |
| Captions and transcript     | Candidate v1 only: timing tolerance, locale tagging, bounds, redaction, retention, toggle, and AT exposure pass on every speech path once `D4` accepts the obligation.                                                                                                                                                                                          |
| Avatar equivalence          | Decorative avatar is hidden; informative avatar exposes name, description, and every state as text.                                                                                                                                                                                                                                                             |
| Media controls              | Labels, roles, states, values, keyboard reachability, and focus restoration pass for every control.                                                                                                                                                                                                                                                             |
| Adapter absence             | Absent, denied, and crashed adapters on Linux, Windows, and macOS leave the baseline and keyboard paths intact and report honestly; FreeBSD reports no AT support by statement.                                                                                                                                                                                 |
| AT-action non-authority     | AT activation cannot approve, grant, mutate terminal state, or publish on the bus; an adapter cannot forge or forward the user activation. Only a Core-observed keyboard or pointer activation of the focused bound decision control through the trusted input path records the verdict, once, and a stale element or adapter-supplied activation fails closed. |
| No single-channel state     | No required state is conveyed only by color, animation, audio, or avatar posture.                                                                                                                                                                                                                                                                               |

These criteria are future implementation evidence. This document records them
as acceptance conditions; it does not claim any of them has been run.

## Resource budgets

### Budget envelope

`start` accepts a budget envelope. The envelope is owned by user or host
policy, requested by the plugin, and enforced at the owning layer. A plugin
cannot raise its own envelope, and a project or model cannot widen it.

| Dimension       | Examples                                                                 | Accountable enforcement owner(s)                                                                 | Exhaustion action                                    |
| --------------- | ------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------ | ---------------------------------------------------- |
| Tokens and cost | Context tokens, output tokens, model cost, speech characters or seconds. | Core owner (shared client quota and ledger) + AI owner (pre-I/O gates)                           | Truncate under CP-5, refuse a new job, or stop.      |
| CPU             | Turn processing, rendering, animation, decoding threads.                 | Core owner (host); Headed Agent owner (render); media presentation owner (decode, owner-pending) | Coalesce, lower quality, or suspend presentation.    |
| Memory          | Event queues, history, snapshots, decoded audio, textures, meshes.       | Owner of the allocation: Core, AI, Headed Agent, or media presentation                           | Evict oldest, drop ephemeral state, or stop the job. |
| GPU             | Texture memory, render targets, mesh and animation buffers.              | Media presentation owner (owner-pending)                                                         | Release surfaces, fall back to 2D, or text-only.     |
| Audio           | Buffer duration, decoded bytes, concurrent playback, device handles.     | Media presentation owner (owner-pending)                                                         | Stop or restart at a safe boundary; degrade to text. |
| Network         | Provider calls, retries, concurrent streams, media downloads.            | Core owner + AI owner                                                                            | Refuse, back off, or fail closed.                    |
| Wall clock      | Turn duration, speech duration, animation duration, reconnect window.    | Core owner + AI owner + Headed Agent owner                                                       | Typed timeout or terminal state; no unbounded wait.  |
| Storage         | Snapshots, cached audio, logs, avatar assets.                            | Core owner (host storage and consent) + AI owner (record semantics); R3 owns retention policy    | Ephemeral default; consent-gated retention only.     |

### Enforcement rules

- There is **no separate AI budget**. Model and speech I/O consume the shared
  per-client quota and appear in the Core ledger; the AI helper applies
  pre-I/O gates before any provider call but does not mint a parallel budget or
  a private quota. Speech character and duration limits are the same envelope's
  token and cost dimensions, attributed to the session and principal.
- Every stored artifact names a retention class. The draft R3 contract records
  its own candidate classes, and this draft does not mint a competing set: where
  the two agree, `ephemeral` is the default and dies with the attachment or job,
  a session-bounded class ends with the session, and an explicit class requires a
  consented recording grant and R3 deletion propagation. These are **candidate
  additions pending R3 alignment**, not an accepted retention rule, and an
  `AssetRef` without a declared class is non-conformant.

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

| Presentation class | Capability                                           | Required behavior                                                                                                                                                                                        |
| ------------------ | ---------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `P-A`              | Audio playback plus GPU-accelerated 2D or 3D avatar. | Full presentation within the declared budgets.                                                                                                                                                           |
| `P-B`              | Audio playback plus 2D or software-rendered avatar.  | Reduced animation and resolution; no 3D requirement.                                                                                                                                                     |
| `P-C`              | Text and captions only; no audio device or no GPU.   | Full session function; speech may produce an `AssetRef` for later playback only when its declared retention class permits and retention was explicitly consented; otherwise the surface stays text-only. |
| `P-D`              | Headless or no presentation attachment.              | Session and tools continue; no rendering, audio, or avatar resources.                                                                                                                                    |

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

| Platform | Expected baseline                                                                                                                                   | Audio                                                       | GPU / avatar                                       | Notes                                                                            |
| -------- | --------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------- | -------------------------------------------------- | -------------------------------------------------------------------------------- |
| Linux    | ADR-0002 `Tier 1` on x86_64; presentation class `P-A` or `P-B` where devices exist.                                                                 | Native audio stack through Core or a media presentation.    | Native GPU surface through the media presentation. | Device loss and compositor restart must degrade without killing the session.     |
| macOS    | ADR-0002 `Tier 1` on ARM64, `Tier 2` on x86_64; presentation class `P-A` or `P-B` where devices exist.                                              | Native platform audio through Core or a media presentation. | Native GPU surface through the media presentation. | App suspension and audio device changes must be handled explicitly.              |
| Windows  | ADR-0002 `Tier 1` on x86_64 with ConPTY; presentation class `P-A` or `P-B` where devices exist.                                                     | Native platform audio through Core or a media presentation. | Native GPU surface through the media presentation. | Audio session and device-change behavior require platform evidence.              |
| FreeBSD  | ADR-0002 `Tier 2` best-effort on x86_64; presentation class `P-C` or `P-D` by default, `P-B` only with detected audio and software or 2D rendering. | Only when a supported audio path is detected.               | Not assumed; software or 2D fallback.              | Missing audio or GPU must fall back to text or headless, never fail the session. |

Presentation class and assistive-technology availability are separate axes. The
AT-SPI/D-Bus, Windows UI Automation, and macOS AX adapters are owned by
`bitty-a11y` under `W-134`; their absence or denial changes no presentation
class, no support tier, and none of the Core baseline.

Platform rules:

- No operating-system API, device path, driver, or GPU backend is named or
  embedded by `bitty-ai`. The Core owner owns host integration where a host
  adapter exists. Device-specific audio, GPU, and decoder adapters belong to the
  owner-pending media presentation, which composes with two accepted ADR-0016
  boundaries: Boundary 2 moves image decode and processing to
  `bitty-graphics` under `W-133` while Core retains bounded intake, placement and
  resource policy, the aggregate image budget, and pre-upload validation;
  Boundary 5 moves notification, URL open, and compositor blur behind the
  platform-service adapter under `W-136` while Core retains each permission gate.
  `bitty-a11y` owns AT adapters under `W-134`. None of those contracts is claimed
  implemented here.
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

| Category                | Examples                                                        | Default     | Revocation behavior                                                            |
| ----------------------- | --------------------------------------------------------------- | ----------- | ------------------------------------------------------------------------------ |
| Model egress            | Sending context to a remote provider.                           | Denied      | Cancel in-flight streams at the next chunk boundary; no new calls.             |
| Provider credential use | Resolving a stored credential for a provider and privacy class. | Denied      | Deny new provider calls; an in-flight stream stops at the next chunk boundary. |
| Tool execution          | Running a proposed tool with a specific target and scope.       | Denied      | Deny pending calls; do not roll back completed effects; reconcile Unknown.     |
| Speech egress           | Sending text to a remote speech provider.                       | Denied      | Stop streaming, discard queued audio, reconcile the job.                       |
| Playback and autoplay   | Producing sound without a fresh user action.                    | Off         | Stop playback and release the device.                                          |
| Recording and retention | Writing audio, visemes, snapshots, or traces to disk.           | Off         | Stop new writes; apply deletion under R3.                                      |
| Avatar media            | Camera, microphone, biometric, or voice-profile use.            | Not granted | Out of scope; requires a separate accepted contract.                           |

Consent rules:

- Consent is granted per authenticated principal `(UID, AgentId)` and per scope,
  with a time bound; it is visible, revocable, and, for provider egress, further
  qualified per provider and privacy class. The accepted IPC ledger binds the
  pair for the instance, and revocation is recorded and survives reconnect. A
  session generation narrows the elevation under that grant; it does not replace
  or renew the grant. A plugin install, a model choice, a project file, a panel
  attachment, or an avatar state never implies consent.
- A provider that resolves to a stored credential requires the matching
  `ai.provider` decision for that principal, provider, and privacy class. Neither
  a tool grant nor a session-wide approval pre-authorizes credential use, and the
  decision is re-checked when the provider or privacy class changes.
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

The **consent surface** — prompt placement, prompt content, grant review,
revocation UI, and how clearly speech egress is distinguished from model egress
— is product decision `D6`. This draft fixes the authorization and revocation
semantics above; it does not claim a prompt design, and consent fatigue is an
explicit input to that decision, not a reason to weaken the floor.

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
- The contract floor is fixed: recording is off by default, any recording is
  consented and redacted, and deletion propagates. Whether the **v1 product**
  exposes record/export controls, and which retention class it offers, is
  product decision `D3`; the two are not in tension. An export path, if one is
  offered, must show a redacted preview before writing and must carry the
  retention class with the artifact.
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
  presentation fails closed to the next lower presentation class or text-only.
  It does not decode untrusted media in-process and does not claim containment.
  A v1 Lua plugin cannot host this decoder: native in-process plugins and native
  module loading are forbidden (`R-017`), and direct `wgpu` device, queue, or
  texture access is outside the plugin boundary.
- No shell command, file path, URL, or environment value from an untrusted
  media payload is executed or resolved by `bitty-ai`.
- A malicious or spoofed panel cannot publish into a reserved topic, forge a
  producer identity, or read another plugin's stream.
- Assistive-technology projection and actions are host-side and bounded. An AT
  adapter cannot publish the projection on the Panel Event Bus, cannot record a
  verdict or grant, and cannot mutate terminal state; its only write path is the
  Core controlled action interface described in
  [AT-action authority](#at-action-authority).

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
`bitty`/`bittyd`; **execution owner** is `bitty-execution` under the accepted
`W-132` direction; **accessibility adapter owner** is `bitty-a11y` under
`W-134`; **AI owner** is `bitty-ai`; **Wheel owner** is `wheel`; **Headed Agent
owner** is `bitty-ai` as product owner, consuming the `bitty-plugins-docs`
plugin-platform contract; **media presentation owner** is owner-pending; and
**Provider plugin owner** is the provider adapter repository. Each row is a
review item for the named owner.

Review roles are cross-cutting, not additional implementation repositories:
**Security owner** is the security reviewer role, **Product owner** is the
product/docs review role, **Accessibility owner** is the accessibility review
role, **terminal docs owner** is `bitty-terminal-docs`, and **Plugin ecosystem
owner** is `bitty-plugins-docs` plus the plugin repositories. **Architecture
owner** and **Documentation curator** are the existing architecture-review and
docs-curation roles. A review role can own a review row without owning
implementation code; the matrix columns still name the repository accountable
for the code.

Wheel 0.1.0 is a standalone harness with no host IPC client and no credential-
handle consumption (Wheel owner decisions D1, D2, D4, and D6; README
"One artifact"/"Optional Bitty integration" and AGENTS.md authority paragraph).
The Wheel column mixes two horizons and labels each cell: **current 0.1.0 roles**
are its own coding workflow, task graph, coding-workflow policy, and its own
records; **every other Wheel cell is a post-0.1.0 sibling design target**, not a
current role. Wheel's own provider adapters and approved secret environment
reads stay at the harness edge in 0.1.0; only a host-brokered credential-handle
role is post-0.1.0.

| Concern                                          | Core / `bittyd` / `bitty-execution` / `bitty-a11y`                                                                                               | AI owner (`bitty-ai`)                                                                                                                                                   | Wheel owner (horizon-labeled)                                                                                                                              | Headed Agent owner (`bitty-ai` product)                       | Media presentation (owner-pending)            | Provider plugin owner                     |
| ------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------- | --------------------------------------------- | ----------------------------------------- |
| IPC framing, auth, instance, rate limits         | Core owner                                                                                                                                       | Consumer                                                                                                                                                                | Post-0.1.0 target                                                                                                                                          | Consumer                                                      | Consumer through Core                         | Consumer through AI and Core              |
| Consent ledger and revocation                    | Core owner                                                                                                                                       | Requests verdicts; never stores grants                                                                                                                                  | Post-0.1.0 target                                                                                                                                          | Presents prompts; never grants                                | Consumes verdicts                             | Consumes verdicts                         |
| Execution authorization and supervision          | Core authorizes, fences, and retains PTY/permission/resource enforcement; `bitty-execution` supervises and executes under `W-132`.               | Owns semantic task model, binding, claims, retry policy, and model-facing execution intent.                                                                             | Post-0.1.0 target                                                                                                                                          | Requests work through the session; never executes or approves | None                                          | None                                      |
| Agent session, `bittyd` session, app handle      | `bittyd` is sole writer of registry/lifecycle/generation/lease if `bitty#2546` places the record there; placement open.                          | Consumes the RFC state names; `bitty-agent-api` is test-only/report-only; the runtime records `Active`/`Completed`/`Failed`/`Canceled`.                                 | Post-0.1.0 target                                                                                                                                          | Holds the application handle and projects the sanitized view  | None                                          | None                                      |
| Session handle and generation                    | Owner of the placement target pending `bitty#2546`; single-writer invariant applies.                                                             | Binds model-facing state to the session generation                                                                                                                      | Post-0.1.0 target                                                                                                                                          | Consumer                                                      | None                                          | None                                      |
| Event sequencing and replay window               | Sequencer is the authority designated by `bitty#2546`; Event Bus transport and observation queues are Core.                                      | Owns AI event schemas and nested turn sequencing                                                                                                                        | Post-0.1.0 target                                                                                                                                          | Consumer of the control stream                                | Consumes media events                         | None                                      |
| Provider registry and routing                    | Core owns publication authority, scopes, and transport.                                                                                          | Owns the authoritative registry, resolution, opaque proof, provider descriptor, capability labels, and privacy classes.                                                 | Post-0.1.0 target                                                                                                                                          | Consumer; no provider selection                               | None                                          | Registers; never widens                   |
| Credentials and secret storage                   | Host-owned storage and grants                                                                                                                    | Uses opaque handles; never reads values                                                                                                                                 | No platform credential-handle role in 0.1.0; own provider adapters and approved secret-env reads stay at the harness edge; host-brokered target post-0.1.0 | Never receives values                                         | Never receives values                         | Resolves at the adapter edge only         |
| Context compilation, quotas, and retention       | Core enforces shared per-client quota and ledger; R3 owns the retention policy.                                                                  | Owns context compilation, pre-I/O gates, AI record semantics, and the speech `AssetRef` instance.                                                                       | Post-0.1.0 target                                                                                                                                          | Consumer                                                      | None                                          | None                                      |
| Context and session snapshots                    | Core owns the host record, consent, and storage.                                                                                                 | Owns compilation, redaction, and snapshot view                                                                                                                          | Post-0.1.0 target                                                                                                                                          | Reads sanitized view                                          | None                                          | None                                      |
| Tool proposal                                    | Core receives and validates                                                                                                                      | Owner of model proposal relay                                                                                                                                           | Post-0.1.0 target                                                                                                                                          | Presents                                                      | None                                          | None                                      |
| Tool approval, execution, result                 | Core owns ledger, verdict, and authorization; `bitty-execution` executes and reports the structured outcome.                                     | Consumes result as untrusted observation                                                                                                                                | Post-0.1.0 target                                                                                                                                          | Presents request and collects decision                        | None                                          | None; tool providers are separate plugins |
| Panel identity, lifecycle, focus                 | Core owner under the accepted Panel Runtime RFC                                                                                                  | Reads references only                                                                                                                                                   | Post-0.1.0 target                                                                                                                                          | Requests panel and composes UI                                | Consumes a panel surface                      | None                                      |
| Panel Event Bus transport and observation topics | Core owner under the accepted Panel Runtime RFC; the bus does not carry approval/stop/tool-result control.                                       | Publishes AI-domain observation topics                                                                                                                                  | Post-0.1.0 target                                                                                                                                          | Publishes only its private observation topics                 | Publishes declared observation topics         | None                                      |
| Accessibility baseline and AT adapters           | Core retains the baseline, projection, focus order, announcements, motion, and contrast; `bitty-a11y` owns AT-SPI/UIA/AX adapters under `W-134`. | Supplies semantic state for its own content; no AT authority.                                                                                                           | Post-0.1.0 target                                                                                                                                          | Conforms; owns no AT backend or terminal mutation             | No AT backend                                 | None                                      |
| Speech job lifecycle                             | Core enforces consent and transport; the `AssetRef` contract is proposed, not accepted.                                                          | Owns request, stream, cancel, and the speech-job asset-reference instance.                                                                                              | None                                                                                                                                                       | Owns eligibility, controls, captions, and transcript          | Owns playback only                            | Synthesizes under the speech contract     |
| Audio device and playback                        | Core may provide a neutral primitive; none claimed.                                                                                              | No ownership                                                                                                                                                            | None                                                                                                                                                       | Requests playback                                             | Owner (owner-pending)                         | None                                      |
| Avatar semantic state and viseme vocabulary      | No ownership                                                                                                                                     | Owns the new candidate vocabulary; not accepted                                                                                                                         | None                                                                                                                                                       | Maps to appearance and animation                              | Renders GPU-backed 2D/3D (owner-pending)      | May emit timing metadata                  |
| 3D rig, shaders, and GPU surfaces                | No ownership                                                                                                                                     | No ownership                                                                                                                                                            | None                                                                                                                                                       | May request                                                   | Owner (owner-pending)                         | None                                      |
| CPU, memory, GPU, audio budgets                  | Core host and transport budgets; shared per-client quota and ledger.                                                                             | Pre-I/O gates and AI record attribution; no separate AI budget.                                                                                                         | Post-0.1.0 target                                                                                                                                          | Presentation budgets                                          | Device and decoder budgets                    | Provider quota reporting                  |
| Platform audio/GPU/AT adapters                   | Core host integration; `bitty-a11y` owns AT adapters under `W-134`.                                                                              | No platform API                                                                                                                                                         | Post-0.1.0 target                                                                                                                                          | No platform API                                               | Device-specific adapter owner (owner-pending) | No platform API                           |
| Plugin identity, category, distribution, signing | No ownership.                                                                                                                                    | Headed Agent product owner chooses whether to publish a package (identity, distribution presence); `bitty-plugins-docs` owns package category and signing/update rules. | Post-0.1.0 target                                                                                                                                          | Consumes the accepted plugin-platform contract                | None                                          | None                                      |
| Coding workflow and task graph                   | No ownership                                                                                                                                     | No ownership                                                                                                                                                            | Owner (current 0.1.0 role)                                                                                                                                 | No `wheel.*` dependency                                       | None                                          | None                                      |
| Retention and deletion                           | Core owns host evidence, consent, storage, and deletion mechanics.                                                                               | Owns AI record semantics and the speech asset-reference instance under draft R3.                                                                                        | Its own records (current 0.1.0 role)                                                                                                                       | Ephemeral media by default                                    | No persistent media by default                | Not owned here; provider policy governs   |

## Security review

A **design security review** was completed at `01a1ad4` and recorded
`APPROVE (design review)` under `PX-1575`. The `3091977` verification (`PX-1581`)
withdrew that approval: AT activation could record a verdict without a trusted
user-activation boundary. This pass makes AT activation non-approving and
requires a Core-observed keyboard or pointer activation; Security re-review is
pending. A design-review approval covers design content only: no implementation
was reviewed, no security gate was executed, and it is not a release or
implementation sign-off. Non-blocking P2/P3 follow-ups from that review are tracked in
[Remaining review follow-ups](#remaining-review-follow-ups). This section records
the threat model and the gates a later implementation review must close; the
document must remain `draft` until every row in
[P0 Review Sign-off](#p0-review-sign-off) records a disposition. That nine-row
table is the single authoritative sign-off set; this section does not create a
smaller or alternate role list.

### Threat analysis

| Threat                                         | Impact                                                                                                                                  | Required gate                                                                                                                                                                                                                                                          | Accountable owner(s)                             |
| ---------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------ |
| Presentation becomes authority                 | A panel, avatar, or attachment can approve or execute without consent.                                                                  | Presentation-never-authority: every effect re-checks the caller's grants.                                                                                                                                                                                              | Core owner + Headed Agent owner                  |
| AT projection or action becomes authority      | An assistive-technology adapter approves on its own, forges a user activation, grants, mutates terminal state, or publishes on the bus. | Adapter is not a principal; AT activation is non-approving; only a Core-observed keyboard or pointer activation of the focused bound decision control through the trusted input path can record a verdict; adapter-supplied activation is ignored; no bus publication. | Core owner + accessibility adapter owner         |
| Stale session, panel, or attachment generation | An old approval, stop, or attachment revives a newer session or writes into a destroyed panel.                                          | Session, panel, `(PanelId, generation)`, and attachment fencing on every mutating operation and event.                                                                                                                                                                 | Core owner + AI owner                            |
| Approval substitution                          | A user approves one call and a different call executes.                                                                                 | Tool identity and version, canonical argument digest, Core-authored prompt, target, scope, expiry, and session-generation binding.                                                                                                                                     | Core owner                                       |
| Same-UID principal confusion                   | A process under the same UID presents another `AgentId` or takes over a session.                                                        | `AgentId` is a validated declaration, not transport authentication; attach and takeover re-authorize grants and consent; cross-principal test.                                                                                                                         | Core owner + Security owner                      |
| Speech egress substitution or replay           | Text, provider, or privacy class changes after approval, or an authorization is reused.                                                 | Core-issued one-shot authorization bound to text digest, provider, privacy class, session generation, principal, and expiry; per-use audit.                                                                                                                            | Core owner + AI owner                            |
| Approval prompt spoofing                       | A presentation surface rewrites the prompt while the bound call differs.                                                                | Core-authored canonical prompt bound to the digest; a mismatch fails closed before the verdict is recorded.                                                                                                                                                            | Core owner + Headed Agent owner                  |
| Rich rendering escape                          | Hostile Markdown, URI, or local resource reaches a desktop capability.                                                                  | `P0-AC-005`, `P0-AC-009`, and `P0-AC-010` controls plus an adversarial corpus.                                                                                                                                                                                         | Headed Agent owner + security reviewer           |
| Secret leakage through snapshots or events     | Credentials or environment values reach a model, log, or UI.                                                                            | Typed redaction before queueing and writing; presence markers only.                                                                                                                                                                                                    | Core owner + AI owner                            |
| Unbounded media parsing                        | A malicious audio or avatar asset exhausts memory or executes code.                                                                     | Mandatory isolated decode, depth limits, and fuzzed decoders; fail closed when isolation is unavailable; no v1 Lua decoder.                                                                                                                                            | Media presentation owner (owner-pending)         |
| Caption or transcript leakage                  | Hidden context or a secret appears in captions, transcript, or export.                                                                  | Same post-redaction text as speech; bounded transcript; redacted export preview; retention class recorded.                                                                                                                                                             | Headed Agent owner + AI owner                    |
| Prompt injection into speech or approval       | Model output causes an egress or an approval.                                                                                           | Egress filtering; approval requires explicit user action, never model text.                                                                                                                                                                                            | AI owner + Core owner + Headed Agent owner       |
| Cross-panel eavesdropping or topic spoofing    | One plugin reads or forges another plugin's stream.                                                                                     | Reserved namespaces, per-topic authorization, producer identity checks.                                                                                                                                                                                                | Core owner                                       |
| Privacy downgrade on fallback                  | A local-only request silently goes remote.                                                                                              | Privacy class is a hard constraint; fallback stays within it and never bypasses Core publication authority.                                                                                                                                                            | AI owner                                         |
| Resource exhaustion or denial of service       | One session starves audio, GPU, or the host.                                                                                            | Hard caps, per-session attribution, containment, and slow-consumer shed.                                                                                                                                                                                               | Core owner + AI owner + media presentation owner |
| GPU or audio device data retention             | One principal's media remains in device memory for another.                                                                             | Release and clear on detach, suspend, and session end.                                                                                                                                                                                                                 | Media presentation owner (owner-pending)         |
| `wheel.*` coupling                             | The Headed Agent surface reaches into Wheel private state.                                                                              | Dependency review; no `wheel.*` import, call, config, or environment key.                                                                                                                                                                                              | Headed Agent owner + security reviewer           |

### Security gates

| Gate                             | Requirement                                                                                                                                                                                                                                                                                                                                                | Evidence required before implementation                                                                                                                                                                                           |
| -------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Presentation-never-authority     | A panel, attachment, avatar state, animation, or speech control cannot approve, execute, or broaden a scope.                                                                                                                                                                                                                                               | Adversarial tests that attempt approval and execution from every presentation surface.                                                                                                                                            |
| AT action non-authority          | An AT adapter cannot approve on its own, grant, mutate terminal state, publish the projection or action outcome on the Panel Event Bus, or forge/forward a user activation. AT activation is non-approving; only a Core-observed keyboard or pointer activation of the focused bound decision control through the trusted input path may record a verdict. | Adapter action corpus, stale-element rejection, Event-Bus negative test, forged/programmatic-activation rejection, Core-observed activation records exactly once, and adapter-supplied activation ignored.                        |
| Accessibility baseline           | Roles, names, states, focus order, announcements, modal confinement, keyboard equivalence, reduced motion, contrast resolution, captions/transcript as candidate, avatar equivalence, and media controls meet the accepted baseline plus tracked candidates.                                                                                               | Semantics map, focus/announcement corpus, reduced-motion and contrast-resolution fail-closed tests, caption/transcript bounds once `D4` accepts them, and missing-adapter tests.                                                  |
| Control/observation separation   | Approval, stop, tool-result, final-turn, and speech-job control events are never carried by the `DropOldest` Panel Event Bus or its observation queues.                                                                                                                                                                                                    | Transport assertion tests and a negative test that a control event cannot be declared as a bus topic.                                                                                                                             |
| Session authority mapping        | A `bittyd` session, an AgentSession, and the application handle are not conflated; `start` creates only what `bitty#2546` permits and daemon-absent operation is supported.                                                                                                                                                                                | Lifecycle mapping tests for daemon-present, daemon-absent, non-terminal worker (if permitted), and terminal-session paths, plus a test that a `bittyd` worker generation never substitutes for an application session generation. |
| Generation fencing               | Stale session, panel, attachment, approval binding, and media-job identities fail closed before effects.                                                                                                                                                                                                                                                   | Property tests and replay corpus across suspend, reconnect, takeover, panel rebuild, and restart.                                                                                                                                 |
| Approval binding                 | Approval binds the exact tool identity and version, validated call digest, Core-authored prompt, target, scope, expiry, session generation, and principal.                                                                                                                                                                                                 | Mutation, prompt-rewrite, replay, expiry, and cross-principal test matrix.                                                                                                                                                        |
| Speech authorization             | Every speech egress consumes one Core-issued authorization bound to the text digest, provider, privacy class, session generation, principal, and expiry.                                                                                                                                                                                                   | Reuse, digest-mismatch, provider-change, privacy-change, stale-generation, cross-principal, and compromised-plugin tests; per-use audit evidence.                                                                                 |
| Routing snapshot integrity       | A caller-supplied routing snapshot version and digest is an assertion validated against the Core publication and the AI helper proof; a mismatch fails the turn closed.                                                                                                                                                                                    | Mutation-invalidates-proof and stale-proof-fails-turn-closed tests on every provider, model, capability, context-window, cost, and alias mutation.                                                                                |
| Media trust split                | A trusted Core extension reaches GPU, window, and audio resources only through Core-owned interfaces; an untrusted post-1.0 helper receives no such handle; neither is a v1 Lua plugin.                                                                                                                                                                    | Handle-surface review per form, capability declaration, isolation proof, and a negative test that no raw handle crosses the plugin or helper boundary.                                                                            |
| Same-UID boundary                | `AgentId` is a validated declaration inside the UID trust domain; attach and takeover re-authorize grants, and no confidentiality between same-UID processes is claimed.                                                                                                                                                                                   | Cross-principal attach and takeover tests, plus a documented same-UID confidentiality boundary.                                                                                                                                   |
| Secret-free surfaces             | Snapshots, events, speech text, avatar state, and traces contain no raw secret or unapproved environment value.                                                                                                                                                                                                                                            | Seeded-secret negative tests on every producer and every write path.                                                                                                                                                              |
| Bounded media                    | Audio, viseme, animation, and avatar inputs are size-, depth-, and time-bounded; decode is mandatorily isolated or fails closed; no v1 Lua decoder or native module load.                                                                                                                                                                                  | Fuzz corpus, malformed asset suite, isolation proof, and fail-closed presentation-class tests.                                                                                                                                    |
| Consent before egress            | Model, speech, and tool egress require the matching grant; autoplay is off unless an explicit user-owned policy enables it; revocation stops the next chunk.                                                                                                                                                                                               | Consent matrix and revocation lifecycle tests.                                                                                                                                                                                    |
| Retention and erasure            | Media and snapshots are ephemeral by default and name a retention class; recording is consented and redacted; source-history deletion propagates to derived media.                                                                                                                                                                                         | Retention-class, export-preview, and deletion-propagation tests.                                                                                                                                                                  |
| Rich rendering controls          | Any Markdown, rich-block, image, or scene surface uses the `P0-AC-005` loader, the `P0-AC-009` URI and launch policy, and the `P0-AC-010` constrained AST/scene with no WebView script.                                                                                                                                                                    | Script, local-resource, hostile-URI, and constrained-scene corpus on every rich surface.                                                                                                                                          |
| Capability fallback fails closed | A missing or unverified platform capability lowers presentation only; it never weakens security, consent, or redaction.                                                                                                                                                                                                                                    | Platform matrix tests and capability-detection negative tests.                                                                                                                                                                    |
| No Wheel coupling                | The Headed Agent surface has no `wheel.*` import, service call, CLI spawn/wrapper, `.wheel/init.lua` or Wheel config/state read, configuration dependency, environment key, or private state access.                                                                                                                                                       | Dependency graph review, path-access review, and a negative integration test per forbidden path, plus a trust-gate bypass test and a Wheel CLI output-parsing test.                                                               |
| Independent review               | Every row in [P0 Review Sign-off](#p0-review-sign-off) differs from the implementer and records a disposition before implementation tasks open.                                                                                                                                                                                                            | Nine review records linked from the owning task.                                                                                                                                                                                  |

### Residual risks

- No headed surface implementation exists, so every gate above is unverified.
  The panel runtime and Event Bus are accepted contracts whose implementation is
  partial and experimental: bounded in-process `PanelEventBus` queues exist in
  bundled-disabled consumers, while cross-process routing, the v1 topic taxonomy,
  and the capability ledger remain open (`RFC-OQ-4`/`RFC-OQ-5`). The accepted
  contract is not evidence that the headed surface is implemented.
- No platform accessibility adapter exists today beyond a headless test backend.
  An absent, denied, or crashed adapter removes platform exposure only; it is
  reported honestly and never described as working AT support. FreeBSD has no AT
  backend and reports that state.
- Media sandboxing and GPU memory isolation are platform-dependent. A
  presentation class that cannot verify the required isolation is disabled
  rather than treated as contained, so the residual risk is reduced
  presentation, not unaudited media parsing.
- Synthetic-speech disclosure is not decided at product level (`D2`). The
  document requires disclosure to exist and forbids silently presenting
  synthetic audio as human; the minimum form remains an owner decision.
- Multi-user, multi-device, and remote headed clients are deferred. They would
  need their own authentication and consent ledger review.
- The `bittyd` session record, application-session-generation issuance, and
  non-terminal worker scope are the subject of `bitty#2546`. `ADR-0019` accepts
  the v0.2.0 staging and `R-B09` fixes the single-writer rule; neither decides the
  field set, the API, or whether a headed agent worker is hosted. A wrong
  placement would still be a contract violation even though it cannot produce two
  writers silently.
- The accepted transport authenticates the UID, not the `AgentId`. Processes
  under the same UID are not isolated from each other, and this draft claims no
  confidentiality between them; that boundary needs its own accepted decision
  if same-UID isolation is required.
- The media presentation category and its packaging are not decided. A v1 Lua
  package, a trusted Core extension component, and an untrusted post-1.0 native
  helper are different supply-chain and permission stories; only the last two are
  compatible with `R-017`, and neither is authorized here.
- The flash threshold, the minimum content-contrast floor, the caption timing
  tolerance, transcript bounds, and retention classes are not fixed numeric
  values in this draft. The flash threshold has no owning contract yet, the
  contrast floor belongs to `OQ-037`, and the caption/retention constants belong
  to a future accepted speech contract; this draft fixes obligations only.

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
  cancellation before and after dispatch. Session-mapping tests cover
  daemon-present, daemon-absent, bittyd-hosted non-terminal worker (if
  permitted), plain AgentSession, and application-handle-only paths without
  conflating them, plus a test that the `bittyd` worker-binding generation never
  substitutes for the application session generation.
- Event-ordering and resynchronization tests: non-lossy gaps, lossy lost-count
  recovery, duplicates, snapshot barriers, replay-window boundaries, reconnect
  during a turn, final-turn control classification, SMO envelope identity,
  attribution, deadline, priority, dedup, cancellation, and expiry, and a
  cross-window attachment attempt that fails closed without the `RFC-OQ-4`
  transport contract.
- Event Bus tests: accepted `owner.name:topic` grammar, manifest-declared
  produce/consume, `UndisclosedTopic`, PR-3 through PR-9 bounds, `DropOldest`
  counting and attribution, per-principal ledgers, and a negative test that an
  approval, stop, tool-result, or speech-job control event cannot be declared or
  delivered as an observation topic.
- Backpressure and soak tests: slow consumer, bounded queues, counted drops,
  audio underflow, and no unbounded memory growth.
- Approval tests: exact tool identity and version, digest binding, Core-authored
  prompt integrity, argument mutation, target change, expiry, cross-principal
  replay, and reconnect with a pending approval.
- Snapshot tests: bounded size, redaction, viewer filtering, immutable
  versions, separation of routing/context/session snapshots, CP-5/AG-2
  generation binding, expansion re-authorization, caller-supplied routing
  version/digest assertions rejected as authority, and a
  mutation-invalidates-proof plus stale-proof-fails-turn-closed test.
- Speech tests: one-shot authorization bound to text digest, provider, privacy
  class, session generation, principal, and expiry; reuse, digest mismatch,
  provider or privacy change, cross-principal, and compromised-plugin failures;
  per-use audit; redaction; first-chunk timeout; cancellation at every boundary;
  exact eight-state status set; `Unknown` reconciliation; asset expiry,
  retention class, and idempotency; no auto-play after reconnect without a
  user-owned policy; and, once `D4` accepts the candidate, synchronized captions
  plus an inspectable transcript with locale, bounds, redaction, retention, AT
  exposure, and toggle coverage.
- Takeover tests: target grants and consent re-authorization, old-generation
  media-job reconciliation, and a cross-principal attempt from the same UID and
  from a different UID.
- Avatar and viseme tests: latest-wins state, bounded animation queue, clock
  tolerance, loss degradation, reduced motion, decorative-versus-informative
  alternative content, state-equivalence announcements, egress prohibition,
  buffer clearing, and release on suspend.
- Accessibility tests: the full verification table in
  [Accessibility verification and acceptance criteria](#accessibility-verification-and-acceptance-criteria),
  including semantics mapping, focus order and restoration, announcement
  coalescing, keyboard equivalence, contrast resolution and the untracked flash
  threshold, missing/denied/crashed AT adapters, FreeBSD AT unavailability, and
  AT-action non-authority including a programmatic-activation rejection and a
  user-activation approval path.
- Media presentation tests: category conformance (trusted Core extension
  component or untrusted post-1.0 helper, never a v1 Lua package), the handle
  split (Core-mediated handles for the extension, none for the helper),
  transcript-free playback, bounded decode, and fail-closed isolation.
- Budget tests: per-dimension hard and soft limits, `FS-AI4` attribution
  fields, shared per-client quota with no separate AI budget, `RC-9`/`RC-10`
  and PR-7 through PR-9 inheritance, retention-class enforcement, degradation,
  and isolation between sessions.
- Rich rendering tests: the `P0-AC-005`, `P0-AC-009`, and `P0-AC-010`
  adversarial corpus on every Markdown, rich-block, image, and scene surface.
- Platform matrix tests: Linux, macOS, Windows, and FreeBSD capability
  detection, ADR-0002 support-tier alignment, AT adapter availability separately
  from presentation class, missing device, device loss, decode-isolation
  failure, and fallback to text or headless.
- Wheel boundary tests: a negative integration test for `wheel.*` import, CLI
  spawn or wrapper, Wheel CLI output parsing, `.wheel/init.lua` read/execute,
  `$XDG_CONFIG_HOME/wheel/init.lua`, and
  `$XDG_STATE_HOME/wheel/trusted_projects.json`; plus a `.wheel/init.lua`
  write-attempt test and a trust-gate bypass test that must fail closed before
  evaluation.
- Security adversarial tests for every gate in the security review table.

A future contract is not accepted because this document exists. Each family
needs its own accepted specification, owner, implementation task, evidence, and
independent review.

## Alternatives considered

| Alternative                                                          | Why it is not selected                                                                                                                                                                                                                 |
| -------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Put headed UI, speech, avatars, or 3D in Wheel                       | Couples high-churn presentation and media to the coding harness, expands Wheel's product scope, and contradicts the requirement that Wheel remain a focused Coding Agent Harness.                                                      |
| Let the Headed Agent depend on Wheel as a library or front end       | Creates a private cross-consumer dependency, makes Wheel's internal API a de facto contract, and prevents independent evolution. Shared needs belong in Core or `bitty-ai`.                                                            |
| Put the Headed Agent inside Core                                     | Makes the terminal platform own AI composition, chat policy, and media presentation, violating the Core-versus-AI boundary and the no-model-I/O rule for Core.                                                                         |
| One global event stream with a total order                           | Creates head-of-line blocking, scales poorly across sessions and panels, and turns unrelated plugins into a shared failure domain. Per-session ordering on the control stream and FIFO within one bus queue suffice.                   |
| Drop-oldest delivery for approvals and tool results                  | Can silently lose an approval, a denial, or a result, which is an authority and safety defect. Control events must be non-lossy.                                                                                                       |
| Carry approval, stop, and tool-result control on the Panel Event Bus | The accepted Panel Event Bus is an observation and lifecycle transport with `DropOldest`; a control event on it can be dropped or overtaken and would need its own accepted amendment. The session control stream is separate.         |
| Ship media decode/render/audio as a v1 Lua plugin                    | Native in-process plugins and native module loading are forbidden (`R-017`); direct GPU, window, and PTY access is outside the plugin boundary. The native path is a trusted Core extension component or an untrusted post-1.0 helper. |
| Define a new `ai.*`/`agent.*`/`mcp.*` capability catalog here        | No accepted catalog exists: the `bitty-ai` extension draft proposes one and `bitty-plugins` has not accepted the manifest grammar. This draft consumes no identifier and adds only presentation classes.                               |
| Describe the surface as a generic future sibling repository          | The recorded predecessor is `ai-panel`, and AI surfaces move to the optional `bitty-ai` extension. A generic sibling framing hides the blocked panel-provider, `bitty-ai` surface, and distribution dependencies.                      |
| Let the plugin resolve providers and hold credentials                | Fragments routing and consent, exposes secrets to presentation code, and bypasses the draft provider boundary and secret invariant.                                                                                                    |
| Let Core render avatars and play speech directly                     | Makes platform code own media policy, GPU lifetime, and accessibility behavior that belong to media presentation and the Headed Agent owner.                                                                                           |
| Inline audio bytes in model context                                  | Bypasses context budgets and redaction, bloats snapshots and events, and contradicts the asset-reference rule in the multimodal boundary.                                                                                              |
| Require 3D or audio for a headed session                             | Excludes headless, low-resource, accessibility, and FreeBSD-style environments. Text, captions, and 2D/software fallbacks are mandatory.                                                                                               |
| Treat panel closure as session stop                                  | Conflates presentation with lifecycle and lets UI absence destroy work. Panel closure detaches; stopping is explicit.                                                                                                                  |

## Affected contracts

No contract is changed by this draft. The table records the expected
relationship for the owning documents and repositories.

| Contract or surface                                                                                                                                                                                                                                                                 | Owner                                                                                     | Effect of this draft                                                                                                                                                                                                                        | Status                                                 |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------ |
| [IPC and Agent RFC](../specifications/ipc-agent-rfc.md)                                                                                                                                                                                                                             | Core owner                                                                                | Consumed unchanged for framing, auth, scopes, bounded messages, consent, streaming, and versioning.                                                                                                                                         | Accepted; no change                                    |
| [Panel Runtime RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/panel-runtime-rfc.md)                                                                                                                                                            | Core owner                                                                                | Consumed unchanged for panel identity, lifecycle, focus, Event Bus grammar, and PR-1 through PR-12.                                                                                                                                         | Accepted; no change                                    |
| [RFC-0006](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/rfcs/RFC-0006-panel-provider-registration-mount.md)                                                                                                                                                | Core owner                                                                                | Consumed unchanged for plugin-facing registration, mount placement, and the closed `panel.*` capability mapping.                                                                                                                            | Accepted; no change                                    |
| [Provider-routing snapshot binding](../architecture/execution-ownership-r1.md#providerrouting-snapshot-binding-contract-accepted-2026-10-17)                                                                                                                                        | Core owner + AI owner                                                                     | Consumed unchanged for routing snapshot domain, bytes, digest, publication authority, and ownership proof.                                                                                                                                  | Accepted section; no change                            |
| [ADR-0013](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/adrs/ADR-0013-core-ontology-identity.md)                                                                                                                                                           | bitty-docs decision owner                                                                 | Consumed for ontology, `AgentId`, `GenerationId`, and the Panel/Execution separation.                                                                                                                                                       | Accepted; no change                                    |
| [ADR-0002](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/adrs/ADR-0002-platform-support-tiers.md)                                                                                                                                                           | bitty-docs decision owner                                                                 | Consumed for support tiers; presentation classes never upgrade a support tier.                                                                                                                                                              | Accepted; no change                                    |
| [P0 acceptance criteria](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/p0-acceptance-criteria.md)                                                                                                                                                            | Security owner                                                                            | Rich, avatar, and media surfaces inherit `P0-AC-005`, `P0-AC-009`, and `P0-AC-010`.                                                                                                                                                         | Normative; no change                                   |
| [Isolation and Resource RFC](https://github.com/bitty-terminal/bitty-plugins-docs/blob/main/runtime/isolation-resource-rfc.md)                                                                                                                                                      | Security owner                                                                            | Inherits `RC-1`, `RC-2`, `RC-9`, and `RC-10`; this draft adds no weaker bound.                                                                                                                                                              | Accepted; no change                                    |
| [Unified authorization backend](../architecture/tool-transport-r2.md#unified-authorization-backend)                                                                                                                                                                                 | Core owner                                                                                | Tool approval events must map to the existing gate order; no new authorization path.                                                                                                                                                        | Draft; no change                                       |
| [Context retention R3](../architecture/context-retention-r3.md)                                                                                                                                                                                                                     | Context management (R3 contract owner) + Core owner (host evidence, consent, and storage) | Speech, viseme, avatar, and attachment data follow the draft consent, retention, and deletion rules; R3 owns the retention policy, not the terminal Core.                                                                                   | Draft; no change                                       |
| [Provider plugin boundary](../providers/provider-plugin-boundary.md)                                                                                                                                                                                                                | AI owner + Provider plugin owner                                                          | Consumed unchanged for privacy classes, capabilities, secret invariant, and project overrides.                                                                                                                                              | Draft; no change                                       |
| [Multimodal inference boundary](../providers/multimodal-inference-boundary.md)                                                                                                                                                                                                      | AI owner                                                                                  | Draft speech and generation-job lifecycle reused; this draft adds only headed request, playback, and cancellation rules.                                                                                                                    | Draft; review follow-up                                |
| [Agent coordination architecture](agent-coordination.md)                                                                                                                                                                                                                            | AI owner + Core owner                                                                     | Aligns panel attachment, suspend, and lifecycle outcomes with the existing reconciliation.                                                                                                                                                  | Draft; candidate alignment                             |
| [Panel environment awareness](../interfaces/panel-environment-awareness.md)                                                                                                                                                                                                         | AI owner + Core owner                                                                     | Sanitized Agent View and no-persistence defaults continue to apply to headed presentation.                                                                                                                                                  | Draft; no change                                       |
| [Execution-projection binding](../specifications/execution-projection-binding-candidate.md)                                                                                                                                                                                         | Core owner + AI owner                                                                     | Candidate design reference only; session attachment is distinguished from execution projection.                                                                                                                                             | Draft; no change                                       |
| [Wheel core and plugin boundary](../specifications/wheel-core-plugin-boundary-candidate.md)                                                                                                                                                                                         | Wheel owner + AI owner                                                                    | Wheel's internal composition candidate, not this sibling boundary; no Wheel work is assigned.                                                                                                                                               | Draft; no change to Wheel                              |
| Core panel and Event Bus implementation                                                                                                                                                                                                                                             | Core owner                                                                                | The panel and Event Bus contracts are accepted by the Panel Runtime RFC and RFC-0006; implementation is partial and experimental, and cross-process routing/taxonomy/capability ledger stay open (`RFC-OQ-4`/`RFC-OQ-5`).                   | Accepted contract; implementation partial/experimental |
| [ADR-0019](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/adrs/ADR-0019-bittyd-staging-advance.md)                                                                                                                                                           | bitty-docs decision owner                                                                 | Consumed for the `bittyd` v0.2.0 staging, session-grained attach/detach, and no restart persistence.                                                                                                                                        | Accepted; no change                                    |
| [bitty#2546](https://github.com/bitty-terminal/bitty/issues/2546) and [bittyd session host requirements](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/bittyd-session-host-requirements-candidate.md)                                              | Core owner                                                                                | Session fields, lifecycle, API, trust, crash/reattach, and single-writer requirements; consumed as open consumer input.                                                                                                                     | Open; no change                                        |
| [ADR-0016](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/adrs/ADR-0016-execution-graphics-accessibility-storage-platform-boundaries.md) Boundaries 1 and 3                                                                                                  | bitty-docs decision owner                                                                 | Consumed for Core authorization/fencing plus `bitty-execution` supervision, and for the Core-retained accessibility baseline plus `bitty-a11y` adapters.                                                                                    | Accepted; no change                                    |
| [Rich Presentation RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/rich-presentation-rfc.md)                                                                                                                                                    | terminal docs owner                                                                       | Consumed for the presentation contract any headed rich surface inherits; `P0-AC-005`/`009`/`010` remain the security floor.                                                                                                                 | Accepted; no change                                    |
| [Accessibility Extraction Contract](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/accessibility-extraction-contract.md) (`W-134`)                                                                                                                  | terminal docs owner + accessibility adapter owner                                         | Consumed unchanged for projection, focus synchronization, controlled actions, and AT adapter responsibilities.                                                                                                                              | Accepted; no change                                    |
| [RFC-0001](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/rfcs/RFC-0001-appearance-configuration.md) and [RFC-0002](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/rfcs/RFC-0002-panel-animations.md)                                 | bitty-docs decision owner                                                                 | Consumed for appearance-pair contrast resolution and the non-overridable reduced-motion/safe-mode contract; the content-contrast floor stays with `OQ-037`.                                                                                 | Accepted; no change                                    |
| [OQ-078](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/open-questions.md)                                                                                                                                                                                   | bitty-docs decision owner + accessibility owner                                           | Referenced for the still-open API/tree and v1 scope; it is not the only open accessibility question (`OQ-037` contrast, unassigned flash threshold); no accepted obligation is reopened.                                                    | Open; no change                                        |
| [OQ-083](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/open-questions.md)                                                                                                                                                                                   | bitty-docs decision owner                                                                 | Consumed as the accepted panel-lease direction; this draft defines no competing lease.                                                                                                                                                      | Accepted; no change                                    |
| [RFC-OQ-4](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/panel-runtime-rfc.md#open-questions)                                                                                                                                                      | terminal docs owner                                                                       | Open: exact v1 bus topic taxonomy, including AI topics and whether cross-window topics route through IPC or an in-process bus. This draft assumes no answer.                                                                                | Open; no change                                        |
| [RFC-OQ-5](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/panel-runtime-rfc.md#open-questions)                                                                                                                                                      | terminal docs owner                                                                       | Open: capability mapping per panel type, especially `panel.overlay` and new `panel.*` families. `panel.*` is the only accepted family cited here.                                                                                           | Open; no change                                        |
| [Product Vision](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/product/vision.md#agent-friendly-not-agent-centric) and [AI Surface Accepted Record](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/ai-surface-accepted-record.md) | terminal docs owner                                                                       | Consumed for agent-friendly/not-agent-centric placement and the `ai-panel` removal/AI-extension direction.                                                                                                                                  | Accepted; no change                                    |
| [Plugin Platform RFC](https://github.com/bitty-terminal/bitty-plugins-docs/blob/main/specifications/plugin-platform-rfc.md)                                                                                                                                                         | Plugin ecosystem owner                                                                    | Consumed for the v1 Lua boundary, `R-017`, closed capabilities, declarative UI, and generation lifecycle.                                                                                                                                   | Accepted; no change                                    |
| [Package Lifecycle RFC](https://github.com/bitty-terminal/bitty-plugins-docs/blob/main/packaging/package-lifecycle-rfc.md) and [Package Follow-up RFC](https://github.com/bitty-terminal/bitty-plugins-docs/blob/main/packaging/package-followup-rfc.md)                            | Plugin ecosystem owner                                                                    | Consumed for activation, integrity, rollback, resolver, registry, and key-management contracts; the contracts are accepted and implementation is missing.                                                                                   | Accepted; no change                                    |
| [Default Distribution RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/default-distribution-rfc.md)                                                                                                                                              | terminal docs owner + Plugin ecosystem owner                                              | Accepted bundle and enabled-by-default model consumed; this draft adds no bundled or enabled entry.                                                                                                                                         | Accepted; no change                                    |
| `bitty-ai` extension capability catalog (`ai.*`, `agent.*`, `mcp.*`)                                                                                                                                                                                                                | AI owner + Plugin ecosystem owner                                                         | Owner-pending: only a draft `bitty-ai` extension proposes it and `bitty-plugins` has not accepted extension-contributed manifest grammar; this draft mints no identifier and no `media.*`/`audio.*`/`gpu.*` family.                         | Not accepted; P-2 dependency                           |
| Plugin identity (`ai-panel` successor or new)                                                                                                                                                                                                                                       | Headed Agent product owner (`bitty-ai`)                                                   | Product decision `P-1`; no identity is chosen here.                                                                                                                                                                                         | Owner-pending                                          |
| Plugin package category                                                                                                                                                                                                                                                             | Plugin ecosystem owner (`bitty-plugins-docs`)                                             | Product decision `P-2`; the v1 Lua package is accepted, out-of-process is unaccepted, and a media category is undecided.                                                                                                                    | Owner-pending                                          |
| Distribution presence                                                                                                                                                                                                                                                               | Headed Agent product owner + Plugin ecosystem owner                                       | Product decision `P-3`; the accepted Default Distribution RFC owns the bundle/enabled-by-default model, and this draft claims no bundled or enabled entry for the surface.                                                                  | Owner-pending                                          |
| Signing and update                                                                                                                                                                                                                                                                  | Plugin ecosystem owner (`bitty-plugins-docs`)                                             | Product decision `P-4`; staged activation/rollback, resolver, registry, and key management are accepted contracts, and implementation is missing.                                                                                           | Owner-pending                                          |
| Media presentation packaging and category                                                                                                                                                                                                                                           | Core owner + Plugin ecosystem owner (category undecided)                                  | Owner-pending; a v1 Lua package is excluded, a trusted Core extension component (`bitty-graphics`/`bitty-a11y` pattern) or an untrusted post-1.0 helper are the only compatible paths, and the two have different handle and trust stories. | Owner-pending                                          |
| Speech and avatar event schemas                                                                                                                                                                                                                                                     | AI owner                                                                                  | Candidate families only; no wire schema, field list, or compatibility promise is accepted.                                                                                                                                                  | Draft                                                  |

### Cross-repository references

The owning repositories are linked once, here. The [References](#references)
section carries the local canonical documents and the security entry point.

- [bitty-docs](https://github.com/bitty-terminal/bitty-docs) owns shared
  governance, the decision register, the open-question register, and the
  normative security corpus. This document links to those owners instead of
  copying or reinterpreting them.
- [bitty-terminal-docs](https://github.com/bitty-terminal/bitty-terminal-docs)
  owns the terminal platform's accepted panel runtime, panel presentation,
  rich presentation, accessibility extraction, and platform-support contracts.
  Panel identity, lifecycle, Event Bus, and AT adapter references here point at
  that accepted contract. The `bitty-execution`, `bitty-graphics`, and
  platform-service focused contracts (`W-132`, `W-133`, `W-136`) also belong
  there; media presentation mechanism references remain owner-pending pointers
  to that repository.
- [bitty-plugins-docs](https://github.com/bitty-terminal/bitty-plugins-docs)
  owns the plugin platform, manifest, capability, package-category, and
  signing/update contracts. It owns the platform contract, not the Headed Agent
  product: `bitty-ai` owns the product and plugin identity choice. Media
  presentation packaging and permission review are split accordingly, and no
  new media package category is assumed. The accepted isolation envelope lives
  at
  [Isolation and Resource RFC](https://github.com/bitty-terminal/bitty-plugins-docs/blob/main/runtime/isolation-resource-rfc.md).
- [bitty-ai](https://github.com/bitty-terminal/bitty-ai) owns implementation
  evidence for the AI runtime, providers, and the Headed Agent product. This
  document claims no implementation status from that repository.
- [bitty-execution](https://github.com/bitty-terminal/bitty-execution),
  [bitty-graphics](https://github.com/bitty-terminal/bitty-graphics),
  [bitty-a11y](https://github.com/bitty-terminal/bitty-a11y), and
  [bitty-storage](https://github.com/bitty-terminal/bitty-storage) are the
  reserved repositories named by ADR-0016 at its acceptance. Their existence is
  not implementation evidence and they are not required for this draft.
  [bitty-platform-services](https://github.com/bitty-terminal/bitty-platform-services)
  is a separately initialized L1 extension crate for the Boundary 5
  platform-service adapters under `W-136`, not one of the four reserved component
  repositories and not a component-pattern example.
- [wheel](https://github.com/bitty-terminal/wheel) owns the Coding Agent
  Harness. Its
  [README](https://github.com/bitty-terminal/wheel/blob/main/README.md),
  [repo.toml](https://github.com/bitty-terminal/wheel/blob/main/repo.toml),
  [SECURITY-MODEL.md](https://github.com/bitty-terminal/wheel/blob/main/SECURITY-MODEL.md),
  and [AGENTS.md](https://github.com/bitty-terminal/wheel/blob/main/AGENTS.md)
  are the authoritative sources for its 0.1.0 position; this document records
  a sibling boundary only and assigns no Wheel implementation work.
- [bitty](https://github.com/bitty-terminal/bitty) owns Core implementation
  evidence and the `bittyd` v0.2.0 slice staging accepted by ADR-0019. Future
  `bittyd` and platform-adapter work require their own accepted contracts; the
  Core panel and Event Bus contracts are accepted by the Panel Runtime RFC and
  RFC-0006, while their implementation remains partial and experimental. No
  `bittyd` binary is claimed.

## Open points

None of these is resolved by this draft. Each needs an owner, a reviewed
decision, and evidence before the related implementation task opens. This table
is a review index: the `Blocks` column names the contract families whose
acceptance depends on the entry, but the table is not itself a blocker register
and carries no authority. Every row names a register, a tracked issue, a review
route, or an explicit deferral; promotion runs through AI-0347
([bitty-ai#553](https://github.com/bitty-terminal/bitty-ai/issues/553)) and the
named owner.

| Open point                                     | Question                                                                                                                                                                                                                               | Likely owner                                                        | Disposition or deferral                                                                                                                                                                                                                                                                                                                                                                | Blocks                                                                               |
| ---------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------ |
| Session authority split                        | Where do the durable `bittyd` session record, generation issuance, reconnect record, and attach leases live, and does `bittyd` host a headed agent worker? How do a `bittyd` session, an AgentSession, and the application handle map? | Core owner + AI owner                                               | [bitty#2546](https://github.com/bitty-terminal/bitty/issues/2546) is the open contract; ADR-0019 accepts the v0.2.0 staging and `R-B09` fixes the sole-writer rule. OQ-061 is adjacent (identity domains), not the primary register. Deferred until #2546 decides placement and worker scope.                                                                                          | `headed.session`, `headed.stream`, `headed.context`, `headed.tools`, `headed.budget` |
| Headed plugin identity and packaging           | Four separate decisions: `P-1` identity (`ai-panel` successor/rename or new), `P-2` package category, `P-3` distribution presence, `P-4` signing/update.                                                                               | `bitty-ai` product owner (P-1/P-3) + `bitty-plugins-docs` (P-2/P-4) | The recorded predecessor is `ai-panel`, a split-later hybrid whose split-later posture the 2026-09-30 removal superseded; the v1 Lua package is accepted, out-of-process is unaccepted, and the accepted Default Distribution RFC owns the bundle/enabled-by-default model. The `ai.*` capability catalog remains a `P-2` dependency. `AI-0347/#553`; deferred.                        | Plugin packaging, media presentation category                                        |
| Media presentation category and ownership      | Is audio/2D/3D/avatar presentation a Core extension component or a post-1.0 native helper, and who owns it? Is any media package category accepted?                                                                                    | Core owner + Plugin ecosystem owner                                 | Owner-pending. A v1 Lua plugin/package is excluded by `R-017`; the trusted Core-extension component path follows the `bitty-graphics`/`bitty-a11y` component pattern (Core-mediated handles), while an untrusted post-1.0 helper excludes GPU/window/PTY/host-Rust handles. `bitty-platform-services` is an L1 crate under `W-136`, not a component example. `AI-0347/#553`; deferred. | `headed.avatar`, `headed.speech`, `headed.platform`                                  |
| Coding capability access                       | If the Headed Agent needs coding intelligence, does it use only public Core/`bitty-ai` contracts, or must the needed mechanism be promoted there? A `wheel.*` runtime dependency is not authorized either way.                         | AI owner + Core owner; Wheel consulted                              | Promote to a public Core/`bitty-ai` contract; Wheel decisions D4/D6 record no host-brokered execution service in 0.1.0; `AI-0347/#553`; deferred                                                                                                                                                                                                                                       | `wheel.*` prohibition, product scope                                                 |
| Replay retention window                        | How long are control events retained for reconnect, and what storage or memory budget applies?                                                                                                                                         | Core owner + AI owner                                               | `AI-0347/#553` for promotion to a Core/AI retention task; AIQ-04/55/5A are adjacent closed(partial) facets; deferred                                                                                                                                                                                                                                                                   | `headed.stream` replay window                                                        |
| Multi-user and remote headed clients           | Are remote, multi-device, or shared-panel sessions in scope? What authentication and consent model applies?                                                                                                                            | Core owner + Security owner                                         | ADR-0008 (`OQ-020` closed, prior decision); out of scope for 0.1.0; promotion tracked by `AI-0347/#553`                                                                                                                                                                                                                                                                                | Identity, consent, transport                                                         |
| Speech privacy classes and provider consent    | Which speech providers are local-only, which require egress consent, and what is the minimum baseline?                                                                                                                                 | AI owner + Security owner                                           | `AI-0347/#553` plus AI/security review; deferred until an accepted speech contract defines the classes                                                                                                                                                                                                                                                                                 | `headed.speech`                                                                      |
| Viseme source and accuracy                     | Is the viseme timeline provider metadata, local analysis, or both? What accuracy and tolerance claims are allowed?                                                                                                                     | AI owner + media presentation owner (owner-pending)                 | `AI-0347/#553` plus media/architecture review; deferred                                                                                                                                                                                                                                                                                                                                | `headed.avatar`, `headed.speech`                                                     |
| 3D renderer placement                          | Is 3D an in-process extension, a separate process, or a remote surface? How is GPU memory isolated between principals?                                                                                                                 | Media presentation owner (owner-pending) + Architecture owner       | `AI-0347/#553` plus plugin-ecosystem packaging review; deferred                                                                                                                                                                                                                                                                                                                        | `headed.avatar`, `headed.platform`                                                   |
| Audio focus policy                             | Is audio focus per panel, per session, or global? What happens when two sessions speak at once?                                                                                                                                        | Core owner + Headed Agent owner                                     | `AI-0347/#553` for promotion to Core host policy; per-device exclusivity is fixed, scope is deferred                                                                                                                                                                                                                                                                                   | `headed.panels`, `headed.speech`                                                     |
| Platform support evidence                      | Which Linux, macOS, Windows, and FreeBSD configurations are actually supported, and who records the evidence for media presentation and AT adapters?                                                                                   | Core owner + terminal docs owner + `bitty-a11y`                     | ADR-0002 accepted tiers; media and AT evidence routes to `bitty-terminal-docs`; AT availability is separate from presentation class; `AI-0347/#553`; deferred.                                                                                                                                                                                                                         | `headed.platform`                                                                    |
| Budget defaults                                | What are the numeric CPU, memory, GPU, audio, token, and cost defaults, and who may change them? There is no separate AI budget; model and speech I/O use the shared per-client quota and Core ledger.                                 | Core owner + AI owner + Security owner                              | `OQ-066` for the context token profile; `AI-0347/#553` for media and host budgets; deferred                                                                                                                                                                                                                                                                                            | `headed.budget`, `headed.stream`, Event Bus queues                                   |
| Approval lifetime and standing grants          | How long does a one-shot approval live, and are session-wide or tool-wide grants allowed?                                                                                                                                              | Core owner + Security owner                                         | `AI-0347/#553` plus security review; OQ-018 is closed for consent scope, not approval lifetime; deferred                                                                                                                                                                                                                                                                               | `headed.tools`, `headed.session`                                                     |
| Synthetic speech disclosure (`D2`)             | What is the minimum disclosure for synthesized audio (spoken, visual, or metadata-only), and is provenance metadata required on export?                                                                                                | Security owner + Product owner                                      | Disclosure itself is required; the minimum form is `D2`, deferred to security/product review under `AI-0347/#553`.                                                                                                                                                                                                                                                                     | `headed.speech`                                                                      |
| Recording and export of speech or avatar state | Which recording, export, and delete rights does the v1 product offer, and which retention class does it name? The contract floor (off by default, consent, redaction, deletion) is fixed; the product surface is `D3`.                 | Core owner + Security owner + Product owner                         | `AI-0347/#553`; AIQ-55/5A adjacent closed(partial) facets; R3 stays draft; interface decision is `D3`, deferred.                                                                                                                                                                                                                                                                       | `headed.speech`, `headed.avatar`, retention                                          |
| Shared chat or transcript contract             | Do Wheel and the Headed Agent share a neutral presentation contract in Core or the plugin ecosystem, or only `bitty-ai` events plus Core panels? A `wheel.*` dependency is excluded.                                                   | Wheel owner + Plugin ecosystem owner                                | `OQ-056` (presentation projection); otherwise `AI-0347/#553`; deferred                                                                                                                                                                                                                                                                                                                 | UI composition, duplication risk                                                     |
| Accessibility baseline                         | Which remaining media, avatar, and caption-presentation details are v1 versus post-v1, and which caption commitments `D4` accepts?                                                                                                     | Accessibility owner + `bitty-a11y` adapter owner                    | ADR-0016 Boundary 3, `W-134`, RFC-0001, and RFC-0002 fix the accepted baseline (focus, announcements, reduced motion, appearance-pair contrast resolution); captions/transcript are candidate v1 only; `OQ-037` owns the content-contrast floor, the flash threshold has no owner yet, and `OQ-078` is not the only open accessibility question. `AI-0347/#553`; deferred.             | `headed.avatar`, `headed.speech`, `headed.platform`                                  |
| Bus topic taxonomy and cross-window transport  | Which topics exist in v1, do AI topics ride the Panel Event Bus at all, and how does an attachment reach a second window?                                                                                                              | terminal docs owner (`bitty-terminal-docs`)                         | `RFC-OQ-4` owns the exact v1 taxonomy and the cross-window decision; the accepted bus is single-process/single-window. This draft assumes no topic list and no cross-window path.                                                                                                                                                                                                      | `headed.panels`, `headed.stream`                                                     |
| Flash threshold ownership                      | What is the numeric flash-rate limit, how is it measured, and which contract owns it?                                                                                                                                                  | Accessibility owner + terminal docs owner                           | No accepted source defines it. The owning task must name the threshold, its measurement method, and its reviewer before any animated surface ships.                                                                                                                                                                                                                                    | `headed.avatar`, `headed.platform`                                                   |

The existing panel-identity and projection questions in the
[AI Unresolved Questions](../product/ai-unresolved-questions.md) register remain
open; this document creates no register identifier and closes none of them. The
register admits no task-level identifier minting, so an entry without a register
row carries an explicit deferral above and is promoted through AI-0347 rather
than asserted here as an authoritative decision.

### Review follow-ups

The following items need an owner decision or implementation evidence that this
docs-only correction cannot supply. They remain open and are not silently
closed.

#### Open product decisions (`D1`-`D7`)

These are product decisions, not contract decisions. The accepted contract
inputs they must respect are listed in
[settled contract inputs](#open-product-decisions-versus-settled-contract-inputs).

| ID   | Decision                                                                                                                                           | Owner                                       | Status                                                                                                                                                       |
| ---- | -------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `D1` | Product thesis and v1 scope: target user, minimum surface (text/speech/avatar), milestone, and relation to the Wheel chat UI and terminal panes.   | Product owner + `bitty-ai` owner            | Open. The candidate scope is recorded above; it is not accepted.                                                                                             |
| `D2` | Synthetic-speech disclosure minimum: spoken, visual, or metadata-only at playback, plus provenance metadata **only if** `D3` provides export.      | Security owner + Product owner              | Open. Disclosure is mandatory; the minimum form is not chosen, and export metadata is conditional on `D3`.                                                   |
| `D3` | Record/export/delete rights and retention class offered by the v1 product, aligned with the fixed contract floor and the export-preview gate.      | Core owner + Security owner + Product owner | Open. The floor (off by default, consent, redaction, deletion) is fixed.                                                                                     |
| `D4` | Accessibility floor beyond the accepted baseline: any additional v1 commitments and which are aspirational.                                        | Accessibility owner                         | Open. The accepted baseline (ADR-0016, `W-134`, RFC-0001, RFC-0002) is not reopened.                                                                         |
| `D5` | Identity and user-facing name only: `ai-panel` successor naming versus a new identity. Repository placement is not part of `D5`.                   | Product owner + `bitty-ai` owner            | Open for identity/name. Placement is fixed by the accepted AI Surface record (optional `bitty-ai` extension).                                                |
| `D6` | Consent surface: prompt placement/content, grant review and revocation UI, speech-versus-model egress distinction, and the consent-fatigue stance. | Product owner + Security owner              | Open. Authorization and revocation semantics are fixed; no UI is defined.                                                                                    |
| `D7` | Gate correction: add Product and Accessibility rows and update the count references.                                                               | Documentation curator + Product owner       | **Discharged by this revision**: nine rows; six design-review approvals and three `NEEDS-FIX` (Security `PX-1581`, Accessibility `PX-1587`, Core `PX-1584`). |

#### Open packaging decisions (`P-1`-`P-4`)

The earlier single "packaging" question conflated four decisions. They are
separated here; none is decided by this draft.

| ID    | Decision                                                                                                    | Owner                                           | Status and dependency                                                                                                                                                                                                                                                          |
| ----- | ----------------------------------------------------------------------------------------------------------- | ----------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `P-1` | Plugin identity: continue the recorded `ai-panel` surface under a successor name, or create a new identity. | `bitty-ai` product owner                        | Open. `ai-panel` was the recorded split-later hybrid; the 2026-09-30 removal superseded that posture.                                                                                                                                                                          |
| `P-2` | Package category: Lua package, out-of-process component, or a new media category.                           | `bitty-plugins-docs`                            | Open. The v1 Lua package is accepted; out-of-process is blocked on plugin IPC open items 2/4/7 and cross-process transport; no media category is accepted. **Accepts the `ai.*`/`agent.*`/`mcp.*` extension catalog grammar** before any AI capability identifier can be used. |
| `P-3` | Distribution presence: bundled, official package, or distribution preset.                                   | `bitty-ai` product owner + `bitty-plugins-docs` | Open. The accepted Default Distribution RFC owns the bundle/enabled-by-default model; this draft claims no bundled or enabled entry and adds no preset.                                                                                                                        |
| `P-4` | Signing and update: signature, provenance, registry, and update channel.                                    | `bitty-plugins-docs`                            | Open. Staged activation, integrity, rollback, resolver, registry, and key-management contracts are accepted; implementation is missing.                                                                                                                                        |

#### Other review follow-ups

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
- The caption timing tolerance, transcript bounds, and retention classes need
  named policy constants in an accepted speech contract; this draft fixes the
  obligations only.
- These tests cannot exist before implementation and are owned by future
  implementation tasks: the `wheel.*` CLI/config negative test, cross-principal
  attach/takeover, compromised-plugin speech authorization, rich-rendering
  adversarial corpus, accessibility verification table, and the nine P0 review
  records.

#### Remaining review follow-ups

These are non-blocking findings from the `01a1ad4` re-reviews. They are recorded
here so they stay tracked instead of being silently dropped; none is a blocker
for this correction pass.

- **Approval binding, partial.** A changed tool implementation version or digest
  is not approval-invalidating today, and there is no execution-time
  revalidation. The owning tools task must bind the implementation identity and
  add revalidation before `tool.started`.
- **Provider-credential consent.** `ai.provider` is now a named category, but the
  per-principal/per-provider/per-privacy-class rule and the standing-grant
  question still need the accepted consent contract; `D6` and the approval-lifetime
  open point own the split.
- **Viseme and avatar privacy.** Inheritance and buffer clearing are specified;
  a remote 3D renderer still lacks an explicit local-only-or-consent rule, and
  shader containment evidence cites the file loader rather than a shader
  containment control.
- **Per-use audit.** Speech has it; playback and recording start/stop still need
  per-use audit entries in the accepted media contract.
- **Threat and gate traceability.** The probe auth/rate/audit shape is partly
  fixed; the threat and gate tables still lack a full `T-01`..`T-14` /
  `R-001`..`R-022` mapping.
- **Idempotency edge.** Asset expiry and start idempotency are specified; a
  same-key/different-parameters retry is not yet explicitly fail-closed.
- **Open enforcement items.** Audio focus scope, budget atomicity/no-double-spend,
  and the clipboard/paste/notification controls (`P0-AC-007`/`008`) remain open
  with their owning tasks.
- **Panel Event Bus implementation.** The accepted contract is implemented only
  as experimental in-process queues; cross-process routing, the v1 taxonomy, and
  the capability ledger stay with `RFC-OQ-4`/`RFC-OQ-5`.
- **Audit of the review gate itself.** `PX-1569`..`PX-1587` are review
  identifiers, not approvals. Six rows record design-review approvals (`PX-1579`,
  `PX-1580`, `PX-1582`, `PX-1583`, `PX-1585`, `PX-1586`); none is an
  implementation approval.

## Acceptance criteria

This design is acceptable as a draft when all of the following hold:

- The document lives in the `agent/` tree, follows the specification spine, and
  carries the required flat frontmatter and status qualifiers.
- The boundary is explicit: Wheel 0.1.0 is an independent Coding Agent Harness
  and not a Bitty plugin; the Headed Agent is an optional `bitty-ai` extension
  and the successor candidate to the removed `ai-panel`, consuming Core and
  `bitty-ai`; no `wheel.*` dependency is defined; no headed UI, speech, avatar,
  or 3D work is assigned to Wheel.
- The product thesis, candidate v1 scope, `ai-panel` predecessor, and the open
  product decisions `D1` through `D7` are stated; open product decisions are
  separated from settled contract inputs.
- Each contract family has an owner, a candidate version, versioning rules, and
  an explicit statement that it is not accepted.
- The accepted IPC and Agent RFC, Panel Runtime RFC, RFC-0006, the scoped
  provider-routing binding, OQ-058 SMO-1 through SMO-4, OQ-083, ADR-0013,
  ADR-0019, ADR-0016 Boundaries 1 and 3, ADR-0002, `W-134`, RFC-0001, RFC-0002,
  the P0 criteria, and the Isolation and Resource RFC are consumed by reference
  and not restated or weakened.
- Session authority distinguishes the `bittyd` session, the AgentSession, and
  the application handle; it separates the application session generation from
  the `bittyd` worker-binding generation, cites `bitty#2546` and the `R-B09`
  sole-writer rule, and does not decide durable placement or worker scope in the
  body.
- Execution ownership follows ADR-0016 Boundary 1: Core authorizes and fences
  and retains PTY/permission/resource enforcement; `bitty-execution` (`W-132`)
  supervises and executes; no implementation is claimed.
- Session lifecycle, session/panel/attachment/AG-2 generation axes, the
  single-writer handoff, two-level session/job outcomes including the runtime
  `Canceled` mapping, cancellation, event ordering, backpressure, and reconnect
  are covered without redefining the accepted `AgentSession` or IPC RFC.
- Provider routing, context/session snapshots, and tool-call/approval events are
  covered without adding a provider loader, credential path, or second
  generation-job lifecycle. The routing publication/proof split and mutation
  invalidation are explicit, and aliases stay outside the strict path.
- Control and approval events are distinguished from Panel Event Bus
  observation topics and from the SMO/IPC event path; no approval, stop,
  tool-result, final-turn, or speech-job control event is defined as a
  `DropOldest` bus topic.
- Panel and Event Bus topics, panel-versus-attachment lifecycle (including
  `(PanelId, generation)` stale-handle rejection), multi-panel suspend/resume,
  and the Core-versus-plugin split are covered by consuming the accepted
  contracts without asserting implementation.
- Speech and avatar contracts cover request, stream, cancel, state, animation,
  visemes, candidate synchronized captions, and an inspectable transcript without
  implementing TTS, rendering, or 3D. `AssetRef` is labeled a proposed shared
  Core contract, and playback is an owner-pending media category.
- The accessibility baseline is explicit: roles/names/states, focus order,
  transition-boundary announcements, modal confinement, keyboard equivalence,
  reduced motion, appearance-pair contrast resolution, contrast floor deferred to
  `OQ-037`, candidate captions/transcript, avatar alternative content, media
  controls, platform AT ownership, absent-adapter behavior, and AT-action
  non-authority are covered with verification criteria.
- Plugin-platform contract ownership (`bitty-plugins-docs`) is separated from
  Headed Agent product ownership (`bitty-ai`); no plugin category, package
  class, or capability identifier is newly defined, and the AI capability
  catalog is recorded as owner-pending rather than accepted.
- Resource budgets, platform fallback, security, privacy, and consent are
  covered, with security gates and residual risks explicit; there is no separate
  AI budget, and every stored artifact names a retention class.
- The ownership matrix and cross-repository references are complete enough for
  the owning repositories to review their own rows.
- Open points are explicit, and none is silently decided across a public
  contract boundary. Each entry names a register, a tracked issue, a review
  route, or an explicit deferral; open product and packaging decisions are in
  the review follow-ups.
- The local documentation gate passes, and every row in
  [P0 Review Sign-off](#p0-review-sign-off) records a disposition before any
  implementation task is opened.

Acceptance of this draft does not accept any contract family for
implementation. Each family needs its own accepted specification and evidence.

## P0 Review Sign-off

This nine-row table is the **single authoritative sign-off set** for this
document. Earlier and later summaries that mention approval, review, or a gate
refer to these rows and do not define a smaller role list.

| Role                   | Reviewer | Status                            | Evidence                                                                                                                                                                                                                                                      |
| ---------------------- | -------- | --------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Architecture owner     | PX-1580  | Approved (design review, PX-1580) | Third-pass review approved the runtime state mapping, `bittyd` generation split, and OQ-058/SMO-versus-Panel-bus separation; the residual status-wording P2 is tracked.                                                                                       |
| Security reviewer      | PX-1581  | Pending — NEEDS-FIX (PX-1581)     | Third-pass review withdrew the earlier approval: AT activation could record a verdict without a trusted user-activation boundary. This pass makes AT activation non-approving and requires a Core-observed keyboard or pointer activation; re-review pending. |
| Core / `bittyd` owner  | PX-1584  | Pending — NEEDS-FIX (PX-1584)     | Third-pass review closed all but three text findings (present-tense `bittyd` status, restart durability, `ai.*` namespace claim); those are fixed here. Final sign-off remains conditional on accepted `bitty#2546`.                                          |
| `bitty-ai` owner       | PX-1582  | Approved (design review, PX-1582) | R3 owner, runtime states, routing-proof gate, ADR-0019 link, and `bitty-agent-api` exports verified.                                                                                                                                                          |
| Wheel owner            | PX-1585  | Approved (design review, PX-1585) | `.wheel/init.lua` trust-gate split, CLI negative tests, matrix horizons, provider/secret wording, and D1/D2/D6 attribution verified.                                                                                                                          |
| Plugin ecosystem owner | PX-1583  | Approved (design review, PX-1583) | `ai.*` catalog owner-pending/`P-2`, component examples, 2D-versus-GPU split, `RFC-OQ-4`/`RFC-OQ-5`, package/distribution status, and `ai-panel` supersession verified.                                                                                        |
| Product owner          | PX-1586  | Approved (design review, PX-1586) | `D5` scope, accepted placement, captions as candidate, `P-C`/`P-D`, predecessor mapping, terminal-pane relation, v1 speech versus post-v1 playback, and `D2`/`D3` verified.                                                                                   |
| Accessibility owner    | PX-1587  | Pending — NEEDS-FIX (PX-1587)     | Prior accessibility findings closed; the AT approval trust boundary remains the blocker. This pass makes AT activation non-approving; re-review pending.                                                                                                      |
| Documentation curator  | PX-1579  | Approved (design review, PX-1579) | Spine order, ADR-0019 link, draft-binding qualification, route/index, ADR-0016 attribution, PR-body counts, and terminology verified.                                                                                                                         |

Six rows record design-review approvals and three record `NEEDS-FIX`. The
approvals accept design content only; they accept no implementation, security
gate result, or release. Security, Accessibility, and Core remain pending:
Security and Accessibility await re-review of the AT trust boundary, and Core
awaits re-review plus accepted `bitty#2546`. The document remains `draft` until
all nine rows record dispositions and any defects are
resolved or tracked.

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
- [ADR-0016 Execution, Graphics, Accessibility, Storage, and Platform-Service Boundaries](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/adrs/ADR-0016-execution-graphics-accessibility-storage-platform-boundaries.md)
  — accepted (2026-10-02), Boundaries 1 and 3.
- [ADR-0019 bittyd Staging Advance to the v0.2.0 Slice](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/adrs/ADR-0019-bittyd-staging-advance.md)
  — accepted (2026-10-22).
- [bittyd session host requirements (candidate)](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/bittyd-session-host-requirements-candidate.md)
  — draft consumer requirements; `R-B09` sole-writer rule.
- [bitty#2546](https://github.com/bitty-terminal/bitty/issues/2546) — open
  `bittyd` session-host contract; no implementation claim.
- [Accessibility Extraction Contract](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/accessibility-extraction-contract.md)
  — accepted (`W-134`).
- [RFC-0001 Appearance Configuration](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/rfcs/RFC-0001-appearance-configuration.md)
  — accepted; appearance-pair contrast resolution. The content-contrast floor stays with `OQ-037`.
- [RFC-0002 Panel Animations](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/rfcs/RFC-0002-panel-animations.md)
  — accepted; `reduced_motion` `auto`/`always`/`never` and safe-mode contract.
- [OQ-078 in the open-question register](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/open-questions.md)
  — open on accessibility API/tree and v1 scope; not the only open accessibility item (`OQ-037`, flash threshold).
- [OQ-083 in the open-question register](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/open-questions.md)
  — accepted panel-lease direction.
- [Product Vision § Agent-friendly, not Agent-centric](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/product/vision.md#agent-friendly-not-agent-centric)
  — accepted product constraint.
- [AI Surface Accepted Record](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/ai-surface-accepted-record.md)
  — accepted; `ai-panel` removal and AI-surface movement.
- [Plugin Platform RFC](https://github.com/bitty-terminal/bitty-plugins-docs/blob/main/specifications/plugin-platform-rfc.md)
  — accepted; `R-017` native in-process prohibition.
- [Package Lifecycle RFC](https://github.com/bitty-terminal/bitty-plugins-docs/blob/main/packaging/package-lifecycle-rfc.md)
  and [Package Follow-up RFC](https://github.com/bitty-terminal/bitty-plugins-docs/blob/main/packaging/package-followup-rfc.md)
  — accepted activation/rollback and accepted resolver, registry, and key management; implementation missing.
- [P0 Security Acceptance Criteria](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/p0-acceptance-criteria.md)
  — normative.
- [Isolation and Resource RFC](https://github.com/bitty-terminal/bitty-plugins-docs/blob/main/runtime/isolation-resource-rfc.md)
  — accepted (2026-08-28).
- [Rich Presentation RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/rich-presentation-rfc.md)
  — accepted.
- [Default Distribution RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/default-distribution-rfc.md)
  — accepted; default bundle and enabled-by-default model.
- [Panel Runtime RFC open questions](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/panel-runtime-rfc.md#open-questions)
  — `RFC-OQ-4` (bus topic taxonomy, cross-window routing) and `RFC-OQ-5`
  (per-panel capability mapping) remain open.
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
