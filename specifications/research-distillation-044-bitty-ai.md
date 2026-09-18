---
title: Execution-supervisor research distillation for bitty-ai (044)
description: Draft bitty-ai distillation of research 044 execution supervisor job service four object model owner subscriber lifetime timeout outcome boundary table
category: specifications
audience: mixed
document_type: research
status: draft
website_publish: false
sidebar_order: 55
---

# Execution-supervisor research distillation for bitty-ai (044)

## Status and recommendation

This is a **draft discussion synthesis**, not a runtime specification, accepted
decision, dependency selection, release commitment, or implementation claim.
It distills only the `bitty-ai`-relevant parts of workspace research record
`044.md` (an execution-supervisor and job-service design discussion): the
four-object model (Task, Execution/Job, Agent, Panel), Job independence from
Panel with mailbox routing, Owner plus Subscriber instead of single ownership,
agent-killed Job disposition through a spawn-time Lifetime, the pre-exit
quiescence check as harness mechanism rather than prompt discipline, the
long-task timeout model with the ExecutionOutcome versus TaskOutcome split, the
Job kind taxonomy, the non-interactive stdin default, supervisor-held output
instead of context stuffing, completion as a Critical Message, the three
network-partition classes, structured OOM and crash outcomes, no-default
retry, progress events that do not wake the model, input-needed as a
first-class event, the persistence split, the full `bitty` versus `bitty-ai`
mechanism-versus-semantics boundary table, the Job-as-wrapper principle with
the T128/J31/E77 worked example, and the v0.1 scope boundary. Harness-survey
material (OpenCode, Codex, Claude Code, Cursor, Kiro, pueue) was read for
design grounding and is distilled only where it constrains the `bitty-ai`
side; concrete tool, method, event, struct, and wire sketches stay unreviewed
vocabulary.

The recommendation is to treat every model below as a candidate input to the
draft [AI Architecture](ai-architecture.md) and its related draft
dispositions, never as an override of the accepted
[IPC and Agent RFC](ipc-agent-rfc.md) or the normative security corpus.
In particular, the `exec` and `job_*` tool sketches, the `execution.*` IPC
sketches, the `ExecutionResult` and `JobResult` struct sketches, the
`Lifetime`, `JobKind`, outcome, and cancel enums, the three-phase rollout,
and the proposed `execution-supervisor.md` or `background-execution-r7.md`
document are **not** accepted by this distillation. Nothing here is promoted
to accepted status, and no implementation is described as shipped.

The source's strongest ideas are the four-object separation with Panel as
projection only (which agrees with the R1 ExecutionContext-primary
direction), the Job-as-wrapper principle with the explicit negative (an
Execution is not inherently an AI Job), the mechanism-versus-semantics split
with two distinct generations (assignment generation on the `bitty-ai` side,
execution generation on the `bitty` side), the ExecutionOutcome versus
TaskOutcome split (a timeout is a fact below and a policy question above),
the no-default-retry rule for unknown or effectful outcomes (consistent with
R1/R5/R6), and the pre-exit quiescence gate as a harness invariant instead of
a prompt reminder. Its weakest claims are the concrete tool and wire sketches
(`exec`, `job_spawn`, `execution.spawn`, and similar), which are unreviewed
interface proposals with no ownership, versioning, or compatibility evidence;
the three-phase rollout, which is a staging opinion with no owner or
milestone evidence; cross-platform OOM attribution, which the source itself
admits needs per-platform backends that do not exist yet; and the control
console mockup, which is a view sketch without a data-contract or
authorization analysis. Those are corrected below.

This synthesis references, without duplicating or modifying, the draft
[AI Architecture](ai-architecture.md) (candidate inputs only),
[Execution ownership R1](execution-ownership-r1.md),
[Tool transport R2](tool-transport-r2.md),
[Panel environment awareness](panel-environment-awareness.md), and
[Provider plugin boundary](provider-plugin-boundary.md) (references only).
The accepted [IPC and Agent RFC](ipc-agent-rfc.md) is unaffected by this
draft. The 041 Capability Layer versus 044 capability-enforcement split stays
consistent with the unmerged-candidates comparison added under CTX-0047 in
the AI Architecture; this draft references that comparison and re-decides
nothing. This document creates no AIQ or OQ identifier and closes none.

## Provenance and evidence boundary

The source is research note `044` (origin), read on 2026-09-16: **2,630
lines**, **46,063 bytes**. Its SHA-256 is
`d00d7f6c5845d9964cd09bcf447c71f2759cf1322fec03a5da4d0e0d8d49c5d0`.
The record is untracked in the research repository, so provenance is by record
number plus fingerprint, not by commit. It was not renamed, edited, or staged by
this task; the `bitty`-side pass still needs it. The body of this document
distills the whole verified file (`044.md:1-2630`); no post-task append existed
at verification time, so no head-versus-tail split applies. Any later append is
uncovered and follows the CTX-0045 pattern (distill the verified head, record
the remainder as uncovered). The hash was verified at task start and re-verified
at task end with no change, so the CTX-0045 growth pattern did not trigger.

Unlike record 039, record 044 is a single pass with twenty-eight numbered
sections plus a four-point follow-up with its own mechanism-versus-semantics
split, boundary table, repository placement, IPC contract, and worked
example; no duplication handling applies. Only the ranges in the coverage
table were distilled. The source is a single-author Chinese-language
discussion with English code sketches; this document is the English-language
draft synthesis, not a translation. The fingerprint identifies the
discussion, not the truth of its claims.

