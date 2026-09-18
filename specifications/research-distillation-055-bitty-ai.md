---
title: Event-Sourced Agent Workspace research distillation for bitty-ai (055)
description: Draft bitty-ai distillation of research 055 event sourced agent workspace six object split agent graph mailbox context share merge task as issue
category: specifications
audience: mixed
document_type: research
status: draft
website_publish: false
sidebar_order: 65
---

# Event-Sourced Agent Workspace research distillation for bitty-ai (055)

## Status and recommendation

This is a **draft discussion synthesis**, not a runtime specification, accepted
decision, dependency selection, release commitment, or implementation claim.
It distills only the `bitty-ai`-relevant parts of one workspace research
summary: `055.md` (Wheel as an Event-Sourced Agent Workspace: agents act only
in headless panels, an immutable event log records facts, context compiles
from facts into a programmable versioned graph, and agents collaborate over
a communication graph with Git-like operations).

The recommendation is to treat every model below as a candidate input to the
draft [AI Architecture](../architecture/ai-architecture.md) and its related draft
dispositions, never as an override of the accepted
[IPC and Agent RFC](ipc-agent-rfc.md) or the normative security corpus.
In particular, the six-object vocabulary, the event names, the mailbox and
bundle shapes, the merge-kind names, the primitive spellings, and the core
principle sentence are **not** accepted by this distillation. Agent
lifecycle, Agent events, and Agent semantics in the source are **discussion
inputs only**: the accepted Agent contract stays entirely with the RFC, which
this draft references without restating normatively. Nothing here is promoted
to accepted status, and no implementation is described as shipped.

The source's strongest ideas are the founding inequalities (Agent is not
Panel, Panel is not Context, Context is not History, Task is not Agent), the
communication-graph versus dependency-graph split (cyclic agent
collaboration kept distinct from an acyclic task graph), the durable
per-call reason as a commit message instead of a thinking dump, the
context-share-as-first-class-object direction with lazy expansion, the typed
Knowledge, Task, and Workspace merge split, the Context GC versus structural
compaction distinction, the Task-as-Issue durable-object direction, and the
core principle that history is immutable, context is programmable, panels are
workspaces, and agents are actors. Its weakest claims are the concrete event
names, which are unreviewed vocabulary with no owner, versioning, or
compatibility evidence; the mailbox, bundle, and manifest field shapes, which
propose no adopted wire, protocol, or schema; the Git-operation mapping,
which borrows vocabulary without an enforcement or versioning owner; and the
primitive spellings, which name no accepted command surface. Those are
corrected below.

This synthesis references, without duplicating or modifying, the draft
[AI Architecture](../architecture/ai-architecture.md) (candidate inputs only),
[Agent Coordination Architecture](../agent/agent-coordination.md),
[Context Management Architecture](../context/context-management.md),
[Command and Tool Architecture](../architecture/command-tool-architecture.md),
[Execution ownership R1](../architecture/execution-ownership-r1.md),
[Context retention R3](../architecture/context-retention-r3.md),
[Task lifecycle R5](../architecture/task-lifecycle-r5.md),
[Tool transport R2](../architecture/tool-transport-r2.md),
[Provider plugin boundary](../providers/provider-plugin-boundary.md), and
[Panel environment awareness](../interfaces/panel-environment-awareness.md). The
accepted [IPC and Agent RFC](ipc-agent-rfc.md) is unaffected by this
draft. The companion
[Wheel decoupling distillation (052)](research-distillation-052-bitty-ai.md)
carries the mechanism-versus-policy rule; the companion
[Wheel-config and Git-model distillation (050-051)](research-distillation-050-051-bitty-ai.md)
carries the Git-inspired content-addressed context model; the companion
[Quality-formula and Context-Compiler distillation (048-049)](research-distillation-048-049-bitty-ai.md)
carries the compiler design; the companion
[Execution-supervisor distillation (044)](research-distillation-044-bitty-ai.md)
carries the supervisor, mailbox routing, and lifetime directions; the
companion
[Wheel scope distillation (046/053/054)](research-distillation-046-053-054-bitty-ai.md)
carries the Coding-scope reconciliation; this draft links to each wherever
agent workspace design touches them, as design input rather than
implementation. This document creates no AIQ or OQ identifier and closes
none.

