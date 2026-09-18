---
title: Dependency Strategy
description: Draft proposal for std-only runtime kernel with post-v0.1 adapter dependency boundaries
category: specifications
audience: contributor
document_type: specification
status: draft
website_publish: false
sidebar_order: 29
---

# Dependency Strategy

> Status: **draft**. This document distills a single-author workspace research
> record into a reviewable proposal. It accepts nothing, describes no shipped
> behavior, adopts no dependency, and authorizes no compatibility promise.
> Every adapter, crate sketch, and version number below is a post-v0.1
> proposal, not a commitment. No new dependency is adopted by this document:
> v0.1 adds zero dependencies per the
> [v0.1 Implementation Profile](../product/implementation-profile-v0.1.md).

**Draft relationship**: Refines the dependency implications of
[AI Architecture](../architecture/ai-architecture.md) MP-3 (Local-first default), TB-1 (MCP as
adapter), TB-3 (Validation before dispatch), TB-4 (Capability and consent per
tool), AG-4 (Least privilege at dispatch), and AG-5 (Orchestration versus
execution). It is gated by the [v0.1 Implementation Profile](../product/implementation-profile-v0.1.md)
single-crate scope and defers transport, intelligence, and storage detail to
[Command and Tool Architecture](../architecture/command-tool-architecture.md),
[Agent Coordination Architecture](../agent/agent-coordination.md),
[Code Intelligence Architecture](../agent/code-intelligence.md), and
[Persistence and Evidence Architecture](../persistence/persistence-evidence.md). These are
topic relationships, not accepted authority.

## Source provenance

Distilled from research note `030` (read 2026-09-15, 678 lines).

The source is written in Chinese
and is preserved untranslated under its existing name; this document is an
English critical distillation, not a translation. The source is shared with
the terminal-docs track and was intentionally not renamed. Source section
numbers below refer to that record. Point-in-time crate versions named by the
source are observations as of September 2026, never pins or approvals; each
needs re-verification against current upstream before any future adoption
decision.

## What this document does not duplicate

Each item below stays owned by its existing document; this proposal references
it and adds only the dependency-boundary facet:

- Provider registry, descriptor, consent, budget, and credential handling stay
  with [AI Architecture](../architecture/ai-architecture.md) MP-1 through MP-11, especially
  MP-3 (Local-first default) and MP-10 (API-key handling). This proposal adds
  no registry field, provider kind, or credential mechanism.
- Tool Bus validation, consent, budgets, and host-only execution stay with
  [AI Architecture](../architecture/ai-architecture.md) TB-1 (MCP as adapter) through TB-7 and
  with [Command and Tool Architecture](../architecture/command-tool-architecture.md). This
  proposal adds no ToolSpec field, authorization backend, or transport
  selection; those stay with AIQ-33, AIQ-36, and AIQ-38.
- Context budget, artifacts, and determinism stay with
  [AI Architecture](../architecture/ai-architecture.md) CP-5 (Budget), CP-6 (Artifacts), and
  CP-7 (Determinism and testability), elaborated by
  [Context Management Architecture](../context/context-management.md) and
  [Prefix-Cache-Friendly Context Design](../context/prefix-cache-context-design.md). This
  proposal adds no serializer, cache key, or budget rule (AIQ-12, AIQ-13).
- Service supervision, Panel and execution ownership, and delegation budgets
  stay with [Agent Coordination Architecture](../agent/agent-coordination.md) under
  AG-4 (Least privilege at dispatch) and AG-5 (Orchestration versus
  execution). This proposal adds no lifecycle state or supervision mechanism.
- LSP sharing, verification fingerprinting, and lint/build/test reuse stay
  with [Code Intelligence Architecture](../agent/code-intelligence.md). This proposal
  adds no broker behavior, overlay rule, or fingerprint field (AIQ-41 through
  AIQ-48).
- Journal representation, backend selection, retention, and replay semantics
  stay with [Persistence and Evidence Architecture](../persistence/persistence-evidence.md).
  This proposal selects no backend and defines no schema (AIQ-51 through
  AIQ-5C).
- The v0.1 scope gate stays with the
  [v0.1 Implementation Profile](../product/implementation-profile-v0.1.md). Anything not
  listed there is out of scope for v0.1, not rejected.
