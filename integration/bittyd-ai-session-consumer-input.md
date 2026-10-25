---
title: bittyd AI session consumer input
description: Draft AI-side decision input separating AgentSession lifecycle from the proposed bittyd session and worker binding
category: specifications
audience: contributor
document_type: specification
status: draft
website_publish: false
sidebar_order: 82
---

# bittyd AI session consumer input

## Document status

> Status: **draft design input**. This page records the `bitty-ai` consumer
> position for the proposed Core-owned `bittyd` session host. It accepts no
> `bittyd` contract, does not change the accepted IPC and Agent RFC, and does
> not claim a daemon, adapter, worker supervisor, or conformance result exists.
> The page is intentionally a candidate input for CTX-1525 and a handoff
> boundary for a possible future CTX-1527; it is not an implementation plan.

The current terminal-platform requirements are read at
[`bitty-terminal-docs` `origin/main` revision `575e8b1d93efb35e6c76b19ade8f034eb64ff434`](https://github.com/bitty-terminal/bitty-terminal-docs/blob/575e8b1d93efb35e6c76b19ade8f034eb64ff434/specifications/bittyd-session-host-requirements-candidate.md).
The staging decision is read at
[`bitty-docs` `origin/main` revision `be68c93cee1638c65932f5dba44cd4e2fadbe265`](https://github.com/bitty-terminal/bitty-docs/blob/be68c93cee1638c65932f5dba44cd4e2fadbe265/docs/decisions/adrs/ADR-0019-bittyd-staging-advance.md).
Those revisions are cited as review inputs, not as accepted implementation
evidence.

## Purpose and scope

This page gives the AI-side consumer a narrow, reviewable boundary for the
proposed `bittyd` session host. It has four purposes:

1. Preserve the accepted `AgentSession` lifecycle and bounded Agent vocabulary
   without treating it as a daemon record.
2. Separate the proposed application session generation from the proposed
   `bittyd` worker-binding generation and from other AI, panel, and execution
   generation axes.
3. State the consumer decisions and negative requirements that must be resolved
   before Core can promote a worker-hosting contract.
4. Define the AI-side review and handoff from CTX-1525 to a future CTX-1527
   without implementing an adapter or worker supervision.

In scope: AI-side identity and lifecycle boundaries, consumer authority,
daemon-absent behavior, restart behavior, state placement, open owner
decisions, and security/conformance dependencies. Out of scope: a `bittyd`
wire schema, a transport adapter, worker spawning or reaping, a fake server,
new capability identifiers, provider I/O, and any change to the accepted IPC
wire or Agent message bounds.

## Normative sources this specification must not weaken

This draft is subordinate to the following sources. A conflict is resolved in
favor of the source named in the table; this page records consumer consequences
only.

| Source                                                                                                                                                                                                             | Consumer consequence                                                                                                                                                                                                                |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [IPC and Agent RFC](../specifications/ipc-agent-rfc.md), accepted at design level                                                                                                                                  | `AgentSession`, bounded history and observation queues, peer-credential authentication, scopes, consent, framing, streaming, and fail-closed behavior remain authoritative. This page adds no method, scope, bound, or credential.  |
| [Headed Agent contracts (candidate)](../agent/headed-agent-contracts-candidate.md)                                                                                                                                 | The Headed Agent lifecycle and application-session generation remain draft. Its three-level separation of `AgentSession`, application handle, and proposed `bittyd` session is the immediate AI-side design input.                  |
| [Session model (candidate)](../specifications/session-model-candidate.md)                                                                                                                                          | A session is not a panel, context, directory, snapshot, or agent. Durable AI session facts and compiled context remain separate candidate concerns.                                                                                 |
| [Execution supervisor (candidate)](../specifications/execution-supervisor-candidate.md)                                                                                                                            | AI coordination and assignment policy remain distinct from host execution facts and worker-binding/execution fencing. No AI-side generation may authorize a host effect by itself.                                                  |
| [AI Architecture](../architecture/ai-architecture.md)                                                                                                                                                              | Agent/plugin generations, context freshness, consent, and model-facing state remain AI-side concerns; host enforcement still happens at the authorized boundary.                                                                    |
| [ADR-0019 `bittyd` staging advance](https://github.com/bitty-terminal/bitty-docs/blob/be68c93cee1638c65932f5dba44cd4e2fadbe265/docs/decisions/adrs/ADR-0019-bittyd-staging-advance.md)                             | The v0.2.0 slice is local, per-user, single-instance, session-grained, versioned, and explicitly excludes restart persistence. Daemon absence remains supported.                                                                    |
| [`bittyd` session-host requirements (candidate)](https://github.com/bitty-terminal/bitty-terminal-docs/blob/575e8b1d93efb35e6c76b19ade8f034eb64ff434/specifications/bittyd-session-host-requirements-candidate.md) | The Core-owned candidate is consumer input, not an accepted contract. Its single-writer, `worker_sessions=false`, adapter, lifecycle, restart, trust, and conformance requirements are recorded here as dependencies and positions. |

## Terminology and authority split

The word **session** is qualified below. An unqualified session must not be
used in an interface or review finding when one of these records is intended.

| Record              | Meaning and lifecycle                                                                                                                                                                                                                                                       | Generation or revision                                                                                                                                                                                                     | Sole authority and status                                                                                                                                                    |
| ------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `AgentSession`      | The accepted IPC and Agent RFC bounded Agent object. It owns one `AgentId`, a `ToolRegistry`, bounded history, and a bounded `SideQueue`; its accepted lifecycle is `Created -> Running <-> WaitingToolResult -> Completed` or `Failed`. Creation grants no tool authority. | Its RFC lifecycle is not a `bittyd` worker fence. Agent/plugin generation and runtime implementation state are separate draft axes.                                                                                        | The accepted RFC defines the contract at design level; no shipped implementation is claimed. Tool effects are separately authorized by the host.                             |
| Application session | The proposed AI-side session used by `start`, `attach`, `detach`, `stop`, approvals, session-scoped jobs, and the Headed Agent application handle. It binds model-facing session state and presentation attachments without making a panel authoritative.                   | Proposed application session generation; attachment generation is local to one attachment. The application generation is not a worker generation, panel generation, Agent generation, routing version, or context version. | `bitty-ai` consumes and records the application semantics. Issuer, durable placement, and sequencing remain owner-pending under `bitty#2546`/CTX-1525.                       |
| `bittyd` session    | The proposed Core daemon record for a registered and supervised host unit. The accepted v0.2.0 staging is terminal-session-only; a worker session is not available in that slice.                                                                                           | Server-assigned session id, proposed worker-binding generation, and separate state revision. The worker-binding generation changes on worker bind or rebind, not on attach or detach.                                      | `bittyd` alone writes registry, lifecycle, worker-binding generation, attach count, leases, and last-seen state if the candidate contract is accepted. No daemon is shipped. |
| Worker/run state    | The run's task, context, tool, provider, consent, model, and audit semantics, including AI interpretation of host execution outcomes.                                                                                                                                       | AI assignment and execution axes remain distinct; a host worker generation only fences which host worker a request targets.                                                                                                | The worker/AI runtime owns its AI state. `bittyd` must not store or mutate it.                                                                                               |

### Accepted AgentSession lifecycle versus proposed host lifecycle

The accepted RFC lifecycle is the AI protocol anchor. It does not become a
`bittyd` state machine by being transported through a daemon:

```text
Accepted AgentSession:
Created -> Running <-> WaitingToolResult -> Completed
                                      \-> Failed

Proposed bittyd host relation:
registered -> running / detached -> terminal or unavailable
```

The second diagram is intentionally illustrative. Its state names, terminal
reason taxonomy, revision semantics, attach leases, and worker binding remain
candidate Core contract material. In particular:

- `AgentSession::Completed` or `Failed` is an AI protocol outcome. It is not
  proof that a `bittyd` worker session exists or that a host process was
  restarted safely.
- A `bittyd` attach or detach is a host-client relation. It must not create,
  complete, cancel, or otherwise mutate an `AgentSession` merely because a
  client or panel disappeared.
- A worker exit, daemon crash, or daemon restart is a host fact. The AI runtime
  must reconcile it into its own run/session outcome rather than infer an
  `AgentSession` terminal state from a lost connection.
- `start` may create an application handle and an `AgentSession` without
  creating a `bittyd` session. Whether a future hosted mode creates all three
  records is an owner decision, not an implication of the operation name.

## Generation separation

There are at least four independent axes in the current AI design input:

| Axis                               | Fences                                                                                                                                                   | Must not be used as                                                                               |
| ---------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| Application session generation     | Session-level mutation, approval, consent/elevation narrowing, session stop, and session-scoped jobs.                                                    | A host worker selector, panel handle, provider routing version, or grant.                         |
| `bittyd` worker-binding generation | Which worker binding a host request targets. It is owned by `bittyd`, does not change for attach/detach, and is invalidated by the host's binding rules. | Application session authority, approval or consent generation, or proof that AI state is current. |
| Agent/plugin generation (`AG-2`)   | AI agent/plugin elevation and context freshness.                                                                                                         | A `bittyd` session or host execution authorization.                                               |
| Panel/attachment generation        | One panel or attachment's local handles, cursor, and presentation resources.                                                                             | Session authority, consent, or worker identity.                                                   |

The non-conflation rule is a promotion blocker: the application session
generation and `bittyd` worker-binding generation must be represented as
different fields, with different issuers and validation rules. If Core places
the application session record in `bittyd`, its contract must add a distinct
application-session-generation field and name its sole writer. Reusing the
`R-S16` worker-binding generation for application authority is rejected. If the
record is not placed in `bittyd`, the adapter must not synthesize a second
application generation from the worker generation.

Generation values are fences, not secrets or capabilities. Possession of any
generation does not grant a scope, consent, control lease, or permission to
invoke a tool.

## AI consumer positions before `bittyd` promotion

The following positions are required inputs for CTX-1525. They do not promote
the candidate Core contract or authorize CTX-1527 implementation.

### v0.2.0 worker capability is explicitly disabled

The AI consumer position is that every v0.2.0 `bittyd` handshake advertises
`worker_sessions=false`. A worker-hosting request must fail with a typed
unsupported-capability result; the adapter must not emulate a worker session as
a terminal session, silently start a second in-process worker, or interpret a
terminal session as an Agent session. Worker hosting requires a later owner
decision, an ADR-0019 amendment, and security review of worker spawn,
environment/credential inheritance, supervision, and cleanup.

This does not prevent `bitty-ai` from using its non-daemon execution path where
that path is separately authorized. It means that a selected `bittyd` transport
cannot claim worker hosting until the capability is true and the conformance
gate is passed.

### No daemon-restart persistence

The AI consumer must treat a daemon instance id as part of the host session
identity. A clean restart, crash, or reboot invalidates old host handles; the
v0.2.0 registry is empty on a new daemon instance and old sessions are not
resumed. The AI side may retain independently governed records, subject to its
own consent and retention policy, but must not present retained AI records as a
live resumed `bittyd` session or replay effects.

Detach and reattach within the same daemon instance are different from restart
persistence. Any future reattach must still pass the current host contract's
instance, session, generation, lease, authorization, and bounded-backlog
checks.

### Daemon-absent operation is supported

`bitty-ai` must have a safe, explicit non-daemon mode. Startup, diagnostics,
headless operation, and the non-hosted AI path must not require `bittyd`. If a
caller explicitly selects the `bittyd` transport and the service is absent,
the result is typed unavailable; the adapter must not silently fall back to a
second worker path while a hosted run might exist. Transport selection and
dual-run prevention belong in the future adapter contract.

This is the AI-side consequence of ADR-0019 D5 and the accepted IPC posture:
absence is a supported state, not permission to weaken authentication or to
create an ambient local endpoint.

### Application and worker generations never conflate

Application operations such as approval, session stop, takeover, and
session-scoped media control use the application session generation. A host
abort or worker-control request uses the current `bittyd` worker-binding
generation in addition to its authenticated principal, scope, and any required
idempotency key. A stale host generation must be reconciled, not silently
replayed against a newer worker. A stale application generation must not be
converted into a host generation and must fail its own AI-side gate.

The AI runtime also keeps `Unknown` reconciliation separate from a terminal
success or failure. A transport timeout or daemon loss is not evidence that a
worker completed, failed, or was cancelled.

### Adapter authority is narrow

The future AI adapter is a transport and mapping boundary, not a second
authority. It may negotiate versions/capabilities/limits, map an explicitly
selected hosted run, attach/detach, read status/events, and request a
generation-checked abort if the caller has already passed AI policy gates. It
must not:

- create or supervise a daemon, registry, lease table, or worker;
- signal a worker pid or use an OS handle as an authority token;
- decide AI consent, provider policy, task ownership, retry policy, or whether
  a cancellation is appropriate;
- treat a daemon scope grant as a substitute for an AI-side policy gate, or an
  AI grant as a substitute for server-side scope evaluation;
- maintain a competing lifecycle truth; its cache is display/reconciliation
  data and must be revalidated before a mutation; or
- silently dual-run, auto-restart, or rebind a worker after uncertainty.

The host remains authoritative for process/PTY ownership, worker supervision,
resource enforcement, and host-side generation checks. The AI side remains
authoritative for task, agent, context, provider, tool, consent, audit meaning,
and retry interpretation.

### No AI state is stored in `bittyd`

The proposed daemon boundary carries only the minimum opaque run-control and
lifecycle metadata required by its host contract. `bittyd` must not store or
mutate AI task graphs, prompts, transcripts, context snapshots, provider/model
selection, credentials, tool arguments/results, consent decisions, agent
roles, workspace contents, or AI audit content. Lifecycle/refusal records may
contain bounded attribution and opaque identifiers, but not AI content or
secrets.

The AI runtime remains the source of record for AI state. Losing a daemon
session record must not erase already-persisted AI work products, and retaining
AI state must not create permission to resume a lost host worker.

## Owner decisions and dependencies

### Decisions required from Core and shared security owners

The following remain open even where this page records an AI position:

| ID                       | Owner decision required                                                                                  | AI consumer position or dependency                                                                                                           |
| ------------------------ | -------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| D-01 / `worker_sessions` | Whether any non-terminal worker is in the v0.2.0 slice.                                                  | Keep `worker_sessions=false`; require a later ADR/security gate before `true`.                                                               |
| D-02, D-03               | Instance/session identity and separate generation versus state-revision semantics.                       | Old instance handles fail closed; worker generation and application generation remain distinct; one writer per fact.                         |
| D-04, D-05, D-06         | Lifecycle/reason taxonomy, daemon stop behavior, crash/restart and orphan policy.                        | No silent worker restart or adoption; daemon restart ends host sessions; AI outcomes reconcile as unknown where appropriate.                 |
| D-07, D-08, D-09         | Attach/abort scopes, client identity, observer/controller cardinality, and control transfer.             | Attachment never grants AI authority; one controller unless Core explicitly accepts another model; attribution is bounded and server-issued. |
| D-10, D-12, D-14         | Event ordering/backlog, heartbeat and lease expiry, and resource/retention limits.                       | Control notifications cannot be silently lost; expiry detaches rather than aborts; limits are published and bounded.                         |
| D-11, D-13, D-18         | Service version negotiation, timeout ownership, and idempotency.                                         | Version mismatch refuses; IPC timeout is not execution failure; mutating retries require reconciliation, generation, and idempotency.        |
| Trust boundary           | Peer-credential re-check, endpoint integrity, same-user residual, platform behavior, and scope families. | No weaker fallback; no remote interpretation; security corpus and Core co-own evidence.                                                      |

The exact Core method names, field names, state spellings, numeric limits, and
scope identifiers are not selected by this page.

### AI-side decisions still required

Before an adapter task can be accepted, `bitty-ai` must decide or document:

1. whether `start` creates an `AgentSession`, an application handle, a hosted
   `bittyd` session, or only references an existing host record in each mode;
2. how the application session generation is issued, persisted, sequenced, and
   invalidated without sharing a counter with the worker generation;
3. how host lifecycle facts map to the accepted `AgentSession` states and to the
   AI runtime's draft `Active`/`Completed`/`Failed`/`Canceled` vocabulary;
4. how daemon absence, daemon restart, transport timeout, worker crash, and
   `Unknown` outcomes are represented without claiming recovery or success;
5. which AI audit fields are retained locally, with consent, redaction, and
   deletion bounds; and
6. which explicit transport-selection policy prevents an in-process and
   hosted worker from running for one run intent.

### Security and conformance dependencies

Promotion and any later implementation depend on evidence outside this draft:

- accepted IPC framing, peer authentication, server-side scopes, RC-9/RC-10
  bounds, bounded queues, and untrusted-observation separation;
- ADR-0019's local daemon gates: peer credentials before privileged actions,
  per-domain single writer, bounded persistence, attribution, secret
  minimization, and daemon-absent `bitty --safe` behavior;
- a Core decision and security review for worker spawn, environment/credential
  inheritance, process-tree cleanup, orphan handling, and worker restart policy;
- no provider credentials, prompt bodies, transcripts, tool payloads, or AI
  state crossing into `bittyd` unless a later separately reviewed contract
  widens the boundary; and
- a conformance suite that tests the real service after a fake-server phase;
  fake-server success is adapter evidence only and is not evidence that
  `bittyd` conforms.

At minimum, the AI consumer review must cover the candidate vectors for
unsupported worker capability, lease expiry without abort, stale generation,
worker crash without restart, daemon restart, daemon absence, direct-signal
prohibition, content-free audit, and distinct cancellation versus timeout.

## Security alignment and traceability

This page preserves the accepted read-only/default-deny Agent posture. A
presentation attachment, application handle, daemon session, worker generation,
or event subscription grants no authority. Model output and terminal content
remain observation data, not instructions. Every effectful operation must still
pass the accepted host authorization and consent path before dispatch.

The principal trust boundary is the local authenticated IPC boundary. A
same-user peer is not thereby a trusted AI principal; the daemon and AI policy
must both fail closed on missing or stale authority. The future adapter must
avoid secret-bearing metadata and must never turn endpoint discovery,
generation possession, or a daemon capability bit into a capability grant.

The security review must explicitly examine the boundary between:

1. AI policy and host enforcement;
2. application session state and daemon lifecycle metadata;
3. worker-binding generations and AI/session generations; and
4. daemon restart/loss and AI retention or resume behavior.

## Verification plan

This document is verified by documentation review, not runtime execution. The
review must confirm that:

1. the accepted RFC lifecycle and bounded semantics remain unchanged;
2. every proposed `bittyd` claim is marked candidate, owner-pending, or a
   consumer position;
3. `worker_sessions=false`, no restart persistence, daemon absence, generation
   separation, adapter authority, and the no-AI-state rule are all explicit;
4. cross-repository links resolve to the cited current revisions and no local
   path, username, socket, or environment value is hardcoded; and
5. no adapter, worker supervisor, daemon, fake server, or conformance result is
   implied by the page.

Future CTX-1527 verification, if authorized, must add deterministic adapter
tests for the selected transport and run the Core-owned conformance vectors
against a labeled fake server and then the real service. It must not treat
passing a fake server as service acceptance, and it must not begin worker
hosting while `worker_sessions=false` or the security dependencies remain
unresolved.

## Alternatives considered

- **Treat every session as an `AgentSession`.** Rejected: the accepted RFC
  object has bounded Agent semantics and no daemon registry, worker lease, or
  process authority.
- **Reuse the `bittyd` worker generation as the application session generation.**
  Rejected: the two values fence different facts, have different owners, and
  must not let a host binding accidentally authorize an AI approval or consent
  decision.
- **Make `bittyd` the AI state store.** Rejected: it creates a second source of
  truth, expands the daemon trust boundary, and couples AI retention to daemon
  restart behavior.
- **Require `bittyd` for every AI run.** Rejected: ADR-0019 requires daemon
  absence to remain supported and a non-daemon path is needed for safe,
  headless, and pre-promotion operation.
- **Implement an adapter and worker supervision in this document's task.**
  Rejected: the Core contract, security evidence, and capability advertisement
  are not yet promoted; implementation belongs only to a separately authorized
  follow-up.

## Affected contracts

This page intentionally changes no accepted contract. It adds a consumer design
input to the AI documentation set and depends on:

- the accepted [IPC and Agent RFC](../specifications/ipc-agent-rfc.md);
- the draft [Headed Agent contracts](../agent/headed-agent-contracts-candidate.md);
- the draft [Session model](../specifications/session-model-candidate.md) and
  [Execution supervisor](../specifications/execution-supervisor-candidate.md);
- ADR-0019 and the terminal-platform `bittyd` candidate requirements; and
- the shared security corpus and the eventual Core-owned `bittyd` contract.

The AI documentation index does not gain an accepted-contract entry from this
page. It remains a draft integration input until the owning reviews decide its
disposition.

## Open points

1. Core must decide D-01 through D-18 (or explicitly supersede them) before a
   worker-hosting adapter can be treated as a compatibility target.
2. The owner of the application session generation must be named, including
   whether its record lives in `bitty-ai`, another AI-owned service, or a
   separately accepted Core record.
3. The exact mapping from host lifecycle outcomes to accepted `AgentSession`
   outcomes must be reviewed without extending the RFC state machine.
4. The service scope family for hosted worker control must be accepted; existing
   terminal scopes must not be stretched by implication.
5. Restart and retention rules must specify how independently retained AI
   records are presented after a daemon instance disappears without implying
   host-session recovery.
6. CTX-1525 must record the Core/security disposition and handoff criteria;
   CTX-1527 must remain blocked until those criteria are met.

## Acceptance criteria

This draft input is ready for independent review when:

- the accepted `AgentSession` lifecycle is separated from the proposed
  `bittyd` session and worker-binding generation;
- the six required consumer positions are explicit: `worker_sessions=false`,
  no restart persistence, daemon absence, no generation conflation, narrow
  adapter authority, and no AI state in `bittyd`;
- owner decisions, security dependencies, and conformance dependencies are
  named without silently deciding them;
- the CTX-1525 and future CTX-1527 handoff is explicit and implementation is
  excluded; and
- the document remains English, draft, self-contained, repository-relative for
  AI sources, and free of generated artifacts or product code.

## P0 Review Sign-off

This candidate has no P0 approval. Before any worker-hosting promotion, review
is required from:

| Reviewer              | Required disposition                                                                                                                                 |
| --------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| `bitty-ai` owner      | Confirm AgentSession/application-state mapping and AI-side retention, generation, and adapter boundaries.                                            |
| Core / `bittyd` owner | Decide the host contract, field authority, capability advertisement, restart behavior, and worker-hosting scope.                                     |
| Security owner        | Review peer authentication, scope separation, worker spawn and credential inheritance, secret minimization, restart loss, and confused-deputy risks. |
| Documentation owner   | Confirm draft status, cross-repository authority, links, terminology, and synchronization with the eventual Core decision.                           |

## References

- [IPC and Agent RFC](../specifications/ipc-agent-rfc.md) — accepted bounded
  Agent and IPC contract.
- [Headed Agent contracts (candidate)](../agent/headed-agent-contracts-candidate.md)
  — AI-side application session, attachment, and generation separation.
- [Session model (candidate)](../specifications/session-model-candidate.md) —
  session/container/history distinctions.
- [Execution supervisor (candidate)](../specifications/execution-supervisor-candidate.md)
  — AI coordination versus host execution and dual-generation design input.
- [AI Architecture](../architecture/ai-architecture.md) — AI generation,
  context, consent, and host-boundary candidate requirements.
- [`bittyd` session-host requirements (candidate), reviewed at `575e8b1d`](https://github.com/bitty-terminal/bitty-terminal-docs/blob/575e8b1d93efb35e6c76b19ade8f034eb64ff434/specifications/bittyd-session-host-requirements-candidate.md)
  — Core consumer requirements, authority matrix, restart and daemon-absence
  positions, and conformance vectors.
- [`bittyd` Session-Host Contract (candidate), reviewed at `575e8b1d`](https://github.com/bitty-terminal/bitty-terminal-docs/blob/575e8b1d93efb35e6c76b19ade8f034eb64ff434/specifications/bittyd-session-host-contract-candidate.md)
  — Core candidate answers and conformance plan; not accepted by this page.
- [ADR-0019 `bittyd` staging advance, reviewed at `be68c93`](https://github.com/bitty-terminal/bitty-docs/blob/be68c93cee1638c65932f5dba44cd4e2fadbe265/docs/decisions/adrs/ADR-0019-bittyd-staging-advance.md)
  — accepted v0.2.0 staging and daemon-absence/restart boundaries.