## Provenance and evidence boundary

The source is research note `055` (summary), read on 2026-09-18: **28
lines**, **7,251 bytes**, SHA-256
`a23dfd8fa93adbb530a4cf4ba881eaa0f9135eb5d00143254d84e6cdd81a09f8`.
The summary landed on the research `main` branch as `e7a18d3`, which
satisfies the CTX-0076 dependency on the landing task; the task description
names CTX-0032 and this record confirms the landed revision instead. The
origin record (`055`) was not read by this task and was not renamed, edited, or
staged; the `bitty`-side, terminal-side, and plugin-side passes still need
their halves. The summary stays Open with owner-pending routing, and
split-owner capture stays Partial until all owned conclusions are accounted
for. Summary line citations below (for example `055:9`) refer to the summary
file, not to the origin. The fingerprint identifies the discussion, not the
truth of its claims.

The summary is a single pass: a status line, a date line, a one-liner, a
background block, a key-conclusions block, a destination-routing line, and an
open-items block; no duplication handling applies. The discussion record
itself is an archive receipt for an undated conversation; the source date is
the receipt date, not the conversation date. This document is the
English-language draft synthesis of the English summary and characterizes
nothing beyond it.

## Authority and reconciliation

The draft [AI Architecture](../architecture/ai-architecture.md) layered models are
candidate inputs only; the six-object vocabulary, the event names, the
mailbox and bundle shapes, the manifest shape, the Git-operation mapping,
the storage split, and the primitive spellings proposed in the source are
**not** accepted by this distillation and must not be read as product,
crate, package, protocol, file-schema, command, or release decisions.
Context assembly, budget, retention, and compaction questions stay with
[Context Management Architecture](../context/context-management.md),
[Context retention R3](../architecture/context-retention-r3.md), and the 048-049
companion; execution and panel-lifetime questions stay with
[Execution ownership R1](../architecture/execution-ownership-r1.md),
[Panel environment awareness](../interfaces/panel-environment-awareness.md), and the
044 companion; tool shape and transport placement stay with
[Command and Tool Architecture](../architecture/command-tool-architecture.md) and
[Tool transport R2](../architecture/tool-transport-r2.md); provider questions stay with
[Provider plugin boundary](../providers/provider-plugin-boundary.md); coordination,
supervision, and task questions stay with
[Agent Coordination Architecture](../agent/agent-coordination.md) and
[Task lifecycle R5](../architecture/task-lifecycle-r5.md); storage and export questions
stay with the 050-051 companion and the storage dispositions; each is
referenced, never duplicated or modified.

The accepted [IPC and Agent RFC](ipc-agent-rfc.md) defines the only
accepted IPC wire, scope, and Agent vocabulary. Every sketch name in the
source (object names, event names, mailbox and bundle fields, manifest
fields, operation spellings, primitive spellings) is a discussion sketch:
this draft records it as input and proposes no file schema, API, command,
tool, event, or wire format. Where the source's sketches overlap RFC-owned
ground (Agent lifecycle, Agent events, Agent semantics, scopes), the RFC
wins without further argument.

Normative security obligations (least privilege, per-action scopes,
capability-based and auditable permission that fails closed, typed
redaction, consented recording, secret minimization) override every
discussion example below. The collaboration and merge sketches are
conceptual vocabulary, not an adopted authorization or distribution
contract. This draft creates or closes no AIQ or OQ identifier; open
questions stay with
[AI Unresolved Questions](../product/ai-unresolved-questions.md) and shared
governance.

## Six-object split and founding inequalities

Source: `055:10` (the Agent, Panel, Event, Context, Task, and Artifact
split with the four founding inequalities).

