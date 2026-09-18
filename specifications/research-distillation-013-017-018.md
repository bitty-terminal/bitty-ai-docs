---
title: AI research distillation from 013, 017, and 018
description: Traceable draft synthesis of AI-relevant research findings and unresolved questions
category: specifications
audience: mixed
document_type: research
status: draft
website_publish: false
sidebar_order: 20
---

# AI research distillation from 013, 017, and 018

This draft records AI-relevant findings from research notes `013`, `017`, and
`018` (read 2026-09-14). The sibling `bitty-ai` repository exists;
commits `b6d3d6c` and `3623c6b` record its documentation gitlink and an
experimental vertical-slice implementation, respectively. This is a discussion
input, not an accepted contract. Observations below identify inspected source;
historical assessments and proposals are not current capability evidence.
The [coverage ledger](../docs/sources/research-coverage-ledger.md) binds source line ranges to
content hashes and distinguishes exclusions, corrections, and unresolved work.

## Stable boundary candidates

The records consistently propose an independent `bitty-ai` runtime: Bitty
Terminal supplies terminal, panel, process, rendering, and transport mechanisms;
the AI runtime supplies model/context policy, agent state, tools, permissions,
and semantic events. This document must not be read as a product capability
claim.

Candidate layering is `core contracts -> model/context/tools -> runtime ->
store/protocol -> CLI`. Keep provider SDK, MCP, database, and terminal I/O out
of the lowest contracts layer. Workspace and UI ownership remain boundary
questions, not settled facts.

## Runtime and context proposals

- Prefer a deterministic, serializable, sans-I/O agent state machine with an
  external driver for model, tool, context, persistence, and client I/O.
- Treat events as first-class observations: model streaming, tool proposals and
  results, permission requests, usage, cancellation, and context changes.
- Assemble context from typed items carrying provenance/trust, sensitivity,
  freshness, priority, and token estimates; budget, deduplicate, redact, and
  compress before provider formatting.
- A code-intelligence tool family could expose project inspection, symbol/range
  reads, references, diagnostics, transactional edits, formatting, and semantic
  rename. Tree-sitter is suited to syntax; LSP to semantic operations; search
  remains a fallback. `ctxctl` and Aider RepoMap are research references, not
  dependencies or accepted interfaces.
- Batch independent tool calls and keep dependent operations serial. Tool-call
  count, model round trips, and context bytes are distinct optimization metrics.

## Providers, tools, and security

The records propose using mature provider/MCP/ACP libraries behind Bitty-owned
traits, while preserving replaceability. These library choices are unaccepted
until license, version, API, and security review. MCP tools must pass the same
capability and permission pipeline as native or remote tools. Permission must
remain capability based, scoped, auditable, and fail closed; proposed effect
analysis refines these required controls. Sandbox and
execution mechanisms may live in Bitty while AI owns policy.

Context collection itself is an attack surface: model output, repositories,
configuration, tool results, and external references are untrusted data.
Secrets must use credential references and filtered environments rather than
being placed in prompts, logs, or durable events. Retry policy must account for
idempotency and must never blanket-wrap destructive tools.

## Persistence and release scope

The records propose SQLite plus FTS5 and an append-only event model before
embeddings/vector memory. A first vertical slice is proposed from input through
context, streaming model, permissioned tool call, result, final answer, event
persistence, and replay. Multi-agent coordination, RAG, browser/voice/image
features, distributed execution, and semantic long-term memory are explicitly
future discussion items.

## Evidence and disagreements

