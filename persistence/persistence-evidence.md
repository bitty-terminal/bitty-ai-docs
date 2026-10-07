---
title: Persistence and evidence architecture
description: Draft journal evidence projection indexing and replay tradeoffs under mandatory privacy controls
category: architecture
audience: contributor
document_type: specification
status: draft
website_publish: false
sidebar_order: 34
---

# Persistence and evidence architecture

## Purpose and scope

This draft compares session journals, execution evidence and event-oriented
storage. These are overlapping design dimensions, not mutually exclusive
backends. No schema, backend, release milestone or persistence requirement is
selected here. The terminal architecture's historical post-1.0 AI scope does
not settle standalone `bitty-ai` release requirements. An ephemeral initial
profile and durable recovery are alternatives needing an explicit scope decision.

**Draft relationships:** [Context management](../context/context-management.md),
[agent coordination](../agent/agent-coordination.md), and
[code intelligence](../agent/code-intelligence.md). These drafts accept no mechanisms.

## Privacy and retention boundary

The normative security corpus, especially P0-AC-026, overrides every recording
example. [AI Architecture PP-2/PP-4](../architecture/ai-architecture.md#privacy-first)
reconciles pre-queue typed redaction and consent for disk recording with that
baseline. A bounded in-memory session and consented durable storage are distinct.

Completeness or losslessness means only fidelity to **authorized, redacted,
still-retained records**. It never promises capture of all conversation, file
reads, input or raw logs. Input recording is off by default and separately opt-in;
clipboard and raw environment are absent by default. Durable recording needs
applicable explicit consent, minimization, redaction before queue/write, user-only
storage and exact export preview. Consent does not authorize unredacted secrets.

Retention is bounded. Deletion/expiry propagates to payloads, evidence, derived
summaries, indexes, caches and referenced artifacts. Tombstones preserve only
permitted absence metadata, not deleted sensitive content. Re-expansion returns
typed unavailable for missing records; neither backup recovery nor replay may
resurrect deleted, expired or never-recorded content. Logical immutability does
not override deletion. The exact propagation and reconstruction protocol is open,
but privacy controls are not optional pending that choice.

## Design dimensions

| Dimension              | Candidate meaning                                                                                      | Limit                                                                            |
| ---------------------- | ------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------- |
| Journal representation | Ordered, append-oriented session entries, including retained conversation and tool evidence references | Completeness limited by consent, redaction, retention and capture bounds         |
| Context projection     | Selected working set or summary built from authorized sources                                          | Compaction changes provider input, not necessarily journal bytes                 |
| Execution evidence     | Results, input fingerprints, execution identity, completeness and attribution                          | May be independent of any conversation, shared only under reader authorization   |
| Storage backend        | SQLite, structured files, append-oriented log or hybrid                                                | No backend selected; transaction and failure semantics need evidence             |
| Search index           | Optional FTS5 or another index over retained fields                                                    | An event log does not require full-text search; deletion must invalidate indexes |
| Replay/reconstruction  | Rebuild supported state from retained ordered records                                                  | Not a permission to rerun tools; no automatic exactly-once or rollback guarantee |

### Session journal and projection

The journal proposal separates captured facts from the next provider view.
Structured outputs, deduplication, selective summaries, compaction entries and
provider-native compression are candidate projection operations. Projection
compaction is compatible with replay when the original journal is retained.
Only destructive journal reduction, deletion, expiry or missing records limit
reconstruction. A summary cannot recreate omitted bytes or certify original tool
outcomes. Backend choice and recovery contract remain open.

### Execution evidence and reuse

Evidence may be stored in a journal, referenced by events, or held separately;
it is not necessarily a subset of a conversation event log. A result records an
execution identity, fingerprint, tool/adapter version, outcome, exit state,
timestamps, completeness and retained evidence references. A candidate sequence:

1. Validate caller, captured target/generation, effect class, budgets and consent.
2. Resolve a bounded input manifest and reuse policy. Unknown relevant inputs
   disable generic caching and effectful coalescing.
3. Coalesce only proven compatible authorized work, with independent waiter
   cancellation and distinct per-request attribution.
4. Record bounded redacted outcome metadata, durably only where consented.
5. Reauthorize each reader and validate freshness before delivery; disclose
   `reused from execution` versus `executed now`.

Fingerprinting includes untracked/generated/deleted inputs, submodule and symlink
state, overlays, tool identity/version, argv, target/cwd, dependency resolution,
environment identity and isolation policy. See [code intelligence](../agent/code-intelligence.md#verification-fingerprinting)
for the full proposal and complexity. Mutable-tree checks cannot prove a coherent
snapshot. Cache hits are not grants, and cached PASS is not independent approval.

### Event-oriented storage and recovery

An ordered event representation can support state reconstruction, with or without
SQL or full-text indexing. Durable intent before an effect is a useful upstream
observation, not sufficient evidence of exactly-once execution. A crash between
an effect and acknowledgement produces `Unknown`; inspect state or obtain user
direction before retry. Cancellation before dispatch prevents effects from
starting; cancellation after dispatch cannot promise reversal. State replay and
effect re-execution are separate operations, and effects require current grants.

**Runtime-bounded reconcile evidence (experimental, reconcile facet only):**
the sibling `bitty-ai` `UnknownReconciler` answers bounded status queries
without re-executing (`ReconcileConfig` default 3 queries, 100 ms base,
5000 ms ceiling, hard caps 16 queries and 30000 ms; exhaustion escalates to
typed `UnknownEscalation` / `AgentError::UnknownUnresolved`); merged in
`bitty-ai` `13ce4c6` (AI-0047). Exactly-once effects and cross-boundary safe
retry stay open — see [AI Unresolved Questions](../product/ai-unresolved-questions.md).
This describes sibling behavior only as read.

**Content-addressed store evidence (experimental, schema and backend facets
only):** the sibling `bitty-ai` `ContentStore` implements a SQLite-backed
content-addressed persistence layer with three tables (`blobs`, `checkpoints`,
`refs`). Blobs are deduplicated by SHA-256 `ContentHash`, bounded by 16 MiB,
and verified on read. Checkpoints are DAG commit nodes with structured
`Rationale` records (bounded cognitive metadata: `why`, `what`, `where_focus`,
`how`, `expected`, `observed`), parent existence verification, tree blob
reference verification, and versioned length-prefixed canonical hashing
(`checkpoint:v2\0`) preventing delimiter-collision attacks. Refs provide
mutable HEAD and branch pointers with existence-verified targets. DAG
operations include backward log traversal and merge-base (lowest common
ancestor) calculation. `INSERT OR IGNORE` provides conflict tolerance for
concurrent writers, and `get_checkpoint` verifies integrity by recomputing the
canonical hash on read (`CorruptCheckpoint` error on mismatch). Facade access
through `AiEngine::open_content_store` and `open_in_memory_content_store`;
merged in `bitty-ai` `9b83315` (AI-0162). Transaction isolation (WAL,
multi-writer), schema migration, event log, effect ledger, and retention
metadata stay open — see [AI Unresolved Questions](../product/ai-unresolved-questions.md).
This describes sibling behavior only as read.

**Task DAG and control-plane evidence (experimental, lifecycle and graph scheduling facets only):**
the sibling `bitty-ai` `TaskEngine` implements a graph-theoretic task scheduling and control layer
with pure Rust graph algorithms and SQLite persistence (`tasks` and `task_dependencies` tables).
Tasks carry typed `TaskId` (sanitized alphanumeric identifiers with validated deserialization),
`TaskStatus` lifecycle states (`Pending`, `Ready`, `Running`, `Blocked`, `Succeeded`, `Failed`,
`Cancelled`), and monotonic `generation` fencing tokens. Cycle detection using BFS fails closed
with typed `CycleDetected` errors prior to insertion, preventing deadlock configurations.
Topological sorting using Kahn's algorithm guarantees deterministic task ordering with priority
tie-breaking. Dynamic execution fencing validates generation tokens on worker completion attempts,
failing closed with `StaleGeneration` when an obsolete or timed-out worker attempts state mutation.
Cascade readiness automatically promotes downstream tasks to `Ready` when all prerequisites succeed,
while cascade blocking propagates `Blocked` status across the entire transitive dependency subgraph
upon task failure or cancellation. Facade access through `AiEngine::open_task_engine` and
`open_in_memory_task_engine`; 10 integration tests in `crates/bitty-ai-slice/tests/task_dag.rs`;
merged in `bitty-ai` `5c3a3dc` (AI-0164). Distributed task scheduling, cross-session worker leases,
and dynamic priority re-evaluation stay open — see [AI Unresolved Questions](../product/ai-unresolved-questions.md).
This describes sibling behavior only as read.

**Three-zone context compiler and Merkle tree evidence (experimental, compilation and prefix-cache facets only):**
the sibling `bitty-ai` `ContextCompiler` and `ContextTree` implement a Three-Zone context compilation
pipeline with deterministic Merkle tree state management. `ContextTree` maps named slot entries
(`TreeEntry`, `EntryKind::Blob`, `EntryKind::Tree`) to immutable content hashes with versioned
length-prefixed canonical encoding (`tree:v1\0`), tree diffing (`diff`), binary reconstruction
(`from_canonical_bytes`), and 3-way semantic slot merging (`merge_3way`) with typed `SlotConflict`
detection. `ContextCompiler` separates the compiled context into three distinct zones: Zone 1
(Stable Prefix) preserves byte-identical system prompts and schemas for LLM vendor prefix-cache hits
and fails closed on budget violation (`Zone1BudgetExceeded`); Zone 2 (Structured State) encodes
the active task, cognitive checkpoints, and context tree slots; Zone 3 (Dynamic Tail) retains the
turn prompt and recent observations. A multi-tier budget reduction pipeline enforces hard ceilings:
Tier 1 prunes unpinned scratchpad slots (`scratch/*`, `temp/*`), Tier 2 compresses older checkpoint
rationales into 1-line digests while retaining recent checkpoints, and Tier 3 cleanly truncates
dynamic tail observations while preserving Zone 1 and the active task. Facade access through
`AiEngine::compile_context`; 8 integration tests in `crates/bitty-ai-slice/tests/context_compiler.rs`;
merged in `bitty-ai` `acfb4a1` (AI-0165). Dynamic vendor-specific cache breakpoint negotiation and
cross-turn cache hit telemetry stay open — see [AI Unresolved Questions](../product/ai-unresolved-questions.md).
This describes sibling behavior only as read.

**Action protocol and auto-spillover evidence (experimental, execution plane and observation formatting facets only):**
the sibling `bitty-ai` `ActionEngine` implements a standardized action protocol separating action
requests from tool execution outcomes. Actions carry structured `ActionIntent` (`Inspect`, `Modify`,
`Execute`, `Verify`, `Custom`) with target validation and bounded parameters. When tool execution
produces stdout or stderr exceeding the inline threshold (`max_inline_bytes`, default 4 KiB),
`ActionEngine` automatically spills the full raw payload into a content-addressed `BlobSink`
(`ContentStore`), generating bounded head/tail UTF-8 previews with explicit truncation indicators.
Structured observation formatting (`format_for_context`) outputs clean Markdown blocks containing
blob content addresses, exit codes, and execution durations optimized for Zone 3 context compilation.
Facade access through `AiEngine::process_action_outcome`; 6 integration tests in
`crates/bitty-ai-slice/tests/action_protocol.rs`; merged in `bitty-ai` `045aa7f` (AI-0166). Interactive
streaming tool input, capability-mediated sandbox revocation, and concurrent tool effect arbitration
stay open — see [AI Unresolved Questions](../product/ai-unresolved-questions.md).
This describes sibling behavior only as read.

**Unified WheelKernel facade and zero-unsafe bridge evidence (experimental, boundary plane and Lua FFI facets only):**
the sibling `bitty-ai` `WheelKernel` converges storage (`ContentStore`), task control (`TaskEngine`),
cognitive state (`ContextTree`), and execution spillover (`ActionEngine`) into a cohesive runtime
orchestrator. Cognitive checkpoints commit Merkle tree canonical blobs and update branch and HEAD
pointers atomically within a single SQLite transaction (`update_refs_atomic`). Persistent kernels
reopen by decoding the active context tree from the HEAD checkpoint's tree blob (`ContextTree::from_canonical_bytes`).
`WheelBridge` provides a safe, zero-unsafe JSON-RPC command dispatch boundary (`kernel.status`, `task.*`,
`slot.*`, `checkpoint.*`, `action.*`, `context.compile`) operating over byte buffers and typed
`BridgeResponse` envelopes with checked integer conversions. An idiomatic Lua client wrapper
(`crates/bitty-ai-slice/lua/wheel/kernel.lua`) provides synchronous host and plugin integration. Facade
access through `AiEngine::open_wheel_kernel` and `open_wheel_bridge`; 14 integration tests across
`tests/wheel_kernel.rs` and `tests/wheel_bridge.rs`; merged in `bitty-ai` `42309b3` (AI-0167). C-ABI
shared-library dynamic loading and asynchronous IPC event multiplexing stay open — see
[AI Unresolved Questions](../product/ai-unresolved-questions.md).
This describes sibling behavior only as read.

**TaskView inlined dependency batch retrieval evidence (experimental, control plane optimization facet only):**
the sibling `bitty-ai` `TaskEngine` and `WheelBridge` implement single-query prerequisite inlining
for task records. `TaskView` enriches `TaskNode` with `dependencies: Vec<TaskId>`, implementing
`Deref<Target = TaskNode>` and flattened serialization. `TaskEngine::list_task_views` batch retrieves
all tasks and their direct prerequisites in $O(|V| + |E|)$ time using a single query across `tasks`
and `task_dependencies`, grouping prerequisite IDs in memory and completely eliminating N+1 query
roundtrips across IPC. `WheelBridge` routes `task.get`/`task.get_view` and `task.list`/`task.list_views`
through `TaskView`, providing immediate dependency graphs to external callers and UI visualizers.
Integration test coverage in `crates/bitty-ai-slice/tests/task_dag.rs` (`task_view_dependencies_inlining_and_serde`),
`tests/wheel_bridge.rs` (`test_bridge_task_dependencies_inlined`), and `tests/wheel_kernel.rs`
(`test_kernel_task_view_dependencies`); merged in `bitty-ai` `34fc526` (AI-0169). Cross-repository
transitive dependency resolution across independent DAGs stays open — see
[AI Unresolved Questions](../product/ai-unresolved-questions.md).
This describes sibling behavior only as read.

**Line-delimited JSON-RPC stdio host runner and cross-process integration evidence (experimental, host runner and IPC drill facets only):**
the sibling `bitty-ai` example binary `wheel_stdio_host.rs` exposes `WheelBridge` over standard
input/output via line-delimited JSON-RPC with optional `--db <path>` SQLite persistence or ephemeral
in-memory operation. Dual-format parsing supports both structured JSON envelopes (`{"command": "...", "payload": ...}`)
and space-delimited command lines (`command payload_json`), with all diagnostics and lifecycle
banners strictly isolated to `stderr`. Deserialization of `TaskDraft` and `TaskView` enforces
resilient visitor decoding (`deserialize_task_deps`), transparently accepting sequences, `null`,
and empty maps (`{}`) from dynamic language serializers (Lua). Verified live in an 11-stage automated
cross-process drill over Unix named pipes (`bitty-terminal/wheel` `tests/e2e_cross_process.lua`, `just e2e`),
exercising handshake, diamond DAG decomposition, dynamic programming topological wave calculation,
monotonic generation fencing (`StaleGeneration`), cascade readiness propagation, Merkle context tree
slots (`tree:v1\0`), action auto-spillover (>8 KiB), Three-Zone context compilation under 64 KiB budget,
6-field structured `Rationale` checkpoints, and multi-process crash/restart SQLite recovery.
Merged in `bitty-ai` `6e062d0` (AI-0170) and `bitty-terminal/wheel` `6f9d09b` (CTX-0009). Socket-based
daemonization, multiplexed bidirectional streaming events, and multi-host consensus stay open — see
[AI Unresolved Questions](../product/ai-unresolved-questions.md).
This describes sibling behavior only as read.

## Verification plan

The inspected `bitty-ai` revision
`3623c6b3ce33e97c1c493109ec6356219d0c9722` has a real experimental slice
(`crates/bitty-ai-slice/src/session.rs:68-136` at that revision; live equivalent:
`crates/bitty-ai-runtime/src/session.rs`), not evidence of this proposed
store/replay runtime. Before enabling persistence/reuse/recovery, obtain an
explicit scoped design and independent security review, then evidence for
consent/redaction, deletion propagation, authorization, bounded storage, crash
reconciliation and replay semantics. Measure storage overhead, query latency and
GC behavior for the selected workload. This document authorizes no prototype or
product implementation and makes no release commitment.

## Open points

Stable identifiers live in the [local AI unresolved-questions register](../product/ai-unresolved-questions.md),
not the accepted global OQ register:

1. **AIQ-51 Schema:** Which representations and transaction boundaries support the selected feature profile?
2. **AIQ-52 Replay:** Which states can be reconstructed, and what evidence distinguishes replay from effect re-execution?
3. **AIQ-53 Backend/index:** Which backend meets bounded storage/query needs, and is optional full-text indexing useful?
4. **AIQ-54 Retention authority:** How are user preferences and tool metadata constrained by host policy?
5. **AIQ-55 Deletion/GC:** How do expiry and deletion invalidate all derived records and references?
6. **AIQ-56 CarryCtx:** Alias of AIQ-10 for lifecycle ownership; the persistence facet asks whether a backend or explicit handoff is appropriate.
7. **AIQ-57 Reconstruction limits:** After retention expiry, deletion or destructive journal reduction, what state remains reconstructible and how is missing evidence disclosed? Projection-only compaction does not require deleting originals.
8. **AIQ-58 Sharing:** Which mechanism proves per-reader non-disclosure across authorization scopes?
9. **AIQ-59 Effects:** How are unknown outcomes reconciled without unsafe retries or fictitious exactly-once guarantees?
10. **AIQ-5A Redaction representation:** What typed markers and invalidation protocol implement mandatory pre-queue/pre-write redaction? The timing requirement is already fixed.
11. **AIQ-5B Observability:** Which bounded authorized queries are required, including explicit truncation and absent evidence?
12. **AIQ-5C Release scope:** Which standalone AI profile needs durable state and when? Neither ephemeral v0.1 nor post-1.0 persistence is selected here.