The retained candidate vocabulary separates six objects: Agent (thinking
actor, mobile across panels), Panel (execution environment, may sit
agentless), Event (immutable fact until GC), Context (recompilable LLM
view), Task (goal and state, may span agents), and Artifact
(content-addressed outputs). The load-bearing inequalities are kept as
candidate design rules: Agent is not Panel, Panel is not Context, Context
is not History, and Task is not Agent.

**Critical judgment:** the object names are vocabulary proposing no type,
schema, registry, or module. The reconciliation with prior art is narrow:
the Agent versus Panel cut agrees with the Agent and Panel lifecycle
separation kept from records 041 and 044 and the R1
ExecutionContext-primary direction; the Panel versus Context cut agrees
with the 025 panel-state-outside-the-prefix invariant; the Context versus
History cut restates the 051 history-versus-active-view slogan and the
context-management `/compact`-changes-the-boundary rule; the Task versus
Agent cut agrees with the 044 process-job versus agent-task separation and
the R5 single-lifecycle-authority disposition without deciding any
additional owner. The Event object is recorded as a pointer: the event-log
shape belongs to the terminal documentation owner (see the pointers
section), and this draft adopts no event vocabulary. The Artifact object is
recorded with the same restraint: content addressing is carried by the
050-051 companion and the storage dispositions, and no hash, path, or
format is adopted here.

## Agent Graph versus Task Dependency Graph

Source: `055:20` (the cyclic communication graph of uniform Agents with
dynamic roles, kept distinct from the Task dependency graph, which stays a
DAG) with `055:21` (mailbox IPC over the agent graph).

The retained direction keeps two graphs apart: the Agent Graph is the
collaboration and communication topology (uniform Agents with per-task
roles such as leader, worker, and reviewer, never agent classes; the team
may be cyclic), while the Task Dependency Graph is the goal-ordering
topology (which should stay acyclic to avoid deadlocks). Fork spawns
parallel agents inheriting context without re-seeding background; the fork
mechanics stay open with the owning task.

**Critical judgment:** the graph names propose no data structure, API, or
scheduler. The reconciliation with prior art is explicit: the
uniform-Agents-with-dynamic-roles sentence agrees with the 046 fixed-domain
open-role composition rule kept in the 046/053/054 companion (one Agent
primitive, roles as configuration); the cyclic-team shape is a candidate
that must still reconcile with the R1 single-agent execution-ownership rule
(one owner per execution) and the 045 Commander-as-Lua-concept vocabulary
cut (Core knows only generic primitives, organization lives in Lua). The
DAG discipline for task dependencies is retained as a candidate invariant
with a deadlock rationale, consistent with the R5 lifecycle disposition and
the 044 no-model-polling and subscription directions; no scheduler,
resolver, or cycle-detection contract follows.

## Mailbox IPC and first-class context share

Source: `055:21` (typed agent mailboxes with context base and refs; lazy
ContextBundles with summary first and expandable decisions, checkpoints,
and artifacts).

The retained direction treats inter-agent messaging as actor-model mailbox
IPC: a message carries request, thread, task, and content identity plus a
context base and references, so a recipient receives the relevant context
graph rather than a bare sentence. Context sharing becomes a first-class
object through lazy bundles: the summary arrives first, and decisions,
checkpoints, and artifacts expand on demand. The bundle mechanics, field
shapes, and expansion protocol stay open.

**Critical judgment:** the mailbox and bundle names are unreviewed
vocabulary proposing no wire, event, or schema. The reconciliation with
prior art is narrow: the mailbox direction agrees with the 044
job-to-mailbox routing and the three-outcome acceptance rule (accepted
into a mailbox, delivered to a live recipient, processed with an
acknowledged result) kept in the 044 companion and in R5; the
context-graph-with-the-message direction agrees with the 050-051
ref-sharing rule (share by reference, not by copy) and the 041 Capability
API unity objective without adopting any of their sketches. Delivery,
durability, auth, and redaction semantics stay with the RFC, the security
corpus, and the R2 and R5 dispositions; this draft proposes none.

