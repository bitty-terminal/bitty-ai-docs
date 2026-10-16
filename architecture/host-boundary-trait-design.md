---
title: HostBoundary trait and lint-gate design
description: Draft candidate design for Core versus Lua enforcement through a HostBoundary trait sketch and a fail-closed lint gate
category: specifications
audience: contributor
document_type: specification
status: accepted
website_publish: false
sidebar_order: 57
---

# HostBoundary trait and lint-gate design

> Status: **accepted** (DEC-0004, 2026-10-15). This document records the
> accepted direction for AIQ-31 facets (a) and (b): enforcement of the Core
> versus Lua split through a Rust `HostBoundary` trait plus a lint gate. It
> authorizes no shipped behavior, closes no Artificial Intelligence Question
> or open-question entry, mints no AIQ or OQ identifier, introduces no
> product code, and changes no accepted document. The accepted
> [IPC and Agent RFC](../specifications/ipc-agent-rfc.md) and the normative security corpus
> override any statement here. Every call-site anchor below is read-only
> survey evidence at `bitty-ai` `main` `97d3125`; no file in the `bitty-ai`
> repository was modified.

## Purpose and scope

This design answers two of the AIQ-31 facets recorded in
[AI Unresolved Questions](../product/ai-unresolved-questions.md): (a) a
trait-level enforcement point for the Core versus Lua split, and (b) a
lint-level gate that rejects direct cross-layer references. Review-time policy
(the remaining review facet) is out of scope here.

Inputs are the draft [Command and Tool
Architecture](command-tool-architecture.md) (Core implements mechanism, Lua
implements policy and workflow), the draft [Provider plugin
boundary](../providers/provider-plugin-boundary.md) (Core owns the contract, plugins own
integration), the draft [Tool transport
R2](tool-transport-r2.md) (unified authorization backend with fail-closed
denial), the Lua-versus-Core candidate design in the
[Lua and Core safety boundary candidate](../specifications/lua-core-safety-boundary-candidate.md)
(mechanism in Rust, Lua decides how, never whether), and the readiness note in [RFC-split
readiness](../integration/rfc-split-readiness.md) that Core/Lua enforcement stays open under
AIQ-31.

## Normative sources

The accepted [IPC and Agent RFC](../specifications/ipc-agent-rfc.md) and the
normative security corpus override any statement here. Design inputs that
this document revises nothing of: [Execution ownership
R1](execution-ownership-r1.md), [Tool transport R2](tool-transport-r2.md),
the [provider plugin boundary](../providers/provider-plugin-boundary.md),
the [Command and Tool Architecture](command-tool-architecture.md), and the
[Lua and Core safety boundary
candidate](../specifications/lua-core-safety-boundary-candidate.md).

## Terminology

This document defines no new glossary terms: `HostBoundary`, binding crate,
lint gate, shell skeleton, and the `*View` / `AuthProof` / `LeaseProof`
sketch names are placeholders scoped to this design. Shared vocabulary
(registry, snapshot, envelope, scope, consent, budget) follows [Execution
ownership R1](execution-ownership-r1.md) and the accepted [IPC and Agent
RFC](../specifications/ipc-agent-rfc.md).

## Call-site inventory (prospective)

The survey below names the seams where host or plugin calls cross or will
cross once a Lua/plugin-loading layer exists. Line anchors are at
`bitty-ai` `main` `97d3125`.

### No Lua or plugin-loading code exists yet

`rg -il 'lua|plugin' crates/bitty-ai-runtime/src/` over the inspected
`bitty-ai` checkout (at `main` `97d3125`) returns only three files —
`agent.rs`, `bridge.rs`, `selection.rs` — and every hit is an incidental
substring: `evaluated`/`evaluation` in `bridge.rs` (`:17`, `:474`, `:536`) and
in `agent.rs` (`:873`), plus the prose line "model plugins own integration
only" in `selection.rs` (`:4`). A stricter whole-word search for loader-shaped
symbols (`mlua`, `wasmtime`, `rlua`, `libloading`, `dlopen`, `dylib`,
`cdylib`, `load_plugin`, `register_plugin`, `lua_state`, `lua_State`) across
`crates/bitty-ai-runtime/src/` and `crates/bitty-ai-slice/src/` returns no
match. There is no Lua runtime binding, no dynamic-library loader, and no
plugin registry in either crate. The inventory below is therefore prospective:
it names the Core-side seams a future host layer must traverse, not calls a
Lua layer makes today.