Security requirements retain the authority of the
[normative sources](../architecture/ai-architecture.md#normative-sources-this-specification-must-not-weaken).
The proposed runtime design cannot relax read-only defaults, consent, project
trust, resource budgets, secret minimization, or host-side enforcement.
013's fixed context-byte limit is not a new global limit: the current
[context contract](../architecture/ai-architecture.md#purpose-and-scope) is token-first, with
the byte default a candidate profile. Neither 018's proposed v0.1 schedule nor
the terminal-facing architecture's historical post-1.0 scope decides the
standalone AI release profile. Persistence/replay requirements remain unresolved;
neither an ephemeral v0.1 nor a post-1.0 persistence deferral is selected here.

## Primary-source inspection ledger

Read-only inspection on 2026-09-14 ran `git rev-parse HEAD` and
`git status --short` inside each available local reference clone. All three had
clean working trees. Revision plus path identifies immutable source.
No reference code was executed. License observations apply to the inspected
files, not every bundled dependency or asset.

| Snapshot     | Full Git revision                          | License files read                                                                                                               |
| ------------ | ------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------- |
| Hermes Agent | `b6b53c69a6ed49cb099cf1bfe76b5e6edd718e5a` | `LICENSE:1–21`, MIT, Nous Research                                                                                               |
| OpenCode     | `95daf90670b7c039c436c85537da5fbfe2205b41` | `LICENSE:1–21`, MIT, opencode                                                                                                    |
| Oh-My-Pi     | `165cede8400d833382177a4dfd2901271c9f0a2c` | `LICENSE`, `packages/agent/LICENSE`, `packages/coding-agent/LICENSE`, each `1–23`, MIT, Mario Zechner / Can Bölük / Stencil Labs |

| Inspected source                                                                                     | Observation                                                                                                                                                                                         | Consequence for the proposal                                                                                                                                                                         |
| ---------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Hermes `agent/turn_tool_round.py:45–140`, `run_tool_round`                                           | Validates and deduplicates calls, stages the assistant message, and persists before effects; failed persistence ends the round.                                                                     | Keep durable intent before effects; an event log alone does not guarantee exactly-once execution or safe replay of tools. This slice does not validate the recording's concurrency counts.           |
| Hermes `agent/conversation_loop.py:1176–1219`, `_apply_context_engine_selection`                     | Optional context hook receives history clones and a token budget; exceptions/invalid results fall back to the original request.                                                                     | Separate optional ranking/compression from mandatory redaction and budget enforcement. Never inherit this fail-open fallback for a security filter.                                                  |
| OpenCode `packages/core/src/permission.ts:76–90,131–162,190–230`                                     | Last matching action/resource rule wins, otherwise ask; missing agent permissions deny. Configured denial is checked before saved grants; assertion waits for approval and cleans pending requests. | Rule ordering and grant persistence need explicit semantics. Saved approvals must not override security ceilings. This is scoped permission evidence, not proof of every dispatcher or sandbox path. |
| Oh-My-Pi `packages/agent/src/agent-loop.ts:533–562,944–981`, `agentLoop` / `runLoop`                 | Copies initial context, emits an agent-start event, launches async loop work, passes an abort signal/stream function, and closes telemetry spans.                                                   | Useful event/driver seams, but this is an asynchronous implementation, not evidence of a serializable sans-I/O state machine.                                                                        |
| Oh-My-Pi `packages/coding-agent/src/tools/index.ts:461–518,622–714`, `BUILTIN_TOOLS` / `createTools` | Session configuration filters a registry of read/edit/bash/LSP and other factories; selected factories initialize concurrently and populate the registry.                                           | Separate tool composition/availability from invocation authorization. Concurrent factory creation does not prove concurrent tool execution or safe multi-file writes.                                |

Aider was genuinely unavailable locally: direct reads of its reference-clone
`.git` and `LICENSE` returned not found, and a workspace search found no
matches. Therefore 018's RepoMap ranking description
remains a secondary-source proposal; no Aider revision or license is asserted.
The actual Hermes guide is `website/docs/developer-guide/agent-loop.md`, not
the recording's abbreviated filesystem path.

### Current Bitty AI evidence

At `bitty-ai` HEAD `3623c6b3ce33e97c1c493109ec6356219d0c9722`,
`crates/bitty-ai-slice/src/session.rs:68–136` implements a provider call,
conditional bounded context collection, tool dispatch, and fragment emission.
This is real experimental code, not an empty scaffold. It does not prove the
proposed context-before-provider, tool-result/model-continuation, SQLite/replay
runtime. The [pressure-test specification](../product/ai-vertical-slice-pressure-test.md)
records its deterministic local provider and loopback host limitations; its
historical test results were not rerun for this documentation task.

Commit `b6d3d6c495607f6bb2660b5442cfefcf350b486e` added `.gitmodules`
and the mode-`160000` `docs` gitlink, still pinned at
`39b4c7568a8807e0940bd298660c972db0cfa92a`. The configured target names
the owning `bitty-ai-docs` repository and tracks `main`. Initial inspection
found an empty, uninitialized mount. After independent review, the commander
initialized the existing gitlink. Read-back verified the same full revision in
`docs`, its `origin` identifying `bitty-ai-docs`, initialized submodule status,
and a clean parent working tree. This checkout does not include the uncommitted
CTX-0003 drafts.

Claims about current crates, versions, vulnerabilities, protocol maturity,
local database measurements, and repository implementation status require
source-level verification before becoming normative. The records disagree on
whether provider transport should be a small set of hand-written protocols or
an adapter over a multi-provider library, and whether orchestration belongs in
Rust or Lua. Preserve both options for an explicit decision; do not infer a
decision from repetition.

## Open questions for discussion

1. Which contracts belong in `bitty-ai` versus a future shared platform crate?
2. Is the initial provider substrate an adapter over an existing library or
   narrowly owned protocol implementations?
3. What is the minimum stable ACP/IPC surface and who owns transport?
4. Which code-intelligence operations are safe read-only defaults, and what
   approval model governs edits, formatting, tests, and remote execution?
5. What event schema, retention/redaction policy, and replay guarantees are
   required for v0.1?
6. Which findings block the next milestone rather than remaining research notes?

## Additional coverage from the full records

The research proposes a **Rust kernel / Lua userspace** boundary: Rust owns
correctness, storage, process and network mechanisms, enforcement, IPC,
secrets, token accounting, and context/tool primitives; Lua composes harnesses,
prompts, workflows, hooks, routing, and context strategy. This is a proposal,
not a decision. A harness is composition policy, while an agent is a runtime
configuration/workflow; neither should weaken Rust enforcement.

The records propose `.wheel/` as declarative, versionable project intent and
policy. Proposed ordinary-setting precedence, from lowest to highest, is
**built-in → user → project → session/CLI** (018:3974–4024); later layers
override earlier ones only within non-overridable security ceilings. System or
distribution security policy, host limits, and capability/consent requirements
cannot be relaxed by any layer. A project declaration requests authority; it
does not grant it. Merge semantics and session-versus-CLI conflicts still need
a reviewed contract. Runtime state belongs in machine-local data storage,
with XDG data/state/cache separation rather than literal paths copied from the
recording. An untrusted
repository `init.lua` or MCP declaration must never execute automatically:
project trust, capability sandboxing, inspection, explicit approval, and
Rust-enforced policy are required. Configuration declares a server or secret
reference; it does not contain implementation or secret material.

Skills describe how to perform work, tools perform capabilities, and MCP
crosses a process boundary. Their configuration, implementation, and trust
models must remain distinct. Native local primitives, Lua composition, and
external MCP services are proposed categories, not an accepted API.
Project-local skills may carry reviewed guidance, scripts, and templates;
global/project shadowing and explicit `builtin:`, `user:`, and `project:`
namespaces are candidate composition rules. Project MCP server source may be
versioned alongside its declaration, but building or starting it is a separate
permissioned effect. The proposed Project Agent Manifest ties together agents,
harnesses, prompts, skills, tools, context sources and policies; its schema and
cross-repository ownership are not settled.

The records recommend tightening `cargo-deny` dependency policy (including
wildcards, default features, and documented informational advisory exceptions),
but this is an unaccepted security-hardening candidate. The proposed
cancellation tree is session → turn → model/tool/context child operations,
supporting cascade cancellation, bounded channels, and backpressure.

Tracing, OpenTelemetry, and GenAI semantic telemetry for model calls, tools,
context, latency, tokens, and cost are proposed observability work, not current
capability. Edit transactions should validate expected hashes, reject stale or
overlapping edits, apply atomically in memory, run permission and syntax
checks, and return compact verification results. The proposed post-edit
pipeline is parse → format → LSP diagnostics → lint → build/check → targeted
tests.

The staged code-intelligence roadmap in 018:2791–2832 is: syntax/project map,
inspection/search/read/graph, multi-file edits, and compact verification;
then LSP definitions, references, implementations, diagnostics, rename, and
code actions; later typed pipelines, persistent indexing, SCIP, and semantic
impact analysis. Earlier sections explore batching and pipelines without
settling their release stage. These are proposals, not implemented features.

Project maps could rank dependency neighborhoods under a token budget, then
read several original-source symbol/range slices with deduplication. Proposed
search modes are text, symbol, structural, and semantic. LSP, formatter,
linter, and build providers remain distinct capabilities; first discover and
configure user-installed tools, deferring a Mason-like installer. Evaluate
reuse of `ctx-symbol` / `ctx-exec` libraries rather than copying their source
or requiring the CLI subprocess, but API/license suitability remains open.
Typed local pipelines are proposed instead of arbitrary-code execution as an
initial batching mechanism, with permissions and budgets on every node.

013's native/embodied workspace vision remains speculative: semantic IDs and
command-completion events could connect bounded observations, reveal/focus
suggestions, panel leases, preserved work, and human handoffs. Rendering,
PTY ownership, and physical session restore remain terminal mechanisms;
agent/task/run identity and consent must not be inferred from panel occupancy.

Candidate libraries mentioned by the records include `genai`, `rig-core`,
`rmcp`, ACP Rust SDKs, `async-lsp`, Tree-sitter, `ast-grep`, `rusqlite`,
`tracing`, OpenTelemetry, `backon`, `secrecy`, `keyring`, and `tiktoken-rs`.
018:712–735 additionally names `tokio`, `tokio-util::CancellationToken`,
`futures`, `serde` / `serde_json`, `schemars`, `jsonschema`, `reqwest` /
`rustls`, and `cap-std`. Candidate test libraries are **`insta`** for event
snapshots, **`wiremock`** for provider contract mocks, and **`proptest`** for
parser, budget, and stream properties (018:731–733). These complement offline
fake providers/tools and adversarial permission/cancellation/replay tests;
mock success alone does not establish live-provider or sandbox correctness.
Token estimation should be provider-specific and distinguish estimates from
billed usage. Curated memory and searchable session history are separate;
`rusqlite` with a database worker is an initial candidate, SQLx a later
concurrency alternative, and embeddings are deferred.
They are evaluation recommendations only; no dependency, version, license, or
security decision is accepted here.

## 017 maturity and governance coverage

Record 017 describes M1 Terminal Truth and M2 Correct Terminal as implemented
or hardening assessments, M3 Usable Terminal as progressing, M4 Workspace/
Panel and M5 Plugin Ecosystem as early/immature runtime areas, M6 Rich/IPC/
Agent as partly implemented with AI design ahead of implementation, and M7
Stable Compatibility and M8 Public Beta as not started. These are dated
research assessments, not this repository's implementation status. Its
verification-first recommendation is to prioritize compatibility evidence,
ownership/lifecycle invariants, plugin validation, and a narrow AI vertical
slice over expanding design.

017 also flags stale project-state snapshots and an open-question register
that should admit new questions only when they block a milestone or respond to
implementation evidence; future ideas should remain research notes.
Its inventory separates agent role, model choice and capability; identifies
task/run/execution identity, workspace overlays and evidence provenance; and
raises CarryCtx backend boundaries and agent growth. These remain distinct
design topics in the existing architecture, not capabilities supplied by this
distillation's proposed event loop or by the experimental slice.
