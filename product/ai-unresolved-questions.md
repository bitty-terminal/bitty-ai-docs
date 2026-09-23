---
title: AI Unresolved Questions
description: Local unresolved choices with feature prerequisites and proposed review routing
category: specifications
audience: mixed
document_type: register
status: draft
website_publish: false
sidebar_order: 22
---

# AI Unresolved Questions

## Purpose

This local draft preserves 53 identifiers, including aliases, not 53 independent
questions. It assigns no owners, release milestones or accepted global OQs.
Promotion requires the canonical [OQ admission rule](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/open-questions.md#use).
All non-alias choices remain open except AIQ-12 and AIQ-13 (Closed, adopted-draft) and the
AIQ-01 snapshot-stream, AIQ-11 L0/L1 enforcement, AIQ-03 store-expiry, AIQ-04 generation-pin, and AIQ-55
store-propagation facets (Closed(partial)); no accepted global decision is made here.

## Disposition

**Prerequisite** blocks enabling the named feature until its mechanism and
evidence satisfy the stated control. A reviewed safe profile may explicitly
exclude that feature; no such release profile is selected here. **Design** is a
choice for the named feature, not a general release blocker. **Scope** requires
a release/profile decision before deriving milestone gates. These replace the
unsupported global Yes/No classifications.

Routing names are proposed reviewer domains, not assigned owners: AI runtime,
terminal/IPC/panel, security, code intelligence, CarryCtx/lifecycle, plugin API,
and standalone AI product. The commander must identify accountable owners before
promotion. Required controls are not reopened: authentication, least privilege,
consent, isolation, budgets, truthful disclosure and secret minimization remain
mandatory. P0-AC-026 and AI PP-2/PP-4 require pre-queue/pre-write redaction and
consented recording; open mechanisms cannot defer those controls.

## Context management

Details: [context management](../context/context-management.md),
[prefix-cache context design](../context/prefix-cache-context-design.md).

| ID     | Open choice                                                                                                         | Blocking feature and rationale                                                                                                                   | Proposed routing               |
| ------ | ------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------ |
| AIQ-01 | Per-request versus incremental context generation — Closed(partial): snapshot-stream facet only; see disposition    | Design: view cadence and invalidation                                                                                                            | AI runtime                     |
| AIQ-02 | Compression backend selection                                                                                       | Design: routing within provider consent and budget                                                                                               | AI runtime, security           |
| AIQ-03 | Artifact expiry and reference invalidation — Closed(partial): store-expiry facet only; see disposition              | Prerequisite: retained artifacts must honor deletion and bounds                                                                                  | AI runtime, security           |
| AIQ-04 | Selection priority versus durable retention authority — Closed(partial): generation-pin facet only; see disposition | Prerequisite: pinning cannot override consent or expiry                                                                                          | AI runtime, security           |
| AIQ-05 | Background maintenance scheduling/consistency                                                                       | Design: preserve bounded responsive admission                                                                                                    | AI runtime                     |
| AIQ-06 | Re-expansion after projection compaction                                                                            | Design: only surviving authorized originals are recoverable; see AIQ-57                                                                          | AI runtime                     |
| AIQ-07 | Cross-session memory retrieval mechanism                                                                            | Prerequisite: consent, freshness and deletion propagation                                                                                        | AI runtime, security           |
| AIQ-08 | MCP schema cache invalidation                                                                                       | Prerequisite: stale schemas cannot authorize changed effects                                                                                     | AI runtime, security           |
| AIQ-09 | Skill format/versioning and ecosystem compatibility                                                                 | Design: loading declarations grants no execution authority                                                                                       | AI runtime, plugin API         |
| AIQ-10 | Task lifecycle authority and CarryCtx backend/handoff                                                               | Design: one lifecycle authority for integration; AIQ-56 is its persistence alias                                                                 | AI runtime, CarryCtx/lifecycle |
| AIQ-11 | Context injection-defense enforcement evidence — Closed(partial): L0/L1 facet only; see disposition                 | Prerequisite: untrusted observations cannot control maintenance policy; L2+ compression and retention facets stay open                           | AI runtime, security           |
| AIQ-12 | Canonical serialization and stable-prefix ordering — Closed (adopted-draft); see disposition                        | Design: deterministic prompt/1 encoding adopted for the prefix-cache prerequisite                                                                | AI runtime                     |
| AIQ-13 | Provider-scoped prefix-cache key and routing scope — Closed (adopted-draft); see disposition                        | Design: provider-scoped CacheKey/CacheScope keying plus measured hit-rate evidence; implicit-vs-explicit routing stays a follow-up policy choice | AI runtime, security           |

### AIQ-11, AIQ-12, and AIQ-13 dispositions (local draft only)

These dispositions close register facets with implementation evidence. They set
no owners or milestones, grant no global promotion, and use Closed and
Adopted-draft wording only.

- **AIQ-12 — Closed (adopted-draft).** Choice: deterministic `prompt/1`
  canonical encoding with five-layer stable order (Core-contract, User,
  Project, Skills-profile, Runtime-turn), length-prefixed sections, sorted
  policy lists, and a trailing `[effective]` block keeping leading bytes
  prefix-stable. Evidence: [Prompt Layering Design](../context/prompt-layering-design.md)
  assembly order (stable-before-dynamic) and [Prefix-Cache-Friendly Context
  Design](../context/prefix-cache-context-design.md) prerequisite; code
  `bitty-ai/crates/bitty-ai-runtime/src/prompt.rs` (`assemble`,
  `render_canonical`); 22 prompt tests covering stable order, byte-exact
  canonical form, sorted LF-only lists, trailing-change prefix stability, and
  prompt-never-grants narrowing; merged in `bitty-ai` `12312ac` (AI-0029).
  Cache-key scope (AIQ-13) and measured hit-rate claims stay out of scope.
- **AIQ-11 — Closed(partial): L0/L1 enforcement facet closed; compression and
  retention facets stay open.** Closed choice: L0 structured-output plus L1
  lossless-pruning enforcement under untrusted observations (trusted-only
  supersede links, full-body `(provider, owner, summary, body)` dedupe with
  deny-by-default survivor, untrusted priority clamp to at most Normal,
  byte-length-only externalization and truncation accounting, no content
  interpretation). Evidence: [Context Management
  Architecture](../context/context-management.md) Level 0, Level 1, and security-boundary
  sections; code `bitty-ai/crates/bitty-ai-runtime/src/context.rs`
  (`assemble`, `effective_priority`); 9 injection-defense negative tests (16
  tests total in `context.rs`) pairing injection variants with same-shape
  benign controls; merged in `bitty-ai` `e1cfdd9` (AI-0028). Stay-open facets
  with reasons: selective compression L2 and above (no summarization backend
  or range-compression mechanism implemented) and durable retention policy (no
  durable store, GC, or cross-session deletion-propagation mechanism
  evidenced); any later pruning-scope gap needs a separate code task.
- **AIQ-13 — Closed (adopted-draft).** Choice: provider-scoped prefix-cache
  key (`CacheKey` pins `provider_id`, `model_id`, `scope`,
  `stable_prefix_hash`, `prefix_len`; equality and hashing cover every
  field so same bytes under a different route compare unequal) with
  three-variant routing scope (`CacheScope`: `Session` reusable across turns
  of one session, `Turn` one turn only, `Round` one provider round only; a
  `Session` key never equals a `Turn` key over the same bytes) and a pinned
  7/10 (70%) repeat-head hit-rate measurement. Evidence: [Prefix-Cache-Friendly
  Context Design](../context/prefix-cache-context-design.md) prerequisite; code
  `bitty-ai/crates/bitty-ai-runtime/src/cache_key.rs` (`CacheKey`,
  `CacheScope`, `CacheKeyError`, `stable_prefix_len` walking length-prefixed
  sections, `fnv1a64`); 8 `cache_key.rs` tests covering the
  provider/model/scope/stable-region inequality matrix, Session/Turn scope
  separation, deterministic rebuild with pinned digest, trailing-only-change
  prefix stability, stable-region-change key break, the 10-round repeat-head
  harness (7 hits / 10 rounds), malformed-input fail-closed construction, and
  the embedded-marker alias proof; merged in `bitty-ai` `fdb37c5` (AI-0082,
  key mechanism plus 7-test harness) and `2b984c4` (AI-0084, length-aware
  boundary plus alias-proof test). Follow-up pointer, not an open facet: the
  implicit-versus-explicit routing half narrows to a pure policy choice
  operating inside non-leaking keys.

### AIQ-01 disposition (local draft only)

This disposition closes a register facet with implementation evidence. It sets
no owners or milestones, grants no global promotion, and uses Closed(partial)
wording only.

- **AIQ-01 — Closed(partial): snapshot-stream facet closed; full
  incrementality and background-maintenance facets stay open.** Closed choice:
  session-pinned snapshot plus per-turn deltas with explicit host-authorized
  refresh rotating the generation pin — the stable snapshot head warms the
  `Session` prefix-cache key while only the turn tail varies, and a refresh
  misses the retired key by construction. Evidence: code
  `bitty-ai-slice/src/snapshot_ingest.rs` (`ingest_snapshot` digest-verified
  ingestion, `RefreshAuthorization::authorize` per-call token,
  `RefreshLedger::issue` strictly increasing issuance with `retired()` audit
  chain and `RefreshError::NotAdvancing` fail-closed replay denial,
  `project_layer_text` marker-prefixed PROJECT rendering embedding the full
  digest, `prompt_snapshot_with_project` binding the record summary to the
  digest); tests `snapshot_delta.rs`
  (`session_pinned_snapshot_with_per_turn_delta`: gen-1 pin, gen-1 delta
  alongside, gen-2 refresh with `StaleGeneration` denial of the stale pin),
  `session_wiring.rs` (`full_session_lifecycle_composes`: ledger issuance,
  tail-only key warmth, refresh miss, retired-generation re-issue denial,
  invalidation fail-closed), `cache_affinity.rs` (`same_digest_warms_same_key`,
  `changed_digest_misses_key`, `project_text_is_stable_prefixed_and_bounded`),
  `hit_rate.rs` (`stable_head_with_varying_tail_hits_consecutively`,
  `refreshed_snapshot_misses_then_rewarms`); runtime `StaleGeneration` (AG-2)
  fail-closed assembly gate underpinning rotation; merged in `bitty-ai`
  `e3bcfe2` (AI-0123, delta cycle), `8427008` (AI-0126, ledger), `f70d3ac`
  (AI-0127, affinity), `7de59d9` (AI-0128, project-layer builder), `a809896`
  (AI-0129, hit-rate), `3858700` (AI-0130, session wiring). Stay-open facets
  with reasons: full incremental view update (each turn reassembles from the
  pinned snapshot plus a recollected delta; no cached view is mutated in
  place and no diff-application mechanism is evidenced) and background
  maintenance scheduling and consistency (ingest keeps no cache and schedules
  no background refresh; the ledger is pure generation arithmetic with no
  bytes, clock, or I/O; refresh timing stays a host decision — overlapping
  AIQ-05, which stays open).

This describes sibling behavior only as read; this repository was not modified
as part of those inspections beyond this register.

### AIQ-03, AIQ-04, and AIQ-55 dispositions (local draft only)

These dispositions close register facets with implementation evidence. They set
no owners or milestones, grant no global promotion, and use Closed(partial)
wording only. Host-side enforcement stays open per the AIQ-11 precedent: the
store enforces removal and expiry, the host decides what to delete and when.

- **AIQ-03 — Closed(partial): store-expiry facet closed; host deletion-timing
  and durable facets stay open.** Closed choice: generation-exact artifact
  expiry plus explicit host-authorized invalidation with typed fail-closed
  resolution — out-of-generation references fail with `ArtifactExpired`,
  dangling or invalidated references fail with `ArtifactUnavailable`, never
  silent substitution or stale bytes. Evidence: code
  `bitty-ai/crates/bitty-ai-runtime/src/context.rs`
  (`ArtifactStore::store(bytes, generation)`,
  `resolve(reference, current_generation)` exact-generation gate, `invalidate`
  dropping bytes and freeing budget, `ArtifactExpired` carrying the reference
  only); 3 store tests (`artifact_expiry_is_generation_exact`,
  `invalidate_drops_bytes_and_frees_budget`,
  `assemble_committed_artifacts_pin_request_generation`) covering
  same-generation resolve, older/future/forged generation denial, idempotent
  double-invalidate, and budget accounting; merged in `bitty-ai` `e4ad1d5`
  (AI-0124). Stay-open facets with reasons: host deletion timing and consent
  (the host decides what to invalidate and when; the store only enforces
  removal), durable GC and retention bounds, and cross-session deletion
  propagation (no durable store, GC, or cross-session mechanism evidenced).
- **AIQ-04 — Closed(partial): generation-pin facet closed;
  durable-retention-authority facet stays open.** Closed choice: every artifact
  pins to its host-supplied generation and retires when the record pin rotates
  (AG-2), so selection priority and `pinned` markers cannot override expiry;
  `pinned` and protection markers cannot override consent, redaction, expiry,
  deletion, or resource ceilings. Evidence: [Context Management
  Architecture](../context/context-management.md) retention-policy consequence;
  code `bitty-ai/crates/bitty-ai-runtime/src/agent.rs` (tool-history artifacts
  stored with `session.generation()`), `context.rs` `assemble` (committed
  artifacts stored with `request.current_generation` after the StaleGeneration
  gate), `bitty-ai-slice/src/snapshot_ingest.rs` (snapshot payloads stored
  with `refresh.generation()`); exact-generation `resolve` gate above; merged
  in `bitty-ai` `e4ad1d5` (AI-0124). Stay-open facets with reasons: durable
  retention-policy authority and host limits constraining user/tool preferences
  (no durable store or GC evidenced); Lua proposes selection policy only, host
  enforcement remains owing.
- **AIQ-55 — Closed(partial): store-propagation facet closed; cross-store and
  host-clone facets stay open.** Closed choice: invalidation retires the bytes
  and every later `resolve` of a derived holder fails closed with typed
  absence; summaries stay inline inert text (digest prefix at most, never
  payload), so no payload survives through them. Evidence: code
  `bitty-ai/crates/bitty-ai-runtime/src/context.rs` `invalidate`
  deletion-propagation contract (host MUST drop cached `AssembledContext`
  values pinning the invalidated generation); test
  `derived_records_fail_closed_after_invalidation` (pre-invalidation resolve,
  post-invalidation `ArtifactUnavailable`, no stale bytes); merged in
  `bitty-ai` `543e7d2` (AI-0125) on top of `e4ad1d5` (AI-0124). Stay-open
  facets with reasons: host-held clones, caches, and indexes (the store cannot
  reach into host-held copies; dropping them is an unenforced host obligation),
  and consistent cross-store removal of caches and indexes (no cross-store
  mechanism evidenced).

These describe sibling behavior only as read; this repository was not modified
as part of those inspections beyond this register.

## Commands and tools

Details: [command/tool architecture](../architecture/command-tool-architecture.md).

| ID     | Open choice                                                  | Blocking feature and rationale                                                | Proposed routing                   |
| ------ | ------------------------------------------------------------ | ----------------------------------------------------------------------------- | ---------------------------------- |
| AIQ-31 | Trait/lint/review enforcement of Core/Lua split              | Design: AI mechanism ownership preserves terminal boundary                    | AI runtime, plugin API             |
| AIQ-32 | Workflow-to-AI-Core promotion review                         | Design: performance never permits terminal AI embedding                       | AI runtime, security               |
| AIQ-33 | Unified authorization/isolation backend                      | Prerequisite: every native/MCP effect needs target, scope, consent and budget | AI runtime, terminal/IPC, security |
| AIQ-34 | Command registration API and versioning                      | Design: compose with accepted plugin API                                      | AI runtime, plugin API             |
| AIQ-35 | Git primitives versus high-level wrappers                    | Design: structured API or bounded authorized execution                        | AI runtime                         |
| AIQ-36 | Native versus MCP tool transport and bridge placement        | Prerequisite: resolve conflicting drafts without direct-spool bypass          | AI runtime, terminal/IPC, security |
| AIQ-37 | Structured exec result schema                                | Prerequisite: disclose failures, truncation and Unknown outcomes              | AI runtime, terminal/IPC           |
| AIQ-38 | Generic execution and registry ownership across repositories | Prerequisite: preserve BA-2/BA-3, no model I/O in bitty-agent                 | AI runtime, terminal/IPC, security |

## Agent coordination

Details: [agent coordination](../agent/agent-coordination.md).

| ID     | Open choice                                       | Blocking feature and rationale                                                        | Proposed routing                                |
| ------ | ------------------------------------------------- | ------------------------------------------------------------------------------------- | ----------------------------------------------- |
| AIQ-21 | Service compatibility-key validation/invalidation | Prerequisite: incompatible targets/overlays must not share state                      | AI runtime, code intelligence                   |
| AIQ-22 | Cross-scope service non-disclosure mechanism      | Prerequisite: prove isolation or exclude sharing; filtering alone is not proof        | code intelligence, security                     |
| AIQ-23 | Lease heartbeat and crash reconciliation          | Prerequisite: bounded supervised ownership cannot rely on destructors                 | AI runtime, terminal/IPC                        |
| AIQ-24 | Atomic ancestor/global budget reservation         | Prerequisite: concurrent delegation cannot overspend or double-spend                  | AI runtime, security                            |
| AIQ-25 | Measured depth/fan-out limits                     | Prerequisite: bounded delegation admission                                            | AI runtime, security                            |
| AIQ-26 | Independent review evidence criteria              | Prerequisite: acceptance cannot derive from self-review/shared PASS                   | AI runtime, CarryCtx/lifecycle                  |
| AIQ-27 | Context graph traversal/cycle mechanism           | Prerequisite: bounded retrieval under adversarial references                          | AI runtime, security                            |
| AIQ-28 | Critical-message acknowledgement and recovery     | Prerequisite: assignment/approval/cancel cannot silently drop or imply effect success | AI runtime, CarryCtx/lifecycle                  |
| AIQ-29 | Optional Panel/execution projection bindings      | Design: presentation movement cannot move execution targets                           | AI runtime, terminal/panel                      |
| AIQ-2A | No-UI execution feature profile                   | Scope: bounded work versus persistent services needs explicit selection               | AI runtime, terminal/IPC, standalone AI product |
| AIQ-2B | Supervisor crash recovery/adoption                | Prerequisite: never adopt arbitrary survivors or repeat Unknown effects               | AI runtime, terminal/IPC, security              |
| AIQ-2C | Interactive writer fencing                        | Prerequisite: takeover/restart must invalidate stale writers before new input         | AI runtime, terminal/IPC, security              |

## Code intelligence

Details: [code intelligence](../agent/code-intelligence.md).

| ID     | Open choice                                          | Blocking feature and rationale                                             | Proposed routing              |
| ------ | ---------------------------------------------------- | -------------------------------------------------------------------------- | ----------------------------- |
| AIQ-41 | Document overlay coordination                        | Prerequisite: conflicting buffers cannot silently share semantic state     | code intelligence             |
| AIQ-42 | Alias of AIQ-22: privileged-server filtering         | Same prerequisite as AIQ-22; retained identifier, no independent closure   | code intelligence, security   |
| AIQ-43 | Incomplete fingerprint handling                      | Prerequisite: disable generic reuse/coalescing when equivalence is unknown | code intelligence, security   |
| AIQ-44 | Cache invalidation granularity                       | Prerequisite: stale inputs cannot produce a falsely current PASS           | code intelligence             |
| AIQ-45 | Effectful coalescing equivalence/isolation mechanism | Prerequisite: every waiter has its own grant; otherwise disable coalescing | code intelligence, security   |
| AIQ-46 | Syntax fallback disclosure format                    | Prerequisite: fallback must be distinguishable from semantic evidence      | code intelligence             |
| AIQ-47 | Diagnostic rate limits and prioritization            | Prerequisite: bounded attributed subscriptions                             | code intelligence, security   |
| AIQ-48 | Warm-service/restart policy                          | Design: bounded supervisor policy within required isolation limits         | code intelligence, AI runtime |

## Persistence and evidence

Details: [persistence/evidence](../persistence/persistence-evidence.md).

| ID     | Open choice                                                                                                      | Blocking feature and rationale                                                              | Proposed routing                            |
| ------ | ---------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------- | ------------------------------------------- |
| AIQ-51 | Schema and transaction boundaries                                                                                | Design: select representation for chosen durable feature profile                            | AI runtime                                  |
| AIQ-52 | State reconstruction versus effect re-execution contract                                                         | Prerequisite: replay must not silently rerun effects                                        | AI runtime, security                        |
| AIQ-53 | Backend and optional search index                                                                                | Design: FTS5 is not inherent to event storage/replay                                        | AI runtime                                  |
| AIQ-54 | Cross-store retention policy authority                                                                           | Prerequisite: host limits constrain user/tool preferences                                   | AI runtime, security                        |
| AIQ-55 | Deletion/expiry and derived-record invalidation — Closed(partial): store-propagation facet only; see disposition | Prerequisite: remove payloads, summaries, caches and indexes consistently                   | AI runtime, security                        |
| AIQ-56 | Alias of AIQ-10: CarryCtx persistence integration                                                                | Same design classification as AIQ-10; backend/handoff facet, not separate lifecycle owner   | AI runtime, CarryCtx/lifecycle              |
| AIQ-57 | Reconstruction after deletion, expiry or destructive journal reduction                                           | Prerequisite: disclose missing evidence; projection-only compaction need not lose originals | AI runtime, security                        |
| AIQ-58 | Per-reader evidence sharing enforcement                                                                          | Prerequisite: cache references cannot leak broader authority                                | AI runtime, security                        |
| AIQ-59 | Unknown effect reconciliation and retry eligibility                                                              | Prerequisite: event log alone grants neither exactly-once nor safe retry                    | AI runtime, terminal/IPC, security          |
| AIQ-5A | Typed redaction markers and invalidation mechanism                                                               | Prerequisite: implement mandatory pre-queue/pre-write redaction, not choose its timing      | AI runtime, security                        |
| AIQ-5B | Bounded authorized observability queries                                                                         | Design: query needs and performance evidence; optional FTS                                  | AI runtime                                  |
| AIQ-5C | Standalone AI persistence/release profile                                                                        | Scope: neither ephemeral v0.1 nor post-1.0 deferral is decided                              | standalone AI product, AI runtime, security |

AIQ-10/56 and AIQ-22/42 are stable aliases, not removed or renumbered IDs.
Any promotion must reconcile all references and retain the alias mapping.
Other overlapping topics (for example context priority and cross-store retention)
retain their distinct facets; this register claims no count of independent OQs.