### Pueue reference inspection

The pueue reference clone was read read-only for design grounding on
2026-09-16 at revision
`193ed2264338bd30a06e347b48183a8800bd178b` with a clean working tree. The
clone carries dual `LICENSE.MIT` (MIT, Arne Beer, 2018) and `LICENSE.APACHE`
(Apache-2.0) files. Inspected slices: `pueue_lib/src/task.rs`
(`TaskStatus` lifecycle, `TaskResult` outcomes, the `Task` struct with
environment snapshot whose `Debug` redacts `envs` as hidden),
`pueue_lib/src/state.rs` (`Group` with `status` and `parallel_tasks`,
daemon-held task and group maps), and `pueue_lib/src/message/request.rs`
(`PauseRequest.wait`). No reference code was executed. Pueue is used only to
ground the `bitty-ai` side (daemon-held results, group isolation, wait
semantics); the source argues borrow-design-not-backend, and this draft
proposes no pueue backend.

## Authority and reconciliation

The draft [AI Architecture](ai-architecture.md) layered models are
candidate inputs only; the Execution Supervisor, the three-phase rollout,
and the proposed supervisor document are **not** accepted by this
distillation and must not be read as crate, package, protocol, or release
decisions. Execution and environment questions stay with
[Execution ownership R1](execution-ownership-r1.md) and
[Panel environment awareness](panel-environment-awareness.md); tool
authorization and transport placement stay with
[Tool transport R2](tool-transport-r2.md); provider questions stay with
[Provider plugin boundary](provider-plugin-boundary.md). Each is
referenced, never duplicated or modified.

The accepted [IPC and Agent RFC](ipc-agent-rfc.md) defines the only
accepted IPC wire, scope, and Agent vocabulary. Every method, event, struct,
and enum name in the source (`exec`, `job_spawn`, `job_get`, `job_read`,
`execution.spawn`, `execution.exited`, `ExecutionResult`, `JobResult`,
`ExecutionOutcome`, `Lifetime`, `JobKind`, `CancelMode`, and similar) is a
discussion sketch: this draft records it as input and proposes no command,
tool, event, or wire format, consistent with the v0.1 scope boundary below.
Where the source's sketches overlap RFC-owned ground (Agent lifecycle, Agent
events, Agent semantics, scopes), the RFC wins without further argument.

Normative security obligations (least privilege, per-action scopes,
capability-based and auditable permission that fails closed, typed redaction,
consented recording, secret minimization) override every discussion example
below. The source's permission names (`execution.observe`,
`execution.read_output`, `execution.signal`, `execution.cancel`, and
similar), operation lists, and console fields are conceptual vocabulary, not
additions to any accepted registry, schema, or protocol. This draft creates
or closes no AIQ or OQ identifier; open questions stay with
[AI Unresolved Questions](ai-unresolved-questions.md) and shared
governance.

## Core thesis: a supervisor, not a stronger spawn

Source: `044.md:1-60` (the `pty_spawn` anecdote with exit notification; the
owner's Multi-Agent questions; the core-conclusion block).

The retained thesis is that Bitty should design an Execution Supervisor and
Job Service that is independent of Agent, Panel, and Conversation, rather
than a stronger `pty_spawn`. Spawning is one execution backend; the
first-class citizens are Job and Execution. The source positions this as the
missing layer under the R1 `Agent -> ExecutionContext <- Panel` direction
and the agent-coordination lifetime separation (mailbox, lease, generation
fencing, handoff, critical message): what 044 adds is how long-lived
processes and jobs are managed under the ExecutionContext.

**Critical judgment:** the thesis is retained as a candidate objective; the
opening harness anecdote is motivation, not evidence. The stable claim is
only the layering (supervision as infrastructure shared by agents, panels,
and future multi-agent, team, watcher, server, training, download, LSP,
debugger, dashboard, headless, and remote-execution work), with ownership,
packaging, and schedule entirely open.

## Harness survey as boundary context

Source: `044.md:61-356` (the six-row harness table; pueue
detachment-at-`76-148`; OpenCode event-driven notification at `149-224`;
Codex process protocol and polling critique at `225-268`; Claude background
tasks, Monitor, and session separation at `269-308`; Cursor Agent/Run
separation, event stream, resume cursor, and subscriptions at `309-356`).

The retained grounding points are: pueue's daemon-held tasks that survive
terminal close, with groups, dependencies, pause, resume, restart, wait,
durable logs, environment snapshots, and callbacks; OpenCode's exit
notification as proof that event-driven delivery beats polling loops;
Codex's process protocol (start, read, write, signal, terminate with output,
exited, and closed events) with the warning that model-side polling
regresses into round-trip and token waste; Claude's separation of process
jobs from agent tasks with Monitor-style event delivery; and Cursor's
Agent-versus-Run split with resumable event streams and subscriptions, which
the source adopts as the target shape (agents do not wait on jobs; agents
subscribe to jobs).

**Critical judgment:** study leads, not findings. The external references
are unverified discussion citations, and the per-harness deficiency claims
are author opinion. The stable claims are the waiting principle (waiting is
the runtime's work, not the model's) and the subscription direction; no
harness is selected as a backend, and pueue in particular is
borrow-design-not-backend per the source and the inspection above.