- Open-question ownership and promotion stay with
  [AI Unresolved Questions](../product/ai-unresolved-questions.md) and the canonical OQ
  admission rule in `bitty-docs`. This proposal reuses existing identifiers
  and proposes no new identifier.

## Kernel principle: std-only runtime with dependency inversion

The source proposes (lines 1-6, 29-88) that the deterministic agent runtime
stay dependency-free and network-free, with third-party crates confined to
boundary adapters.

**Draft disposition: adopt:**

- `bitty-ai-runtime` keeps its current shape: an agent kernel and state
  machine over traits and domain types (`agent`, `context`, `provider`,
  `session`, `stream`, `tool` in the source sketch), not an I/O framework.
  It stays without an async runtime, HTTP client, TLS stack, MCP SDK,
  parser, or database in v0.1 and, as a working hypothesis, stays without
  network dependencies permanently.
- Dependency inversion at the provider boundary: the runtime knows a
  `ModelProvider` trait and domain tool/context/session types, but does not
  know HTTP, vendor APIs, `reqwest`, `tokio`, or TLS. The source sketch
  (lines 65-82) lists exactly what the runtime must not name: HTTP, vendor
  identities, request libraries, async runtimes, and TLS. Transport,
  pooling, timeout, redirect, proxy, chunked bodies, and SSE framing belong
  to a provider adapter, never to the kernel.
- The same inversion applies outward: the Tool Bus knows `ToolSpec`,
  `ToolCall`, `ToolResult`, schema, authorization, and dispatch; it does not
  own MCP framing, LSP lifecycle, file walking, parsing, or storage. Each of
  those enters only through a narrow adapter behind validation,
  authorization, budget, and redaction.

This matches the existing posture that no network call exists until a
provider whose privacy class permits it is selected and the corresponding
capability is granted (MP-3 (Local-first default)), that every native and MCP
effect needs the unified authorization backend (AIQ-33), and that the v0.1
profile runs behind a `FakeProvider` with no network access in v0.1 code
paths.

## Adapter boundary map

Future third-party dependencies, if ever adopted, belong to adapters outside
the kernel. Nothing in this section is adopted now; every row is a post-v0.1
proposal gated on its own contract, OQ resolution, and implementation
evidence.

| Capability                 | Candidate crates (observations, not adoptions) | Owning adapter boundary              | v0.1 status |
| -------------------------- | ---------------------------------------------- | ------------------------------------ | ----------- |
| Deterministic agent kernel | None; standard library only                    | `bitty-ai-runtime`                   | Keep as is  |
| Serialization and wire     | `serde`, `serde_json`                          | Provider / IPC adapter / Tool Bus    | Post-v0.1   |
| Error boilerplate          | `thiserror`                                    | Each adapter / runtime crate         | Post-v0.1   |
| Async runtime              | `tokio`, `tokio-util`                          | Provider / MCP / LSP adapters only   | Post-v0.1   |
| Async streaming            | `futures-core`, `futures-util`                 | Provider streaming adapter           | Post-v0.1   |
| HTTP providers             | `reqwest`                                      | Provider adapter                     | Post-v0.1   |
| MCP client                 | `rmcp`                                         | MCP tool adapter                     | Post-v0.1   |
| Tool schema and validation | `schemars`, `jsonschema`                       | Tool registry / Tool Bus             | Post-v0.1   |
| Logging and spans          | `tracing`                                      | Runtime adapter layer                | Post-v0.1   |
| Workspace file traversal   | `ignore`                                       | Context / code tools adapter         | Post-v0.1   |
| Syntax and symbols         | `tree-sitter` plus grammar crates              | Code intelligence adapter            | Post-v0.1   |
| Language protocol          | `async-lsp`, `lsp-types`                       | Code intelligence broker             | Post-v0.1   |
| Fingerprint and cache keys | `blake3`                                       | Evidence / code intelligence adapter | Post-v0.1   |
| Diff primitives            | `similar`                                      | Diff / context / review primitives   | Post-v0.1   |
| Embedded persistence       | `rusqlite` evaluated first, if ever            | Store adapter, only if decided       | Not now     |
| Test fixtures              | `tempfile` as dev-dependency                   | Tests only                           | Post-v0.1   |