## Merge, Rebase, and Diff semantics

Source: `055:19` (Git analogues applied to versionable context, never to
the immutable event log; squash and checkpoint compression keep provenance
and stay expandable) with `055:21` (Knowledge, Task, and Workspace merge
kinds; context diff across decisions, artifacts, files, facts, questions,
and task state).

The retained candidate operations apply Git vocabulary to the versionable
context graph only: fork and branch for parallel hypotheses, checkpoint as
commit, merge with explicit conflicts, cherry-pick for findings, rebase
with validation, diff for inspectability, stash, tag, reflog, and GC from
live roots. The non-rewriting rule is load-bearing: squash and checkpoint
compression change the active view while keeping provenance and staying
expandable; the immutable event log is never rewritten. Merges split into
three typed kinds (Knowledge, Task, and Workspace), and context diff
compares decisions, artifacts, files, facts, questions, and task state for
the dashboard.

**Critical judgment:** every operation name is vocabulary proposing no
command, API, or storage format; the 051 companion already carries the
same Git-inspired model with the same restraint, and this draft defers to
it rather than restating it. The value this record adds is the three-way
merge-kind split and the six-way diff inventory, both recorded as candidate
input: the kind split must still reconcile with the R5 lifecycle authority
(Task merges change owned state and need an owner, not just a merge
algorithm) and with the R3 consent and retention rules (a Knowledge merge
must not smuggle expired or unconsented material back into the active
view). The Rebase value is the validate-before-replay direction already
kept from 051; the Diff value is the inspectability rule already kept from
051, extended here to the dashboard surface, which stays terminal-owned
(see the pointers section).

## Context GC versus structural compaction

Source: `055:22` (Context GC with Hot, Warm, and Cold lifetimes, so that
model context and persistent history diverge by design) with `055:23`
(checkpoint, fork, share, diff, squash, and merge as first-priority
primitives) and the 048-049 companion's compaction sections.

The retained distinction separates two operations that must not be
confused: Context GC reclaims versioned context objects from live roots
with a grace policy, while structural compaction (the `/compact` family)
changes the active view boundary and records compaction metadata without
deleting retained history. Hot, Warm, and Cold lifetimes govern what the
model sees; persistent history follows retention and consent policy, and
the two diverge by design. Recovery or re-expansion requires the original
authorized records still to exist; retention expiry and user deletion
remain effective.

**Critical judgment:** the lifetime labels are vocabulary proposing no
policy, threshold, or scheduler. The reconciliation with prior art is
narrow: the GC direction agrees with the 051 rooted-collection rule kept
in the 050-051 companion; the compaction direction agrees with the
context-management `/compact`-changes-the-boundary rule and the 025
structural-compaction invariant; the Hot, Warm, and Cold stratification
agrees with the 049 Cold, Warm, and Hot worlds kept in the 048-049
companion. This draft adds no lifetime value, grace period, trigger, or
backend, and re-decides none of the open retention questions (which stay
with R3 and the AIQ register).

## Task-as-Issue and reason-as-commit-message

Source: `055:23` (Tasks as Issue-like durable objects claimable by any
replacement agent; the user dashboard as a company-console view with ask,
inspect, pause, cancel, fork, message, handoff, diff, and merge) with
`055:13` (the per-call reason as durable rationale: what the action exists
to verify, the expected outcome, and links; internal thinking stays
ephemeral).

The retained candidate direction models a Task as an Issue-like durable
object with status, owner, collaborators, dependencies, context,
artifacts, progress, and blockers, claimable by any replacement agent so
that work survives agent turnover. The dashboard direction (ask, inspect,
pause, cancel, fork, message, handoff, diff, merge, with no subagents
exposed) is recorded as terminal-owned surface vocabulary, not as an
accepted view: the operation names propose no command or wire. The
per-call reason is retained as durable rationale in the shape of a commit
message (what the action exists to verify, the expected outcome, and
links), while internal thinking stays ephemeral and out of long-term
history.