## Four-object model

Source: `044.md:357-411` (the Task, Execution/Job, Agent, Panel table; the
T-128 style composition diagram; the Panel-loss, Panel-switch, compaction,
and agent-absence invariance claims).

The retained ontology fixes four distinct objects: Task as semantic work
(such as fixing an issue), Execution/Job as a real OS process or process
tree, Agent as the deciding and working logical subject, and Panel as the
human observation and interaction projection. Panels observe jobs; agents own
and subscribe to them; conversations compact and agents relocate without the
job necessarily stopping. The source claims this matches the existing
identity separation already defined for agents, executions, workspaces,
services, evidence, and panels.

**Critical judgment:** the separation is retained as a candidate objective
consistent with R1; the invariance claims (job continues across panel loss,
agent switch, compaction, and even agent absence) are author proposals
conditioned on the Lifetime policy below, not guarantees. Lifetime scoping
decides each case.

## Job independence from Panel and mailbox routing

Source: `044.md:412-476` (the origin-panel versus owner distinction; the
observer list; the job-to-mailbox-to-session route).

The retained rule is that a Job does not belong to a Panel: creation may
record an origin panel ("started here") plus observers for UI purposes, but
completion routes to the Agent mailbox and then to the live Agent session,
never back to the origin panel. An agent that moved panels still receives
its own job completions.

**Critical judgment:** the routing direction is retained as a candidate
objective; the mailbox, session-delivery, and observer-list mechanics are
undecided and stay with the owning tasks. The stable claim is only the
negative (origin panel is not the owner) plus the delivery target (the
agent, not the panel).

## Owner plus Subscriber instead of single ownership

Source: `044.md:477-523` (the owner plus subscribers sketch; the
observe/read/signal/cancel/adopt/delete permission split; the lease reuse
note).

The retained model gives each Job one Owner and any number of Subscribers
(the owning agent, a team lead, a reviewer, the human UI), with observation
rights separated from control rights: visibility into a job never implies
the right to kill it. The source proposes reusing the existing lease
principle (coexisting read leases, a single fenced interactive writer with
generation-fenced transfer).

**Critical judgment:** the ownership shape is retained as a candidate input;
the permission vocabulary is unreviewed and enforcement placement stays with
the multi-agent security split below (`bitty` enforcement, `bitty-ai`
coordination). No permission is adopted here.

## Agent-killed disposition through Lifetime

Source: `044.md:524-593` (the four-variant Lifetime sketch with the
Agent, Task, Workspace, and Detached behaviors and examples).

The retained direction is that spawn declares the Job's lifetime scope, and
agent death follows the declared policy: Agent-scoped jobs cancel, Task-
scoped jobs continue while the task lives, Workspace-scoped jobs continue
while the workspace lives, and Detached jobs continue under the supervisor
independently. The source's examples (a test run as Task-scoped, a dev
server as Workspace-scoped, model training as Detached, a one-shot shell
command as Agent-scoped) are illustrations of the policy vocabulary, not
defaults.

**Critical judgment:** the explicit-policy direction is retained as a
candidate; the variant names, the example bindings, and any Detached
durability beyond process life are proposals. Detached jobs that outlive the
application need the persistence and daemon work that the v0.1 boundary
below explicitly defers.

## Pre-exit check as mechanism, not prompt discipline

Source: `044.md:594-654` (the quiescence-gate sketch; the wait, detach,
handoff, and cancel choices; the forced-stop fallback to Lifetime policy).

The retained rule is that an agent preparing to finish must pass a harness
quiescence gate: with no live owned jobs it may finish, and with live jobs
it must explicitly choose wait, detach, handoff, or cancel per job before
finishing normally. Forced stops fall back to the Lifetime policy. The
source's explicit point is kept: this must be a harness invariant, never a
system-prompt reminder to "remember to check background tasks."

**Critical judgment:** the invariant direction is retained as a candidate
objective consistent with R5 handoff and acknowledgement rules; the gate
mechanics, choice vocabulary, and forced-stop semantics are undecided and
need owning-task design plus security review before any enforcement claim.

## Long-task timeout model and the outcome split

Source: `044.md:655-723` (hard, idle, and retention separation with
training, test, download, and server examples; the rejection of a uniform
background default) and `044.md:2264-2366` (supervisor-held deadlines with
the `ExecutionLimits` sketch; `bitty-ai` policy selection; the
Timeout-versus-Failed analysis with the `ExecutionOutcome::TimedOut`
proposal and the ExecutionOutcome versus TaskOutcome split).

The retained model separates three clocks: a hard timeout, an idle timeout,
and a retention TTL, set per workflow (training with no hard or idle limit
and a multi-day retention, tests with a tens-of-minutes hard limit,
downloads with a long hard limit plus a short idle limit, servers with no
hard limit and a service kind). Deadlines must be held by the Execution
Supervisor so they survive agent disconnection. The load-bearing split is
kept: a timeout is recorded below as `ExecutionOutcome::TimedOut` (a fact),
while what it means above (task failed, blocked, paused, retried, or normal
policy termination) is a `bitty-ai` policy decision, never a folding of
timeout into generic failure.