### Provider adapter

When real providers arrive, the source advises (lines 92-148) against
hand-implementing TCP, HTTP, HTTP/2, TLS, chunked bodies, SSE, proxying,
pooling, timeouts, and redirects. A provider adapter owns `tokio` plus
`reqwest` (with `futures-core` / `futures-util` for streaming and `tracing`
for spans) behind the `ModelProvider` trait, starting as one consolidated
provider crate with per-protocol modules and splitting per vendor only if
lifecycles and release cadences genuinely diverge. Provider kinds remain
transport adapters, never capability grants: remote kinds still require the
accepted network grant and provider consent, and a `local-only` provider
performs no network I/O (MP-3 (Local-first default); MP-10 (API-key
handling)).

### Tool Bus adapter

The source proposes (lines 218-308) deriving tool argument types into JSON
Schema with `schemars` and validating model-produced JSON arguments with
`jsonschema`, so hand-written field checks do not become the validation
story. **Draft disposition: adopt with the existing order preserved:** size bound,
then schema validation per TB-3 (Validation before dispatch), then typed
deserialization, then permission and effect classification per TB-4
(Capability and consent per tool), then dispatch. Validation never widens
authority, and dispatch stays under AG-4 (Least privilege at dispatch) with
the unified backend from AIQ-33.

### MCP adapter

The source proposes (lines 152-215) against re-implementing initialize,
capability negotiation, tools, resources, prompts, notifications, tasks,
subscriptions, transports, JSON-RPC correlation, and version negotiation, and
points at the official Rust MCP SDK as the future client behind an
`McpToolAdapter` beside a `NativeToolAdapter` under the Tool Bus. MCP stays
an adapter, not an internal protocol, per TB-1 (MCP as adapter); every
MCP-mediated effect still passes the same schema, caller and target
authorization, consent, budget, redaction, and outcome rules as native tools.
Transport selection and backend ownership stay open under AIQ-36 with generic
execution ownership under AIQ-38.

### Code intelligence adapter

The source proposes (lines 311-462) against re-implementing recursive
directory walking with ignore semantics, syntax parsing, and the LSP
lifecycle, pointing at `ignore` for workspace scanning, `tree-sitter` plus
grammars for outline and symbols feeding progressive disclosure, `async-lsp`
plus `lsp-types` for protocol types and framing if a broker is built, and
`blake3` for fingerprints and cache keys. Bitty-owned work stays at the
broker, authorization, sharing, snapshot, and progressive-disclosure layer;
protocol, parsing, and traversal stay with maintained crates. LSP detail
(initialize through shutdown, overlays, restarts, generations) and reuse
eligibility stay with [Code Intelligence Architecture](../agent/code-intelligence.md)
and AIQ-41 through AIQ-48.

### Store adapter

The source advises restraint (lines 464-514): with representation, backend,
and durable-recovery scope still undecided, no `sqlx`, `rusqlite`,
`sled`/`redb`, or `RocksDB` enters now. If a local single-process SQLite
profile with WAL is ever selected, `rusqlite` is evaluated before a full
async SQL framework, because the shape is an embedded database with a
controlled schema, not a database abstraction layer. Backend, schema,
retention, and replay-contract choices stay with AIQ-51 through AIQ-5C, and
any durable recording stays under PP-4 (No on-disk persistence without
consent) with PP-2 (Typed redaction).

## Provider and transport separation

This section distills only the `bitty-ai`/network-relevant tail of research
note `029` (read 2026-09-15, lines 1574-2018 of 2018 lines). The source is
written in Chinese and is preserved untranslated; this section is an English
critical distillation, not a translation. The record is shared with the
terminal-docs track. Lines 1-1573
(`bitty-core` network-free shape, plugin-via-git) are `bitty`-side and are
excluded here: the terminal-docs repository owns them. Within lines
1574-2018, every `bitty`-side row (terminal core, Lua plugin gateway, weather
plugin, Plugin Manager external git) is marked out of scope below; this
section decides only the `bitty-ai` side. The tail names no crate versions;
every provider name, transport kind, and endpoint shape below is a
point-in-time observation from September 2026, never a pin or approval.