**Critical judgment:** the Task-field list and the dashboard-operation
list are illustrations proposing no schema, command, or view. The
reconciliation with prior art is explicit: the Task-as-Issue direction
agrees with the 044 Job-as-wrapper principle and the R5 single-authority
disposition (claimable-by-replacement needs the R5 ownership split,
fencing, and handoff contract, which this draft does not restate); the
reason-as-commit-message direction tightens the 044 tool-run record
(input, reason, status, duration, summary plus blob references) into a
rationale-first convention without adopting any field or format. The R5
independent-review criteria and critical-message acknowledgement rules
override any handoff or claim sketch below.

## Core principle

Source: `055:23` (history is immutable, context is programmable, panels
are workspaces, agents are actors) with `055:19` (Git analogues never
rewrite the log) and `055:20` (agents act only in headless panels, per the
AI-side reading of the headless rule).

The retained candidate principle has four clauses: history is immutable
(the event log records facts and is never rewritten); context is
programmable (the model prompt is a compiled, versioned, budget-checked
materialized view over the fact graph); panels are workspaces (execution
space with identity and lifecycle, distinct from both the actor and the
view); agents are actors (mobile behavior units collaborating over
mailboxes with dynamic per-task roles). The Batch plus DAG multi-tool
shape, the output pipeline (raw output to blob store to structured parser
to result summary to context compiler), and the running-task status
discipline (status, elapsed time, and parsed progress with full logs on
demand) are recorded as compatible context for the compiler and tool
dispositions, not as adopted contracts.

**Critical judgment:** the principle is a slogan proposing no
architecture, packaging, or team split. Its reconciliation with prior art
is cumulative: immutability restates the 051 and R6 append-ordered journal
rules; programmability restates the 048-049 compiler-as-view direction and
the 025 stable-first layering; panels-as-workspaces restates the 039
Panel-as-host boundary with the R1 projection-only rule; agents-as-actors
restates the 044 and 045 placement (mechanism in the runtime, organization
in Lua) with the RFC as the only accepted vocabulary. The headless-only
agent clause is recorded with its terminal-owned half as a pointer (see
below): headed panels stay human-owned and agent touch means forking an
execution snapshot, never typing into the user's panel.

## Owner-pending pointers for non-AI halves

The rows below route conclusions this draft does not distill. They are
pointers with inline summaries, not links and not decisions.

| Source | Topic                                                                                                               | Owning destination                                                               |
| ------ | ------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| 055:11 | Headed versus headless panel ownership; shared-panel lease and capability gating; detached-panel history            | Terminal documentation owner; owner approval pending                             |
| 055:11 | Fork-an-execution-snapshot rule for touching user work (cwd, worktree, environment, commands, outputs)              | Terminal documentation owner; owner approval pending                             |
| 055:12 | Immutable event-log vocabulary and shape (task, agent, tool, artifact, decision, share, message, checkpoint events) | Terminal documentation owner; owner approval pending; event names stay RFC-owned |
| 055:14 | Tool-call detachment from terminal input; Wheel tool runtime under policy, capability, and resource control         | Terminal and runtime owners jointly; owner approval pending                      |
| 055:15 | Batch plus DAG multi-tool shape (sequence, parallel, dependency, conditional, retry, timeout, cancel)               | Tool and runtime owners; mechanics, fields, and policies stay open               |
| 055:16 | Running-task status discipline; UI collapse decoupled from context materialization                                  | Terminal documentation owner (dashboard and views)                               |
| 055:17 | Output pipeline stage placement (blob store, structured parser, result summary)                                     | Compiler and storage owners; stage contracts stay open                           |
| 055:22 | Hybrid storage layout (SQLite index plus content-addressed object store; JSON and JSONL for interchange)            | Storage and terminal owners; schema, paths, and formats stay open                |
| 055:23 | Dashboard mechanics (company-console view; ask, inspect, pause, cancel, fork, message, handoff, diff)               | Terminal documentation owner; operation names are vocabulary only                |

