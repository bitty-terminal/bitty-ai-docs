---
title: Shared workspace services and agent coordination research
description: Critical synthesis of research 021 on tooling reuse teams context messaging and panel lifecycles
category: specifications
audience: mixed
document_type: research
status: draft
website_publish: false
sidebar_order: 21
---

# Shared workspace services and agent coordination research

## Status and recommendation

This is a **draft discussion synthesis**, not a runtime specification, accepted
decision, dependency selection, release commitment, or implementation claim.
The recommendation is to separate logical agents, supervised executions,
authorized workspace services, evidence storage, and UI projections. Share
expensive work only when its inputs, trust domain, and lifecycle are compatible.
Start with a small coordination model; add organization structure only when
measured coordination costs justify it.

The source's strongest ideas are consumer-independent tooling lifetimes,
reference-first context, bounded structured communication, and independent
agent/panel lifecycles. Its weakest claims are unconditional deduplication,
automatic control of headless panels, globally shared authority, and persistence
without a retention or recovery contract. Those are corrected below.

This elaborates the draft [AI architecture](ai-architecture.md), especially code
services, execution identity, context recovery, message delivery, and leased
workstations. The [IPC and Agent RFC](ipc-agent-rfc.md) remains the accepted
transport and consent contract. The
[pressure test](ai-vertical-slice-pressure-test.md) is experimental evidence, and
the [browser/agent pre-study](browser-agent-pre-study.md) remains a draft.

## Provenance and evidence boundary

The entire research note `021` was read on 2026-09-14: **3,378 lines**,
including the final blank line. Its SHA-256 is
`b6607d330887f42133e989be70dd4791aaac0ec1723441d01461fb9631403da9`.
All source ranges below are inclusive and refer to that record. The fingerprint
identifies the discussion, not the truth of its claims.

Prior discussion inputs were read in the uncommitted CTX-0003 worktree:
[research distillation 013/017/018](research-distillation-013-017-018.md) and
the [coverage ledger](../docs/sources/research-coverage-ledger.md). They remain
separate drafts and
are not copied into this task or promoted to accepted authority. They already
cover sans-I/O runtime direction, code-intelligence adapters, evidence,
transactional edits, optional CarryCtx integration, and deferred multi-agent
work. This synthesis adds lifecycle, sharing, and coordination detail rather
than replacing that foundation.

### Inspected implementation and upstream sources

Read-only Git inspection found clean working trees for both sources below.
No upstream scripts, tests, binaries, or installers were executed. Paths in this
table are relative to the named repository; revision plus path identifies the
source independently of machine layout.

| Source and revision                                                           | Inspected range                                            | Observation and limit                                                                                                                                                                                                                                                |
| ----------------------------------------------------------------------------- | ---------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Sibling `bitty-ai`, `3623c6b3ce33e97c1c493109ec6356219d0c9722`                | `crates/bitty-ai-slice/src/session.rs:68–136`              | `run_turn` calls a provider, conditionally collects bounded context, dispatches an optional tool, and emits fragments. This is real experimental code. It does not implement this proposed service supervisor, organization graph, context compiler, or lease model. |
| Same sibling                                                                  | Tracked `crates/` inventory and `docs` gitlink             | Core and experimental slice sources exist; the initialized docs gitlink is `39b4c7568a8807e0940bd298660c972db0cfa92a`. Repository-existence denials in older guides are stale. Inventory is not a full behavior audit.                                               |
| Read-only OpenCode reference clone `95daf90670b7c039c436c85537da5fbfe2205b41` | `packages/opencode/src/lsp/lsp.ts:112–117,145–206,208–297` | State has clients and an in-flight spawn map. Initialization registers a shutdown finalizer. Lookup reuses root/server-ID clients and in-flight spawns; failed initialization and duplicate creation stop the newly created process.                                 |
| Same upstream                                                                 | `LICENSE:1–21`                                             | MIT, copyright 2025 opencode. This license observation is for the inspected snapshot, not a dependency or bundled-asset audit.                                                                                                                                       |

**Disagreement with 021:115–155:** the user's report of memory growth and
surviving language-server processes is valid incident input, but the inspected
OpenCode source does not support the blanket explanation that every subagent
necessarily owns an independent server. Instance-scoped reuse and cleanup paths
exist. This inspection does not prove cleanup on every crash, determine which
version the user ran, or reproduce the incident. A future diagnosis needs
version, instance/worktree identities, spawn ancestry, shutdown events, and
memory measurements; this task does not touch the user's running processes.
The source's per-server memory estimates and aggregate savings are hypothetical,
not benchmarks.