Duplicate-check outcome: the kernel-no-network rule (Kernel principle), the
consolidated provider adapter map (Provider adapter), and the MSRV decision
points above already cover this document's prior surface. This section adds
only the delta: four-layer provider/transport layering with an `HttpTransport`
sketch and test transports, a feature-flag isolation sketch, shared-transport
with separate permission models, a unified internal model protocol as future
direction, and draft dispositions for the tail's three no-network rules. It
fits here because each item is a dependency-boundary facet of the same
std-only kernel proposal; no new specification is created and no navigation
change is needed. Everything below is a post-v0.1 proposal, not a
commitment. The v0.1 posture is restated, not weakened: v0.1 runs behind a
`FakeProvider` with no network access in v0.1 code paths per the
[v0.1 Implementation Profile](../product/implementation-profile-v0.1.md).

### Agent to transport layering as direction

The tail proposes (lines 1574-1674) a four-layer shape rather than letting
the agent call an HTTP client directly:

```text
Agent
  ↓
Model abstraction
  ↓
Provider
  ↓
Transport
```

The rationale (lines 1637-1652) is that not every provider needs the public
internet. Observed transport kinds from the source, unverified here: OpenAI,
Anthropic, Gemini, and OpenRouter over HTTPS; Ollama, LM Studio, and
`llama.cpp` over localhost HTTP or a local process; embedded and mock
providers with no network at all. No endpoint URL, port, or protocol version
is adopted by this section.

The tail sketches two async provider traits (lines 1613-1631), `Provider`
with `complete` and `LanguageModel` with `stream`. Judgment: both sketches
are future direction only. The current runtime keeps its sync provider-turn
shape (`ProviderTurn`/`Fragment` lineage in the experimental slice,
`FakeProvider` with no network in v0.1); no async runtime, HTTP client, or
TLS stack enters `bitty-ai-runtime` now. Any async adoption needs its own
reviewed contract, OQ resolution, and implementation evidence, consistent
with the split-only-on-real-boundary sequencing.

The tail's crate-boundary sketch (lines 1777-1822: `bitty-ai-core`,
`bitty-ai-agent`, `bitty-ai-tools`, `bitty-ai-context`,
`bitty-ai-provider-api`, per-vendor provider crates,
`bitty-ai-transport-http`) is illustrative future shape, not a plan. Stale
`bitty-ai-core` crate naming below is translated to the current
`bitty-ai-runtime` single-crate scope; the former core crate name is deleted
on the implementation track per AI-0011 (not verified here). The
anti-pattern stays: no `bitty-ai-runtime` depending on `reqwest`, directly
or transitively, because every dependent would then inherit the HTTP/TLS
tree. Per-vendor splits happen only on genuinely divergent lifecycles,
release cadences, or feature sets.

### HttpTransport split and test transports

The tail proposes (lines 1678-1733) separating vendor logic from HTTP
mechanics so `OpenAiProvider` owns protocol mapping while an `HttpTransport`
abstraction owns bytes on the wire, with `ReqwestTransport`,
`CurlTransport`, `MockTransport`, `ProxyTransport`, and `RecordedTransport`
as future backends. **Draft disposition: adopt as test-value direction, post-v0.1 only:**

```rust
trait HttpTransport {
    async fn request(
        &self,
        request: HttpRequest,
    ) -> Result<HttpResponse>;
}

struct OpenAiProvider<T: HttpTransport> {
    transport: T,
}
```

Unit tests would then drive `Agent` through `OpenAIProvider` over a
`MockHttpTransport` or `RecordedTransport` without reaching a vendor
endpoint. No transport trait, backend, or vendor crate is adopted now; each
needs its own contract, redaction evidence under PP-2 (Typed redaction),
and fail-closed tests showing no ambient filesystem, process, or network
authority leaks past MP-3 (Local-first default) and AG-4 (Least privilege
at dispatch).

### Feature-flag isolation sketch as direction

The tail sketches (lines 1737-1773) Cargo features such as
`provider-openai`, `provider-anthropic`, and `http-native`, so a local-only
build like `cargo build --no-default-features --features provider-ollama`
yields agent, tools, context, and Ollama with no public-internet client, and
a future Unix-socket or subprocess Ollama path could drop the HTTP client
entirely. Recorded as direction only: no feature names, crate names, or
default-feature choices are adopted, and the sketch does not authorize
removing or adding any dependency. Any future flag layout must preserve the
v0.1 zero-new-dependency gate until its own increment explicitly adopts an
adapter.