## Sections read for boundary accuracy

The summary was distilled whole; the ranges below were read so that
non-`bitty-ai` content is not silently absorbed.

- `055:1-5` status, date, and one-liner (Open capture state; archive
  receipt dated 2026-09-18 for an undated discussion; workspace definition
  with headless panels, immutable log, programmable context, and
  communication graph): scope framing; motivates the synthesis.
- `055:6-8` background (owner context-management question; headless panels
  with mutual visits; flat main-plus-subagents shape; log volume versus
  model output; structured per-panel storage with Git-style operations and
  per-tool-call reasons): boundary context; the terminal-owned halves
  route to the pointers above.
- `055:11-17` panel ownership, event vocabulary, reason convention, tool
  detachment, Batch plus DAG shape, status discipline, and output
  pipeline: read as terminal, tool, and compiler boundary context; only
  the AI-side consequences are distilled above, and no event, field,
  command, or stage contract follows.
- `055:22-23` storage split and dashboard mechanics: read as storage and
  terminal boundary context; only the AI-side consequences are distilled
  above, and no schema, path, format, or view contract follows.
- `055:24` destination routing (panel, event-log, tool-runtime, pipeline,
  storage, and dashboard halves to the terminal owner; agent graph,
  mailbox, context, Git-like primitives, and task semantics to the AI
  owner): followed by this draft; no capture claim is asserted beyond it.
- `055:25-28` open items (owner approval of the split, the headless rule,
  the event vocabulary, the Git mapping, and the storage division;
  primitive-spelling acceptance; graph-split enforcement; origin stays
  unrenamed; split-owner capture stays Partial): retained as the
  promotion gate; this draft creates or closes no identifier.

## Source coverage and critical disposition

Retain means retain as a **proposal**, not accept as normative. Improve
means retain the objective with the correction above. Reject means reject
that mechanism or absolute claim; defer means no commitment pending named
evidence.

| Source lines | Topic                                                                                                | Disposition, rationale, and alternative                                                                      |
| ------------ | ---------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| 055:1-5      | Status, date, one-liner; Open capture state with owner-pending routing                               | Retain routing; this draft follows it and asserts no capture claim beyond the AI slice.                      |
| 055:6-8      | Owner context-management question; flat-shape limits; log volume; Git-style storage proposal         | Boundary context; retain the workspace framing as candidate; terminal halves route to pointers.              |
| 055:10       | Six-object split with four founding inequalities                                                     | Retain vocabulary and inequalities as candidate; reconcile with 041, 044, R1, R5, 025, 051 per the section.  |
| 055:11       | Headless-only agent rule; headed, shared, and detached panel kinds; fork-snapshot touch rule         | Read for boundary accuracy; agent-side consequence retained; panel mechanics route to terminal owner.        |
| 055:12       | Immutable event-log vocabulary                                                                       | Read for boundary accuracy; shape routes to terminal owner; no event name adopted here.                      |
| 055:13       | Per-call reason as durable rationale (commit-message shape); thinking stays ephemeral                | Retain rationale convention as candidate; tighten the 044 tool-run record; no field adopted.                 |
| 055:14       | Tool-call detachment from terminal input; Wheel tool runtime under policy and control                | Read for boundary accuracy; placement routes to terminal and runtime owners; no contract adopted.            |
| 055:15       | Batch plus DAG multi-tool shape                                                                      | Compatible context for tool dispositions; mechanics and policies stay open with tool owners.                 |
| 055:16       | Running-task status discipline; UI collapse decoupled from materialization                           | Read for boundary accuracy; status direction retained; views route to terminal owner.                        |
| 055:17       | Output pipeline (blob store, parser, summary, compiler) with hashed stable blocks                    | Compatible context for the 048-049 compiler; stage contracts stay open.                                      |
| 055:18       | Context Manifest with parent lineage as a selectable tree; prompt as budgeted materialized view      | Retain manifest direction as candidate; reconcile with 048-049 IR and zones; no schema adopted.              |
| 055:19       | Git analogues on versionable context only; squash keeps provenance and stays expandable              | Retain non-rewriting rule as load-bearing; defer to the 051 companion for the model; no command adopted.     |
| 055:20       | Agent Graph (cyclic, uniform Agents, dynamic roles) distinct from Task DAG; fork without reseeding   | Retain split as candidate; reconcile with 046 roles, R1 ownership, 045 vocabulary cut.                       |
| 055:21       | Mailbox IPC with context base and refs; lazy ContextBundles; typed merges; six-way context diff      | Retain directions as candidate; reconcile with 044 mailbox, 050-051 ref sharing, 041 unity; no wire adopted. |
| 055:22       | Hybrid storage (SQLite index plus content-addressed store; JSON and JSONL interchange); GC lifetimes | Read for boundary accuracy; storage routes to owning repos; GC direction retained per the section.           |
| 055:23       | Task-as-Issue durable objects; company-console dashboard; first-priority primitives; core principle  | Retain Task and principle as candidate; dashboard routes to terminal owner; no schema adopted.               |
| 055:24       | Destination routing to terminal and AI owners                                                        | Retain routing; this draft follows it; no canonical page is claimed as capturing beyond this draft.          |
| 055:25-28    | Open items; origin stays unrenamed; split-owner capture stays Partial                                | Retain; origin stays unrenamed and split-owner capture stays Partial.                                        |

