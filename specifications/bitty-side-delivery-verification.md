---
title: Bitty-side delivery verification
description: Draft read-only verification mapping bitty main deliveries to BII inputs without acceptance decision
category: specifications
audience: contributor
document_type: research
status: draft
website_publish: false
sidebar_order: 42
---

# Bitty-side delivery verification

> Status: **draft**. This note records read-only verification of `bitty`
> `main` against the [Bitty-Side Integration Input](bitty-side-integration-input.md)
> (BII-01 through BII-10 with research 023 G-2 and G-3). It describes what
> was delivered and what remains missing. It grants no acceptance, sets no
> `bitty`-side priority, closes no open question, and changes the status of
> no AIQ entry. Acceptance, sequencing, and mechanism decisions stay with
> the `bitty` repository through its own review. No file in the `bitty`
> repository was read for writing or modified for this note; no file in the
> `bitty-ai` implementation repository was modified either.

## Scope and method

Inspection point is `bitty` `main` at `eef983e` (read-only), covering six
landed pull requests in merge order:

- `327064f` docs reconciliation (#702).
- `9bbc1a6` bounded `terminal.snapshot` service (#703).
- `50b32ec` host tool dispatch with per-tool consent (#705).
- `cf3ac8b` generic execution backend with structured outcome (#707).
- `a61c291` publishable bridge client boundary (#709).
- `eef983e` bounded scene-fragment ingestion transport (#711).

Method per item was first-hand source read (`outline`, then targeted
`symbol` or narrow `read`), integration and unit test enumeration, and a
read-only comparison against the `bitty-ai` slice consumer
(`bitty-ai/crates/bitty-ai-slice`, pinned `bitty-ipc` git revision
`3c9cfea6f9481aac32cbf5b657043f94d3fe273b`). Line numbers below refer to
`bitty` `main` at the inspection point. Test counts count `#[test]`
attributes in the cited files.

## BII-01 Bounded terminal snapshot host service

Request: host-registered, zone-scoped, bounded `terminal.snapshot` handler
behind the existing `terminal.inspect` scope, returning a bounded snapshot
data object rather than internal grid objects.

Delivered by #703 (`9bbc1a6`): `crates/bitty-ipc/src/snapshot.rs` (710
lines) plus `crates/bitty-ipc/tests/snapshot_service.rs` (244 lines).
Evidence bar: `SNAPSHOT_METHOD` (`snapshot.rs:64`), `DetailLevel`
(`snapshot.rs:93`), `SnapshotRequest` (`snapshot.rs:235`),
`SnapshotData` (`snapshot.rs:317`), `TerminalSnapshot`
(`snapshot.rs:338`), `SnapshotService` (`snapshot.rs:470`), `dispatch`
(`snapshot.rs:548`). Scope mapping reuses the generic registry
(`terminal.snapshot` under `terminal.inspect` in `scope.rs:383`). Tests: 8
unit plus 10 integration, including missing-handler fail-closed
(`host_without_snapshot_handler_fails_closed`), missing-scope denial,
unknown-method denial, terminal mismatch denial, bounded budgets, zone
narrowing, char-boundary truncation, and untrusted-surface labeling.

Gap: the service is pure data, bounded, and headless with a caller-supplied
provider function; there is no live terminal binding, no live
generation-change read test, and no redaction implementation in the
inspected module. Deterministic snapshots under live generation change and
redaction remain sequel host wiring.

## BII-02 Generic host tool dispatch method

Request: host-registered generic tool-dispatch method with per-tool consent
and bounded results, usable by any agent or plugin caller through
authenticated IPC.

Delivered by #705 (`50b32ec`): `crates/bitty-ipc/src/tool_dispatch.rs`
(894 lines) plus `crates/bitty-ipc/tests/tool_dispatch_service.rs` (350
lines). Evidence bar: `ToolSpec` (`tool_dispatch.rs:197`), `ToolRequest`
(`tool_dispatch.rs:271`), `ToolOutput` with `ToolExecution`
(`tool_dispatch.rs:346`, `tool_dispatch.rs:393`), `ToolDispatchService`
(`tool_dispatch.rs:491`), `dispatch` (`tool_dispatch.rs:564`). Dispatch
denies unknown tools, missing scopes, missing or expired consent, and
effect tools without explicit `allow_effects`, and bounds names, schemas,
arguments, results, targets, and client identity. Tests: 9 unit plus 11
integration, including unknown-tool, missing-scope, missing-consent, and
missing-handler denials, effect opt-in paths, oversized argument and result
denials, target mismatch denial, and shared per-client scope consent
(`consent_is_shared_per_client_scope_not_per_tool`).

Gap: providers are caller-supplied test doubles; no real capability backend
(filesystem, process, terminal, view, configuration, plugin effects) is
wired, and no MCP path is proven. Live tool effects remain sequel work.

## BII-03 Authorization gate before every real effect

Request: every real tool effect passes one common host effect gateway in a
fixed order (schema validation, captured target and generation resolution,
scope check, consent ledger check, effect policy, budget reservation,
attributed outcome).

Partially delivered by #705 (`50b32ec`) as a fixed dispatch prefix:
request validation, client identity bounds, registry lookup, scope check,
explicit effect opt-in, consent ledger check, provider call, captured
target match, output validation, and attributed outcome
(`tool_dispatch.rs:564`). Tests prove each denial class leaves no partial
state.

Gap: the inspected order has no schema-version check, no captured
generation resolution, no effect policy beyond the `allow_effects`
boolean, and no budget reservation or accounting. The consent type
(`ConsentLedger`) is reused across modules, but a single shared ledger and
a single shared budget path across native and MCP paths with a shared
gate-order proof are not shown. Effectful tools therefore remain gated by
shape, scope, opt-in, and consent only.

## BII-04 Generic supervised execution backend

Request: generic `bitty`-owned supervised execution primitive behind an
`ExecutionRequest` shape that owns process and PTY handles, generations,
isolation, and cleanup, with optional Panel projection attached later.

Delivered by #707 (`cf3ac8b`) as shapes plus a headless service:
`crates/bitty-ipc/src/execution.rs` (1531 lines) plus
`crates/bitty-ipc/tests/execution_service.rs` (343 lines). Evidence bar:
`ExecutionRequest` (`execution.rs:444`), `EnvPolicy`
(`execution.rs:359`), `ExecutionService` (`execution.rs:980`),
`dispatch` (`execution.rs:1047`), `EXECUTION_SCOPE` as `process.spawn`
(`execution.rs:155`). The request carries executable, arguments, working
directory, environment policy, captured target, timeout, output budget,
and explicit effect opt-in. Bounds reuse accepted contracts and the
environment policy is closed (`Isolated` or `Explicit` with no ambient
inheritance). Tests: 13 unit plus 11 integration, including scope,
consent, opt-in, budget, target mismatch, and closed environment proofs.

Gap: the module header states it owns no socket, spawns no process,
performs no I/O, and depends on no workspace crate beyond `bitty-ipc`
itself. There is no PTY or process handle ownership, no isolation or
cleanup wiring, no executable allowlist enforcement, and no Panel
projection wiring. Headless execution without Panel or shell is the
default provider shape only.

## BII-05 Structured execution outcome with Unknown reconciliation

Request: host execution backend returns the full structured outcome
disclosure classes (`Succeeded`, `Failed`, `Cancelled`, `Unknown`) with
bounded redacted evidence references, reconciled by status inspection or
user direction before retry.

Delivered by #707 (`cf3ac8b`) as outcome types plus a query path:
`ExecutionStatus` (`execution.rs:161`), `EffectState`
(`execution.rs:220`), `RawExecutionOutput` with `Unknown` agreement
validation, `ExecutionResult` (`execution.rs:781`), and stored-outcome
`reconcile` plus `resolve` on `ExecutionService`. The `Unknown` agreement
(status and effect state agree on `Unknown`, and `Unknown` carries no exit
code) is enforced fail-closed, same-id re-dispatch is rejected, and there
is deliberately no retry primitive. Tests: `unknown_agreement_is_enforced`
(unit) and `unknown_reconciles_through_the_query_path_never_blind_retries`
plus `resolve_preserves_attribution` (integration).

Gap: cancellation races on both sides of live dispatch and post-dispatch
`Unknown` outcomes against real processes are not proven; reconciliation
is a stored-outcome query over headless doubles. No rollback is claimed,
which matches the BII request.

## BII-06 Versioned bridge client SDK for out-of-process consumers

Request: versioned, out-of-process bridge client SDK exporting the
accepted envelope, method registry, scope, and consent types, replacing
pinned Git revision plus dependency-exception consumption for stable
releases.

Delivered by #709 (`a61c291`) as a publishable boundary:
`crates/bitty-ipc/Cargo.toml` sets `publish = true` (line 11) and
`crates/bitty-ipc/src/bridge.rs` (401 lines) plus
`crates/bitty-ipc/tests/bridge_client.rs` (165 lines) add `BridgeClient`
(`bridge.rs:98`) with `call` (`bridge.rs:193`), `take_request`,
`answer`, `take_response`, and expiry draining. The client composes the
client-side dispatch prefix (routing, authorization, consent, budget,
envelope, attribution, outcome) with bounded params
(`MAX_BRIDGE_PARAMS_BYTES`, `bridge.rs:81`) and bounded client identity.
Tests: 9 unit plus 10 integration, including unknown-method, scope,
consent, expiry, params-bound, correlation, and no-partial-state proofs.

Gap: the crate version follows the workspace version rather than an
independent SDK versioning proof, and no external consumer build without
an internal-crate Git dependency is shown in the inspected repositories.
Read-only comparison shows `bitty-ai` still pins `bitty-ipc` by Git revision
(`bitty-ai-slice/Cargo.toml:18`, rev `3c9cfea`) with the prior
consumption shape. Stable-release substitution therefore remains sequel
consumer work.

## BII-07 Bounded rich scene-fragment transport

Request: bounded scene-fragment ingestion method with a consumable
fragment contract (for example Markdown, diff, and tool-card fragments).

Delivered by #711 (`eef983e`) as a text-chunks-first transport:
`crates/bitty-ipc/src/rich_fragment.rs` (494 lines) with `FragmentData`
(`rich_fragment.rs:95`), `RichFragment` (`rich_fragment.rs:116`), and
`FragmentIngestService` (`rich_fragment.rs:205`) with `ingest`
(`rich_fragment.rs:252`) and `drain_bounded`. The dedup key is
(`terminal_id`, `generation`, `seq`), text is bounded per fragment at the
grid-text bound with char-boundary truncation, queue depth is bounded, and
duplicates are rejected before capacity. Fragments reuse the snapshot zone
vocabulary and are always labeled untrusted. Tests: 11 unit tests with no
separate integration file, covering budgets, truncation, NUL rejection,
grammar rejection, duplicate-before-capacity, capacity, generation-scoped
sequence sharing, trust labeling, over-ceiling DTO rejection, and FIFO
drain.

Gap: the contract carries text plus zone only; there is no Markdown, diff,
or tool-card typed fragment. No wire method is registered (`scope.rs`
unchanged), and live render or projection wiring to scene blocks is
explicitly sequel work. Authorization, consent, and provider-echo checks
for a future serving method are absent by design.

## Panel capability decoupling for AI-specific surface

Request BII-08 asks review of the existing AI-named surface in the
terminal runtime Panel code against a generic panel provider service
capability, demoting AI-specific core APIs to plugin or SDK primitives.

Delivered by #702 (`327064f`) as a candidate design record only:
`specifications/ai-surface-reconciliation.md` (277 lines) inventories the
`ai_panel.rs` surface, maps 17 items to Keep or Demote verdicts with
specified follow-up changes D-1 through D-6 and F-1 through F-4, and
records the shipped, accepted, draft, open, and candidate ledger. The
record explicitly authorizes no implementation and edits no Core API.

Gap: the task is docs-only, so every AI-specific Core API remains in
place and the requested negative evidence (no core AI-specific API beyond
generic primitives on the reviewed path) is not yet available. Demotion
work is specified but not executed.

## BII-09 Parallel work split across the two tracks

BII-09 records the proposed division of labor: the `bitty-ai` track
completes the agent kernel against deterministic doubles while the
`bitty` track completes the host capability gateway (BII-01 through
BII-05 first), meeting at the bridge when the gateway exists.

Observed: the `bitty` side landed the gateway shapes in the proposed
grouping (snapshot, then dispatch with consent, then execution with
structured outcomes), followed by the bridge boundary and the fragment
transport, with the Panel review as a docs-only record. The `bitty-ai`
slice comparison still runs against deterministic doubles through the real
`bitty-ipc` bridge with `terminal.snapshot` fail-closed coverage. This
note makes no claim about track completeness on either side.

## BII-10 Suggested build order for the bitty track

BII-10 proposes the `bitty`-track order: bounded `terminal.snapshot`
(BII-01), generic dispatch with authorization and consent (BII-02 with
BII-03), execution backend with structured outcomes (BII-04 with BII-05),
bridge client SDK (BII-06), fragment ingestion (BII-07), Panel cleanup
(BII-08).

Observed: `bitty` `main` follows that group order for code items
(#703, #705, #707, #709, #711), with the Panel reconciliation record
(#702) landed first as docs-only input rather than last as code cleanup.
Each code group landed with its linked evidence bar of bounded validation
and fail-closed tests. Whether the order satisfies the `bitty`
repository is for that repository to decide.

## Research 023 G-2 and G-3 confirmation

Research 023 G-2 (`terminal.snapshot` host service) maps to BII-01 and is
addressed in shape by #703 as a bounded service under `terminal.inspect`.
Research 023 G-3 (generic Tool Bus dispatch) maps to BII-02 with BII-03
and is addressed in shape by #705 as generic dispatch with per-tool
consent. The remaining 023 gaps stay where the BII input puts them:
packaging (G-1) is addressed in shape by the publishable bridge boundary
(#709) without consumer substitution proof, transport (G-4) is addressed
in shape by text-first fragments (#711) without typed fragments or render
wiring, and design reconciliation items stay candidate docs (#702).

## Explicit non-requests and open-question disposition

This verification changes the status of no register entry. AIQ-33,
AIQ-36, AIQ-37, AIQ-38, AIQ-08, AIQ-29, AIQ-2A, AIQ-10 with the AIQ-56
alias, AIQ-26, and AIQ-28 stay open under the canonical admission rule
cited by [AI Unresolved Questions](ai-unresolved-questions.md). The
[Execution ownership R1](execution-ownership-r1.md) registry-split
framing, the [Tool transport R2](tool-transport-r2.md) open surface, and
the [Task lifecycle R5](task-lifecycle-r5.md) lifecycle boundary are
unchanged; this note is a versioned observation input, not a lifecycle
transition in either repository.

## Conclusion: what was delivered and what remains missing

Delivered in shape on `bitty` `main`: bounded snapshot, generic dispatch
with per-tool consent, execution shapes with a structured `Unknown`
query path, a publishable bridge client, text-first fragment ingestion,
and a candidate Panel reconciliation record, each with deterministic
bounded tests and fail-closed denials.

Still missing for a live-host claim: live terminal and process or PTY
wiring, real capability backends, unified gate order with generation,
schema, policy, and budget accounting, live cancellation and
acknowledgement-loss proofs, consumer substitution off the pinned
revision, typed rich fragments with render wiring, and executed Panel
demotion with negative-evidence code proof. No acceptance decision for
the `bitty` repository is made here; follow-up scope belongs to the
commander and the owning repositories.

## References

- [Bitty-Side Integration Input](bitty-side-integration-input.md) (Draft):
  BII-01 through BII-10 with G-2 and G-3 confirmation.
- [Execution ownership R1](execution-ownership-r1.md) (Draft): execution
  ownership and registry-split framing.
- [Tool transport R2](tool-transport-r2.md) (Draft): unified backend and
  transport open surface.
- [Task lifecycle R5](task-lifecycle-r5.md) (Draft): lifecycle authority
  and handoff fencing.
- [AI Vertical Slice Pressure Test](ai-vertical-slice-pressure-test.md)
  (Draft): gap wording and slice evidence.
- [AI Unresolved Questions](ai-unresolved-questions.md) (Draft): open
  register entries, unchanged by this note.
- [AI Architecture](ai-architecture.md) (Draft): bridge process model and
  scene path.