### Shared transport implementation, separate permission models

The tail proposes (lines 1826-1894) sharing the transport implementation
between plugin HTTP and AI providers while keeping their permission models
separate: a plugin HTTP gateway with a permission layer (permission check,
host allowlist, sandbox, rate limit, user consent) beside a trusted provider
path for Bitty's own provider component. Judgment: only the `bitty-ai` half
is in scope here. The `bitty`-side rows — weather plugin over `bitty.http`,
the Lua plugin HTTP gateway, the `bitty-http-core` naming, and Plugin
Manager external git — belong to the terminal-docs and plugins-docs tracks
and are marked out of scope; nothing here decides them.

On the `bitty-ai` side, "trusted" in the source means the provider does not
pass through the plugin sandbox, not that it carries ambient authority. A
post-v0.1 provider path still requires the accepted gates unchanged:
MP-3 (Local-first default) for the network grant, MP-10 (API-key handling)
for credential references with typed redaction, TB-3 (Validation before
dispatch) and TB-4 (Capability and consent per tool) at the Tool Bus, AG-4
(Least privilege at dispatch) at dispatch, and PP-2 (Typed redaction) with
PP-4 (No on-disk persistence without consent) for any diagnostic, trace, or
recorded transport payload. Sharing a transport implementation must never
share or widen consent scope.

### Unified internal model protocol as future direction

The tail proposes (lines 1898-1983) a Bitty-internal protocol so the agent
never handles vendor framing: providers differ in authentication, streaming
protocol, tool-calling format, reasoning fields, usage accounting, and cache
metadata, with future candidates named as observations (Claude Messages API,
Gemini API, Responses API, OpenAI-compatible, local `llama.cpp`, Ollama, AWS
Bedrock, Azure OpenAI, Vertex AI, custom enterprise endpoints). Each vendor
stream (SSE, HTTP chunks, Anthropic events, OpenAI events, Gemini
candidates) would be consumed inside the provider adapter and re-emitted as
a uniform event stream over a uniform request shape, sketched in the source
as:

```rust
struct ModelRequest {
    messages: Vec<Message>,
    tools: Vec<ToolDefinition>,
    temperature: Option<f32>,
    max_tokens: Option<u32>,
}
```

```rust
enum ModelEvent {
    TextDelta(String),
    ReasoningDelta(String),
    ToolCallStart { /* ... */ },
    ToolCallDelta { /* ... */ },
    ToolCallEnd { /* ... */ },
    Usage(Usage),
    Finished,
}
```

Recorded as future direction only. The current sync provider-turn shape
stays; these async request/event sketches are not adopted, add no
`ModelRequest` or `ModelEvent` type, and decide no streaming, tool-schema,
or accounting semantics. Canonical serialization, stable-prefix ordering,
cache-key, and routing-scope rules stay with AIQ-12 and AIQ-13; provider
transport and bridge placement stay with AIQ-36 with generic execution
ownership under AIQ-38; per-action authorization stays with AIQ-33. No new
identifier is proposed: each facet reuses its existing OQ.

### Draft dispositions for the tail's three no-network rules

The tail closes (lines 1985-2018) with three rules and an expanded matrix.
Recorded here as draft dispositions, translating stale naming and marking
`bitty`-side rows out of scope. The source text, with `bitty-ai-core`
translated in brackets:

> 1. `bitty-core` has no network dependency.
> 2. `bitty-ai-runtime` [`bitty-ai-core` in the source] has no network dependency.
> 3. Network exists only behind explicit transport/provider boundaries.

Dispositions:

1. `bitty-core` has no network dependency: `bitty`-side, out of scope.
   Terminal-docs owns it; recorded here only as a dependency of the
   layering, not decided.
2. `bitty-ai-runtime` has no network dependency: proposal rationale
   consistent with the Kernel principle and the v0.1 `FakeProvider`
   no-network posture, not a new normative requirement. The stale
   `bitty-ai-core` crate name maps to the current `bitty-ai-runtime`
   single crate, whose former core-crate name is deleted on the
   implementation track per AI-0011 (not verified here).