## Proposed validation and promotion path

These are future evidence requirements, not tests executed by this
documentation task. They keep the design falsifiable before any agent
workspace proposal constrains `bitty-ai`.

| Campaign            | Required observation                                                                                                                                                                                                     |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Object split        | Six named objects appear in a reviewable design with no conflation (an agent relocates panels, a panel sits agentless, a view recompiles without touching history, a task survives agent turnover), in reviewable tests. |
| Graph split         | A cyclic agent team completes a scoped task while its task dependencies stay acyclic with no deadlock, and a role change needs no agent-class change, in reviewable tests.                                               |
| Mailbox and share   | A recipient completes a follow-up from a mailbox message plus its context bundle alone, with lazy expansion covering decisions, checkpoints, and artifacts, in reviewable tests.                                         |
| Typed merge         | Knowledge, Task, and Workspace merges resolve the same conflict differently with explicit conflicts and no silent concatenation, in reviewable tests.                                                                    |
| Diff                | A context diff across checkpoints names exactly the changed decisions, artifacts, files, facts, questions, and task state with no silent omission, in reviewable tests.                                                  |
| GC discipline       | A rooted collection reclaims only unreachable context with a grace period while a compacted episode stays restorable from history, in reviewable tests.                                                                  |
| Task durability     | A replacement agent claims an Issue-like task and completes it from the recorded state with no private handoff, in reviewable tests.                                                                                     |
| Reason durability   | A later reviewer reconstructs why a tool ran from its recorded reason alone (verified outcome, expected result, links) with no thinking-dump access, in reviewable tests.                                                |
| Principle adherence | History shows no rewrite across checkpoint, squash, and merge operations while the active view recompiles under budget, in reviewable tests.                                                                             |
| Boundary integrity  | A panel, storage, dashboard, or tool-runtime change ships with no `bitty-ai` contract change, and an AI-side change ships with no terminal contract change, in reviewable tests.                                         |

Promotion needs independent AI architecture, context-management,
coordination, terminal-owner, docs-curator, and security review. Route
object vocabulary, graph enforcement, mailbox and bundle contracts, merge
kinds and rules, diff shape, GC policy, task schema, reason fields,
primitive spellings, pipeline stage contracts, and storage layout to scoped
owner tasks. This draft changes no normative contract and authorizes no
product code.
