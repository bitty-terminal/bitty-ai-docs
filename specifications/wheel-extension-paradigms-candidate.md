---
title: Wheel extension paradigms (candidate)
description: Candidate Wheel extension paradigms with verified real, candidate, and unimplemented labels
category: specifications
audience: mixed
document_type: specification
status: draft
website_publish: false
sidebar_order: 70
---

# Wheel extension paradigms (candidate)

## Purpose and scope

This is a **draft candidate design record**, not a runtime specification,
accepted decision, or implementation claim. It records four extension
paradigms for Wheel — tool packs, model providers, interface and dashboard
composition, and strategy — and labels each paradigm element as observed
behavior, candidate direction, or unimplemented discussion. Labels are
verified against the owning implementation repositories; anything not
observed is kept explicitly out of the observed set.

Wheel is the official Agent Harness plugin for the Bitty terminal, scoped to
software engineering only. That role is real, and the harness itself is a
pre-implementation scaffold: a static manifest, one Lua entry point, and a
quality gate, with no harness behavior shipped. Nothing here promotes a
sketch to an API, a package, a protocol, or a release commitment, and
nothing here overrides the accepted
[IPC and Agent RFC](ipc-agent-rfc.md) or the normative security corpus.

This document creates no AIQ or OQ identifier and closes none.

## Status vocabulary

| Status        | Meaning in this document                                                                                            |
| ------------- | ------------------------------------------------------------------------------------------------------------------- |
| Real          | Observed in the owning implementation repository. Experimental-slice evidence only, never a shipped-behavior claim. |
| Candidate     | Proposed by a design direction only; no review has accepted it.                                                     |
| Unimplemented | Named in discussion but with no mechanism anywhere; explicitly not claimed.                                         |
| Owner-pending | Belongs to another repository owner; recorded here as a pointer, never as content.                                  |

## Wheel position and harness role

Status: **real role, scaffold implementation.**

Wheel is the official Agent Harness plugin for the Bitty terminal
(`bitty-terminal.wheel`), scoped to software engineering: its task domain is
always software development, and roles vary by configuration rather than by
class explosion. The scaffold carries a static manifest, one Lua entry
point, and a CI quality gate. The manifest requests only the notification
capability behind the generated example command: no filesystem, process,
network, clipboard, terminal-input, or persistent-state authority, and no
install-time code execution. Sibling future plugins for provider and model
management, the agent dashboard, and other surfaces are separate
repositories, never this one. Repository existence, a manifest, or a
proposed file tree is not evidence of usable plugin behavior.

