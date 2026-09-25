---
title: Provider transport adapter contract
description: Draft consumer-side contract for a provider transport adapter covering its input envelope guarantees and network delegation
category: specifications
audience: contributor
document_type: specification
status: draft
website_publish: false
sidebar_order: 46
---

# Provider transport adapter contract

> Status: **draft**. This document records the `bitty-ai` side of the
> provider transport adapter contract: what Core passes to an adapter, what an
> adapter guarantees back, which network behaviors are delegated to the
> network layer rather than reimplemented, and which concerns stay with the
> adapter. It proposes no accepted architecture, adopts no dependency, no
> crate, no trait, and no numeric limit, authorizes no shipped behavior, and
> closes no Artificial Intelligence Question entry. AIQ-33 and AIQ-36 stay
> open. No product code is introduced or described as implemented. The
> normative security and IPC obligations linked from
> [AI Architecture](../architecture/ai-architecture.md) override any
> experimental adoption stated here.

## Purpose and scope

A provider transport adapter is the only place in the AI-core sub-platform
where network I/O may occur. This document fixes the contract that adapter
owes in both directions, so that integration is mechanical once the network
layer it depends on exists:

- The input envelope Core hands to an adapter, including the opaque
  credential handle that replaces any raw secret (MP-10, MPC-2).
- The guarantees an adapter owes Core: deterministic timeouts (MP-8),
  provider-independent errors, the CP-5 budget gate enforced before any
  provider I/O, and no vendor-specific branching inside Core.
- The delegation boundary with the network layer, so that redirect
  re-authorization, proxy precedence, TLS policy, and transfer budgets are
  consumed rather than reimplemented.
- The concerns that remain adapter-owned: connection pooling, chunked bodies,
  and SSE framing.