**Critical judgment:** the clock separation and the outcome split are the
strongest timing claims and are retained as candidate inputs; the limit
values, struct shape, and per-kind bindings are unreviewed sketches. No
default timeout policy is adopted: a uniform background kill rule is
explicitly rejected.

## Job kind taxonomy

Source: `044.md:724-765` (the Command, Interactive, Service, and Watch
sketch with the cargo-test, training, REPL, dev-server, and CI-watch
examples).

The retained direction is that Jobs carry a kind so the runtime holds the
right expectation: long-lived Watch and Service jobs are normal, not stuck.
The examples are illustrations of the vocabulary, not a kind registry.

**Critical judgment:** a candidate heuristic, not a taxonomy decision. Kind
names, semantics, and any scheduling or limit consequences per kind are
open; kinds must not silently widen authority or change enforcement.

## Non-interactive stdin default

Source: `044.md:766-808` (the closed-stdin default with the background-stdin
hang and orphan-process motivation; PTY plus writable stdin only for
interactive jobs).

The retained rule is that non-interactive Jobs default to no TTY and closed
stdin (pipes for standard output and error), with PTY and writable stdin
only for explicitly interactive Jobs. The source presents this as both a
hang-avoidance and a safety default.

**Critical judgment:** the default-closed direction is retained as a
candidate consistent with least privilege; the exact spawn-options shape is
undecided. Unexpected prompts in background tasks should surface as failure
or an input-needed event (below), never as a silent permanent hang.

## Supervisor-held output, not context stuffing

Source: `044.md:809-849` (the ring-buffer plus persisted-log sketch; the
bounded completion payload with evidence reference; the explicit-read and
filtered-read direction).

The retained model keeps execution output with the supervisor (a bounded
ring buffer plus an optional persisted log) and delivers only a bounded
completion summary (identifier, exit code, line count, tail, evidence
reference) to the agent, which then explicitly reads tails or filtered
slices on demand. Full logs are never pushed into agent context by default.

**Critical judgment:** the retention-and-reference direction is retained as
a candidate consistent with R6 file-held artifact bytes; buffer bounds, log
placement, filter vocabulary, and retention policy are entirely open and
stay with the owning tasks.

## Completion as a Critical Message

Source: `044.md:850-918` (the R5 three-outcome reuse; stable event
identifiers with at-least-once delivery plus idempotent handling; the
Unknown rule for crash between effect and acknowledgement).

The retained rule routes job completion through the existing critical-message
machinery: accepted into the mailbox, delivered to a live recipient, and
processed with acknowledgement stay three distinct outcomes; delivery is
at-least-once with stable event identifiers and idempotent handling rather
than exactly-once; and a crash between an effect and its acknowledgement
yields Unknown, never a fabricated result. The source explicitly presents
this as reuse of R5/R6, not a new mechanism.

**Critical judgment:** retained as a consistency note, not a new contract.
All critical-message semantics stay owned by R5; this draft adds only the
consequence that job completion events belong on the critical path while
progress observations do not.

## Three network-partition classes

Source: `044.md:919-997` (provider disconnection at `923-947`; job-network
failure at `948-976`; Bitty IPC disconnection with resume cursor at
`977-997`).

The retained distinction is threefold. Provider disconnection must not
disturb running jobs: the supervisor continues and the agent finds
completions in its mailbox on recovery, which is the point of an
independent supervisor. Job-network failure must not be invented by the
supervisor: unless the OS or tool reports it, the record is exit code,
signal, standard error, and timeout for the agent to interpret. IPC
disconnection between executor and AI runtime must retain events for
cursor-based resume on reconnect.

**Critical judgment:** the partition taxonomy is retained as a candidate
input; the resume-cursor mechanics, mailbox durability across the outage,
and any retry that follows are undecided. No network-error inference by the
supervisor is adopted.

## Structured OOM and crash outcomes

Source: `044.md:998-1052` (the outcome-enum sketch; the only-if-certain OOM
rule with per-job cgroup as a future Linux backend; the per-platform
process-management split; the kill-the-owned-tree rule).

The retained direction is that abnormal endings get structured outcomes
(success, exit code, signal, spawn failure, cancellation, timeout, OOM,
supervisor loss, unknown) instead of a bare exit code, with OOM recorded
only when positively determined. Platform backends differ (process groups
with pidfd and optional cgroup v2 on Linux, process groups with wait
primitives on macOS, Job Objects with ConPTY on Windows), and cancelling a
job must terminate the owned process tree, never a single PID.

**Critical judgment:** the structure direction is retained as a candidate;
the enum shape is an unreviewed sketch and OOM attribution currently has no
backend evidence (the source itself notes the cgroup path is a deferred
candidate that must not be presented as a cross-platform guarantee).
Killing by process-tree rather than PID is consistent with existing
supervision guidance.

## No-default retry

Source: `044.md:1053-1110` (the effectful-command counterexamples; the
idempotent-or-approved precondition; the Unknown-inspect-first rule).

The retained rule is that retry defaults to none: re-running is safe for
some commands and destructive for charges, deletions, applies, pushes, and
migrations, so automatic retry needs either an explicit idempotency
declaration or explicit user or agent approval, and Unknown outcomes require
inspection before any retry. The source presents this as preservation of the
existing R1/R5/R6 stance.