One generic background rule is cited here without becoming a Wheel claim.
The plugin-ecosystem direction states that Lua decides policy and
composition while Rust enforces capability and mechanism: plugins choose
what should happen and compose host-provided services, while the host
decides whether it may happen and how it is bounded
([Plugin system](https://github.com/bitty-terminal/bitty-plugins-docs/blob/main/extensibility/plugin-system.md)).
That sentence describes the ecosystem's layering posture. It is not a Wheel
contract, and no Wheel-specific paraphrase of it is stated anywhere in this
document.

## Headless container invariant

Status: **real invariant, scaffold wiring.**

Under the execution-ownership direction, the execution context is primary:
panels describe presentation and view projections, never execution
authority. Every Wheel agent defaults to running in an isolated headless
panel working container whose identifier embeds the agent name, with the
headless flag defaulting to true. Presentation panels are optional
observation projections onto that container, never a second execution
authority.

**Critical judgment:** the invariant above is the stable claim. Placement
semantics, lifecycle coupling, and reconciliation rules stay with
[Execution ownership R1](../architecture/execution-ownership-r1.md) and
[Panel environment awareness](../interfaces/panel-environment-awareness.md).
This section describes the container invariant only; lifecycle machinery is
described separately below, and no combined construct is stated.

## Task lifecycle machinery

Status: **real state machine, scaffold agents.**

The slice-layer kernel is a runtime orchestrator coordinating four planes:
transactional content storage as the data plane, the task graph engine as
the control plane, the Merkle context tree as the compilation plane, and the
action pipeline as the execution plane. Session-only material such as recent
actions and the active task identifier resets on every open and never
survives a reopen; durability covers content, checkpoints, references, and
tasks only.

Task execution follows a seven-state machine: a task with unfinished
prerequisites waits; a task whose prerequisites are all satisfied is ready
to dispatch; a dispatched task runs; a task whose prerequisites failed or
were cancelled is blocked; finished tasks are succeeded, failed, or
cancelled, and only those three states are terminal. Completion, failure,
and cancellation carry generation fencing so stale transitions fail closed.

**Critical judgment:** the orchestrator shape and the state machine above
are observed. Agent role loops, autonomous turn execution, streaming
callbacks, and checkpoint publishing exist only as scaffold Lua with no host
wiring and are not behavior evidence. Coordination semantics stay with
[Agent Coordination Architecture](../agent/agent-coordination.md) and
[Task lifecycle R5](../architecture/task-lifecycle-r5.md).

## Tool packs

Tool contribution is the most implemented paradigm: the declaration,
registry, and execution seam are all observed, while pack distribution is
not.

Status: **real mechanism, candidate packaging, unimplemented distribution.**

Real today: a bounded tool declaration carries a name, a human-readable
description, an opaque bounded JSON Schema for arguments, a required
capability scope, and a side-effect-free flag, with the default dispatch
profile admitting only side-effect-free tools at the inspect tier. The
registry refuses duplicates fail-closed with no replacement and no
shadowing, refuses malformed or over-bound declarations with typed errors,
and refuses further registration past the per-session bound; every refusal
leaves the registry unchanged. The runtime never executes a tool: execution
belongs to a separate host and runtime seam with a deterministic test peer,
and dispatch is presented with an attribution handle the runtime issues.
The scaffold Lua registry mirrors this shape at illustration level:
registration requires a name plus an execute function, listing returns name,
intent, description, and parameters, a default core-tool set is registered
at construction, and role gating keeps read-only roles fail-closed.

Candidate: distributing tools as packs that projects compose declaratively,
with same-named tools from different provenances resolving under
distinguishable identity and correctly scoped permission.

Unimplemented: any pack registry, distribution channel, versioning, or
installation surface.

## Model providers

Provider abstraction is observed as a trait and test peer; provider choice
as a named service surface is not observed anywhere.

Status: **real trait, scaffold adapters, unimplemented service surface.**

Real today: a provider trait mediates authorized model requests only,
backed by descriptor, message, tool-call-request, sampling, reasoning, and
turn shapes with validation, plus a scripted deterministic test peer for
offline and CI use. The transport direction keeps the frozen turn surface
separate from any future adapter envelope.

Scaffold only: in-memory mock, commercial REST and streaming adapters, and a
subprocess CLI runner, with a line-buffered streaming parser and per-vendor
tool-schema formatters. None of this is wired to a host and none of it is
behavior evidence.

Unimplemented: a named model-provider service surface. No such namespace
exists in the runtime crates, the slice harness, or the scaffold, so this
document proposes no service name, registration call, or routing rule for
one.

Candidate: provider-shaped tool contribution and a provider-observability
pointer (cost, cache-hit, input, context, and timing signals flowing to a
plugin surface), owned by a future adapter task that also owns every
endpoint, field, and metric contract.

## Interface and dashboard composition

Surfaces observe execution; they never own it. Queryable state is
observed, event plumbing beyond in-memory scaffold illustration is not.

Status: **real query surface, scaffold plumbing, candidate dashboard.**

Real today: the kernel client speaks task, slot, checkpoint, action, and
context-compile commands over its dispatch boundary, and the task engine
exposes task views for listing and inspection. Those queries are the only
observed read path from execution state to an observer.

Scaffold only: an in-memory context event bus with subscribe, unsubscribe,
and publish verbs, and an ASCII task-graph and telemetry renderer. Both are
unwired illustration.

Candidate: the dashboard as an installable plugin fed by events, with
interface independence from agent placement (an agent completes a scoped
task headless while its interface renders from another panel, with no
agent-side interface dependency) and a minimal-default posture in which a
minimal installation can omit interface plugins entirely.

Candidate UI independence (direction only, no mechanism): one UI may front
agents across several panels, complementing the headless-agent-elsewhere-UI
case ([Wheel-core and Plugin-boundary candidate design](wheel-core-plugin-boundary-candidate.md#ui-dashboard-tools-and-memory-as-installable-plugins)).
The dashboard — local or remote — is a projection, not a runtime: it renders
the same agent panels, runs no model, and hosts no agent
([Wheel-to-runtime coupling (candidate)](wheel-to-runtime-coupling-candidate.md#remote-frontend-candidate)).

Candidate interaction surfaces (direction only, no mechanism): a headless-panel
agent keeps two full interaction lines plus one projection surface. The CLI
line is observed only as an implemented drill — the `wheel_stdio_host -- --db`
stdio host runner (bitty-ai AI-0170) exposing the bridge over stdin/stdout for
integration drills, never a product surface — and the IPC line is the bridge
JSON-RPC dispatch carried over the `bitty-ipc` frame. The GUI is
projection-only: it renders agent panels and forwards commands, runs no model,
and hosts no agent
([Wheel-to-runtime coupling (candidate)](wheel-to-runtime-coupling-candidate.md#remote-frontend-candidate)).
Layering follows the posture cited above — Lua decides policy and composition
while Rust enforces capability and mechanism
([Plugin system](https://github.com/bitty-terminal/bitty-plugins-docs/blob/main/extensibility/plugin-system.md)) —
and this extends the installable-plugin direction for tool providers, UI
flavors, and memory in the companion boundary design with no new mechanism
([Wheel-core and Plugin-boundary candidate design](wheel-core-plugin-boundary-candidate.md#ui-dashboard-tools-and-memory-as-installable-plugins)).

Candidate lifecycle decoupling (vocabulary only): a closed panel does not
unload its parent plugin; the agent survives closure with an explicit
transition; panel suspend and resume are host operations with stale handles
failing closed; agents may relocate across panels and panels may sit
agentless. Panel and agent stay independent entities joined by association —
attach, spawn-headless, release, reattach
([Wheel-to-runtime coupling (candidate)](wheel-to-runtime-coupling-candidate.md#the-lifecycle-decoupling-table);
[Wheel-config and Git-model candidate design](wheel-config-and-context-git-model-candidate.md#wheel-modes)).

Unimplemented: any event wire format, subscription contract, or dashboard
plugin.

## Strategy through context hooks

Compilation machinery is observed; hook points on it are candidate
vocabulary with no registration or invocation mechanism.

Status: **real compiler, candidate hooks, unimplemented mechanism.**

Real today: a three-zone context compiler separates the invariant stable
prefix (kept byte-stable for vendor prefix-cache reuse) from structured
cognitive state and the dynamic turn tail, under a multi-tier budget
pipeline of scratchpad pruning, rationale compression, and tail truncation.
Named context slots live in a Merkle tree with deterministic digests, tree
diffing, and three-way semantic merging, with optimistic concurrency
control on updates.

Candidate: strategy expressed as hook points across task, tool, compile,
and spawn sites with explicit grading, so ordinary project configuration
cannot silently break compiler invariants: safe hooks any project may use,
advanced hooks requiring review, and capability-gated hooks that demand a
grant. The accompanying rule is policy over code: users declare context
policy (budgets, preferences, pinning, exclusion) rather than hand-rolling
per-prompt compiler algorithms, and every rule the runtimes enforce
deterministically is a rule the core prompt no longer has to carry.

Unimplemented: hook registration, invocation, ordering, and grading
enforcement. No hook machinery exists in the crates, so this document
proposes no hook name, signature, or tier enforcement.

## Security review

- Read-only by default: the default dispatch profile admits only
  side-effect-free tools at the inspect tier, and role gating keeps
  read-only roles fail-closed.
- Least privilege at dispatch: every tool declares its required capability
  scope, duplicates and malformed declarations fail closed without partial
  state, and authorization hooks deny by default.
- No ambient authority for project or plugin Lua: configuration and
  strategy code reaches process, filesystem, or network only through
  mediated functions, never directly.
- Providers, tool results, stored context, prompts, and external references
  are untrusted until a narrow capability or policy grants access; model
  output is observation data, never instructions.
- A safe startup path works without third-party providers, and the minimal
  installation omits every optional plugin without losing the harness.
- Generation fencing on task transitions means stale strategy or
  observation handles fail closed rather than rebinding implicitly.

## Relation to existing systems

The draft [AI Architecture](../architecture/ai-architecture.md) layered
models are candidate inputs only; no paradigm sketch here is accepted by
this candidate design and none of it may be read as a crate, package,
protocol, file-schema, or release decision. Tool shape and transport
placement stay with
[Command and Tool Architecture](../architecture/command-tool-architecture.md)
and [Tool transport R2](../architecture/tool-transport-r2.md); provider
questions stay with
[Provider plugin boundary](../providers/provider-plugin-boundary.md);
execution and environment questions stay with
[Execution ownership R1](../architecture/execution-ownership-r1.md) and
[Panel environment awareness](../interfaces/panel-environment-awareness.md);
coordination questions stay with
[Agent Coordination Architecture](../agent/agent-coordination.md) and
[Task lifecycle R5](../architecture/task-lifecycle-r5.md); compilation and
budget questions stay with
[Context Management Architecture](../context/context-management.md) and the
companion
[Quality-formula and Context-Compiler candidate design](quality-and-context-compiler-candidate.md);
core-versus-plugin layering and composition stay with the companion
[Wheel-core and Plugin-boundary candidate design](wheel-core-plugin-boundary-candidate.md).
Placement, binding, and lifecycle questions stay with
[Execution-projection binding (candidate)](execution-projection-binding-candidate.md),
[Panel and agent workspace boundary (candidate)](panel-workspace-candidate.md),
[Shared workspace services and agent coordination (candidate)](shared-workspace-services-candidate.md),
and
[Wheel-to-runtime coupling (candidate)](wheel-to-runtime-coupling-candidate.md).
Each is referenced, never duplicated or modified.

The accepted [IPC and Agent RFC](ipc-agent-rfc.md) defines the only
accepted IPC wire, scope, and Agent vocabulary. Every sketch name in the
paradigm directions (pack names, service names, event names, hook names,
field names) is a discussion sketch: this draft records it as input and
proposes no file schema, API, command, tool, event, or wire format.

Normative security obligations (least privilege, per-action scopes,
capability-based and auditable permission that fails closed, typed
redaction, consented recording, secret minimization) override every
discussion example throughout this document. The paradigm sketches are
conceptual vocabulary, not an adopted authorization or distribution
contract. This draft creates or closes no AIQ or OQ identifier; open
questions stay with
[AI Unresolved Questions](../product/ai-unresolved-questions.md) and shared
governance.

## Open items

These are future evidence requirements, not tests executed by this
documentation task. They keep each paradigm falsifiable before any of them
constrains `bitty-ai`.

| Campaign           | Required observation                                                                                                                                             |
| ------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Tool packs         | A contributed tool pack resolves with distinguishable provider identity and correctly scoped permission, with duplicates still refused, in reviewable tests.     |
| Provider surface   | A provider change ships with no Wheel change and no caller-visible difference, against a real adapter rather than the mock, in reviewable tests.                 |
| Dashboard events   | An agent completes a scoped task headless while its interface renders from another panel, with no agent-side interface dependency, in reviewable tests.          |
| Strategy hooks     | A declared hook changes compiled output identically to its hand-rolled equivalent, and a capability-gated hook without its grant is denied, in reviewable tests. |
| Headless placement | Execution authority stays with the container while projections come and go, with no projection able to recreate or steer execution, in reviewable tests.         |
| Lifecycle fencing  | A stale task transition is refused and a blocked task records its blocking prerequisite explicitly, in reviewable tests.                                         |
| Minimal default    | A minimal installation runs the harness while an extended composition adds exactly its declared set, in reviewable tests.                                        |

Promotion needs independent AI architecture, context-management, terminal
and plugin-owner, docs-curator, and security review. Route pack shapes,
service names, event contracts, hook tiers, and placement semantics to
scoped owner tasks. This draft changes no normative contract and authorizes
no product code.

## References

- [Plugin system](https://github.com/bitty-terminal/bitty-plugins-docs/blob/main/extensibility/plugin-system.md) (Owner-pending): generic background for the policy and composition versus capability and mechanism posture; cited, never restated as a Wheel claim.
- [Wheel-core and Plugin-boundary candidate design](wheel-core-plugin-boundary-candidate.md) (Draft): composition layers, distribution direction, and the `.wheel` composition role; referenced, never duplicated.
- [Wheel-config and Git-model candidate design](wheel-config-and-context-git-model-candidate.md) (Draft): configuration classes, trust, and the portable-versus-native split; referenced, never duplicated.
- [Quality-formula and Context-Compiler candidate design](quality-and-context-compiler-candidate.md) (Draft): compiler-side observability direction; referenced, never duplicated.