> Phase-0 note (2026-10-10): the survey above is anchored at `bitty-ai`
> `main` `97d3125` and predates the sibling lint-gate evidence recorded in
> the [AIQ-31 disposition](../product/ai-unresolved-questions.md#aiq-31-disposition-local-draft-only).
> That disposition records a landed shell-based gate
> (`scripts/host-boundary-lint.sh`, `just host-boundary-lint`) closing
> macro-mediated, source-graph, allowlist, and path-dependency escapes. Where
> the sketch patterns below and that landed gate differ, the landed gate plus
> review decides, and this document is revised first.

### Seam 1: extension-point declarations (`extension.rs`, AI-0079)

`crates/bitty-ai-runtime/src/extension.rs` (`366` lines) declares the
extension seam as data only: `ExtensionPoint` (`:61`), `Manifest` (`:95`),
and eight declaration-only point traits — `ModelPoint` (`:110`),
`ToolPoint` (`:121`), `ContextPoint` (`:130`), `MemoryPoint` (`:140`),
`CompactorPoint` (`:149`), `AgentPoint` (`:158`), `CommandPoint` (`:167`),
and `UiPoint` (`:176`) — landed in `bitty-ai` `7b76c27` (AI-0079). The module
header states there is no registry, no resolution algorithm, no versioning
logic, and no permission grant; every trait method is required, there are no
supertraits between the point traits, and there are no blanket impls, so a
probe implementing one point satisfies no other point's bound.

Host-crossing candidate: whatever future registry resolves a `Manifest` into
attached capabilities, and whatever future loader instantiates a point
implementation, must cross the boundary through an explicit host call. The
point traits themselves stay declarations; they never become the crossing
mechanism.

### Seam 2: ToolBus, registry, and spec surface (`tool.rs`, AI-0090)

`crates/bitty-ai-runtime/src/tool.rs` (`1107` lines) owns the vocabulary and
dispatch surface a Lua tool layer would eventually reach for:
`ToolRegistry` with `register`/`lookup` (`:275-347`, `register` at `:301`),
`ToolSpec` with constructor and `schema_digest` (`:201-273`), `ToolBus` with
`new`/`with_authorizer`/`precheck`/`dispatch` (`:578-760`), `ToolAuthorizer`
and `DenyAllAuthorizer` (`:390-409`), and `ToolExecutor` with
`FakeToolExecutor` (`:495-551`). Stale-schema denial was proved in
`bitty-ai` `a4cc1e6` (AI-0090).

Host-crossing candidates: a future Lua `register_tool` or tool-call path must
resolve to `ToolRegistry::register` (with schema-digest invalidation) for
registration and to `ToolBus::precheck` plus `ToolBus::dispatch` (never
directly to a `ToolExecutor`) for execution. Direct construction of
`ToolSpec` from untrusted input bypasses validation and is a lint target.

### Seam 3: provider and selection surface (`provider.rs`, `selection.rs`, AI-0085)

`crates/bitty-ai-runtime/src/provider.rs` (`1051` lines) defines the
`ModelProvider` trait (`:768`: `provider_id`, `list_models`, `complete`,
plus test-observability accessors) with `FakeProvider` (`:801`) as the
deterministic test double, and the sampling contract `SamplingParams` (`:373`)
on `TurnRequest` (`:411`) validated pre-I/O (`validate_sampling`, `:677`;
`check_sampling` ahead of `:768`), landed in `bitty-ai` `f0c1c5e` (AI-0085).
`crates/bitty-ai-runtime/src/selection.rs` (`1562` lines) owns the Core-side
registry, capability-subset matching, alias resolution, ordered fallback, and
typed-error policy over `ProviderRegistry`: `register` (`:402`),
`register_alias` (`:464`), `lookup` (`:519`), `resolve_alias` (`:538`),
`select` (`:570`).

Host-crossing candidates: a future Lua model-selection or provider-call path
must resolve through `ProviderRegistry::select` (never by model-name string
alone) and then through exactly one `ModelProvider::complete` call with
pre-I/O sampling validation. Constructing provider calls that skip
`validate_sampling` or bypass `select` is a lint target.

### Seam 4: cache-key, fingerprint, fencing, and fallback primitives

- `cache_key.rs` (`268` lines): `CacheKey` (`:129`, constructor `:154`),
  `CacheScope` (`:56`), `CacheKeyError` (`:77`); provider-scoped key
  mechanism landed in `bitty-ai` `fdb37c5` (AI-0082). A future Lua
  cache-control or prefix-management path must construct keys only through
  the validated constructor with explicit scope; ad-hoc key strings are a
  lint target.
- `fingerprint.rs` (`177` lines): `InputFingerprint` (`:68`, constructors
  `:85`/`:110`, reuse check `:140`); completeness with disable-on-unknown
  landed in `bitty-ai` `8c5418b` (AI-0095). A future Lua caching or reuse
  path must reuse only through `allows_reuse`; comparing raw digests or
  skipping the unknown-disable is a lint target.
- `fencing.rs` (`228` lines): `WriterLease` (`:62`), `WriterRefusal`
  (`:77`), and the pure predicate `check_writer` (`:142`); interactive
  writer fencing landed in `bitty-ai` `d4d0738` (AI-0092). A future Lua
  write path must present a lease through `check_writer` before issuing a
  write; lease-free writes are a lint target.
- `fallback.rs` (`364` lines): `FallbackEnvelope` (`:121`, constructor
  `:138`, `to_bytes` `:183`, `from_bytes` `:211`) and the scrubbing
  constructor `fallback_for` (`:340`); the minimal-envelope syntax fallback
  landed in `bitty-ai` `97d3125` (AI-0097). A future Lua disclosure or
  degraded-output path must build envelopes through `fallback_for`; direct
  `FallbackEnvelope` literal construction with unscrubbed text is a lint
  target.

## Proposed `HostBoundary` trait shape (illustrative sketch)

The block below is an illustrative design sketch, not product code. It must
not be copied into a crate as an implementation; it exists so reviewers can
judge the shape before any migration task is scoped. Names, bounds, and
error types are placeholders.

```rust
// Illustrative sketch only: candidate HostBoundary shape, not product code.
// Every cross-layer call in the inventory above would traverse this trait;
// Core internals keep calling each other directly.
pub trait HostBoundary {
    type Error;

    // Registration side: plugin and Lua declarations enter only here.
    fn register_tool(&mut self, spec: ToolSpecCandidate) -> Result<ToolName, Self::Error>;
    fn register_alias(&mut self, alias: AliasCandidate) -> Result<(), Self::Error>;

    // Execution side: selection, provider turn, tool dispatch, writes.
    fn select_model(&self, request: &SelectRequestView) -> Result<SelectedModelView, Self::Error>;
    fn complete_turn(&mut self, request: &TurnRequestView) -> Result<TurnOutcomeView, Self::Error>;
    fn dispatch_tool(&mut self, call: &ToolCallView, auth: &AuthProof) -> Result<ToolOutcomeView, Self::Error>;
    fn checked_write(&mut self, lease: &LeaseProof, bytes: &[u8]) -> Result<(), Self::Error>;

    // Cache and disclosure side: keys, fingerprints, fallback envelopes.
    fn cache_key(&self, scope: &CacheScopeView, region: &[u8]) -> Result<CacheKeyView, Self::Error>;
    fn check_reuse(&self, stored: &FingerprintView, candidate: &FingerprintView) -> bool;
    fn fallback(&self, unreadable: &[u8]) -> Result<FallbackView, Self::Error>;
}
```

Why compile-time enforcement beats convention: a trait bound is checked on
every build, while a "call the right function" convention is checked only by
reading the diff. Once the lint gate (below) forbids direct references to
the Core items, the only way for host or plugin code to compile is to go
through an implementor of this trait, so a future Lua binding crate can hold
a `&dyn HostBoundary` (or a generic `H: HostBoundary`) and the compiler
rejects any path that bypasses it. The existing seams already point this
way: `ToolBus` denies by default without an authorizer, `select` fails
closed on name-only resolution, and `check_writer` refuses stale leases, so
the trait composes fail-closed primitives behind one choke point instead of
inventing new policy.

What the trait does not cover:

- Data-format agreement. The trait fixes who may call, not what bytes mean:
  schema shape, serialization, and stable-prefix ordering stay with AIQ-12
  and the registry digest rule (AI-0090).
- Host policy. Budgets, consent scope evaluation, capability grants, and
  redaction stay with the R2 unified authorization backend, MP-10, and the
  security corpus; the trait carries an `AuthProof`, it never mints one.
  Forward condition: when the view, `AuthProof`, and `LeaseProof` types are
  defined for real, views must be credential-free and `AuthProof` must be
  host-minted, opaque, and unforgeable by binding or plugin code.
- Transport and presentation. Wire framing, Panel projection, and rendering
  stay with the accepted [IPC and Agent RFC](../specifications/ipc-agent-rfc.md) and AIQ-29;
  the trait returns outcome views, never pixels or wire bytes. The draft
  terminal-to-helper surface (ops, budgets, consent, redaction handoffs) is
  sketched in [Execution ownership R1](execution-ownership-r1.md#scoped-helper-ipc-surface-draft),
  not here.
- Loader mechanics. How Lua code is loaded, sandboxed, versioned, or
  reloaded is a separate loader design; this trait only constrains what the
  loaded code may reach once running.

## Lint-gate rule specification

The gate is a fail-closed textual check over the host and plugin binding
surface: any direct reference to a Core-owned item outside the Core crate
fails the build. It applies to host-side and plugin-side binding crates, not
to Core internals (the `bitty-ai-runtime` crate itself) and not to this
documentation repository.

Banned reference patterns (concrete `rg` patterns, run from the `bitty-ai`
repository root against host/plugin binding paths only):

- `rg -n 'use\s+bitty_ai_runtime::(tool|provider|selection|cache_key|fingerprint|fencing|fallback)::' <binding-paths>`
  catches direct imports of the seven inventoried Core modules.
- `rg -n 'bitty_ai_runtime::(tool::(ToolRegistry|ToolBus|ToolSpec|ToolExecutor)|provider::(ModelProvider|FakeProvider)|selection::ProviderRegistry|cache_key::CacheKey|fingerprint::InputFingerprint|fencing::(check_writer|WriterLease)|fallback::(FallbackEnvelope|fallback_for))' <binding-paths>`
  catches fully qualified uses that dodge the import form.
- `rg -n 'impl\s+(ToolExecutor|ToolAuthorizer|ModelProvider)\s+for' <binding-paths>`
  catches out-of-Core implementations of the Core seam traits; new seam
  implementations land in Core or in a reviewed adapter crate named by the
  gate allowlist, never ad hoc in a plugin.
- `rg -n 'unsafe\s*\{' <binding-paths>` catches unsafe blocks in binding
  code; any exception needs an explicit allowlist entry with a safety note.

Allowlist: exact file paths with a one-line justification each, reviewed with
the design. Test doubles inside Core (`FakeProvider`, `FakeToolExecutor`)
are Core-owned and never referenced from binding paths.

Where the gate lives (`just` recipe plus CI step sketch, both illustrative):

```sh
# Illustrative sketch: a future bitty-ai just recipe for the lint gate.
# Fails closed: any hit exits nonzero and fails the build.
just host-boundary-lint:
    #!/usr/bin/env bash
    set -euo pipefail
    rg -n 'use\s+bitty_ai_runtime::(tool|provider|selection|cache_key|fingerprint|fencing|fallback)::' crates/bitty-ai-host crates/bitty-ai-lua --glob '!allowlisted.rs'
    test "${PIPESTATUS[0]}" -eq 1  # rg exits 1 on no match: no hit means pass
```

```yaml
# Illustrative sketch: a future CI step in the bitty-ai repository.
# Not a change to this repository's CI.
- name: Host-boundary lint gate
  run: just host-boundary-lint
```

Fail-closed behavior: the gate exits nonzero on any banned hit, on any `rg`
invocation error (missing path, bad pattern), and on any unlisted binding
crate directory (new host/plugin crates must be added to the scanned paths
explicitly, so silence never means skipped). A passing gate proves only that
no banned textual reference exists; it does not prove semantic confinement,
which stays with review and the trait bound.

## Migration plan

1. Land the lint gate first. It is cheap (textual patterns, no refactor),
   immediate (runs on the current tree with zero hits expected, since no
   binding crates exist yet), and ratchets: every future host or Lua binding
   crate is born inside the gate. Phase-0 note (2026-10-10): this step is
   recorded as satisfied in the sibling `bitty-ai` repository by the
   [AIQ-31 disposition](../product/ai-unresolved-questions.md#aiq-31-disposition-local-draft-only)
   (shell-based gate with macro, source-graph, allowlist, and path-dependency
   escape closures). The gate applies from day one to the Phase 1 crate below.
2. Accept this design through review (DEC-0004). No trait migration starts
   before acceptance; the sketch above must survive review unchanged in
   intent or be revised here first.
3. Phase 1 (approved direction, draft record): crate-first
   `crates/bitty-ai-host` inside the `bitty-ai` repository, per the
   owner-approved host-helper schedule (2026-10-10;
   [CTX-0136 issue](https://github.com/bitty-terminal/bitty-ai-docs/issues/237)),
   with the host-boundary lint gate covering the new crate from day one.
   This record grants no implementation permission: the implementing task is
   separately scoped.
4. Phase 2 (per-seam migration backlog; each item is a separately scoped task
   after acceptance): provider registry plus model I/O; selection; ToolBus
   and dispatch; context providers; extension-registry wiring; cache and
   disclosure primitives; MCP placement per AIQ-36 and AIQ-38;
   slice adapter routing, including the `LocalEndpoint` route question
   (whether the slice `LocalProvider` adapter path routes helper-side or
   provider-plugin-side under the `local-only` privacy class). Each migration
   task wires its seam through the accepted trait, extends the gate patterns
   if new bypass spellings appear, and closes no AIQ entry by itself.
5. Revisit the review facet separately. Trait plus lint cover mechanism;
   human review policy for promotions and new seams stays with AIQ-32 and
   AIQ-34 and is not claimed here.

## Phase 1 scope (candidate shell, no behavior)

This section records the candidate scope of the Phase 1 step named in
[Migration plan](#migration-plan) step 3: what a shell-only
`crates/bitty-ai-host` crate inside the `bitty-ai` repository would contain
and what it would exclude. It is draft direction only: it proposes no
behavior, authorizes no implementation, closes no AIQ or OQ entry, mints no
identifier, and changes no accepted document. Acceptance records direction
only; even the shell skeleton described below exists solely by a separately
scoped task. Where any sentence here conflicts with a normative source, the
normative text wins.

### What the shell would contain

- Workspace membership (candidate): the shell would live as
  `crates/bitty-ai-host` in the `bitty-ai` repository workspace, alongside
  the existing runtime crate, so the host-boundary lint gate in
  [Migration plan](#migration-plan) step 1 and the
  [AIQ-31 disposition](../product/ai-unresolved-questions.md#aiq-31-disposition-local-draft-only)
  would cover the new crate from day one.
- Dependency policy (candidate rule): the shell would start std-only,
  consistent with the
  [std-only kernel direction](../providers/dependency-strategy.md#kernel-principle-std-only-runtime-with-dependency-inversion);
  any future network, TLS, or IPC dependency would need its own scoped task
  plus supply-chain review before adoption. Stated here as a candidate rule,
  not a gate decision.
- Crate-level deny-all defaults (candidate): the shell would carry no ambient
  authority by construction, composing the fail-closed primitives surveyed in
  [Call-site inventory](#call-site-inventory-prospective) (deny-by-default
  dispatch, fail-closed selection, lease-checked writes) behind one choke
  point, so an empty shell would grant nothing.
- Seam skeleton, module listing only (candidate names, zero behavior): the
  shell would contain empty module declarations only — names without
  signatures, logic, or wiring — for the seams a later phase could traverse:
  provider registry; model selection; ToolBus and dispatch; context
  providers; extension-registry wiring; cache and disclosure primitives; MCP
  placement. Each name points at the corresponding seam in
  [Call-site inventory](#call-site-inventory-prospective) and at the Phase 2
  backlog in [Migration plan](#migration-plan); per-seam behavior stays with
  separately scoped Phase 2 tasks.

### Gate from day one (candidate)

- The shell would enter the same quality posture as its sibling crates from
  its first commit (candidate): formatting, `clippy` with denied warnings,
  tests, and dependency deny/audit checks, wired into the `bitty-ai`
  repository's `just check` and CI the way sibling crates are.
- Durable posture (candidate): the shell would create no durable path,
  consistent with the v0.1 ephemeral profile in
  [Release profile and AIQ-5C disposition](persistence-profile-r6.md#release-profile-and-aiq-5c-disposition)
  and [v0.1 non-goals](../product/implementation-profile-v0.1.md#v01-non-goals);
  any durable store would need its own scoped task and gates.

### Red lines inherited (by reference, not restated)

- `bitty-agent` no-selection, no-model-I/O, no-keys rule: see the
  [R1 registry-ownership disposition](execution-ownership-r1.md#mp-1-registry-ownership-versus-ba-2-agent-versus-ai-split-and-ba-3-bridge-process-model-disposition).
- BA-6 pressure-test gate: see
  [Sub-platform staging](ai-architecture.md#sub-platform-staging-proposed).
- No ambient authority: see the isolation rule in the
  [provider boundary security review](../providers/provider-plugin-boundary.md#security-review)
  and the draft op families in
  [Scoped helper IPC surface](execution-ownership-r1.md#scoped-helper-ipc-surface-draft).
- Secrets never in errors, logs, or traces: see the
  [Secret invariant](../providers/provider-plugin-boundary.md#secret-invariant)
  and the
  [AIQ-5A disposition](../product/ai-unresolved-questions.md#aiq-5a-disposition-local-draft-only).

### Explicitly open (no choice made here)

The items left open by this section now live at top level; see
[Open points](#open-points). This section adds no per-seam item and rewrites
none.

This record grants no implementation permission: acceptance records
direction only, and even the shell skeleton described above exists solely by
a separately scoped task.

## Extension activation policy (candidate extension of the Phase 2 backlog)

This section extends the Phase 2 per-seam backlog in
[Migration plan](#migration-plan) step 4 by reference only: it records the
candidate activation policy the extension-registry wiring item would carry.
It adds no per-seam item beyond that policy stub, rewrites no backlog entry,
and grants no implementation permission. It is a candidate proposal only: it
mints no identifier, closes no AIQ or OQ entry, and claims no implemented
behavior. Where any sentence here conflicts with a normative source, the
normative text wins.

### Candidate activation sequence

A candidate point activation would run strictly after the R2 gate order in
[Tool transport R2](tool-transport-r2.md#decision), in this candidate order:

```text
grant parse -> authorization -> tool registration -> consent evaluation
```

Each step would be a precondition for the next, and the sequence would start
only after the R2 gates have passed. A point declaration (one of the eight
declaration-only point traits surveyed in
[Call-site inventory](#call-site-inventory-prospective) Seam 1) would never
substitute for ToolSpec registration, authorizer grant, or the consent
triple: the declaration would name the capability shape, while registration,
grant, and consent evaluation would each remain a separate check owned by the
R2 backend.

### Deny-all activation stub (candidate shape)

The candidate stub shape would be deny-all: no auto-activate, explicit
enable per point, and fail-closed on an unknown point:

```rust
// Illustrative sketch only: candidate deny-all activation stub, not product code.
enum CandidateActivation {
    Enabled,
    Denied,
}

fn activate_candidate(point: &str, explicitly_enabled: &[&str]) -> CandidateActivation {
    if explicitly_enabled.contains(&point) {
        CandidateActivation::Enabled
    } else {
        CandidateActivation::Denied
    }
}
```

An empty enable list would therefore deny every point, and a point name the
stub does not recognize would deny rather than enable. The stub would carry
policy only; it would perform no registry lookup and no resolution.

### What the future AI-0282 S3 would implement against

The future AI-0282 S3 extension-registry work would implement against this
policy stub as its gating contract: activation order, deny-all default, and
per-point explicit enable. Registry population and point resolution behavior
would stay with that separately scoped task; this section defines the policy
stub only and specifies no registry or resolution behavior here.

## Classification delta for gate-v2 (candidate, not accepted)

This section is candidate wording only: it proposes a classification delta
for the host-boundary lint gate v2, and it authorizes no implementation,
closes no AIQ or OQ entry, mints no identifier, revises no accepted text in
this document, and changes the acceptance recorded in [Acceptance
criteria](#acceptance-criteria) in no way. Where any sentence here conflicts
with a normative source, the normative text wins. The implementing gate-v2
work stays with the separately scoped `bitty-ai` task (AI-0310); this
section records the classification basis that task would build against,
nothing more. Survey anchors below are read-only evidence at `bitty-ai`
`main` `7879757`; no file in the `bitty-ai` repository was modified.

### Inverted topology (candidate classification)

The candidate classification would invert the topology assumed in
[Lint-gate rule specification](#lint-gate-rule-specification): the host
crate would be recorded as a std-only vocabulary crate — no behavior, no
dependencies — and the endpoint crate would be recorded as the privileged
consumer. Survey evidence: `crates/bitty-ai-host/Cargo.toml` would carry an
empty `[dependencies]` section (comment lines only, std only), while
`crates/bitty-ai-endpoint/src/lib.rs` would consume `51`
`bitty_ai_runtime::` references plus `45` `bitty_ai_host::` references,
gated on `bitty-ipc-api` schema validators.

The candidate binding-crate inventory would therefore list the endpoint
alongside the host and any future plugin binding crates, with the endpoint's
allowed-consumption set declared as the lawful seams below. Each seam would
be allow-listed by name; every reference outside the set would fail the gate:

- `ProviderRegistry::select` for model selection (never name-only
  resolution; surveyed as the single non-test `select` call site through an
  injected registry view).
- The fallback seam: host `FallbackDirectiveEnvelopeRequest::resolve`
  (`Advance` versus `Stop`) with the runtime `fallback_directive` held as a
  drift-locked test reference.
- The ToolBus-versus-ToolExecutor seam: the endpoint would consume the
  `ToolExecutor` handle only as an injected reference inside a
  caller-supplied view (surveyed `ToolView`), never `ToolBus`, `precheck`,
  or `dispatch` (`ToolBus` would have zero references in the endpoint).
- `FakeProvider` and `FakeToolExecutor` as test-only doubles (surveyed
  exclusively under `#[cfg(test)]` modules).
- S2 envelope constructors for echo fixtures: `SnapshotEnvelope::new`,
  `PinEnvelope::new`, `ToolListSnapshot::from_imported`, and
  `list_version` reads.
- Snapshot vocabulary: `SnapshotVersion`, `SnapshotRefusal`, and
  `check_snapshot_version`.
- S-C schema validators through `bitty-ipc-api` (surveyed
  `validate_ai_result` call sites plus the `MAX_SCHEMA_*` bound set).
- The `AgentRefusal` table for mapping refusals to IPC errors.
- `ChunkReassembler` for test-side reassembly (surveyed under
  `#[cfg(test)]`; reassembly TTL and expiry semantics would stay owned by
  `bitty-ipc-api`).
- `ConsentLedger::inspect` as the single read path (surveyed as the only
  non-test ledger call; every `.grant(` call site would sit under
  `#[cfg(test)]`).

### Day-one-red case (candidate record)

The candidate record would note that a verbatim port of the AI-0145 gate
(which bans `use bitty_ai_runtime::tool::...` in binding paths) would flag
red on day one against the surveyed import at endpoint `lib.rs:3274`
(`ToolError as RuntimeToolError, ToolExecutor as RuntimeToolExecutor`).
Gate-v2 would therefore allow-list — not ban — the endpoint's declared
consumption set above, and would ban everything outside it, including at
least: sockets, frames, process spawn, clocks, threads, network and
filesystem access; provider or registry construction outside tests;
ledger mutation outside tests; and cost settlement (surveyed as
recorded-but-never-settled, with `estimate_cost` appearing only under
`#[cfg(test)]`).

### E0 tests keep-or-remove (candidate recommendation)

The candidate recommendation would keep the exactly two manifest and pin
tests in the endpoint (`host_manifest_declares_zero_dependencies` at
`lib.rs:4117` and `endpoint_ipc_pin_stays_at_owner_decision` at `lib.rs:4144`,
pinning `bitty-ipc-api` at `f201911`) as the gate-v2 seed rather than
superseding them, for three candidate reasons stated in POLICY-style
wording: the hermetic manifest read (`include_str!`, no process spawn and
no filesystem read at test time) would keep the seed cheap; the
workspace-relative include path would keep silent relocation breaking
loudly; and the pin assertion would keep a stale-pin bump a separately
scoped owner decision rather than silent drift. Superseding them would
require the gate-v2 task to re-prove each property, so keeping them would
be the smaller change.

### Gate-v2 check inventory (candidate list only, no implementation)

The checks below are candidate acceptance statements only; they specify no
implementation, and they would be implemented by the separately scoped
`bitty-ai` gate-v2 task:

- Endpoint consumption allow-list: every `bitty_ai_runtime::` and
  `bitty_ai_host::` reference in the endpoint would resolve to a member of
  the declared consumption set; any reference outside the set would fail.
- Host zero-dep tripwire: the host manifest `[dependencies]` section would
  stay comment-and-blank-lines only; any dependency entry would fail.
- Pin tripwire: the endpoint manifest would keep pinning `bitty-ipc-api`
  at the owner-decided revision; any drift without an owner decision
  would fail.
- No-new-edge without classification update: a new binding-crate
  directory or a new cross-crate edge not listed in the classification
  would fail until the classification names it.
- Test-only confinement: seam-trait implementations, test doubles,
  ledger grants, and registry or provider construction outside the
  declared set would appear only under `#[cfg(test)]`; any non-test
  occurrence would fail.

Candidate non-goal: crate-level deny-attributes alone would be
insufficient — textual bans plus review would still be required, because a
deny-attribute would not catch a new smuggling spelling that the textual
patterns have not yet named.

## Security review

Security-lens verdict (2026-10-15, note PX-0568): APPROVE-WITH-NITS — no
normative control weakened; deny-all defaults hold; enforcement placement is
consistent with the R2 gate order; no new attack surface. The two conditions
are recorded in this document: Phase 2 registry/snapshot wiring carries the
R1 snapshot constraints (see [Open points](#open-points)), and the
view/`AuthProof` forward condition above (see `What the trait does not
cover`). Threat substance stays owned by the security lens.

## Verification plan

- Three-lens review actually performed on 2026-10-15: architecture lens
  (note PX-0566), docs-curator lens (note PX-0567), security lens (note
  PX-0568); findings addressed before acceptance (see [Acceptance
  criteria](#acceptance-criteria)).
- Repository gates: `just check` green (formatting, lint, links, metadata,
  language, hygiene). This repository holds no product code, so no test suite
  applies to this document.
- Fail-closed behavior stays with the gate definition above; semantic
  confinement stays with review and the trait bound.

## Alternatives considered

- Convention-only enforcement (rejected): a "call the right function"
  convention is checked only by reading the diff, while a trait bound is
  checked on every build, so a future Lua binding crate can hold a
  `&dyn HostBoundary` and the compiler rejects bypass paths.
- Trait-first migration order (rejected): the lint gate lands first because
  it is cheap, immediate, and ratchets every future host or Lua binding crate
  from day one.
- S-2 protocol-only terminal side (archived): the registry-split alternative
  to the adopted S-1, archived per the [R1
  ruling](execution-ownership-r1.md#ruling-2026-10-15); revived only by a new
  ruling.

## Affected contracts

Inputs only, unrevised: [Execution ownership R1](execution-ownership-r1.md),
[Tool transport R2](tool-transport-r2.md), the [provider plugin
boundary](../providers/provider-plugin-boundary.md), the [Command and Tool
Architecture](command-tool-architecture.md), the [Lua and Core safety
boundary
candidate](../specifications/lua-core-safety-boundary-candidate.md), and the
accepted [IPC and Agent RFC](../specifications/ipc-agent-rfc.md) — none
revised by this document. The `bitty-ai` implementation at `97d3125` was
surveyed read-only. No AIQ or OQ entry is closed by this acceptance.

## Open points

- S-1 adopted by reference: the S-1 versus S-2 registry-split facet is
  decided — S-1 adopted, S-2 archived — per the [R1
  ruling](execution-ownership-r1.md#ruling-2026-10-15); no trait change was
  needed. Phase 2 backlog items touching registry or snapshot wiring
  (extension-registry wiring, MCP placement per AIQ-36/AIQ-38) must carry the
  R1 snapshot constraints (read-only, versioned, credential-free, never used
  for selection execution or model I/O) plus the required invalidation path
  and fail-closed-on-stale behavior as their acceptance criteria.
- `LocalEndpoint` route question: stays open; see the Phase 2 backlog in
  [Migration plan](#migration-plan) and the [Transport taxonomy
  proposal](../providers/provider-plugin-boundary.md#transport-taxonomy-proposal).
- Per-seam behavior: stays with the Phase 2 backlog in
  [Migration plan](#migration-plan); each item is a separately scoped task.
- Review facet: human review policy for promotions and new seams stays with
  AIQ-32 and AIQ-34 and is not claimed here.

## Acceptance criteria

Status: ACCEPTED as DEC-0004 (2026-10-15). Three independent review lenses
each approve — none the author — acting under 2026-10-15 user delegation
with the prior category roles vacant:

- Architecture lens: APPROVE after FIX-1 (shell-skeleton authorization
  harmonized to direction-only) and FIX-2 (S-1 recorded adopted by reference,
  no trait change) — verdict note PX-0566.
- Docs-curator lens: APPROVE-WITH-NITS, all eight nits addressed by the
  canonical headings added above — verdict note PX-0567.
- Security lens: APPROVE-WITH-NITS, both conditions recorded (Phase 2
  registry/snapshot wiring carries the R1 snapshot constraints, see
  [Open points](#open-points); view/`AuthProof` forward condition, see `What
the trait does not cover` above) — verdict note PX-0568.

Decided context: the S-1 registry-split ruling adopted in [Execution
ownership R1](execution-ownership-r1.md#ruling-2026-10-15) (2026-10-15). This
acceptance closes no AIQ or OQ entry: AIQ-31 stays open, as do AIQ-32 through
AIQ-38 where referenced (the AIQ-38 split-facet closure belongs to CTX-0156,
not to this acceptance), and it mints no new identifier. The accepted
[IPC and Agent RFC](../specifications/ipc-agent-rfc.md) is unaffected: no normative framing,
scope, lifecycle, or consent statement here overrides it. Sibling draft
dispositions ([Execution ownership R1](execution-ownership-r1.md), [Tool
transport R2](tool-transport-r2.md), [Provider plugin
boundary](../providers/provider-plugin-boundary.md)) are referenced as inputs only and
are unrevised by this document. The `bitty-ai` implementation at `97d3125`
is surveyed read-only; nothing here describes that slice as the complete
proposed runtime. Recording Phase 1 and the Phase 2 backlog above grants no
implementation permission.

## P0 Review Sign-off

| Role              | Reviewer                                                        | Verdict                                | Evidence / scope                                                                            | Date       |
| ----------------- | --------------------------------------------------------------- | -------------------------------------- | ------------------------------------------------------------------------------------------- | ---------- |
| architecture lens | acting reviewer (2026-10-15 user delegation; prior role vacant) | APPROVE                                | FIX-1, FIX-2 addressed; layering, composition, S-1 compatibility; note PX-0566              | 2026-10-15 |
| docs-curator lens | acting reviewer (2026-10-15 user delegation; prior role vacant) | APPROVE-WITH-NITS, nits addressed      | eight spine-section nits resolved by canonical headings; note PX-0567                       | 2026-10-15 |
| security lens     | acting reviewer (2026-10-15 user delegation; prior role vacant) | APPROVE-WITH-NITS, conditions recorded | Phase 2 snapshot-constraint condition plus view/`AuthProof` forward condition; note PX-0568 | 2026-10-15 |

Three independent reviewers, none the author. No AIQ or OQ entry closed by
this acceptance.

## References

- [AI Unresolved Questions](../product/ai-unresolved-questions.md) (Draft): AIQ-31 and
  the AIQ-32 through AIQ-38 facets; no entry status changes here.
- [Command and Tool Architecture](command-tool-architecture.md) (Draft):
  Core-mechanism versus Lua-policy placement.
- [Tool transport R2](tool-transport-r2.md) (Draft): unified authorization
  backend and fail-closed denial the trait composes.
- [Provider plugin boundary](../providers/provider-plugin-boundary.md) (Draft): Core owns
  the contract, plugins own integration.
- [Lua and Core safety boundary (candidate)](../specifications/lua-core-safety-boundary-candidate.md) (Draft):
  mechanism-in-Rust plus Lua-decides-how direction.
- [RFC-split readiness](../integration/rfc-split-readiness.md) (Draft): AIQ-31 stays open;
  no split promoted here.
- [IPC and Agent RFC](../specifications/ipc-agent-rfc.md) (Accepted): normative IPC framing
  that overrides any statement here.