3. Network exists only behind explicit transport/provider boundaries:
   proposal rationale consistent with dependency inversion and the adapter
   boundary map, not a new normative requirement. Enforcement still flows
   through the existing controls: MP-3 (Local-first default), MP-10
   (API-key handling), TB-3 (Validation before dispatch), TB-4
   (Capability and consent per tool), AG-4 (Least privilege at dispatch),
   PP-2 (Typed redaction), and PP-4 (No on-disk persistence without
   consent), which this section does not weaken.

The expanded matrix (lines 1998-2014: terminal core without HTTP/TLS, AI
runtime without HTTP/TLS, AI provider with optional HTTP, Lua plugin with
optional network capability, Plugin Manager with external git) is treated
the same way: the AI-runtime row restates the v0.1 posture as proposal
rationale, the AI-provider row is a post-v0.1 proposal, and the
terminal/plugin/manager rows are out of scope here.

## MSRV decision points are open, not actions

The workspace baseline in the source is Rust `1.85` for both `bitty` and
`bitty-ai`. Point-in-time observations from September 2026 need decisions
before any adoption; this document bumps nothing and pins nothing.

| Observation (September 2026, unverified) | Fit against workspace `1.85` | Open decision, not an action                                                                                                                                                  |
| ---------------------------------------- | ---------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `reqwest 0.13.5` declares MSRV `1.85`    | Fits the current baseline    | No MSRV decision needed for the provider path on this point alone; adoption still needs its own contract and OQ resolution                                                    |
| `rmcp 3.x` needs Rust `1.88`             | Above the current baseline   | Keep `1.85` and pin a compatible SDK, raise the project MSRV at the MCP stage, or isolate the MCP adapter at a higher MSRV; source leans against pinning an old SDK long-term |
| `tree-sitter 0.27.0` declares `1.90`     | Above the current baseline   | Do not raise the whole workspace for one parser now; revisit as an MSRV decision when code intelligence lands, or when the toolchain has moved naturally                      |

The three-way MCP framing (stay on `1.85`, raise the project, or isolate the
adapter) and the tree-sitter deferral come directly from the source
(lines 191-215, 384-410). Each is recorded here as an open choice faceted to
existing transport and intelligence questions (see below), not as a new
crate, version pin, or toolchain decision.

## Split only on a real dependency boundary

The source closes (lines 636-657) with an explicit sequencing rule: the v0.1
single-`bitty-ai-runtime` shape exists to stabilize interfaces first; crates
split along dependency boundaries only when a real external dependency
arrives. The six-crate sketch in the source (runtime, provider, tool-bus,
MCP, code, store) is an illustrative future shape, not an implementation
plan. Premature vendor-per-crate splits and speculative workspace layouts
are rejected: consolidate providers first, then split only on genuinely
divergent lifecycles, release cadences, or feature sets.

## The three closing principles

The source states three principles (lines 659-668), adopted here as proposal
rationale only, not as normative requirements:

> `bitty-ai-runtime` stays free of network dependencies for as long as
> possible, ideally long-term near std-only.
>
> HTTP, MCP, LSP, and database access are adapters; they do not enter the
> agent kernel.
>
> Reuse maintained crates for commodity infrastructure (protocols, parsers,
> HTTP/TLS, JSON Schema, LSP, file traversal); reserve Bitty-owned design for
> agent lifecycle, context projection, tool authorization, execution and
> panel integration, and evidence semantics.

## Identity naming is bridge input, not a docs decision

The source observes (lines 516-587) that `bitty-agent::AgentId` as
`owner.name` protocol principal and `bitty-ai-runtime` numeric `AgentId`
alongside `RunId`, `SessionId`, and `ExecutionId` share a name with different
meanings: external protocol identity versus runtime-local logical handle. It
sketches explicit renames (`AgentPrincipalId` versus `AgentInstanceId`, or
`ProtocolAgentId` versus `RuntimeAgentId`) and a bridge mapping from protocol
identity through authorization into runtime identity and its run, session,
and execution handles.