The governing constraint is the kernel principle recorded in
[Dependency Strategy](dependency-strategy.md#kernel-principle-std-only-runtime-with-dependency-inversion):
`bitty-ai-runtime` stays a std-only agent kernel and state machine over traits
and domain types, with no HTTP client, TLS stack, async runtime, or network
dependency. The v0.1 expression of that principle — no network access in v0.1
code paths, with a `FakeProvider` covering all tests — stays as stated in the
[v0.1 Implementation Profile](../product/implementation-profile-v0.1.md).
The architecture-level mandates that keep transport, pooling, timeout,
redirect, proxy, chunked bodies, and SSE framing out of the kernel stay in
force under MPC-5 in
[AI Architecture](../architecture/ai-architecture.md). This document does not
weaken either.

Inputs are MP-1 through MP-11 and MPC-1, MPC-2, and MPC-5 in
[AI Architecture](../architecture/ai-architecture.md); CP-5 in the same
document; the Core-owned surface, transport taxonomy, and secret invariant in
[Provider plugin boundary](provider-plugin-boundary.md); the kernel principle,
adapter boundary map, and provider-and-transport separation in
[Dependency Strategy](dependency-strategy.md); the unified authorization
backend in [Tool transport R2](../architecture/tool-transport-r2.md); and the
register in [AI Unresolved Questions](../product/ai-unresolved-questions.md).

Out of scope here, and unchanged by this document: the network layer's own
contract and its numeric policy values, the host secret store, the Model
Manager panel, plugin-registry mechanics, and every decision owned by the
terminal and plugin-ecosystem tracks. Where this document needs a decision
that belongs to another owner, it names the owner and stops.

## Normative sources this specification must not weaken

- [Security Overview](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/overview.md):
  default posture that external providers are untrusted until a narrow grant,
  invariants 1 through 10, and the rule that deferral must not create a
  bypass.
- [Threat Model](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/threat-model.md):
  T-10 and R-013 for untrusted-observation labeling of provider output.
- [Security Risk Register](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/risk-register.md):
  R-012 for child credential leak and R-014 for secret exposure via traces.
- [P0 Security Acceptance Criteria](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/p0-acceptance-criteria.md):
  P0-AC-021 through P0-AC-026 whole, including mandatory typed redaction
  before queue and before write.
- [IPC and Agent RFC](../specifications/ipc-agent-rfc.md) (Accepted): bounded
  framing, scope families, consent ledger, and streaming chunking that
  provider I/O shares under MP-9.
- [Core and Plugin Boundaries](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/architecture/core-boundaries.md):
  the rule that AI and Agent layers remain outside the core.

Where this document refines a threshold or an encoding for the adapter edge, it
refines those sources. If a mechanism stated here weakens a normative control,
the normative text wins and this document must be corrected.

## Terminology

| Term                   | Meaning here                                                                                                                                                                                                                              |
| ---------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Adapter input envelope | The single bounded value Core hands to an adapter for one turn: identity inputs, a budget-resolved request, and a credential handle. Nothing else crosses the edge.                                                                       |
| Credential handle      | The opaque reference an adapter receives instead of a secret value, defined by the secret invariant in [Provider plugin boundary](provider-plugin-boundary.md#secret-invariant).                                                          |
| Transport kind         | The Core-owned descriptor-declared class of access (`HttpApi`, `LocalEndpoint`, `CliHarness`, `ManagedAccount`, `Router`) whose taxonomy is owned by [Provider plugin boundary](provider-plugin-boundary.md#transport-taxonomy-proposal). |
| Network layer          | The `bitty-network` extension: a light contract layer for request/response types, capability definitions, and service traits, plus a default-off implementation behind it.                                                                |
| Delegated behavior     | A network behavior the adapter consumes from the network layer and must never implement, reimplement, or bypass locally.                                                                                                                  |
| Adapter concern        | A behavior that stays with the adapter because it is per-provider protocol semantics, not transport policy.                                                                                                                               |

The authoritative definitions of `ModelProvider`, `ModelDescriptor`, privacy
class, the capabilities vocabulary, the context budget, and the error kinds
stay with [AI Architecture](../architecture/ai-architecture.md) and
[Provider plugin boundary](provider-plugin-boundary.md). This document links
them and adds no second definition.

## Input envelope: what Core passes to an adapter

An adapter is entered only through the Core-owned `ModelProvider` surface
(MP-1, MP-4 through MP-7). The envelope below is the complete set of inputs;
an adapter that needs anything else is requesting an authority the boundary
does not grant.

### Identity and routing inputs

- `provider_id`: the bounded `owner.name` descriptor identity, validated at
  registration and bounded to at most 64 bytes matching
  `^[a-z][a-z0-9_-]*$` (MP-2). A descriptor entry that fails validation never
  reaches an adapter (FS-AI7).
- `model_id`: a registry-known model name, never a free-form string
  (MP-5). An unknown model fails closed before adapter entry.
- Transport kind: the descriptor-declared class of access, not a vendor
  product name. Core branches on transport kind and on declared capabilities;
  it does not branch on vendor identity (see
  [No vendor branching inside Core](#no-vendor-branching-inside-core)).
- Descriptor facts the adapter needs and cannot invent: the declared
  capabilities subset, the context window, and the privacy class. A
  `local-only` entry never performs network I/O (MP-3).

### Bounded request

- Messages, context references, and tool names as resolved by Core under the
  CP-5 request contract. The adapter does not collect context, re-truncate, or
  widen the set; truncation counts and provenance stay with the context layer.
- The MP-5 sampling contract in its validated form. A field the backend does
  not support is refused with a typed outcome before I/O, never silently
  defaulted (the deferred sampling-matrix disposition in
  [Provider plugin boundary](provider-plugin-boundary.md#v01-interface-freeze-ai-0135)).
- The caller's `now_ms` and the resolved deadline for the whole turn
  (MP-8), plus the effective transfer bound for the response.
- A budget reservation proving the CP-5 gate already admitted this request.
  The reservation is the adapter's authorization to begin I/O; its absence is
  a fail-closed condition, not a default.

### Opaque credential handle

- Configuration declares a credential reference, never a value
  (MPC-2). The envelope carries the resulting opaque handle plus the evidence
  that the dedicated `ai.provider` grant was evaluated; a raw secret value
  never crosses into the envelope (MP-10).
- The handle's lifetime inside the adapter is bounded to request signing. The
  adapter must not copy the resolved value into any structure it retains,
  including connection pool keys, error values, diagnostics, traces, journals,
  snapshots, or cache entries, and must not expose it to a child process
  environment, a `BITTY_*` variable, a discovery file, or agent-visible
  context (PP-2, PP-5, FS-AI5, R-012).
- Where the value is injected into the outgoing request is not decided here.
  The register's AIQ-5A disposition already pins one raw-value path,
  `expose_for_adapter`, at the host adapter edge, and the secret invariant
  fixes host-side resolution; whether the substitution itself is performed by
  the adapter, by a host-side signer, or by the network layer is a
  secret-store and network-layer question owned elsewhere and tracked by the
  handoff item in
  [Provider plugin boundary](provider-plugin-boundary.md#bitty-side-handoff-not-a-decision).
  This document fixes the obligation — handles in, values out of reach — and
  not the mechanism.

### What the envelope never carries

- A raw secret value, a decrypted token, or a credential file path that
  authorizes a read.
- Ambient filesystem, process, or network authority. A provider adapter is not
  a second authorization path; it does not widen caller, target, capability,
  consent, or budget scope (R2, AG-4).
- Vendor negotiation state. Endpoint construction, request signing, and
  account resolution are adapter-internal and never round-trip through Core.
- A model instruction derived from provider output. Provider output is
  untrusted observation data (T-10, R-013), never policy.

## Guarantees an adapter owes Core

### Deterministic timeouts

- The deadline Core supplies governs the entire adapter call, covering
  connection, negotiation, redirects, and body reading, not only the first
  response byte.
- The observed bound is the MP-8 profile: `DEFAULT_REQUEST_TIMEOUT_MS = 5 s`
  by default, `DEFAULT_MCP_TIMEOUT_MS = 10 s` for tool-mediated streaming,
  and `MAX_REQUEST_TIMEOUT_MS = 30 s` as the hard ceiling an adapter may never
  exceed. The kernel remains wall-clock-free, so every deadline decision is
  made from the caller `now_ms` (CP-7).
- Request-level retry inside the adapter stays inside the same deadline; a
  retry may not extend it, and a retry count is never unbounded.
- Deadline expiry is reported as a Core-owned typed outcome with the elapsed
  budget attributed, never as a vendor code or a transport-specific string
  (FS-AI4).
- Any additional per-stream or per-chunk idle bound is required by this
  contract but is not pinned here; a numeric value requires the same review as
  the MP-8 profile and no new number is adopted by this document.

### Provider-independent errors

- Vendor status codes, transport failures, and vendor message text are mapped
  at the adapter edge into the Core-owned, transport-neutral error kinds
  (`BudgetExceeded`, authorization denial, cancellation, `Unknown`). No raw
  vendor code, header, or body text crosses the adapter into agents, panels,
  journals, or traces.
- Retryable versus terminal classification is a Core-owned vocabulary
  decision, because Core owns ordered fallback semantics. An adapter reports
  the classification it is given and never invents a local taxonomy or silently
  substitutes a different model.
- A failure that is indeterminate after the request left the machine is
  reported as `Unknown` and reconciled before any retry (MP-7); an adapter
  never reports success it did not observe and never claims rollback of an
  effect that already happened.
- Containment holds: a fault in one adapter call affects only its owning
  session or stream (MP-11, FS-AI3). Sibling sessions, terminals, and plugin
  virtual machines stay responsive.
- Streaming outcomes keep the MP-6 framing: `seq`/`total`/`final` chunks under
  the `256 KiB` decoded-byte ceiling, backpressure that sheds oldest buffered
  chunks with a countable metric, and no silent loss.

### CP-5 budget gate before provider I/O

- The gate is ordered in Core before adapter entry: caller and target
  authorization, then consent, then budget resolution, then adapter entry,
  then provider I/O. A request that would exceed the resolved CP-5 budget
  fails with a typed `BudgetExceeded` at the boundary and no I/O occurs
  (MP-5, CP-5).
- The adapter treats a missing or already-exhausted reservation as fail-closed:
  it starts no I/O and reports the typed denial rather than truncating,
  downgrading, or retrying.
- Consumption is charged against the same per-client quotas as IPC and MCP
  traffic, with no separate model-specific budget (MP-9). The adapter reports
  usage; Core owns the ledger semantics.
- An adapter-declared limit may only tighten an effective bound. It may never
  raise a caller, transport, or network limit, and the effective bound is the
  tighter of the two.

### No vendor branching inside Core

- Adding a vendor shape must not require a Rust change in Core. A new provider
  is a new adapter plus declarative preset data; a base-URL change, a rename, a
  custom header, or a private gateway stays data (MPC-5).
- A Core condition that matches a provider name, an endpoint shape, or a
  vendor wire format is a conformance defect, not a feature. Core may branch
  on Core-owned vocabulary only: transport kind, declared capabilities,
  privacy class, and the error and outcome kinds.
- Core never accumulates per-vendor defaults, status-code tables, or
  retry tables. Those are adapter data, and the shared policy that constrains
  them is the one enumerated in this document.

## Network behavior delegated to the network layer

The adapter is a client of the network layer, not a second network stack. It
consumes the network layer's contract surface — request and response types,
capability definitions, and service traits — and never depends on the network
implementation crate directly. The kernel depends on neither. The network
runtime is default-off, so a network-capable adapter must never be part of a
default build.

Nothing below is decided here; each row records that the behavior is owned by
the network layer and consumed by the adapter, so that integration is a
matter of wiring to a published contract rather than a second design. The
issue tag on each row names the network-layer delivery it depends on, as
recorded in
[Precondition: network-layer delivery](#precondition-network-layer-delivery).

| Behavior                                                                                                                      | Owner   | Adapter obligation                                                                                                              | Not in the adapter                                                             | Precondition |
| ----------------------------------------------------------------------------------------------------------------------------- | ------- | ------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------ | ------------ |
| Redirect following, per-hop re-authorization against the capability, hop limit, cross-origin sensitive-header strip or denial | Network | Declare the destination once and pass the request through; treat any surfaced redirect as a policy outcome, not a hint to retry | No redirect follower, no hop counting, no header-preservation rule             | #37          |
| Proxy resolution precedence and proxy authentication, including refusing to dial an HTTPS proxy as plaintext                  | Network | Express its proxy requirement; accept the resolved policy                                                                       | No proxy stack, no environment reading of proxy secrets, no plaintext fallback | #38          |
| TLS policy: verification, protocol and cipher posture, and the authorized override path                                       | Network | Consume the enforced posture                                                                                                    | No TLS bypass, no verification override, no custom trust store                 | #37, #38     |
| Transfer budgets for declared and chunked bodies, aggregate limits, and WebSocket frame and message limits                    | Network | Request its bound and honor the enforced ceiling                                                                                | No unbounded buffering, no self-selected larger limit                          | #37, #38     |
| Connection deadline preservation across receive, send, and close                                                              | Network | Rely on the enforced per-connection deadlines when pacing reads, writes, and close                                              | No deadline extension, no idle-hold of a connection                            | #38          |
| Destination resolution: DNS, resolution deadline, and resolver cancellation                                                   | Network | Consume the resolved destination and its failure                                                                                | No resolver, no address cache, no deadline re-implementation                   | #39          |
| Tunnel and protocol framing details: CONNECT leftover handling and bounded subprotocol offers                                 | Network | Use only the offered, bounded surface                                                                                           | No hand-rolled tunnel, no unbounded protocol offer                             | #39          |
| Diagnostic hygiene: redaction of headers, bodies, userinfo, and control-bearing hosts, with safe correlation data retained    | Network | Add typed `SecretField` redaction to its own records (PP-2)                                                                     | No raw request or response dump in an error string                             | #39          |

Issue numbers in the last column refer to the `bitty-network` repository. TLS
policy and destination resolution are not introduced by those issues; they are
listed here because the same precondition establishes that the adapter has a
single enforced posture to consume rather than a second one of its own.

### Anti-growth rule

The delegation table is a ceiling, not a starting point. If the adapter needs
a behavior the network contract does not offer, the correct response is a
change request to the network owner, not a local implementation. An adapter
that grows its own redirect follower, proxy stack, TLS bypass, resolver, or
unbounded buffer is out of contract by construction, and a review finding of
such code is a defect regardless of the feature it enables.

### Policy precedence

Effective behavior is the intersection of the network layer's enforced policy
and the adapter's declared requirement. The adapter may narrow — one
endpoint, one protocol, one smaller ceiling — and may never widen. A request
the network layer refuses is reported as a typed outcome to Core; the adapter
does not route around the refusal with a second path, and a refused
destination never becomes a reason to try a different provider inside the
adapter (Core owns fallback order).

### Destination and privacy class

A `local-only` provider performs no network I/O at all (MP-3), including no
loopback HTTP call. A loopback destination is a destination the network layer
evaluates under its own policy, not a private shortcut the adapter may assume;
an adapter that treats loopback as an exemption from network policy is out of
contract.

## Concerns that remain with the adapter

### Connection pooling

Pooling and reuse are per-provider concerns because they depend on endpoint,
protocol, and credential scope, so they stay with the adapter. The adapter
owns a bounded pool with idle eviction, keyed so that a connection is never
reused across credential scopes, provider identities, or privacy classes, and
never across a revoked grant. Pooling must not retain a body, a header, or a
resolved credential value, and a cancelled call must not leave a pooled entry
holding request state.

### Chunked bodies

Reading a provider's chunked body incrementally, and stopping at the
enforced bound instead of materializing the whole response, is an adapter
concern; the bound itself is a network-layer budget. The adapter may
accumulate only up to the effective ceiling, must report a typed failure when
the ceiling is reached, and must never treat a truncated body as a complete
turn.

### SSE framing

Parsing server-sent event framing — event and data field lines, comment
lines, vendor stop sentinels, and per-event payload limits — and translating
vendor event types into the Core-owned stream shape is an adapter concern.
The adapter enforces the Core-owned fragment ceilings and framing
(`seq`/`total`/`final`, byte ceiling, countable shed) on what it emits; it
does not set the transport's buffering limits.

### The line between the two layers

Framing is the adapter's; limits are the network layer's; policy is the
network layer's; protocol semantics are the adapter's. Endpoint construction
from a descriptor base URL, request and response mapping for a vendor shape,
request signing, request-level retry inside one deadline, model discovery for
the providers an adapter implements, and account flows behind opaque handles
are adapter concerns, per the Core-versus-plugin boundary in
[Provider plugin boundary](provider-plugin-boundary.md#core-versus-plugin-boundary)
and the transport taxonomy recorded there.

## Precondition: network-layer delivery

No real adapter is written until the following issues in the `bitty-network`
repository have merged:

| Issue                             | Title                                                       | Bears on this document                                                         |
| --------------------------------- | ----------------------------------------------------------- | ------------------------------------------------------------------------------ |
| `bitty-terminal/bitty-network#37` | Re-authorize redirects and enforce HTTP response budgets    | Redirect re-authorization, per-hop policy, and transfer budgets                |
| `bitty-terminal/bitty-network#38` | Bound WebSocket messages and preserve proxy/deadline safety | Proxy precedence, proxy authentication, and message bounds                     |
| `bitty-terminal/bitty-network#39` | Harden CONNECT, DNS, subprotocol, and diagnostic boundaries | Destination resolution, tunnel and subprotocol framing, and diagnostic hygiene |

This is a stated precondition, not a schedule. It carries no date, no
milestone, and no owner assignment here; the milestone and sequencing those
issues carry are that repository's own planning and are not adopted by this
document. The precondition exists so that when the network layer lands, the
adapter's network-facing obligations are wired to a published contract rather
than designed a second time. It does not authorize implementation, does not
resolve AIQ-33 or AIQ-36, and does not settle the numeric limits or the
redirect policy — those belong to the network contract, which is also where
the issues above state that those values must be decided first.

Evidence basis, read 2026-09-25: the three issues' own titles, labels, and
stated problems in the `bitty-network` repository, and the public layout of
that repository, which separates a contract layer (request and response types,
capability definitions, service traits, no implementation dependencies) from a
default-off implementation (async runtime, transport, HTTP and WebSocket, TLS,
DNS, proxy, policy). No code from that repository was executed, imported, or
modified, and no claim here describes its internal behavior beyond that public
description.

## Explicit non-claims

- No implementation exists. No adapter, transport, network client, or vendor
  integration described here is built, and no sentence here implies otherwise.
- No dependency, crate, trait, feature name, or version is adopted. The
  `HttpTransport` and provider-trait sketches in
  [Dependency Strategy](dependency-strategy.md#httptransport-split-and-test-transports)
  remain future direction; this document states obligations, not Rust types.
- No numeric timeout, transfer limit, hop count, pool size, or policy value is
  adopted or changed.
- No AIQ entry is closed, narrowed, or promoted, and no new identifier is
  proposed. Open-question ownership and promotion stay with
  [AI Unresolved Questions](../product/ai-unresolved-questions.md).
- No owner, milestone, or delivery commitment is set. The registry split, the
  secret store, the Model Manager panel, and plugin-registry mechanics stay
  with their owning repositories as handoff input.
- The v0.1 posture is restated, not changed: no network access in v0.1 code
  paths, with a `FakeProvider` covering all tests.

## Security review

- **Credentials.** The handle-not-value rule (MP-10, MPC-2) is the load-bearing
  control. A raw value crossing the envelope, reaching a child environment, a
  `BITTY_*` variable, a discovery file, a trace, a journal, or an agent
  workspace is a release-blocking defect (PP-2, PP-5, FS-AI5, Invariant 9,
  P0-AC-026).
- **Redaction timing.** Typed `SecretField` redaction applies before queue and
  before write, in the adapter as much as in the kernel. The container-level
  redaction facet is `Closed(partial)` in the register under AIQ-5A, whose
  timing and marker/invalidation facets stay open there; this document neither
  reopens nor extends that disposition.
- **No ambient authority.** Consuming the network layer adds no filesystem,
  process, or network authority to the kernel or to any plugin virtual machine.
  A network-capable adapter stays behind the same caller, target, capability,
  consent, and budget gates as any other effect (AG-4, R2).
- **Minimization.** The adapter sends only the budget-resolved request. Adding
  a dependency never justifies sending more context than the task needs
  (PP-1), and no adapter widens a consent scope by being "trusted" — trusted
  means the provider path is not sandboxed, not that it carries authority.
- **Untrusted output.** Provider output is observation data, never an
  instruction (T-10, R-013). A prompt fragment arriving over the adapter is
  labeled by the existing pipeline, not by adapter-local string inspection.
- **Fail closed.** If bounding, redaction, or consent machinery cannot start
  or is detected disabled, the adapter refuses to serve rather than serving
  unbounded or unredacted (FS-AI7, FS-AI1).
- **No TLS or policy bypass.** Any adapter-side attempt to relax network
  policy, verify nothing, follow a redirect unchecked, or buffer without a
  bound weakens a P0 trust boundary and returns `NEEDS-FIX` at review.

## Verification plan

Acceptance of this contract requires reviewed evidence in the owning
implementation repository. No code exists yet, so the plan below is the bar a
future adapter must meet, not evidence of passing tests.

- Envelope evidence: seeded fixtures showing an adapter receives exactly the
  enumerated inputs, that an unvalidated descriptor, unknown model, or missing
  budget reservation is refused before I/O, and that no request begins without
  a reservation.
- Handle evidence: negative tests with a seeded sentinel secret proving it
  never appears in adapter errors, diagnostics, traces, journals, pool keys,
  snapshots, or child environments, and that a handle is unusable after its
  grant is revoked.
- Deadline evidence: connect, negotiation, redirect, and body-read phases each
  bounded by the caller deadline, a retry that cannot extend it, and the hard
  ceiling never exceeded.
- Error-mapping evidence: a table of vendor statuses, transport failures, and
  indeterminacy cases, each mapping to a Core-owned kind with attribution
  recorded, and no vendor text or status code in any agent-visible surface.
- Budget evidence: gate-order traces showing authorization, consent, then
  budget, then I/O, with `BudgetExceeded` before any socket is opened, usage
  charged to the shared per-client quotas, and a truncated body reported as a
  typed failure.
- Delegation evidence: static review showing no redirect follower, proxy
  stack, TLS override, resolver, or unbounded buffer in the adapter, plus
  integration fixtures proving a refused destination or an enforced ceiling is
  surfaced as a typed outcome rather than routed around.
- Kernel evidence: a dependency graph in which the kernel crate reaches
  neither the network contract nor its implementation, directly or
  transitively, and a default build that contains no network backend.
- No-branching evidence: a test asserting that registering and calling a second
  provider shape requires no Core change, plus a grep-level check that Core
  contains no provider-name or vendor-shape condition.
- Containment evidence: a failing adapter call leaves sibling sessions,
  terminals, and plugin virtual machines responsive (MP-11, FS-AI3), and safe
  startup still works with no provider configured (FS-AI6).
- Determinism evidence: seeded `now_ms`, in-memory descriptor snapshots, and a
  mock or recorded network client driving the full turn with no wall-clock,
  filesystem, or network I/O in the kernel (CP-7).

## Alternatives considered

- **Let the adapter own its HTTP stack.** Rejected: it duplicates network
  policy in a second place, creates two TLS postures, and puts transfer
  budgets and redirect rules outside the layer that already enforces them
  fail-closed. The dependency direction in
  [Dependency Strategy](dependency-strategy.md) also argues against
  hand-implementing TCP, HTTP, TLS, chunked bodies, SSE, proxying, pooling,
  timeouts, and redirects.
- **Move transport into the kernel.** Rejected: it violates the kernel
  principle and makes every dependent inherit the HTTP and TLS tree. The
  dependency-inversion rule stands.
- **Put the adapter in a separate process behind an IPC boundary.** Considered
  and not selected here. It adds a process boundary that the network layer
  already provides as a contract boundary, and its placement interacts with
  execution ownership, which is open under AIQ-38. This document does not
  settle it.
- **Branch on vendor identity inside Core.** Rejected explicitly by MPC-5: it
  would make Core a second registry and turn every new vendor into a Rust
  change.
- **Let the adapter own proxy selection from its configuration.** Rejected:
  proxy precedence and proxy authentication, including the refusal to treat an
  HTTPS proxy as plaintext, are network-layer policy; a per-adapter
  configuration would create divergent proxy postures.

## Affected contracts

- [AI Architecture](../architecture/ai-architecture.md) (Draft): MP-1 through
  MP-11, MPC-1, MPC-2, MPC-5, and CP-5 are elaborated here, not changed.
  MP-8 timeouts, MP-9 quota sharing, MP-10 credential handling, MP-11
  containment, CP-5 pre-I/O budget, CP-7 determinism, PP-1 through PP-6,
  FS-AI1 through FS-AI7, and MPC-5's retained transport mandates all keep their
  current force.
- [Provider plugin boundary](provider-plugin-boundary.md) (Draft): the
  Core-owned surface, the transport taxonomy, the Core-versus-plugin table, the
  secret invariant, and the bitty-side handoff list are unchanged; this
  document is the consumer-side elaboration of the transport row.
- [Dependency Strategy](dependency-strategy.md) (Draft): the kernel principle,
  the adapter boundary map, the provider-and-transport separation, and the
  post-v0.1 adapter status are unchanged; no dependency is adopted.
- [Tool transport R2](../architecture/tool-transport-r2.md) (Draft): the
  unified authorization backend is a precondition for adapter entry, not a
  choice this document makes.
- [v0.1 Implementation Profile](../product/implementation-profile-v0.1.md)
  (Draft): unchanged; the zero-network v0.1 posture and the
  single-`bitty-ai-runtime` scope gate stand.
- The `bitty-network` contract: consumed, not defined. This document creates
  no obligation for that repository and changes no value it owns.

## Open points

This document closes no register entry and proposes no new identifier:

- **AIQ-33 (unified authorization and isolation backend) stays open.** The
  adapter must sit behind the unified backend for every effect it triggers, but
  the backend mechanism is an unselected open choice. Nothing here narrows it.
- **AIQ-36 (native versus MCP tool transport and bridge placement) stays
  open.** This document governs the provider transport path only and decides
  nothing about tool-transport placement or bridge placement.
- AIQ-5A (typed redaction markers and invalidation mechanism) keeps its
  existing `Closed(partial)` disposition: the handle-not-value obligation is
  fixed here, the timing and marker/invalidation facets stay where the register
  put them, and nothing is reopened or extended.
- AIQ-38 (generic execution and registry ownership across repositories) stays
  open: the registry split that decides where the adapter's registration lives
  is undecided.
- AIQ-02 (routing within provider consent and budget), AIQ-13
  (provider-scoped cache key and routing scope), and AIQ-24 (budget
  reservation) keep their register entries; this document consumes their
  outcomes and decides none of them.
- The numeric transfer limits, hop policy, proxy precedence, and TLS override
  path are owned by the network contract, not here.
- The mechanism that substitutes a credential value into an outgoing request
  is owned by the secret-store and network owners, not here.
- Open risk: an adapter that quietly becomes a second HTTP stack. Mitigation:
  the anti-growth rule, static review evidence, and the delegation evidence in
  the verification plan.
- Open risk: a network-capable adapter widens consent scope by being exempt
  from the plugin sandbox. Mitigation: "trusted" means unsandboxed, not
  authorized; the gate order is unchanged.

## Acceptance criteria

- Draft author: CTX-0116 (`ai-docs-commander`).
- Acceptance requires independent review by the architecture category owner,
  the docs curator, and a security reviewer, plus the repository's own gates
  (`just check`) green on the branch.
- This document is complete as a contract statement while the feature remains
  unimplemented. It says so in [Explicit non-claims](#explicit-non-claims) and
  in the status block.
- Suggested follow-ups, each as a separately scoped and separately authorized
  task: the network-layer integration plan once the precondition issues
  merge; the credential-substitution mechanism with the secret-store and
  network owners; and the first adapter contract instantiation.

## P0 Review Sign-off

No P0 sign-off is claimed by this document. It is a draft contract for a trust
boundary — provider network access and credential handling — so a security
reviewer is required before it is relied on, together with the architecture
category owner for the delegation boundary and the docs curator for taxonomy,
metadata, and links. Sign-off is recorded here only when it exists; this
section records none.

## References

- [AI Architecture](../architecture/ai-architecture.md) (Draft): MP-1 through
  MP-11, MPC-1 through MPC-6, CP-5, CP-7, PP-1 through PP-6, and FS-AI1
  through FS-AI7.
- [Provider plugin boundary](provider-plugin-boundary.md) (Draft): Core-owned
  surface, transport taxonomy, Core-versus-plugin boundary, secret invariant,
  v0.1 interface freeze, and bitty-side handoff.
- [Dependency Strategy](dependency-strategy.md) (Draft): kernel principle,
  adapter boundary map, provider-and-transport separation, and the
  `HttpTransport` sketch as future direction.
- [Tool transport R2](../architecture/tool-transport-r2.md) (Draft): unified
  authorization backend and path-selection contract as the adapter's
  precondition.
- [AI Unresolved Questions](../product/ai-unresolved-questions.md) (Draft):
  AIQ-02, AIQ-05A, AIQ-13, AIQ-24, AIQ-33, AIQ-36, and AIQ-38, each keeping its
  existing register disposition.
- [v0.1 Implementation Profile](../product/implementation-profile-v0.1.md)
  (Draft): single-crate scope and the no-network v0.1 posture.
- [IPC and Agent RFC](../specifications/ipc-agent-rfc.md) (Accepted): bounded
  framing, scopes, consent ledger, and streaming chunking.
- `bitty-network` Issues 37, 38, and 39: the delivery precondition recorded in
  [Precondition: network-layer delivery](#precondition-network-layer-delivery).