**Critical judgment:** retained as a consistency note with a load-bearing
consequence: no background runner may weaken the existing retry and Unknown
rules. Idempotency vocabulary, approval mechanics, and backoff policy are
open.

## Progress events must not wake the model

Source: `044.md:1345-1410` (the training-epoch motivation; the Observation
versus Critical split; the coalesce, drop-oldest, and UI-only handling; the
R5 consistency claim).

The retained rule separates Observation events (standard output, progress,
resource usage, heartbeats), which may coalesce, drop oldest with counters,
or stay UI-only, from Critical events (completion, failure, input need,
permission need, timeout, ownership change, cancellation), which require
reliable delivery. Progress must never cost the model a reasoning turn.

**Critical judgment:** retained as a consistency note with R5; the event
taxonomy, coalescing counters, and delivery mechanics are undecided. The
stable claim is only the split and the no-wake direction.

## Input-needed as a first-class event

Source: `044.md:1411-1455` (the confirmation, password, selection, and
breakpoint cases; the WaitingInput state sketch; the sensitive-input
boundary; the closed-stdin consequence).

The retained direction is that a job blocked on input surfaces an explicit
event with a prompt hint rather than hanging, while secrets stay behind the
existing sensitive-input boundary (the agent must not read user input). For
background tasks with closed stdin, an unexpected prompt becomes failure or
WaitingInput, not a stall.

**Critical judgment:** a candidate input-need direction, not an event
contract. State names, prompt-hint shape, and secret-handling mechanics are
open and need security review; nothing here relaxes the normative
redirection and minimization obligations.

## Persistence split

Source: `044.md:1456-1499` (the SQLite-for-metadata plus filesystem-logs
plus artifact-reference sketch with bounded ring buffer; the R6 consistency
claim).

The retained split keeps metadata, events, subscriptions, ownership, and
output indexes in SQLite, log bytes on the filesystem, larger outputs behind
artifact references, and a bounded ring buffer in memory, so standard output
can never inflate the database to tens of gigabytes. The source presents
this as consistent with the R6 journal plus file-held artifact bytes
direction.

**Critical judgment:** retained as a consistency note; all of it is
post-v0.1 scoping under the R6 ephemeral v0.1 and the boundary below. Store
choice, schema, index shape, and retention policy are entirely open.

## v0.1 scope boundary

Source: `044.md:1500-1554` (the R6-ephemeral alignment; the deferred daemon,
persistence, crash adoption, and detached week-long jobs; the three-phase
sketch) and `044.md:1654-1730` (the proposal to promote the material to a
dedicated supervisor document).

The carried boundary is that job-submission plumbing is out of v0.1: no new
commands, tools, events, or wire formats ship in v0.1, and durability
(durable journal, persistent evidence store, crash adoption, detached
long-lived jobs, a supervisor daemon) stays post-v0.1 under the R6
ephemeral-v0.1 profile. The source's three phases (in-memory supervisor with
event-driven completion, panel independence, mailbox delivery, and
process-tree cleanup; then persistent metadata with restart reconciliation,
Unknown handling, and resume cursors; then a detached daemon with multi-day
survival, handoff, adoption, and scheduling) are retained as the author's
staging opinion, not a schedule: phases have no owner, milestone, or
acceptance evidence. The proposal to promote this material to a dedicated
supervisor document is recorded as an author proposal; this task creates no
such document and adopts no such plan.

**Critical judgment:** the boundary is the load-bearing scope claim of this
draft. Phase 1 items are proposals for post-v0.1 scoping like everything
else here, and none of them constrains the v0.1 profile.

## Typed result contract as two layers

Source: `044.md:1852-2011` (the anti-conflation rule at `1852-1875`; the
authoritative `ExecutionResult` and `ExecutionOutcome` sketches at
`1876-1931`; the semantic `JobResult` sketch at `1932-1984`; the artifact
split at `1985-2011`).

The retained rule is that AI-generated summary and OS execution result must
not be one object. The `bitty` side produces the authoritative
`ExecutionResult` (identifier, outcome, timestamps, output and error
references, artifact references, resource usage, truncation flag) with facts
only the process supervisor can know (exit code, process tree, signal, OOM
determination, timeout). The `bitty-ai` side produces the semantic
`JobResult` (execution reference, structured summary, diagnostics, task
effect, progress note), whose credibility is explicitly lower than the
execution fact. Artifacts split the same way: storage, digest, and retention
references are generic host mechanism, while what an artifact means for a
task is `bitty-ai` semantics. The UI consequence is kept as illustration:
failure shows the factual outcome with the agent summary and evidence links
side by side, never a bare summary sentence.

**Critical judgment:** the two-layer split is among the strongest claims
and is retained as a candidate input; both struct shapes are unreviewed
sketches. The stable claims are the anti-conflation rule and the
credibility ordering (fact above interpretation).

## Multi-agent security split and dual generations

Source: `044.md:2012-2181` (the no-bare-check rule and the authorized
request shape at `2012-2070`; the capability-enforcement list at
`2071-2094`; the claim and ownership semantics at `2095-2123`; the two
staleness kinds with two generations at `2124-2181`).