### Evidence status distinction

This synthesis distinguishes three categories of material:

1. **Verified implementation behavior:** observations from inspected source code
   at named revisions (sibling `bitty-ai` experimental slice, upstream OpenCode
   LSP manager). These describe what exists, not what should exist.
2. **Accepted/normative documentation:** the accepted IPC/Agent RFC and the
   normative security corpus/P0 acceptance criteria override discussion suggestions.
   The AI architecture is a separate **draft**, not an accepted contract; its
   proposed mechanisms and this synthesis require acceptance. Its links to
   normative requirements do not elevate the whole document's status.
3. **Proposed controls requiring future implementation evidence:** design
   recommendations in this draft for service sharing, lease models, context
   compilation, bounded delegation, and panel lifecycle separation. These need
   review, prototyping, security testing, and acceptance before becoming
   contracts. None closes existing open risks R-011 through R-014.

The dispositions (retain/improve/reject/defer) apply to the research source's
proposals and distinguish strong ideas worth advancing from weak claims needing
correction.

## Authority and reconciliation

The architecture's
[normative source map](ai-architecture.md#normative-sources-this-specification-must-not-weaken)
routes to the canonical security overview, threat model, risk register, and
P0 acceptance criteria. Their requirements override every discussion example:

- P0-AC-021/022: authenticated local IPC and per-action server-side scopes;
  neither an object identifier nor a connection grants authority.
- P0-AC-023/024: filtered child authority, read-only Agent/MCP defaults,
  per-client elevation, and terminal output kept in observation channels.
- P0-AC-026: secret minimization, redacted sensitive fields, opt-in input
  recording, user-only storage, and export preview. Raw environments and
  clipboard contents are absent by default.
- Resource bounds, no hot-path extension work, contained failures, and safe
  startup remain mandatory. Sharing does not create an exemption.

These are existing obligations. The additional mechanisms proposed below need
review; none closes R-011 through R-014 or supplies security test evidence.

The source's `agent.spawn`, `team.create`, `code.rename`, and `panel.*` names
are conceptual vocabulary, not additions to the accepted method registry.
Adapters must map operations to reviewed scopes; unsupported operations fail
closed. Do not replace the accepted `AgentMessage` union with the source's
organization-message enumeration merely because both use the same name.

The earlier claim that the shared register stopped at OQ-032 is stale.
Reinspection on 2026-09-14 found later entries, including OQ-084 (ontology),
OQ-085 (trust levels) and OQ-086 (sensitive input). Use the live canonical
register for each entry's status; this draft creates or closes none. Local
AIQ identifiers need admission and owner routing before promotion to global OQs.

## Agents and shared workspace services

### Identity and ownership

Retain `Agent != OS process != model conversation` (021:41–112). A logical
agent can be driven by an async state machine, consistent with earlier
sans-I/O proposals. The executor choice does not itself isolate faults or
memory: blocking analysis needs bounded workers, and untrusted tools require
host-enforced process/sandbox boundaries. A panic boundary does not contain
process-wide allocation failure. Do not load external plugins into the terminal
process to save memory.

Candidate ownership separates:

| Object           | Owner and lifetime                                                                                       |
| ---------------- | -------------------------------------------------------------------------------------------------------- |
| Agent/run        | AI runtime; reasoning state, attributed actions, and bounded cancellation tree                           |
| Execution        | Supervising execution backend; captured target, process ownership, effect outcome, and cleanup           |
| Workspace view   | Authorized content/environment identity, not a compositor tab or an ephemeral AgentWorkspace             |
| Tooling instance | Service supervisor; leased by compatible consumers, bounded by workspace/session policy                  |
| Evidence record  | Context/evidence store; retention and access independent of a live process                               |
| Panel/view       | Terminal/plugin presentation owner; refers to executions and observations without owning PTY descriptors |

`WorkspaceView` is a candidate term for a filesystem/worktree/overlay view.
It must not alias the existing compositor `WorkspaceId` or `ViewId`, and must
not turn ephemeral `AgentWorkspace` data into durable project state. A runtime
may use opaque typed handles and resolve roots from target metadata; it should
never derive execution cwd from the currently focused panel.

### Sharing key and authorization

Improve the source's `(workspace, language, config_hash)` key and its later
worktree correction (021:156–205,641–700). A candidate compatibility key includes:

- execution-target identity and generation;
- repository/worktree or overlay identity and canonical root identity;
- server executable/version identity, language, negotiated features and encoding;
- toolchain, relevant environment profile, initialization/configuration hashes;
- document-overlay namespace and access/isolation domain.

Do not hash raw credentials into public cache keys or logs. Use protected opaque
environment identities, and treat any secret-dependent operation as unsuitable
for generic reuse until its confidentiality policy is established.

Same commit, same language, or same visible workspace is insufficient. Different
unsaved buffers, worktrees, containers, toolchains, or access domains need
separate state unless the adapter proves isolation. A privileged server may
index more files than a narrow caller may read: result filtering alone is not
automatically a proof against cross-scope leakage. Prefer separate service
domains when safe sharing cannot be established.

Authorize each acquisition, query, result delivery, and subscription against the
current caller. Never union the capabilities of all attached agents. A lease
references a service; it is not a transferable permission token.

### Stateful language-service mediation

Retain a semantic tool surface rather than arbitrary model-generated LSP calls
(021:206–261). Return source identity, snapshot/document version, method,
provider, freshness, confidence, and truncation. Syntax search is a useful
fallback but cannot silently claim semantic equivalence to language-service
references or rename.

A proposed broker acts as the protocol client and owns initialization, document
open/change/close ordering, versioned overlays, request correlation, bounded
notifications, cancellation, and shutdown. Many agents are consumers of this
client, not independent writers into a shared server stream. Conflicting edits
to one URI require a single authoritative overlay or separate service state.
Late diagnostics carry their document version; unknown freshness is visible.
Server restart invalidates outstanding request generations and rehydrates only
authorized document state.

Formatting, rename, code actions, server-command execution, and server-initiated
edits are effectful proposals. Apply through the existing permission and
expected-revision ChangeSet path, never implicitly because a server returned an
edit. Starting a supposedly read-only analysis server may execute project build
logic; tool availability does not authorize process creation, project code, or
network access.

### Supervision, pressure, and cancellation

Retain consumer-independent lifetime, but reject “shared and persistent” as an
unbounded invariant (021:457–640,742–756). Proposed states are Starting,
Ready, Idle, Draining, Stopped, and Failed, with a fresh generation on restart.
Admission counts starting instances so concurrent requests cannot exceed the
process budget before initialization completes. Reuse a single in-flight start
for a compatible key; failure wakes every waiter with an attributed error.

Agent exit releases its handles and requests. A service stays warm only within
an explicit idle/memory/process policy. Active requests and authorized
long-running executions count as leases; a zero UI attachment count means
nothing about safety to terminate. Expiring leases need heartbeat/reconciliation
for crashed clients, not only reference counts and destructors.

Under pressure, stop admitting optional work, evict eligible idle instances,
and bound restarts with backoff and a circuit breaker. Active work can be
cancelled only through an explicit priority/budget policy with observable
outcomes. Cancellation of one waiter must not cancel a shared operation still
needed by another authorized waiter. Revocation immediately prevents new reads
or effects for the revoked principal even if a service remains alive for others.

Use bounded graceful protocol shutdown followed by termination and reaping of
the owned process family when needed. Track launch identity/generation and
descendants using the supervising backend, not process-name matching or a stale
PID alone. Never kill an editor's language server merely because its executable
matches. Supervisor crash and host shutdown need an owned-process recovery plan;
durable handles are not sufficient to safely adopt an arbitrary survivor.

Linux cgroup resource accounting is a deferred backend candidate (021:552–584),
not a cross-platform guarantee or authorization mechanism. Controller availability,
delegation, child escape, OOM attribution, and platform-specific containment need
separate evidence. If a required isolation limit cannot be enforced, refuse that
execution profile rather than silently running it unrestricted. The source's
idle duration, memory budget, idle-server count, and process-tree names are
illustrations, not adopted configuration defaults.

## Lint, build, test, and evidence reuse

Retain bounded scheduling and request coalescing (021:262–456), but distinguish
four operations: sharing a language server, joining an in-flight check, reading
an existing result, and skipping a new execution because a cache is eligible.
They need different correctness and consent rules.

The proposed `WorkspaceRevision = HEAD + dirty hashes + config` is incomplete.
A verification fingerprint also needs relevant untracked/generated files,
submodule state, deletions, symlink identities and target policy, unsaved overlay,
tool executable/version, exact argv, cwd/target, dependency resolution/lockfiles,
environment profile, and applicable isolation policy. An input manifest must
state what was included and what could not be captured. Changing only ignored
build inputs can change an outcome without changing Git HEAD.

Prefer immutable authorized snapshots for reusable work. On a mutable tree,
before/after validation can detect many races but does not prove that the tool
observed a coherent snapshot during execution. Mark such evidence accordingly;
do not certify a PASS for the current tree when its input identity is unknown.
Clock-, network-, randomness-, external-service-, and machine-dependent tests
are non-cacheable by default unless a reviewed adapter constrains those inputs.

Candidate eligibility sequence:

1. Validate caller, target, tool, effect class, budgets, and current consent.
2. Resolve a bounded input manifest and reuse policy; unknown inputs disable
   generic caching and in-flight merging of effectful work.
3. Coalesce only compatible executions with independent waiter cancellation;
   retain per-request attribution without counting one physical run as several.
4. Store an immutable result with execution ID, fingerprint, tool/adapter version,
   status, exit code, timestamps, completeness, and evidence references.
5. Reauthorize the reader and validate freshness/reuse policy before delivery.
   Report `reused from execution` distinctly from `executed now`.

Tests and build scripts can mutate files or contact services. A cache hit is not
an authorization grant for the original command; conversely reading an already
authorized, redacted result need not rerun it. Shared target directories need
tool-aware serialization/isolation; do not coalesce `build`, `check`, `test`,
and `clippy` as interchangeable results. Independent review may reuse evidence
but must retain the option or requirement to request a fresh run. Cached PASS
never supplies independent approval.

Use per-target queues with fairness, explicit priority aging, bounded admission,
and tool-aware exclusive resources. Deduplication is an optimization subordinate
to correctness, not the source's unconditional MUST. If equivalence cannot be
proved, queue a separate authorized execution or return unsupported.

For a manifest scanning B input bytes, full fingerprint construction is O(B)
time and O(F) metadata space for F files; incremental invalidation can reduce
work but must handle watcher loss through rescan. Hash-map lookup is expected
O(1) after key construction. Shared execution saves duplicate work only for
eligible requests; no numerical savings are claimed.

## Teams, delegation, and organization graphs

Retain explicit task ownership, independent review, scoped delegation, budgets,
and compact reporting (021:773–1434). Improve the company analogy into typed
relations rather than a mandatory hierarchy of departments.

- A task has one accountable owner at a time, a versioned assignment, scoped
  implementers, reviewers, consulted participants, and subscribers. RACI is
  useful metadata; it does not create capabilities or approval authority.
- Keep the delegation/authority lineage acyclic and single-parent for budget
  attribution. Task dependencies are a separately validated DAG. Review and
  consultation links can form a general graph without becoming authority edges.
- A lead can request a child only when its granted delegation profile permits
  it. Child authority is attenuated, never inherited wholesale or increased by
  depth, team membership, model choice, or role title. Earlier restrictive
  reviewer/implementer profiles remain the default; recursive leaders require
  an explicitly reviewed profile.
- Reserve global and ancestor token/cost, active-agent, process, queue, and
  deadline budgets atomically before admitting children. Reconcile unused
  reservations on exit. A parent handing a budget to a child cannot spend it
  again. A bounded depth and direct-report limit need measurements, not the
  source's arbitrary default of five.
- On leader failure, fence the old assignment generation, reconcile outstanding
  work and uncertain effects, and hand off accountable ownership explicitly.
  A replacement leader must not duplicate every child or retry unknown effects.
- Separate implementation, independent review, and final acceptance. Identical
  model families or shared artifacts do not constitute independence on their
  own; the reviewer needs the requirement and evidence, not only the author's
  summary. A manager may inspect evidence when needed; banning all deep reading
  would weaken review.

Structured design reviews should contain proposal, evidence, dissent, verdict,
and action items with bounded rounds and a decision owner. Avoid recursive
agent chatter. Machine-verifiable status changes need no LLM call. Completed
temporary teams release resources; retained decisions/artifacts follow consent,
retention, and provenance rules rather than becoming permanent global memory.

Defer automatic promotion from a small task into an “AI Organization Runtime.”
First compare a single agent and a coordinator with a small bounded worker set
against a hierarchical design on completion quality, coordination latency,
duplicate work, cost, and recovery. Mission, Organization, and Department need
not be separate runtime services merely to reproduce a company chart.

## Context compilation and progressive disclosure

Retain context as a selected working set plus references, state, memory, and
artifacts, distinct from the complete transcript (021:1436–1805,2155–2400).
The proposed six layers are selection categories, not six competing stores:

| Source layer    | Reconciliation with existing context design                                                  |
| --------------- | -------------------------------------------------------------------------------------------- |
| Identity        | Small role/instruction snapshot; effective capabilities remain host state, not prompt claims |
| Mission         | Bounded objective and constraints relevant to the current task                               |
| Workspace       | Authorized target/view, source revision, tool availability, and bounded map                  |
| Task            | Assignment generation, acceptance criteria, dependencies, evidence pointers                  |
| Working set     | Actual provider input assembled for one turn under the resolved token/byte budgets           |
| Episodic memory | Opt-in retained observations and approved knowledge; retrieved only as needed                |

Use the existing environment/knowledge/temporal planes and instruction epochs
instead of inventing a second authority model. Runtime custody of context does
not erase per-agent ownership, access, or deletion rules. A context graph is an
index over records; the dependency subgraph may be acyclic while consultation
and evidence references need cycle detection and bounded traversal. A relational
store with typed edges is a simpler initial alternative to a dedicated graph
database.

Candidate compilation pipeline:

1. Capture task/run, instruction epoch, caller policy, target snapshot, provider
   profile, and current budget. Preserve explicit user requirements and unresolved
   blockers before optional retrieval.
2. Discover authorized references and resolve bounded fragments. A ContextRef
   carries source/owner, target generation, revision or content hash, selector,
   trust/sensitivity, freshness, and retention state. A URI-shaped string is not
   a filesystem/network capability and cannot trigger arbitrary scheme loading.
3. Recheck access, redact, deduplicate by identity and compatible provenance,
   rank by task relevance, and select under the budget. Missing/deleted/expired
   references produce typed absence; `latest` must resolve to an immutable
   identity before use. Never silently substitute another revision.
4. Build a deterministic bundle for captured inputs, carrying included and omitted
   IDs, truncation, errors, and source references. Reserve room for tools,
   provider formatting, output, and model-specific overhead. Distinguish token
   estimates from billed usage.
5. Apply mandatory policy and size checks after optional summarization and before
   provider I/O. Failure of redaction or budgeting fails closed; an optional
   ranker failure can use a bounded policy-safe fallback, never the raw history.

Reference-first communication reduces copying, not necessarily provider input:
the model ultimately needs resolved content. Count retrieval, ranking,
summarization calls, cache misses, and all agents' usage when evaluating cost.
Stable context ordering helps caching only when it does not delay revocation or
hide a changed input.

Use progressive code reads (map, outline, symbol, original range, expanded file)
and command records (summary, structured diagnostics, bounded raw ranges).
Compression is lossy: preserve failures, warnings, skipped tests, unknown exit
state, truncation, and the evidence route. Reject “raw output never enters
context unless explicitly requested” as an absolute: bounded failure excerpts
may be needed immediately. The controlling rule is minimum sufficient,
authorized, attributed context, not a ban on useful raw evidence. Terminal text
and agent summaries of it remain untrusted observations.

Retention is bounded and consented; durable task/evidence metadata is distinct
from ephemeral scratch and raw logs. Archive references can outlive a panel only
while their records remain retained. Deletion needs tombstones and invalidation
of derived summaries/caches according to policy; do not promise eternal replay
or restore secrets from an archived environment. Candidate bounded graph
selection visits V nodes and E edges in O(V + E) time and O(V) space before
ranking; sorting K candidate fragments is O(K log K). Traversal limits cap V,
E, fetched bytes, and elapsed work before allocation.

## Messages, IPC, and recovery

Retain typed request/result/finding/review/blocker/notification messages and
reference payloads (021:1806–1937,2257–2400). Reuse the accepted local transport
and bounds; an in-process channel does not remove caller checks. Workspace and
global routing are logical scopes, not a new globally privileged socket,
registry, or default TCP service.

Candidate coordination envelopes add message ID, authenticated sender and
recipient generation, task/assignment version, correlation/causation ID, kind,
bounded payload/reference list, deadline, and delivery state. Sender claims are
validated at admission; spoofed `from` or `Decision` text cannot authorize an
effect. Recipient context expansion reauthorizes every referenced item rather
than inheriting the sender's read scope.

Distinguish three outcomes: accepted into a mailbox, delivered to a live
recipient, and processed with an acknowledged result. None means that a tool
effect succeeded. Loss-tolerant progress observations can coalesce or drop with
counters. Task assignments, approvals, cancellation, and result acknowledgements
must not silently disappear under observation-stream DropOldest rules. Refuse
new requests or backpressure within accepted caps and expose the refusal.

If durable delivery is later selected, use bounded retention, deduplication,
an outbox/inbox or equivalent transactional record, expiry, and explicit
reconciliation. Do not claim exactly-once effects: a crash after execution but
before acknowledgement leaves an Unknown outcome that needs status inspection
or user direction. Retries use stable IDs and payload conflict checks; ordering
is per task/recipient where needed, not a global total-order promise. Dead or
stale routes fail closed. Backpressure never silently escalates into spawning
another agent.

Keep communication records separate from the model transcript. Automatic
context enrichment is bounded retrieval, not unconditional injection of every
related graph node. Cross-workspace inspection requires explicit target grants;
movement of a UI surface neither moves executions nor broadens context access.

## Panels, executions, leases, and human control

### Reconcile the source's two panel models

The source first calls a Panel a projection (021:1938–2154) and later calls it
an owner of cwd, environment, I/O, and history (021:2402–2490). Prefer the
existing architecture's **Agent -> ExecutionContext <- Panel** model. A panel
can project an execution's state/history but does not own PTY descriptors or
become an execution merely because it has no visible UI.

Retain many-to-many observation and independent lifetimes; reject mandatory
one-headless-panel-per-agent. A structured build tool or manager may need no
panel and no shell. A background execution may be shown later without being
recreated. Headed/headless describes presentation, not authority or persistence.
The host's accepted lifecycle vocabulary remains unchanged by this research;
Created/Active/Idle/Archived/Destroyed is a proposed execution/history policy,
not a replacement Panel Runtime state machine.

No-UI agent execution is distinct from a terminal daemon surviving GUI exit,
detach/reattach across restarts, or remote UI. Those broader features remain
subject to the accepted ADR 0008 deferral and trust-boundary gate linked by the
[existing architecture](ai-architecture.md#execution-profiles-targets-and-provider-negotiation).

### Leases and access modes

Improve the source's observe/assist/control ladder (021:2705–2835). These can be
UX profiles over capabilities, but `control` cannot bundle input, spawning,
interruption, cwd/environment modification, closure, and filesystem writes.
Each effect needs its actual scope and consent. Observation itself is bounded:
it does not grant raw environment, all history, input capture, or sibling logs.

Reject automatic headless control (021:2773–2804). A fresh agent remains
read-only. A separately approved task profile may permit particular actions in
its execution target, whether or not it has a visible panel. Hiding a panel
cannot manufacture consent or lock the human out of oversight.

A candidate lease binds principal/run, target/execution and generation, purpose,
permitted operations, expiry, and assignment version. Several read leases can
coexist. Interactive input has one fenced writer; transfer invalidates the old
writer generation before the new writer is admitted. Human takeover revokes or
pauses automated input and reconciles pending actions instead of interleaving
keystrokes or replaying queued commands into a newly changed shell. Closing or
archiving needs lifecycle authority and a check for other leases and active work;
holding any lease is not ownership of the whole resource.

The sensitive-input interlock in the IPC RFC remains a candidate, not an
implemented defense. Observation or grant possession must never be used to
claim that secret-input automation is already safe.

### Lifecycle outcomes

| Event                            | Recommended outcome                                                                                                                    |
| -------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| Panel hidden, moved, or detached | Change presentation/attachment only; execution target and authority stay fixed                                                         |
| Agent finishes or fails          | Release its leases; cancel its owned transient work or explicitly hand off an authorized retained execution; unrelated work continues  |
| Execution exits                  | Record actual outcome and bounded evidence; attached agent may continue with another execution                                         |
| Last lease expires               | Evaluate supervised idle policy; do not infer a long-running server is safe to destroy from UI absence                                 |
| User requests closure            | Resolve exact resource/generation and impact; perform separately authorized lifecycle action; report evidence-retention outcome        |
| Archive requested                | Quiesce or explicitly stop owned execution, record outcome, then retain policy-approved evidence; archive is not a live shell snapshot |
| Runtime restarts                 | Recover authorized metadata and reconcile uncertain executions; stale handles/grants do not resurrect processes or effects             |

Reject the source's absolute “any node exits without affecting another”
(021:3312–3359). Identity lifetimes are independent, but cancellation,
revocation, process failure, and resource disposal necessarily propagate bounded
effects. An agent may survive panel closure while losing a tool stream or
waiting for a replacement target. State that transition explicitly.

### User assistance and control console

For “why did my command fail?”, resolve the explicitly selected terminal and
command under read consent, obtain bounded redacted diagnostics and references,
and preserve unknown exit/boundary state when shell integration is absent.
Analysis does not automatically rerun the command. Proposed remedies are
separate from dispatch, and any execution uses a captured target, current
authorization, and its own evidence record.

Logical attachment, visible presence, presentation, and focus are distinct.
Background attachment should not steal focus. Notifications are bounded and
policy-controlled; showing a decision surface is a request to the host, not
permission for an agent to open arbitrary windows. Users should be able to
inspect or stop agent work through their authorized control surface even when
execution is headless; automation resumes only through an explicit handback.

Retain the proposed console's organization/task/agent/panel/resource views,
bidirectional attachment indexes, purpose labels, and correlated timelines
(021:3059–3377). Derive them from authoritative event/state records, not two
independently writable indexes. Show target, assignment generation, actual
execution state, reused versus fresh evidence, queue/dropped-event counts,
CPU/memory attribution, estimated/billed tokens, blockers, and consent requests.
Progress percentages need a defined denominator; unknown progress is preferable
to fabricated precision. Kill/reassign/archive/message/focus UI actions pass
the same authorization and generation checks as any other client. Keyboard
bindings, official-plugin packaging, and distribution status are deferred.

## Source coverage and critical disposition

Retain means retain as a **proposal**, not accept as normative. Improve means
retain the objective with the correction above. Reject means reject that
mechanism or absolute claim; defer means no commitment pending named evidence.
Repeated diagrams and illustrative numbers are condensed into these topics.

| Source lines | Topic                                         | Disposition, rationale, and alternative                                                                     |
| ------------ | --------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| 1–40         | Incident and shared infrastructure            | Retain motivation; improve attribution using source inspection rather than inferring per-subagent ownership |
| 41–112       | Agent/process/model distinction               | Retain logical identity; improve async proposal with bounded workers and isolation boundaries               |
| 113–205      | LSP ownership, memory, broker key             | Improve workspace-scoped pooling; reject unverified universal OpenCode diagnosis and memory savings         |
| 206–261      | Semantic code tools and fallback engines      | Retain; improve provenance and effects separation rather than raw protocol access                           |
| 262–320      | Lint fan-out                                  | Improve with compatible in-flight execution and independent authorization/cancellation                      |
| 321–407      | Revision cache and reviewer test reuse        | Improve with complete input manifests, snapshot semantics, nondeterminism policy, and visible reuse         |
| 408–456      | Build broker                                  | Retain bounded scheduling; improve resource locking and effect-aware command distinction                    |
| 457–516      | Refcount and idle shutdown                    | Improve with supervised leases, crash reconciliation, bounded stop/reap, and generation fencing             |
| 517–584      | Memory pressure and Linux cgroups             | Retain pressure objective; defer backend/defaults pending enforcement and portability evidence              |
| 585–640      | Process diagram                               | Improve logical/physical separation; reject implication that an actor is an OS sandbox                      |
| 641–700      | Worktree-aware WorkspaceView                  | Retain; improve target, overlay, environment and access-domain identity                                     |
| 701–772      | Shared services and MUST deduplicate          | Improve conditional sharing; reject universal persistence/dedup requirement                                 |
| 773–874      | Multi-leader organization graph               | Retain typed relations; defer mandatory multi-tier organization                                             |
| 875–966      | Span of control and recursive spawn           | Improve bounded admission and attenuated grants; defer example defaults                                     |
| 967–1059     | Manager/worker roles and summaries            | Retain role-aware summaries; reject denying managers evidence drill-down                                    |
| 1060–1143    | Structured meetings and departments           | Retain bounded review artifacts; defer automatic organizational escalation                                  |
| 1144–1213    | Shared infrastructure and team budgets        | Improve trust-scoped reuse and atomic ancestor/global budget reservations                                   |
| 1214–1293    | RACI and independent review                   | Retain ownership/accountability separation; verdict is evidence, not self-acceptance                        |
| 1294–1434    | Temporary teams and organization runtime      | Retain resource release; defer extra runtime layers and permanent memory assumptions                        |
| 1436–1667    | Context versus history and six layers         | Retain working-set model; improve mapping to existing planes/epochs                                         |
| 1668–1724    | ContextRef and reference messages             | Improve immutable identity, authorization, freshness, and missing-reference handling                        |
| 1725–1805    | Command/file progressive disclosure           | Retain; improve warning preservation and bounded original-source fallback                                   |
| 1806–1937    | Message kinds and three IPC layers            | Retain logical routing; reject new ambient global bus or implied daemon                                     |
| 1938–2039    | Many-to-many panel projections                | Retain; defer floating-panel UX and keep focus host-controlled                                              |
| 2040–2154    | Execution home, attachments, context lifetime | Improve captured per-execution target; context retention is independent but bounded                         |
| 2155–2256    | Graph and role-aware compiler                 | Retain selection; improve bounded traversal, mandatory filtering, and simple-store alternative              |
| 2257–2400    | Message enrichment and invariants             | Improve delivery/ack semantics; reject token-savings certainty and absolute raw-output ban                  |
| 2402–2490    | Panel as environment versus projection        | Reject environment ownership in Panel; use existing ExecutionContext and terminal owners                    |
| 2491–2659    | Default lease and portable context            | Improve optional leases and retained evidence; no mandatory shell or eternal context                        |
| 2660–2772    | User-panel error investigation and modes      | Retain explicit observation; improve granular scope mapping and no automatic rerun                          |
| 2773–2835    | Headless control and shared leases            | Reject default control; improve fenced writer and separately consented effects                              |
| 2836–2955    | Archive and Panel/PTY separation              | Retain separation; improve quiescence, bounded retention, and no live-process resurrection                  |
| 2956–3058    | Agent panel management and multiple surfaces  | Improve target-scoped lifecycle grants; a lease does not authorize closing others' work                     |
| 3059–3216    | Purpose, console, timelines and indexes       | Retain observable projections; defer UI bindings/packaging and require authoritative state                  |
| 3217–3311    | Human actor, attachment versus focus          | Retain shared protocol concept; improve human takeover and per-principal authority                          |
| 3312–3378    | Lifecycle invariants and console conclusion   | Improve independent identities with explicit failure propagation; defer official-plugin designation         |

## Proposed validation and promotion path

These are future evidence requirements, not tests executed by this documentation
task. They keep the design falsifiable before any implementation is authorized.

| Campaign                  | Required observation                                                                                                                                                                        |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Sharing and startup races | Compatible concurrent consumers create one service; different worktree/overlay/access domains cannot share incompatible state; failed starts release all reservations                       |
| Document ordering         | Conflicting buffers, late diagnostics, server restart, cancellation, and stale generations never yield silently current evidence or unauthorized edits                                      |
| Process lifecycle         | Agent crash, supervisor crash, final waiter exit, timeout and pressure leave no unowned survivors; only recorded owned process families are stopped                                         |
| Cache soundness           | Dirty/untracked/generated changes, toolchain/environment changes and external inputs invalidate reuse; Unknown/partial outcomes never become PASS                                           |
| Shared authorization      | Narrow callers cannot acquire privileged indexed content, join unauthorized effects, or obtain revoked results through another waiter's grant                                               |
| Delegation and recovery   | Concurrent child creation cannot overspend ancestor budgets; stale leaders cannot reassign/spawn; review cannot approve its own implementation                                              |
| Context and messaging     | Seeded secrets/injection, graph cycles, expired references, dropped observations, duplicate/reordered messages and failed persistence remain bounded and attributed                         |
| Panel and human takeover  | UI movement cannot change cwd/scope; stale input writer is fenced; close/archive cannot kill another owner's work; hidden work stays inspectable under user authority                       |
| Performance               | Compare cold/warm memory, process count, idle reclaim, swap/pressure, queue latency, duplicate execution and total model tokens/cost against a fixed baseline; report platform and workload |

Recommended sequence: authorize and verify a narrow single-agent execution and
evidence path; add one compatible tooling-service lease; prove isolation and
cleanup; introduce conservative check-result reuse; validate reference-based
context; then bounded delegation and UI projections. A multi-team scheduler,
cross-instance persistence, remote execution, and an official console remain
later decisions. Resource and security enforcement belong in each stage, not a
hardening phase after unrestricted execution.

Promotion needs independent AI architecture, terminal/panel-owner, docs-curator,
and security review. Resolve ownership of generic workspace services without
moving language tooling into the terminal hot path or making standalone AI
depend on the GUI. Route missing OQ/register links, accepted lifecycle conflicts,
retention policy, platform supervision, lease fencing, and cache eligibility to
scoped owner tasks. This draft changes no normative contract and authorizes no
product code.
