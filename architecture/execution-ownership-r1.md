---
title: Execution ownership R1
description: Single-agent execution ownership disposition with an accepted provider-routing snapshot binding contract
category: specifications
audience: contributor
document_type: specification
status: draft
website_publish: false
sidebar_order: 35
---

# Execution ownership R1

> Status: **draft**, with only the provider/routing snapshot binding contract in
> [Provider/routing snapshot binding contract](#providerrouting-snapshot-binding-contract-accepted-2026-10-17)
> accepted as a scoped maintained working contract. The broader R1 material
> remains a draft disposition for single-agent execution ownership, its P0
> sign-off slots remain pending, and it does not authorize shipped behavior.
> Normative security and IPC obligations override any experimental adoption
> stated here. Multi-agent scope is frozen and deferred; see [Frozen
> multi-agent scope](#frozen-multi-agent-scope).

## Purpose and scope

This decision covers single-agent execution ownership only:

- `ExecutionContext` as primary execution record versus headless-panel-first
  ownership.
- Validity of no-shell and no-panel execution.
- Cancellation split between pre-dispatch prevention and post-dispatch
  `Unknown` reconciliation without rollback.
- Draft disposition of the MP-1 (Registry ownership) versus BA-2 (Agent versus
  AI split)/BA-3 (Bridge process model) registry conflict, preserving
  the rule that `bitty-agent` performs no model I/O.

Inputs are the single-agent sections of
[Agent coordination architecture](../agent/agent-coordination.md), the exec and Panel
material in [Command and tool architecture](command-tool-architecture.md), MP-1
(Registry ownership), BA-2 (Agent versus AI split), BA-3 (Bridge process
model), and MP-7 (`cancel`) in [AI Architecture](ai-architecture.md),
AIQ-29, AIQ-2A, and AIQ-38 in
[AI Unresolved Questions](../product/ai-unresolved-questions.md), and
[v0.1 Implementation Profile](../product/implementation-profile-v0.1.md). The accepted
[IPC and Agent RFC](../specifications/ipc-agent-rfc.md) and the canonical security corpus linked
by [AI Architecture](ai-architecture.md) remain overriding authority.

No product code is introduced or described as implemented.

## Normative sources this specification must not weaken

This draft R1 material does not replace or relax the accepted security, consent,
authorization, IPC framing, redaction, budget, and lifecycle requirements
linked by [AI Architecture](ai-architecture.md) and the accepted
[IPC and Agent RFC](../specifications/ipc-agent-rfc.md). The existing R1
requirement for a read-only, versioned, credential-free metadata snapshot,
required invalidation, and fail-closed behavior on stale or missing state
continues to govern this contract. A conflict is resolved in favor of those
normative sources; this document is not an implementation authorization.

## Terminology

- **Provider/routing snapshot**: The accepted same-domain projection of the
  populated provider registry used to bind provider-list observations. It is
  not an MCP tool-list snapshot, context snapshot, session snapshot, or
  completion-request snapshot.
- **Domain**: The fixed binding namespace carried with a publication. The
  first accepted value is `provider-routing`; a version or digest without
  this domain is not a provider/routing binding.
- **Canonical bytes**: The exact UTF-8 JSON bytes produced by the canonical
  projection rules below. The SHA-256 digest covers these bytes, including
  their exact field names, ordering, and escaping.
- **Equality proof**: Helper-side evidence that the validated publication
  projection is byte-for-byte equal to the helper's populated registry
  projection at the publication boundary. A matching version alone is not
  equality proof.
- **Opaque binding capability**: A helper-issued, non-forgeable in-memory
  proof that an exact publication acknowledgement and equality proof passed.
  It is not a caller-supplied credential, a provider credential, or a routing
  decision.

## Decision

**R1 selects option A as draft disposition: `ExecutionContext` primary with
optional Panel projection.**

The reconciled flow is:

```text
agent.exec()
  -> authorized ExecutionContext with captured target and generation
  -> supervised structured process, or PTY only when required
  -> bounded redacted execution evidence
  -> structured result

optional Panel -> projection of execution and retained evidence
```

**Option B is rejected as an ownership model.** The mandatory
`agent.exec -> Headless Panel -> PTY/process` path (via
[Command and tool architecture](command-tool-architecture.md))
is not adopted. A headless panel must not be required per agent, per execution,
or as the authority for process ownership, cleanup, or consent. Headed and
headless describe presentation only, not authority, persistence, or lifecycle.

This aligns with the existing draft reconciliation in
[Agent coordination architecture](../agent/agent-coordination.md) and
[Command and tool architecture](command-tool-architecture.md), and with the
experimental adoption row already recorded in
[v0.1 Implementation Profile](../product/implementation-profile-v0.1.md).

## Rationale

- An agent acquires an authorized execution target, not authority through panel
  occupancy. Deriving working directory, environment, or capability from the
  currently focused panel would conflate presentation with ownership.
- A structured build, test, or analysis task needs neither a shell nor a panel.
  Requiring either would force unnecessary PTY, shell, and UI surface for work
  that is fully described by a captured target and a supervised process.
- The supervising execution backend owns process and PTY handles, generations,
  and cleanup. A Panel may later project the same execution without recreating
  it, and many-to-many observation keeps independent lifetimes.
- PTY carries human-compatible terminal semantics only: stdout and stderr
  merge into a single byte stream at the PTY/kernel layer, so a terminal
  emulator cannot reliably attribute spans to streams. Structured command
  results (stdout, stderr, exit code, artifacts, task outcome) must therefore
  travel via the Execution Service, Process Supervisor, Shell Integration,
  typed outcome, or IPC — never be derived from PTY output. This is why
  execution, agent task/job state, and PTY stay separate: the PTY shows, the
  supervisor knows.
  Current-state note (2026-10-04): per ADR-0016 Boundary 1, the Process
  Supervisor and Execution Service mechanism has been extracted from Core into
  the dedicated extension repository `bitty-execution`
  (`bitty-terminal/bitty-execution`). Core executes interactive terminal PTYs
  only and does not supervise agent background execution.
- Presentation movement (hide, show, move, detach) changes attachment only. It
  never moves the execution target, broadens context access, or manufactures
  consent.

## No-shell and no-panel validity

Both no-shell and no-panel executions are valid when authorized:

- No-shell execution covers supervised structured processes without shell
  parsing, shell history, or terminal emulation.
- No-panel execution covers background or headless work with no visible UI.
  Such an execution may later be projected by a Panel without being recreated.
- No-panel execution does not imply a persistent terminal daemon, detach and
  reattach across restarts, remote UI, or any new transport authority. Those
  broader features remain subject to the accepted headless deferral (ADR 0008)
  and trust-boundary gate cited by [Agent coordination architecture](../agent/agent-coordination.md)
  and [IPC and Agent RFC](../specifications/ipc-agent-rfc.md).
- No-UI scope for v0.1 is bounded task execution under explicit authorization,
  budgets, and evidence retention. Long-running servers, persistent services,
  and background daemons require an explicit feature-profile selection beyond
  this decision.

## Cancellation contract

Cancellation follows MP-7 (`cancel`) with an explicit dispatch boundary:

- **Pre-dispatch:** cancellation prevents any tool effect from starting.
  Incomplete streamed tool arguments and cancelled proposals are never
  dispatched and cause no effect.
- **Post-dispatch:** cancellation stops further admission and requests bounded
  cancellation of owned work. Already-started effects may have happened.
  The outcome is the actual recorded result or `Unknown`; reconcile uncertain
  effects by status inspection or user direction before retry.
- Cancellation never promises rollback or exactly-once effects.
- Cancellation of one waiter never cancels shared execution still required by
  another authorized waiter.
- Structured exec results must disclose failures, truncation, and `Unknown`
  outcomes with evidence references; a reference permits retrieval only after
  reader authorization and only while redacted records remain retained.

## MP-1 (Registry ownership) versus BA-2 (Agent versus AI split) and BA-3 (Bridge process model) disposition

**Red line preserved: `bitty-agent` performs no model selection, no model I/O,
and no API-key handling.**

MP-1 describes a host-owned and host-validated `ai.model` registry with
deferred owning-crate alternatives. Read literally as placing provider
implementation, model selection, LLM I/O, or API keys inside terminal Core,
those alternatives conflict with BA-2 and BA-3. This decision resolves the
conflict as draft disposition:

- BA-2 (Agent versus AI split) and BA-3 (Bridge process model) win on placement. Provider registry implementation and all
  model I/O belong in the independent AI helper behind scoped IPC, consistent
  with the bridge process model. The terminal side never loads AI code into
  the main process. Under [DIR-030](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/index.md)
  the helper is the native component `ai` (executable `bitty-ai`), resolved
  from the component install root (never `PATH`), digest-verified, and spawned
  as a stdio coprocess; the current-state notes in [AI Architecture](ai-architecture.md)
  and the accepted [IPC and Agent RFC](../specifications/ipc-agent-rfc.md) record that
  terminal Core no longer links `bitty-agent`, whose intended consumer is that
  out-of-process component.
- A terminal-side registry, if retained, validates generic service metadata
  and mediates authorized requests only. It is not a provider implementation
  and holds no credentials.
- Exact split, narrowed to two draft options (AIQ-38 stays open; this section
  grants no permission to implement either option):
  - **Option S-1 (metadata-only terminal registry).** The terminal side keeps
    a generic service-metadata view (provider and model names, declared
    capabilities, routing facts) validated at registration; it mediates
    authorized requests and enforces scopes, consent, and budgets at the IPC
    boundary. Registry implementation, selection-policy execution, provider
    I/O, and credentials stay helper-side.
  - **Option S-2 (protocol-only terminal side).** The terminal side keeps no
    registry state; every AI request is forwarded over scoped IPC to the
    helper, which owns registry, selection, and I/O. The terminal side still
    enforces scopes, consent, and budgets at the IPC boundary and still
    exposes no AI-specific API beyond generic primitives.
  - Both options preserve the red line above, the BA-6 pressure-test gate in
    [AI Architecture](ai-architecture.md) (no terminal-Core change for AI; each
    demanded Core AI-specific API is a Plugin API gap, not a feature), and the
    rule that every effect passes caller/target authorization, consent,
    budgets, redaction, and attributed outcomes. Choosing between them needs
    the architecture owner: the deciding evidence is which option keeps
    failure semantics (post-dispatch `Unknown` reconciliation across IPC),
    the consent ledger, and the budget ledger in one auditable place without
    duplicating registry state.
- Transport selection between native tools and MCP routing stays open under
  AIQ-36 and AIQ-38. Neither path bypasses the common authorization,
  target and generation binding, schema and effect validation, consent,
  budgets, redaction, and attributed outcomes required for every effect.

## Scoped helper IPC surface (draft)

This section sketches the draft IPC surface between the terminal side and the
`bitty-ai-host` helper. It is a candidate derived from BA-3 in
[AI Architecture](ai-architecture.md), DIR-030, and the accepted
[IPC and Agent RFC](../specifications/ipc-agent-rfc.md). On any conflict the
RFC's normative statements win, and this section grants no implementation
permission.

- **Transport shape.** Under DIR-030 the helper is the native component `ai`
  (executable `bitty-ai`): terminal Core resolves it from the component
  install root (never `PATH`), verifies its digest, and spawns it as a stdio
  coprocess. The helper may use the bitty-network crates only under a
  Core-issued network capability grant, which it never widens. IPC stays
  local-user-only by default with an explicit scope per operation, and framing
  stays bounded per the RFC's transport and framing rules; exact helper-side
  values are not pinned here.
- **Op families (candidate).** Provider-turn ops (selection plus
  complete/stream behind the helper, with streamed turns in sequenced,
  byte-ceiled chunks); tool-effect dispatch ops (real execution happens in the
  host/runtime under capability-checked dispatch, rate limits, per-client
  scopes, consent prompts, and audit — `bitty-agent` itself never executes a
  tool); observation and context reads (read-only by default; terminal content
  is untrusted observation data, never instructions); consent-elevation ops
  (per scope, ledgered, revocable); and budget/accounting ops (usage reported
  helper-to-terminal against the same per-client quotas). No op family widens
  authority: possession of the channel grants nothing beyond presenting a
  request for server-side evaluation.
- **Budget handoff.** Terminal-side per-client quotas follow the RFC's rate
  limit and budget rules (request rate and payload caps, stream chunk
  ceilings, newest-first shed with attributed records and countable metrics).
  Helper-side pre-I/O gates follow the CP-5 rule (budget gates every request
  before provider I/O) with typed `BudgetExceeded` failure per MP-5/MP-9.
  Usage is reported, never self-granted, on both sides of the boundary.
- **Consent handoff.** Default deny for a fresh client identity; elevation
  requires a separate grant per scope showing the exact method set it enables
  (no bundled admin grant); every grant is ledgered (who, agent identity,
  scope, method set, grant and expiry time, granter) and visible on an
  inspection surface; revocation is immediate; terminal-output content is kept
  separate from filesystem- or network-authorized dispatch per the RFC's
  confused-deputy rule.
- **Redaction handoff.** Scrubbed views are required at every log or IPC
  crossing (redaction marker, bounded scrubbed-output size per the RFC's
  recorded scrubbing evidence, which stays experimental review evidence, not a
  normative bound); raw arguments and results are retained only for host
  dispatch, never for logs or IPC; typed secret-field machinery and the
  secret-exposure-via-traces rule apply unchanged across the boundary.

## Provider/routing snapshot binding contract (accepted, 2026-10-17)

This section promotes the existing provider/routing snapshot binding candidate
in place. It accepts the same-domain contract and acceptance matrix only. It
does not claim that publication, invalidation, helper registry population, or
live Core-to-helper acknowledgement wiring exists, and it grants no
implementation permission for Core transport, wire, or Lua integration.

The historical candidate record remains recognizable through the original
projection shape, the `provider-routing` domain, the helper-side equality seam,
and the fail-closed matrix below. The candidate's open decisions are resolved
here; residual transport and implementation questions are explicit in [Open
points](#open-points).

### Domain boundary

The provider/routing snapshot is a distinct domain named `provider-routing`.
It must not be inferred from, substituted by, or compared only by version to
any other snapshot domain, including:

- the MCP tool-list/context domain represented by
  `SnapshotEnvelope.list_version`;
- a context or terminal-observation snapshot;
- a session or execution snapshot; or
- a provider completion, request, or response snapshot.

Every publication, acknowledgement, binding capability, and strict request
correlator must carry or be associated with the domain tag. The MCP
`list_version` is never a provider/routing binding. A same numeric version in
two domains is a mismatch, not a match. Current AI-0303 behavior at
`bitty-ai@6757b1e21ed27941b910ce0452834806fafc9c4b` correctly keeps the MCP
list version separate and refuses the strict seam as `Unbound` until this
same-domain contract is accepted and wired.

### Canonical projection and bytes

The accepted initial strict target is `ai.provider_list`. Its provider/routing
projection is exactly the populated ProviderRegistry model data, with no
provider implementation, selection result, request, credential, or secret
material. The projection contains:

- each registered model's `provider_id`;
- each registered model's `name`;
- its closed capability labels (`text`, `streaming`, `tool_use`, `image_input`,
  `audio_input`, `audio_output`, `video_input`, and `reasoning`);
- its `context_window_tokens` value, where `0` retains the registry meaning
  of unknown; and
- its relative `input_cost_weight` and `output_cost_weight` values, where
  zero retains the registry meaning of an uncalibrated baseline weight.

The accepted projection does not include provider display names or free-form
`routing_facts`: those are accepted by the current metadata-publication seam,
but are not fields in the current ProviderRegistry projection used by
`ai.provider_list`. It also excludes aliases initially. Aliases are a
separate registry table and must not affect a binding silently. A later alias
policy may include them only if alias names, candidate order, and all target
references are projected, bounded, canonicalized, hashed, and included in the
same equality proof. Until then, alias mutation invalidates any binding if it
can affect the populated registry state, and the initial strict contract does
not claim alias-aware coverage.

The accepted payload shape is the following canonical JSON object:

```json
{
  "domain": "provider-routing",
  "providers": [
    {
      "id": "provider-id",
      "models": [
        {
          "name": "model-name",
          "capabilities": ["text"],
          "context_window_tokens": 4096,
          "input_cost_weight": 1,
          "output_cost_weight": 1
        }
      ]
    }
  ]
}
```

The accepted canonical JSON profile is deterministic and is the contract, not
an incidental serializer choice:

1. The projection is serialized as exact UTF-8 JSON with no insignificant
   whitespace. The only accepted top-level field order is `domain`, then
   `providers`; provider field order is `id`, then `models`; model field order
   is `name`, `capabilities`, `context_window_tokens`, `input_cost_weight`,
   then `output_cost_weight`.
2. Provider objects are sorted by `id`, models within a provider by `name`, and
   capability labels by their canonical label value. Each ordering is ascending
   bytewise order over the UTF-8 bytes of the string. Duplicate providers,
   models, or capability labels are rejected. Registry insertion order is never
   preserved.
3. JSON strings are Unicode scalar sequences encoded as UTF-8. The canonical
   serializer emits non-ASCII scalar values unescaped and uses the shortest
   JSON escape for `"`, `\\`, and U+0000 through U+001F: `\\b`, `\\t`, `\\n`,
   `\\f`, and `\\r` for those named controls, and lowercase `\\u00xx` for the
   remaining controls. Invalid UTF-8, surrogate code points, and unescaped
   control characters are rejected. No alternate escape spelling is
   canonical.
4. The three numeric fields are unsigned JSON integers in the registry's
   `u32` range, encoded in base-10 with no leading zeroes except the value
   `0`. Fractional, negative, non-finite, duplicate, unknown, and out-of-schema
   values are rejected before canonicalization. `0` retains the registry
   meanings of unknown context window and uncalibrated baseline cost weight.
5. The digest is lowercase hexadecimal SHA-256 over exactly the canonical
   UTF-8 bytes described above. The digest does not cover a parsed value,
   pretty-printed rendering, publication frame, acknowledgement, or caller
   parameters. Domain separation is structural and mandatory: the canonical
   bytes contain the exact `"domain":"provider-routing"` member, and a
   publication from any other domain is rejected before equality proof. No
   MCP `list_version` or other domain can satisfy this check.

These rules intentionally match the proven AI-0324 helper projection in
`bitty-ai@6757b1e21ed27941b910ce0452834806fafc9c4b`: the implementation
evidence emits raw non-ASCII UTF-8, escapes JSON controls, uses the field order
above, sorts the closed capability labels, and hashes the resulting bytes. A
future change to the profile requires a new accepted decision and invalidates
existing bindings.

The publication frame and every nested collection remain bounded. The accepted
profile reuses the established publication ceiling of 256 KiB for the
frame and canonical payload, limits model entries to the registry's bounded
maximum of 64 total entries, limits capabilities to 8 labels per model, and
limits identity and label strings to 128 bytes unless a later accepted
canonicalization decision selects stricter bounds. A publication exceeding a
frame, field, collection, or nesting bound is refused before registry mapping
or handler contact. The exact bound values are contract inputs, not hints for
unbounded buffering.

The projection is read-only and credential-free. Core and helper must not
request, load, serialize, hash, log, or transmit provider credentials, API
keys, bearer tokens, secret values, provider implementation state, or model
request content as part of this publication. This prohibition is about the
defined fields and access path; it does not claim that a generic string scan
can detect every possible secret. Validation must reject fields outside the
contract and must use the existing typed secret and redaction controls where
the surrounding system handles sensitive data.

### Ownership and proof

The accepted ownership split is:

- **Core** owns publication authority: it derives and publishes the exact
  canonical provider/routing projection, mints the monotonic
  `snapshot_version` and never-reused publication identity, and owns
  invalidation when its source view mutates. Core's generic
  `snapshot_version` fields on outbound `ai.*` parameters do not by themselves
  constitute this publication or its acknowledgement wiring.
- **The helper** owns the authoritative populated `ProviderRegistry` for this
  contract. It validates domain, identity, lifecycle, canonical bytes, digest,
  schema, and bounds; projects the actual registry consumed by
  `ai.provider_list`; and compares that projection byte-for-byte with the Core
  publication. The helper issues the acknowledgement after exact publication
  validation, then issues the opaque proof only after consuming that exact
  acknowledgement and proving registry equality. A version, digest,
  projection, or acknowledgement echoed by a caller is never authority.
- **Mutation** of any registry state invalidates the proof before strict use,
  including an alias-table mutation even though aliases are excluded from the
  initial projection. Aliases cannot affect strict `ai.provider_list` behavior
  until a later accepted alias projection decision.
- **Transport placement** remains unassigned. A shared DTO or canonicalizer
  belongs in `bitty-ipc` only if a future accepted transport design proves that
  both sides must share it. This acceptance assigns no crate, wire field, Core
  transport, or Lua integration work.

The existing endpoint metadata publication is validation vocabulary and state
only. Its provider ID/name/model/capability/routing-fact payload is not this
projection and has no provider-routing domain tag. The AI-0324 implementation
evidence at `bitty-ai@6757b1e21ed27941b910ce0452834806fafc9c4b` proves a pure
helper-side projection and equality seam for a populated ProviderRegistry,
while strict dispatch remains `Unbound`. The Core snapshot-version
implementation evidence at `bitty@08aa903e85e148a04a57ac2b3e7ebd04c0405098`
emits a Core-owned generic version in outbound `ai.*` parameters, but does not
publish this canonical provider/routing payload or wire live publication and
acknowledgement to the helper.

Publication acceptance and proof issuance require an authenticated
Core-origin publication ingress bound to the authorized Core component and
channel. Caller-supplied identity, version, digest, projection, or metadata
cannot establish Core origin. Missing or invalid Core-origin authentication
fails closed before publication acknowledgement or proof issuance. The
authentication and transport mechanism remains a residual implementation open
point; this contract grants no permission to choose or implement it.

### Lifecycle and mode behavior

The accepted lifecycle is:

- **Initial publish:** Core publishes a new `provider-routing` projection with
  its Core-issued monotonic version and never-reused identity. After the first
  publication, each replacement is strictly greater than the current Core
  version. The helper validates the bounded frame and retains the exact
  publication as pending, then returns an acknowledgement containing exactly
  the received identity, domain, version, and digest.
- **Bind:** The helper consumes only that exact pending acknowledgement, proves
  equality against the current ProviderRegistry projection consumed by
  `ai.provider_list`, and issues the opaque helper proof. No strict handler is
  contacted before proof.
- **Republish and supersession:** A changed valid projection receives a new
  identity and greater version. Publishing a newer version supersedes the
  previous pending publication; the old acknowledgement and proof cannot be
  used. A duplicate identity is refused even when its payload is unchanged.
- **Mutation invalidation:** Any provider, model, capability, context-window,
  cost-weight, or alias-table mutation invalidates the current proof before
  strict use. Core must publish a fresh exact projection and the helper must
  re-acknowledge and re-prove it before strict provider-list use resumes.
- **Fail-closed lifecycle:** Missing, stale, future, replayed, duplicate,
  superseded, wrong-domain, wrong-digest, malformed, oversized, or
  projection-mismatched publications and acknowledgements are refused. “Future”
  means a version or identity that is not the current Core-authorized lifecycle
  state for this helper, including one presented before Core has published it;
  the helper never infers authority from a caller-supplied number. Missing or
  invalid Core-origin authentication is refused before acknowledgement or
  proof issuance. Any strict refusal happens before handler contact, provider
  implementation or provider I/O, credential access, tool execution, or any
  other side effect. Refusal preserves invalidation and never falls back to a
  prior or guessed snapshot.

Strict mode fails closed on every missing or invalid same-domain proof. The
accepted initial strict domain is `provider-routing`, and `ai.provider_list` is
the only strict enablement target. This acceptance does not itself enable live
strict behavior; a separately scoped implementation task must wire it subject
to this contract and the evidence plan below. MCP list, context, session,
complete, tool, and extension domains remain tolerant or not wired until each
receives its own accepted same-domain contract. Their versions must not be
silently cross-bound to `provider-routing`, and MCP `list_version` must never be
reused. The existing default and standalone path remains tolerant and continues
to serve its current behavior without requiring a publication.

### Acceptance matrix

| Case                                                                                                   | Required result                                                                                                                                               |
| ------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Exact same-domain publication, digest, version, and equal registry projection                          | A future strict `ai.provider_list` implementation may proceed only after exact acknowledgement and opaque helper proof issuance.                              |
| Same version with the wrong domain, including MCP `list_version`                                       | Refuse as a domain mismatch; do not issue a capability or contact a handler.                                                                                  |
| Same domain/version with the wrong digest                                                              | Refuse before mapping or handler contact.                                                                                                                     |
| Projection mismatch in a field, provider/model order, capability order, numeric value, or alias policy | Refuse equality proof and remain fail-closed.                                                                                                                 |
| Stale, missing, or future version                                                                      | Refuse; no fallback to a prior or guessed snapshot.                                                                                                           |
| Replay, duplicate identity, or superseded publication/acknowledgement                                  | Refuse and preserve the prior invalidation state.                                                                                                             |
| Provider/model/capability/context/cost mutation                                                        | Invalidate the old binding before strict use; require fresh publication and proof.                                                                            |
| Any strict refusal                                                                                     | Refuse before `ai.provider_list` or other handler contact, provider implementation/provider I/O, credential access, tool execution, or any other side effect. |
| Oversized frame, field, collection, nesting, or count                                                  | Refuse before allocation or registry mapping beyond the declared bounds.                                                                                      |
| Credential field, secret access, or provider implementation access                                     | Reject the publication/access path; do not treat heuristic secret scanning as universal detection.                                                            |
| Standalone or tolerant call without a publication                                                      | Preserve current tolerant behavior and do not require the capability.                                                                                         |

This matrix is part of the accepted contract and is acceptance evidence for a
future implementation, not evidence that the current AI-0303 seam passes any
row beyond its documented fail-closed behavior. The current AI-0303 seam at
`bitty-ai@6757b1e21ed27941b910ce0452834806fafc9c4b` and AI-0324 helper
projection/equality seam at
`bitty-ai@6757b1e21ed27941b910ce0452834806fafc9c4b` is implementation evidence
only; the current Core generic-version path is not exact publication or live
acknowledgement wiring.

## Frozen multi-agent scope

Per the CTX-0009 task scope and the v0.1 non-goals in
[v0.1 Implementation Profile](../product/implementation-profile-v0.1.md), the following
are frozen and excluded from this decision:

- Manager and reviewer agents, teams, and organization graphs.
- Service sharing and LSP broker reuse beyond single-agent bounds.
- Cross-agent mailbox, messaging durability choices, and routing.
- Multi-agent budgets, hierarchical delegation, depth and fan-out policy.
- Independent-review separation criteria for multi-agent acceptance.

Pointers:

- Team, delegation, budget, and organization-graph proposals stay in
  [Agent coordination architecture](../agent/agent-coordination.md).
- v0.1 exclusion and sequencing stay in
  [v0.1 Implementation Profile](../product/implementation-profile-v0.1.md).
- Multi-agent scope returns only after single-agent primitives prove stable.

## Security review

The accepted provider/routing contract preserves the existing trust boundary: the snapshot is
read-only metadata, carries no credentials, and grants no provider I/O or
selection authority. Domain separation prevents an MCP tool-list version from
being confused with provider/routing state. Canonical bytes, exact digest
verification, bounded parsing, mutation invalidation, and no-handler-contact
on refusal address substitution, replay, drift, and resource-exhaustion risks.

The credential prohibition is precise but not a claim of universal secret
detection. A future implementation must use the accepted typed secret,
redaction, authorization, and IPC controls at every access path, and a
security reviewer must verify that no provider implementation, request
content, or credential-bearing state enters the projection. The projection
has no credential field or access path; bounded marker checks and the existing
typed secret/redaction controls apply where the surrounding system handles
sensitive data, without claiming universal credential detection. Any weakening
of fail-closed stale/missing behavior requires the normative security review and
an accepted decision outside this contract.

## Verification plan

A future implementation claiming this disposition must show, at minimum:

- Headless execution without a Panel or shell under an authorized captured
  target and generation, with Panel projection attached later without
  recreating the execution.
- Cancellation races on both sides of dispatch: pre-dispatch starts no effect;
  post-dispatch reports actual or `Unknown` outcomes with reconciliation
  before retry and no rollback claim.
- Shared-execution evidence that one waiter's cancellation does not stop work
  still required by another authorized waiter.
- Structured exec results disclosing failures, truncation, and `Unknown`
  outcomes with bounded redacted evidence.
- Negative evidence that no structured result is derived from PTY output:
  exit codes, stream separation, and artifacts arrive via the supervised
  backend's typed outcome path, consistent with the single-stream limit
  above (a sibling conformance point: `bitty-agent`'s `AgentObservation`
  queue carries `ProcessExited { code }` without stream bytes and never
  places PTY descriptors in the queue).
- Negative evidence that `bitty-agent` performs no model selection, model I/O,
  or API-key handling, and that no Core AI-specific API exists beyond generic
  primitives.
- Deterministic headless coverage with seeded inputs and fail-closed budget,
  consent, and redaction checks.

The accepted provider/routing contract additionally requires evidence for every
row of the acceptance matrix: canonical-byte and SHA-256 reproducibility,
exact same-domain equality, wrong-domain and wrong-digest refusal,
deterministic ordering and escaping, field/count/frame bounds, mutation
invalidation, stale/future/replay/duplicate/supersession handling, no handler
contact on strict refusal, credential/access prohibition, and preservation of
the tolerant standalone path. Evidence must distinguish the current AI-0324
helper-side projection seam at
`bitty-ai@6757b1e21ed27941b910ce0452834806fafc9c4b` and AI-0303 `Unbound`
behavior from any future wired behavior.

## Alternatives considered

- **Use MCP `SnapshotEnvelope.list_version` as the binding.** Rejected. MCP
  tool-list state is a different domain and a same numeric version does not
  prove provider/routing equality.
- **Bind only a version or digest without projecting the registry.** Rejected.
  Equality requires canonical provider/model data and a helper-side proof;
  caller-supplied correlators are not authority.
- **Include aliases without binding their table.** Rejected for the initial
  contract. Alias names and ordered candidates are separate state and must be
  projected, bounded, hashed, and compared together before they can affect a
  strict binding.
- **Treat the current metadata-publication payload as the final projection.**
  Deferred. It is a useful validation seam, but its provider name and
  `routing_facts` fields do not equal the current ProviderRegistry projection,
  and it has no domain tag.

## Affected contracts

The accepted contract affects the future Core publication contract, the helper-side
provider/routing binding and `ai.provider_list` strict admission contract, and
the eventual acknowledgement path over helper IPC. It does not alter the
current tolerant endpoint behavior, MCP list-version contract, provider
selection implementation, or any live Core/helper wire. `bitty-ipc` is an
affected boundary only if an accepted transport design places a shared DTO or
canonicalizer there; this document assigns no implementation location or
implementation permission.

## Open points

This document changes the status of no register entry:

- AIQ-29 (optional Panel and execution projection bindings) stays a design
  choice. Presentation movement cannot move execution targets.
- AIQ-2A (no-UI execution feature profile) stays a scope choice. Bounded work
  versus persistent services needs explicit profile selection.
- AIQ-38 (generic execution and registry ownership across repositories) stays
  a prerequisite for its still-open facets. BA-2 and BA-3 are preserved; the
  S-1/S-2 registry-split facet is decided — S-1 adopted, S-2 archived as a
  considered-and-archived alternative — per the Ruling (2026-10-15) below,
  which also settles the helper-liveness prerequisite (resident singleton).
  Transport selection (native versus MCP placement) and every other AIQ-38
  facet stay open under AIQ-36 and AIQ-38.

Promotion of any of these identifiers requires the canonical admission rule
cited by [AI Unresolved Questions](../product/ai-unresolved-questions.md).

Candidate note (2026-10-11, choice open): the S-1/S-2 selection above stays
open pending a helper-liveness prerequisite question. A resident singleton
helper favors S-1 because a long-lived process can hold the
metadata-only terminal view against a versioned helper-side snapshot; an
on-demand spawned helper favors S-2 because no registry state survives
between spawns and every request forwards over scoped IPC. Commander lean
(not a decision): S-1 with a versioned snapshot, reusing the AI-0211
pattern as read-only reference. The lean changes no status, closes no
identifier, and grants no implementation permission; the choice needs
architecture-owner review under AIQ-38.

### AIQ-38 S-1/S-2 options and recommendation (candidate, 2026-10-14)

This candidate extends the S-1/S-2 open text above without changing it. It
mints no identifier, closes nothing, and grants no implementation permission.
AIQ-38 stays open and the choice needs architecture-owner review. Nothing here
claims implemented behavior; both options are candidate readings only.

- **Exact split (candidate reading).**
  - **S-1 (metadata-only terminal registry).** The terminal side would keep a
    generic, read-only service-metadata snapshot only: provider and model
    names, declared capabilities, and routing facts, validated at registration
    and tagged with a registry version. It would mediate authorized requests
    through scoped request envelopes and handle mediation at the IPC boundary,
    enforcing scopes, consent, and budgets there. Registry implementation,
    selection-policy execution, provider I/O, and credentials would stay
    helper-side as the authoritative registry.
  - **S-2 (protocol-only terminal side).** The terminal side would keep no
    registry state and no snapshot. Every AI request would be forwarded over
    scoped IPC to the helper through generic request envelopes, with the
    helper owning registry, selection, and I/O. The terminal side would still
    enforce scopes, consent, and budgets at the IPC boundary and would still
    expose no AI-specific API beyond generic primitives.
  - In both readings, request envelopes and dispatch handles live at the
    boundary; registry authority and snapshots do not: only S-1 keeps a
    versioned snapshot, and neither option places provider implementation,
    selection execution, or credentials on the terminal side.
- **Deciding evidence (what the owner would compare).**
  - **Post-dispatch `Unknown` reconciliation across IPC.** Under S-1, an
    `Unknown` outcome could cite the snapshot version plus the request envelope
    identifier, giving status inspection a stable correlator on both sides of
    the boundary. Under S-2, reconciliation would correlate by envelope
    identifier alone with no local snapshot version, so every retry or status
    check would depend on a helper round trip for current registry facts.
  - **Consent and budget ledger placement.** Both options keep enforcement at
    the IPC boundary with helper-side pre-I/O gates and terminal-side
    per-client quotas, consistent with the budget and consent handoffs above.
    S-1 would let boundary checks cite a pinned snapshot version for audit;
    S-2 would cite only the forwarded envelope, keeping one fewer versioned
    artifact in the audit trail at the cost of no local routing context.
  - **No state duplication.** S-2 duplicates nothing by construction. S-1
    duplicates metadata by design and is acceptable only while the snapshot
    stays read-only, versioned, credential-free, and never used for selection
    execution or model I/O.
  - **Drift and invalidation needs.** S-2 needs no invalidation path. S-1
    would need one: a defined invalidation or refresh on helper registry
    change, plus fail-closed behavior on stale or missing snapshot versions so
    a drifted view cannot authorize or route as if current.
- **Candidate recommendation (lean, not a verdict).** The candidate lean is
  toward S-1 with a versioned, read-only, fail-closed snapshot, because it
  keeps boundary enforcement auditable against a pinned version, makes drift
  explicit through a required invalidation path, and gives `Unknown`
  reconciliation a stable correlator without duplicating registry authority,
  selection, I/O, or credentials. S-2 remains viable where zero terminal-side
  registry state is preferred over those audit and correlation benefits. The
  lean changes no status, closes no identifier, and grants no implementation
  permission; the architecture owner accepts, rejects, or redirects it under
  AIQ-38.
- **Red lines (by reference, unchanged).** The red lines in the MP-1 versus
  BA-2/BA-3 disposition above stay unchanged: `bitty-agent` performs no model
  selection, no model I/O, and no API-key handling; the terminal side never
  loads AI code in-process; every effect still passes caller and target
  authorization, consent, budgets, redaction, and attributed outcomes.

### Ruling (2026-10-15)

Acting architecture owner under 2026-10-15 user delegation (prior owners
departed). This ruling records a decision; it grants no implementation
permission beyond the decided record and claims no implemented behavior. It
closes ONLY the S-1/S-2 registry-split facet of AIQ-38; every other AIQ-38
facet stays open. The S-1/S-2 option definitions above are unchanged as
historical record.

- **Adopted: S-1 (metadata-only terminal registry) with a resident singleton
  helper.** The terminal side keeps a read-only, versioned service-metadata
  snapshot (provider/model names, declared capabilities, routing facts),
  credential-free, never used for selection execution or model I/O. Registry
  implementation, selection execution, provider I/O, and credentials stay
  helper-side.
- **Invalidation/refresh path REQUIRED with fail-closed behavior.** A defined
  invalidation/refresh path is required, with fail-closed behavior on stale or
  missing snapshot versions (AI-0211 deny-stale pattern as read-only
  reference), so a drifted snapshot view cannot authorize or route as if
  current.
- **Helper liveness settled: resident singleton.** On-demand spawn would have
  forced S-2; the resident singleton prerequisite is settled in favor of S-1.
- **S-2 archived as a considered-and-archived alternative.** Revisit clause:
  if a resident helper proves infeasible in implementation, S-2 may be revived
  only by a new ruling.
- **Rationale.** Post-dispatch `Unknown` reconciliation gets a stable
  snapshot-version + envelope-id correlator on both sides; consent/budget
  boundary checks can cite a pinned snapshot version for audit; the
  duplication cost is bounded (read-only metadata + one proven invalidation
  pattern).
- **Cross-links.** bitty-ai-docs#277 (ruling request); bitty-ipc#21 (Q2
  answered: S-1 adopted; the snapshot-version + envelope-id correlator lands
  later as an optional params field).

### Provider/routing residuals

- Live Core publication/version/invalidation events, helper registry
  population, transport acknowledgement, and strict-handler wiring remain
  implementation work and are not claimed here.
- Wire placement, frame shape, and whether a shared DTO/canonicalizer belongs
  in `bitty-ipc` remain a follow-up transport decision. This acceptance grants
  no Core transport, wire, or Lua integration permission.
- Alias-dependent strict behavior remains forbidden until a later decision
  projects, bounds, canonicalizes, hashes, and equality-checks the alias table
  and its target references.
- `ai.provider_list` strict enablement requires a separately scoped
  implementation-task authorization subject to this contract and its evidence
  plan. No other domain is enabled by this decision.
- AIQ-29, AIQ-2A, and the non-provider-routing facets of AIQ-38 remain open in
  their existing register entries. This contract introduces no identifier,
  owner assignment, milestone, or register closure.

## Acceptance criteria

- The provider/routing snapshot contract is accepted as a design contract;
  acceptance does not claim shipped behavior or authorize implementation.
- The canonical projection contains exactly `provider-routing`, provider IDs,
  model names, closed capability labels, context window tokens, and relative
  input/output cost weights, with aliases excluded initially.
- Canonical bytes are exact UTF-8 JSON under the fixed field order, sorting,
  escaping, Unicode, integer, bound, and lowercase SHA-256 rules above.
- Core owns exact publication, monotonic versioning, identity, and invalidation;
  the helper validates and acknowledges the exact publication, then issues the
  opaque proof only after registry equality succeeds. Caller echoes are never
  authority.
- Mutation invalidates proof; stale, missing, future, replayed, duplicate,
  superseded, wrong-domain, wrong-digest, malformed, oversized, and mismatch
  cases fail closed before handler contact, provider implementation/provider
  I/O, credential access, tool execution, or any other side effect.
- Publication acceptance and proof issuance require authenticated Core-origin
  ingress bound to the authorized Core component and channel; missing or
  invalid authentication fails closed before acknowledgement or proof issuance,
  and caller-supplied metadata cannot establish origin.
- Standalone default behavior remains tolerant. MCP/context/session/complete/
  tool/extension domains remain tolerant or not wired, and MCP `list_version`
  is not reused.
- Live `ai.provider_list` strict enablement, Core transport/wire integration,
  Lua integration, and any shared DTO/canonicalizer are separately scoped
  follow-ups, not permissions granted by this acceptance.
- The document remains self-contained, English-only, synchronized with the
  architecture route, and free of external provenance anchors.
- Independent architecture-owner, docs-curator, and security-reviewer review
  records no blocking finding before implementation follow-ups rely on this
  contract.

## P0 Review Sign-off

The provider/routing contract is accepted as a design record; the following
three independent review slots remain explicit for the acceptance handoff and
implementation follow-ups:

- **Architecture category owner:** Pending independent review of projection,
  ownership, lifecycle, and strict-target boundaries.
- **Documentation curator:** Pending independent review of status, spine,
  self-containment, terminology, and route synchronization.
- **Security reviewer:** Pending independent review of fail-closed behavior,
  bounds, domain separation, no-handler-contact refusal, and credential/access
  controls.

Repository gates do not substitute for these reviews, and this sign-off does
not claim implementation authorization or live strict enablement.

## References

- [Agent coordination architecture](../agent/agent-coordination.md) (Draft):
  single-agent Panel reconciliation and supervision vocabulary.
- [Command and tool architecture](command-tool-architecture.md) (Draft):
  exec and Panel integration and native versus MCP transport status.
- [AI Architecture](ai-architecture.md) (Draft): MP-1 (Registry ownership),
  MP-7 (`cancel`), BA-2 (Agent versus AI split), BA-3 (Bridge process model),
  and the existing registry and transport reconciliation.
- [AI Unresolved Questions](../product/ai-unresolved-questions.md) (Draft): AIQ-29,
  AIQ-2A, AIQ-36, AIQ-37, and AIQ-38.
- [v0.1 Implementation Profile](../product/implementation-profile-v0.1.md) (Draft):
  experimental scope, non-goals, and blocking questions.
- [IPC and Agent RFC](../specifications/ipc-agent-rfc.md) (Accepted): bounding IPC framing,
  scopes, and lifecycle contracts.