Judgment: record the options and the bridge sketch as input to the in-flight
code task AI-0013. This document adopts no rename, assigns no identifier, and
decides no mapping. The `bitty-agent` protocol-identity side belongs to the
`bitty` repository and is out of scope here; any change there needs its own
reviewed contract in the owning repository.

## Point-in-time versions are observations only

All versions below are second-hand observations from the source as of
September 2026, with source-cited links left in the research record. None is
a pin, approval, or recommendation; upstream state must be re-verified before
any future decision.

| Crate         | Observed version in source | Observation date | Status here                                         |
| ------------- | -------------------------- | ---------------- | --------------------------------------------------- |
| `reqwest`     | `0.13.5`                   | September 2026   | Fits `1.85` per source; not adopted                 |
| `rmcp`        | `3.x`                      | September 2026   | Needs `1.88` per source; decision open, not adopted |
| `schemars`    | `1.2.2`                    | September 2026   | Derive direction noted; not adopted                 |
| `ignore`      | `0.4.33`                   | September 2026   | Walker direction noted; not adopted                 |
| `tree-sitter` | `0.27.0`                   | September 2026   | Needs `1.90` per source; decision open, not adopted |
| MCP protocol  | `2026-07-28`               | September 2026   | SDK support claim from source; not verified here    |

`jsonschema`, `tokio`, `tokio-util`, `futures-core`, `futures-util`,
`tracing`, `async-lsp`, `lsp-types`, `blake3`, `similar`, `rusqlite`, and
`tempfile` are named without versions in the source and carry no version
observation at all.

## Privacy and normative controls

This proposal must not contradict P0-AC-026 ([P0 Security Acceptance Criteria](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/p0-acceptance-criteria.md)),
PP-2 (Typed redaction), or PP-4 (No on-disk persistence without consent):

- PP-2 (Typed redaction) requires redaction before queuing. Adapter choice
  changes nothing: no provider, MCP, LSP, file-walker, parser, or store
  payload may carry inline secrets into diagnostics, traces, snapshots, or
  child environments, and configuration declares credential references, never
  values (MP-10 (API-key handling)).
- PP-4 (No on-disk persistence without consent) governs every cache, index,
  fingerprint store, and snapshot any adapter implies. A faster prefix cache,
  a shared LSP result, or a persistent evidence sketch never authorizes
  durable recording on its own.
- Minimization (PP-1, where referenced by the architecture) still prefers the
  smallest budget-bound set. Dependency convenience never justifies sending
  more context than the task needs or widening any consent scope.
- Least privilege stays at dispatch per AG-4 (Least privilege at dispatch):
  adding an SDK must not add ambient filesystem, process, or network
  authority, and any new tool surface passes the unified backend from AIQ-33.

No clause here weakens the normative security corpus linked from
[AI Architecture](../architecture/ai-architecture.md). P0 trust boundaries stay release
blockers.

## v0.1 scope boundary

Consistent with the [v0.1 Implementation Profile](../product/implementation-profile-v0.1.md)
(single `bitty-ai-runtime`, `FakeProvider`, L0+L1, single-agent scope):

- In scope for v0.1 discussion: the std-only kernel direction, the
  dependency-inversion rule, the adapter boundary map as a planning aid, the
  split-only-on-real-boundary sequencing, and the identity-naming bridge
  input to AI-0013.
- Beyond-v0.1 proposals (not commitments): every adapter crate named above,
  including provider networking, MCP client support, schema validation
  crates, file traversal, parsers and grammars, LSP broker dependencies,
  fingerprint and diff crates, any store backend, and any MSRV bump. Network
  access, persistence, and LSP-backed intelligence are not in v0.1; each
  needs its own reviewed contract, OQ resolution, and evidence before any
  implementation claim.

This document adds zero dependencies to v0.1. The `Cargo.toml` std-only
posture stays until a later increment explicitly adopts an adapter.

## Runtime evidence

No implementation of this proposal is claimed. The source cites upstream
documentation and repository pages for version and protocol observations;
none was independently verified here. The sibling `bitty-ai` repository was
not inspected for this task (out of scope: this task must not touch the
`bitty-ai` repository), so no statement here describes sibling behavior. Any
future implementation requires the v0.1 authorization backend (AIQ-33),
redaction evidence under P0-AC-026, and code with tests in the owning
implementation repository; no sentence here implies that code exists.