The retained split is that `bitty` performs final capability enforcement
while `bitty-ai` owns coordination: the AI side decides logically (owner,
reviewer, subscriber, handoff) but every sensitive operation crosses into
the host as an authorized, versioned request (principal, execution,
operation, generation) that the host admits or denies, because a bare AI-side
check would let one AI bug kill any process. Enforcement-side capabilities
(observe, read output, write input, signal, cancel, attach, transfer) are
independent. Claim and ownership semantics (claims, assignments, stale-agent
detection, lead reassignment, task and reviewer binding) stay AI-side. The
load-bearing distinction is two staleness kinds with two generations: agent
claim staleness (lost heartbeat, expired lease, commander-decided handoff)
is `bitty-ai` coordination with an assignment generation, while execution
handle staleness (an identifier rebuilt at a newer generation) is host
detection with an execution generation that the host must reject.

**Critical judgment:** the enforcement-versus-coordination split is retained
as a candidate consistent with the 041 IPC-permission direction and the
CTX-0047 comparison (capability enforcement at the boundary, coordination
above it); the capability names, claim shapes, and request envelope are
unreviewed sketches. This section re-decides nothing about the Capability
Layer.

## Cancel and timeout: mechanism below, policy above

Source: `044.md:2182-2263` (the policy-versus-mechanism cut; the graceful
escalation ladder owned by the host; the cancel-protocol and cancel-outcome
sketches) and `044.md:1756-1851` for the binding context (the Task, Agent,
Execution, and workspace binding owned AI-side with target resolution and
enforcement owned host-side).

The retained cut is that `bitty-ai` decides whether a cancel is appropriate
(task policy, team ownership, whether another agent still needs the job)
while `bitty` executes it (signal escalation with grace periods, process-group
termination, descendant reaping) and reports a structured cancel outcome, so
the AI side never infers "it stopped because I sent Ctrl+C." Timeout clocks
follow the same cut with supervisor-held deadlines on one side and
workflow-chosen policies on the other. The binding context is kept: task,
agent, execution, dependency, claim, handoff, and progress-note semantics
are AI-side orchestration the terminal never needs to understand, while
worktree and working-directory targets arrive as references the host
resolves, authorizes, captures, and executes (never a raw path the core
blindly runs).

**Critical judgment:** the cut is retained as a candidate input; the cancel
modes, grace periods, outcome names, and binding shapes are unreviewed
sketches. The stable claims are host-executed termination with reported
(never assumed) outcomes and host-resolved execution targets.

## Boundary table

Source: `044.md:2367-2396`. The full table is retained as a candidate
input; every cell naming an interface, identifier, or mechanism is
discussion vocabulary, not an adopted contract.

| Capability                                          | `bitty`                             | `bitty-ai`                              |
| --------------------------------------------------- | ----------------------------------- | --------------------------------------- |
| Process spawn                                       | Owns                                | Requests                                |
| PTY                                                 | Owns                                | Uses                                    |
| Standard output and error capture                   | Owns                                | Consumes                                |
| Process tree                                        | Owns                                | Views the authorized projection         |
| Working directory, environment, and worktree target | Resolves, validates, and captures   | Selects the target                      |
| Execution identifier and generation                 | Owns                                | References                              |
| Task identifier                                     | Semantically blind                  | Owns                                    |
| Job-to-Task binding                                 | Opaque metadata or reference only   | Owns                                    |
| Job-to-Agent ownership                              | Capability principal                | Owns the coordination semantics         |
| Claim and handoff                                   | Enforcement primitive               | Owns the policy                         |
| Subscription                                        | Event mechanism                     | Subscription policy                     |
| Read and kill permissions                           | Final enforcement                   | Decides and requests scope              |
| Graceful cancel                                     | Executes                            | Selects the policy                      |
| Timeout clock                                       | Executes                            | Sets the policy                         |
| Process-tree termination                            | Owns                                | Requests                                |
| Exit, signal, and OOM                               | Authoritative fact                  | Interprets                              |
| Raw artifacts and evidence                          | Produces and references (mechanism) | Interprets and associates with the task |
| Summary                                             | Produces no AI semantics            | Owns                                    |
| Progress note                                       | Does not interpret                  | Owns                                    |
| Retry                                               | Provides the re-execution primitive | Decides whether to retry                |
| Stale execution                                     | Detects                             | Responds                                |
| Stale agent claim                                   | Not responsible for Agent semantics | Detects and coordinates                 |
| Task completion                                     | Not responsible                     | Owns                                    |

## Repository placement, the wrapper principle, and the worked example

Source: `044.md:2397-2526` (the AI-agnostic execution subsystem sketch with
its generic-identifier rule; the AI-side job module sketch; the wrapper
principle sentence) and `044.md:2527-2630` (the T128, J31, and E77 binding
illustration with the principal-view closing).

The retained placement keeps the host execution subsystem AI-agnostic
(process, PTY, supervisor, outcome, limits, signals, output, capability
with generic principal, execution, target, capability, and opaque-metadata
identifiers only) and the AI-side job layer semantic (task, agent, team,
assignment, execution, and workspace references). The architecture sentence
is kept verbatim as a principle: **every Job may reference an Execution,
but an Execution is not inherently an AI Job**, because ordinary users, Lua
plugins, and terminal features execute processes too (a user-started test
run is an Execution without necessarily being a `bitty-ai` Job). The shared
IPC names generic execution semantics rather than agent ontology, consistent
with the existing command-tool boundary that keeps AI-specific semantics
out of the terminal process. The worked example is retained as the binding
illustration: Task T128 (fix a parser bug) owned by Agent A in workspace W5
binds to Job J31, which spawns Execution E77 at generation 3 through
target-resolving IPC; the exit event (outcome with output references) flows
back through J31 to T128 as a progress note while T128 stays in progress;
throughout, the host knows only an authorized principal and its
capabilities, never what T128 means or which agent role A plays.