## Open questions and risks

All choices below reuse existing identifiers; no new identifier is proposed.
Duplicate-check outcome against [AI Unresolved Questions](../product/ai-unresolved-questions.md):
MSRV and adapter-timing questions are facets of existing transport,
intelligence, persistence, and serialization items, not independent
questions. Version observations create no new OQ.

1. Which MSRV path covers MCP adoption: project-wide bump, adapter
   isolation, or a compatible SDK generation? (Facet of AIQ-36.)
2. Which MSRV path covers parser adoption when code intelligence lands?
   (Facet of the code-intelligence prerequisites AIQ-41 through AIQ-48.)
3. Which store backend, if any, survives the persistence-profile decision,
   and what MSRV and indexing consequences follow? (Facet of AIQ-53 with
   scope AIQ-5C.)
4. Which reviewed backend enforces per-action capability, target, consent,
   isolation, and budget for every native and MCP effect once adapters
   exist? (AIQ-33; execution ownership AIQ-38.)
5. How are MCP schema staleness, cache invalidation, and changed-effect
   authorization handled once an SDK is in play? (Facet of AIQ-08 with
   AIQ-36.)
6. What are the canonical serialization, stable-prefix ordering, cache-key,
   and routing-scope rules that adapters must reuse rather than reinvent?
   (AIQ-12, AIQ-13.)
7. Risk: an adapter SDK pulls ambient authority (filesystem, process,
   network) into the runtime. Mitigation: keep adapters behind TB-3
   (Validation before dispatch), TB-4 (Capability and consent per tool), and
   AG-4 (Least privilege at dispatch) with fail-closed tests.
8. Risk: supply-chain breadth grows unchecked once adapters land.
   Mitigation: split only on a real dependency boundary, prefer the smallest
   maintained surface that covers the protocol, and record each adoption with
   its own review and lockfile evidence.

## Related specifications

- [AI Architecture](../architecture/ai-architecture.md) (Draft): MP-3 (Local-first default),
  MP-10 (API-key handling), TB-1 (MCP as adapter), TB-3 (Validation before
  dispatch), TB-4 (Capability and consent per tool), AG-4 (Least privilege at
  dispatch), AG-5 (Orchestration versus execution), CP-5 (Budget), PP-2
  (Typed redaction), PP-4 (No on-disk persistence without consent).
- [v0.1 Implementation Profile](../product/implementation-profile-v0.1.md) (Draft):
  single-crate zero-new-dependency scope gate marking every adapter in this
  proposal as post-v0.1.
- [Command and Tool Architecture](../architecture/command-tool-architecture.md) (Draft):
  Core-versus-Lua boundary and AIQ-33, AIQ-36, AIQ-38.
- [Agent Coordination Architecture](../agent/agent-coordination.md) (Draft):
  supervision and execution ownership under AG-4 and AG-5.
- [Code Intelligence Architecture](../agent/code-intelligence.md) (Draft): broker,
  fingerprint, and reuse prerequisites AIQ-41 through AIQ-48.
- [Persistence and Evidence Architecture](../persistence/persistence-evidence.md) (Draft):
  representation, backend, retention, and replay prerequisites AIQ-51 through
  AIQ-5C.
- [AI Unresolved Questions](../product/ai-unresolved-questions.md) (Draft): AIQ-08,
  AIQ-12, AIQ-13, AIQ-33, AIQ-36, AIQ-38, AIQ-41 through AIQ-48,
  AIQ-51 through AIQ-5C reused; no new identifier proposed.

## Evidence and verification boundary

This specification distills one workspace research record plus critical
judgment. It is not implementation evidence. Acceptance requires independent
review, and any future adapter adoption requires:

- A reviewed contract showing the adapter sits outside the kernel, passes
  validation, authorization, budget, and redaction at the boundary, and adds
  no ambient authority, with fail-closed tests.
- Re-verified upstream version, MSRV, and protocol observations with
  lockfile evidence, not reliance on September 2026 notes.
- Privacy evidence that redaction-before-queue, minimization, and
  consent-gated recording hold with the adapter enabled.
- Runtime code and tests in the owning implementation repository; no promise
  here implies that code exists.