**Critical judgment:** placement and principle retained as candidate inputs;
crate, module, and file sketches are proposals with no packaging evidence,
and the IPC names are discussion vocabulary that proposes no wire method.
The stable claims are the agnosticism rule (no agent or task ontology in
the host) and the wrapper principle.

## Sections read but excluded (bitty-side or unreviewed surface)

Source ranges below were read in full for exclusion accuracy.

- `044.md:1-36` (the harness anecdote and the owner's opening questions):
  boundary context; motivates the synthesis, not a distilled claim.
- `044.md:1111-1188` (the `exec` and `job_*` tool-method sketches with the
  synchronous-window-to-handle escalation): excluded as unreviewed tool
  surface; proposes no accepted command or tool per the v0.1 boundary.
  The no-polling direction is already retained through the harness survey
  and subscription sections.
- `044.md:1189-1233` (explicit argument vectors versus default shell
  execution): read, not distilled; shell-versus-argv attribution detail is
  `bitty`-side execution design, and its permission-checking consequence is
  already R2-owned ground.
- `044.md:1234-1286` (the multi-agent visibility console mockup): excluded
  as an illustration without a data contract or authorization analysis; the
  organization views it resembles are already retained as derived,
  authorization-checked views in existing coordination material.
- `044.md:1287-1344` (owner disappearance with handoff and generation):
  read; restates existing R5 handoff and fencing for Jobs with no new
  mechanism, retained only as the consistency note in the security split
  above.
- `044.md:2500-2526` (the shared-IPC sketch): read; retained only as the
  generic-semantics direction in the placement section above; every
  operation name is discussion vocabulary proposing no wire method.

## Source coverage and critical disposition

Retain means retain as a **proposal**, not accept as normative. Improve means
retain the objective with the correction above. Reject means reject that
mechanism or absolute claim; defer means no commitment pending named
evidence. Boundary sections read but not distilled are marked as such.

| Source lines | Topic                                                       | Disposition, rationale, and alternative                                                               |
| ------------ | ----------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| 1-36         | Harness anecdote and opening questions                      | Boundary context in this distillation; motivates the synthesis, not a distilled claim.                |
| 37-60        | Core thesis: supervisor and Job Service, not stronger spawn | Retain layering as candidate objective; anecdote is motivation; defer ownership, packaging, schedule. |
| 61-148       | Harness table; pueue detachment, groups, dependencies, logs | Retain waiting and subscription principles; borrow-design-not-backend; defer all backend selection.   |
| 149-224      | OpenCode event-driven notification                          | Retain event-over-polling direction; PTY-session shape is backend detail, not the abstraction.        |
| 225-268      | Codex process protocol; polling critique                    | Retain protocol cleanliness and no-model-polling rule; deficiency claims are author opinion.          |
| 269-308      | Claude background tasks, Monitor, session separation        | Retain process-job versus agent-task separation; product specifics are context, not contract.         |
| 309-356      | Cursor Agent/Run split, event stream, subscriptions         | Retain subscription direction as target shape; cloud specifics excluded.                              |
| 357-411      | Four-object model and invariance claims                     | Retain separation as candidate; invariance conditioned on Lifetime policy, not guaranteed.            |
| 412-476      | Job not belonging to Panel; mailbox routing                 | Retain routing direction; mailbox and session mechanics undecided.                                    |
| 477-523      | Owner plus Subscriber; observe versus control               | Retain ownership shape; permission names unreviewed; enforcement stays host-side.                     |
| 524-593      | Agent-killed disposition; Lifetime sketch                   | Retain explicit-policy direction; variants and bindings are proposals; Detached durability deferred.  |
| 594-654      | Pre-exit quiescence gate as mechanism                       | Retain invariant direction; gate mechanics undecided, need owning-task design.                        |
| 655-723      | Hard, idle, and retention clocks; anti-default rule         | Retain clock separation; values unreviewed; uniform kill rule rejected.                               |
| 724-765      | Job kind taxonomy                                           | Retain as heuristic needing per-case review, not a decision procedure.                                |
| 766-808      | Closed-stdin default; PTY only for interactive              | Retain default-closed direction; spawn-options shape undecided.                                       |
| 809-849      | Supervisor-held output; bounded completion                  | Retain retention-and-reference direction; bounds and placement open.                                  |
| 850-918      | Completion as Critical Message; Unknown rule                | Consistency note; critical semantics stay R5-owned; no new mechanism.                                 |
| 919-997      | Three partition classes and resume cursor                   | Retain taxonomy; cursor mechanics and retry undecided; no supervisor inference.                       |
| 998-1052     | Structured outcomes; OOM rule; tree kill                    | Retain structure direction; enum unreviewed; OOM backends unevidenced.                                |
| 1053-1110    | No-default retry; Unknown-inspect-first                     | Consistency note with load-bearing consequence; vocab and backoff open.                               |
| 1111-1188    | `exec` and `job_*` tool sketches                            | Excluded; unreviewed tool surface; proposes no accepted command or tool.                              |
| 1189-1233    | Argument vectors versus shell default                       | Read, not distilled; `bitty`-side detail with R2-owned consequences.                                  |
| 1234-1286    | Visibility console mockup                                   | Excluded; illustration without data contract or authorization analysis.                               |
| 1287-1344    | Owner handoff with generation                               | Read; restates R5 fencing for Jobs; no new mechanism.                                                 |
| 1345-1410    | Observation versus Critical events; no-wake rule            | Retain split and no-wake direction; taxonomy and counters open.                                       |
| 1411-1455    | Input-needed event; WaitingInput; secret boundary           | Candidate direction; states and hints open; needs security review.                                    |
| 1456-1499    | SQLite metadata with filesystem logs and artifact refs      | Consistency note; all post-v0.1 under R6; schema and policy open.                                     |
| 1500-1554    | v0.1 boundary; three-phase sketch                           | Boundary carried as scope; phases are staging opinion, not schedule.                                  |
| 1555-1653    | Borrow-design synthesis and architecture diagram            | Retain composition direction; proposes no backend or dependency.                                      |
| 1654-1730    | Dedicated-supervisor-document proposal                      | Recorded as author proposal; no document created or adopted here.                                     |
| 1731-1755    | Mechanism versus semantics framing                          | Retain framing as candidate; the split below carries the content.                                     |
| 1756-1851    | Job-to-Task binding cut with workspace refs                 | Retain cut; binding shapes unreviewed; raw-path execution rejected.                                   |
| 1852-1931    | Authoritative `ExecutionResult` layer                       | Retain anti-conflation and credibility ordering; struct unreviewed.                                   |
| 1932-2011    | Semantic `JobResult` and artifact layers                    | Retain interpretation-above-fact layering; shapes unreviewed.                                         |
| 2012-2094    | Host capability enforcement                                 | Retain split consistent with the CTX-0047 comparison; names unreviewed.                               |
| 2095-2123    | AI-side claim and ownership semantics                       | Retain coordination ownership; shapes unreviewed.                                                     |
| 2124-2181    | Dual staleness with dual generations                        | Retain distinction as load-bearing; mechanics undecided.                                              |
| 2182-2263    | Cancel mechanism versus policy                              | Retain cut; modes and periods unreviewed; outcomes reported, never assumed.                           |
| 2264-2316    | Supervisor-held deadlines; policy selection                 | Retain deadline-holding direction; limits unreviewed.                                                 |
| 2317-2366    | Timeout-versus-Failed analysis; outcome split               | Retain split as strongest timing claim; interpretation stays policy.                                  |
| 2367-2396    | Full boundary table                                         | Retain as candidate input; interface cells are vocabulary.                                            |
| 2397-2499    | Repository placement; wrapper principle                     | Retain agnosticism rule and principle; crates and modules are proposals.                              |
| 2500-2526    | Shared generic IPC direction                                | Retain direction; operation names propose no wire method.                                             |
| 2527-2630    | T128, J31, and E77 worked example                           | Retain as binding illustration; identifiers are illustration, not registry.                           |

## Proposed validation and promotion path

These are future evidence requirements, not tests executed by this
documentation task. They keep the design falsifiable before any supervisor
proposal constrains `bitty-ai`.

| Campaign             | Required observation                                                                                                                                                           |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Object separation    | A job completes to its agent's mailbox after its origin panel closes and after the agent switches panels, with panel lifetime independent of job lifetime in reviewable tests. |
| Mailbox routing      | Completion delivery names the agent rather than any panel, and redelivery after reconnect deduplicates on stable event identifiers.                                            |
| Lifetime policy      | Each declared lifetime scope behaves as specified on agent death, with forced stops falling back to the declared policy in reviewable tests.                                   |
| Quiescence gate      | An agent with live owned jobs cannot finish normally without an explicit per-job disposition, enforced by the harness rather than by prompting.                                |
| Timeout split        | A timed-out execution records a timeout fact below while task disposition above varies by policy, with no folding into generic failure.                                        |
| Closed stdin         | A non-interactive job never blocks forever on standard input, and interactive capability requires an explicit grant.                                                           |
| Output discipline    | A multi-thousand-line job delivers a bounded completion with an evidence reference, and full output enters context only through an explicit, authorized read.                  |
| Critical completion  | Completion, failure, input-need, and cancellation events survive agent disconnect and reconcile Unknown outcomes by inspection, never by assumption.                           |
| Retry discipline     | An effectful or Unknown-outcome job is never retried without an explicit idempotency declaration or approval, in reviewable tests.                                             |
| Boundary enforcement | An AI-side bug or compromised agent cannot exceed its granted execution capabilities, because the host enforces every sensitive operation independently.                       |
| Wrapper principle    | A non-AI execution exists without any AI Job wrapper, and host code and identifiers carry no agent or task ontology in review.                                                 |

Promotion needs independent AI architecture, execution-owner, terminal and
plugin-owner, docs-curator, and security review. Route tool and method
names, event catalog, struct and enum shapes, limit values, kind taxonomy,
console views, phase ownership and milestones, and the supervisor-document
proposal to scoped owner tasks. This draft changes no normative contract
and authorizes no product code.
