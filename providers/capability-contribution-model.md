---
title: Capability-contribution model
description: Draft record of the S6 AI capability-contribution model restoring AI families ceilings and the credential adapter over the AI-free Core seed
category: specifications
audience: contributor
document_type: specification
status: draft
website_publish: false
sidebar_order: 47
---

# Capability-contribution model

> Status: **draft**. This document transcribes the S6
> capability-contribution model implemented in the experimental `bitty-ai`
> slice at `bitty-ai@4058215` (`AI-0172`), module
> `crates/bitty-ai-slice/src/ai_contrib.rs`, wired at
> `LiveBittyHost::from_binding` in `crates/bitty-ai-slice/src/live_host.rs`.
> It restores AI authority additively over the AI-free Core seed: three
> capability families with seven heads and exact parameter polarity, two
> non-empty role-ceiling rows, and the provider-credential adapter as its
> canonical home. This document accepts no Request For Comments, closes no
> Artificial Intelligence Question entry, proposes no new identifier, and
> changes no product code. The normative security and IPC obligations linked
> below override any statement here.

## Purpose and scope

After the S4 cutover the Core seed is AI-free: the `agent`, `mcp`, and `ai`
families, their seven heads, and the Commander and Implementer ceiling rows
that named them left the Core tables, and Core-only installs reject all of
them fail-closed. The S1 and S5 slices added the additive extension hooks
that let a loaded extension contribute families, heads, parameter rules, and
role ceilings back without changing any Core default. S6 is the `bitty-ai`
side of that contract: the fixed contribution tables, the credential
adapter move, and the single extension-load call site.

This document records:

- The AI-free Core seed that S6 builds on.
- The additive fail-closed registration hooks S6 calls.
- The exact contributed set: three families, seven heads, and the
  parameter polarity where only `mcp.invoke` and `agent.memory` require a
  parameter.
- The contributed role ceilings: Commander regains `ai`, `mcp`, and
  `agent`; Implementer regains `agent`; the default authorization path stays
  gate-only and the ceiling gate is opt-in.
- The provider-credential adapter and its canonical home in `bitty-ai`.
- The extension-load wiring site and its failure discipline.
- The S4, S5, S6, and S7 commit pointers that bound the sequence.

Explicitly out of scope here: trust domains, Core event kinds, and new
secret tiers are not contributed; `FakeHost` is deliberately not wired; and
the two wrong documentation pointers inside `bitty-ai` code
(`live_host.rs` and `fake_host.rs` link text, `prompt.rs` header paths) are
recorded as follow-up in [Open points](#open-points) rather than fixed,
because code repositories are outside this repository's scope.

Sources inspected read-only for this document:

- Implementation: `bitty-ai` checkout at `origin/main` commit `4058215`
  (`AI-0172`), module `crates/bitty-ai-slice/src/ai_contrib.rs` (family
  contributions, role ceilings, registration entry points, credential
  adapter) and `crates/bitty-ai-slice/src/live_host.rs` (extension-load
  call site).
- Dependency pin: `bitty-ai@4058215`
  `crates/bitty-ai-slice/Cargo.toml` (typed Core dependency at the S5 merge
  revision).
- Core hooks: `bitty` checkout at `origin/main`, module
  `crates/bitty-package/src/catalog.rs` (additive fail-closed family
  registration), module `crates/bitty-plugin-host/src/roles.rs` (AI-free
  ceilings, ceiling-contribution table), and module
  `crates/bitty-plugin-host/src/effective.rs` (gate-only default path,
  opt-in ceiling path).

No file in the `bitty-ai` or `bitty` repositories was modified. Revision
pins above are anchors at those revisions and drift with later commits.

## Normative sources this specification must not weaken

- [Security Overview](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/overview.md):
  default posture that external providers are untrusted until a narrow
  grant, and the rule that deferral must not create a bypass.
- [Threat Model](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/threat-model.md):
  T-10 and R-013 for untrusted-observation labeling of provider output.
- [Security Risk Register](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/risk-register.md):
  R-012 for child credential leak and R-014 for secret exposure via traces.
- [P0 Security Acceptance Criteria](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/p0-acceptance-criteria.md):
  typed redaction before queue and before write, including P0-AC-026.
- [IPC and Agent RFC](../specifications/ipc-agent-rfc.md) (Accepted):
  bounded framing, scope families, consent ledger, and lifecycle contracts
  that every capability and ceiling contribution shares.
- [Core and Plugin Boundaries](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/architecture/core-boundaries.md):
  the rule that AI and Agent layers remain outside the core.

Where this document refines a behavior for the contribution edge, it
refines those sources. If a mechanism stated here weakens a normative
control, the normative text wins and this document must be corrected.

## Terminology

| Term                | Meaning here                                                                                                                                 |
| ------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| Core seed           | The closed Core tables (`CAPABILITY_FAMILIES`, `CLOSED_CAPABILITY_HEADS`, `AgentRole::capability_ceiling`) with no AI content since S4.      |
| Contribution        | An additive registration through a Core extension hook that a loaded extension performs against a fresh Core catalog.                        |
| Family              | A capability family label (for example `ai`).                                                                                                |
| Head                | A bare capability identifier within a family (for example `ai.provider`), without a `:PARAMETER` suffix.                                     |
| Polarity            | The per-head parameter rule: whether the head requires a `:PARAMETER` (`requires_param` true) or forbids one (false).                        |
| Ceiling             | The set of families a role may exercise at most, intersected with grants elsewhere; a ceiling narrows and never grants.                      |
| Effective ceiling   | Core defaults plus contributed families for one role.                                                                                        |
| Gate-only path      | The default authorization path that runs the role gate without a ceiling gate (pre-S5 behavior).                                             |
| Opt-in ceiling path | The explicit authorization path that additionally intersects the effective ceiling.                                                          |
| Credential adapter  | The provider-credential configuration surface moved from Core into the slice, naming where a credential comes from without carrying a value. |
| Canonical home      | The single owning module for a surface after its move; the prior Core location is staged, then removed.                                      |
| Extension-load site | The single product construction path that performs the AI registration before any other host state exists.                                   |

The authoritative definitions of capability families, heads, enforcement
points, and the credential reference vocabulary stay with the owning Core
modules. This document links them and adds no second definition.

## AI-free Core seed (S4)

Since S4 the Core seed carries no AI authority. The closed family table
holds the non-AI families only, and the role ceilings name no AI family:

| Role        | Core ceiling families (AI-free)        |
| ----------- | -------------------------------------- |
| Commander   | `fs`, `process`, `network`, `terminal` |
| Implementer | `fs`, `process`, `terminal`            |
| Tester      | `fs`, `terminal`                       |
| Reviewer    | `terminal`                             |

A fresh `CapabilityCatalog::core()` accepts exactly the closed heads, and a
fresh `RoleCeilingCatalog::core()` admits exactly the rows above, so a
Core-only install denies every AI family, every AI head, and every AI
ceiling extension fail-closed. AI authority returns only through an
explicitly extended catalog; until then every ceiling above is AI-free by
construction.

## Additive fail-closed registration hooks (S1, S5)

S6 calls two Core hooks and changes neither. Both hooks are additive only
and validate the whole call before mutating, so a failed call leaves the
catalog unchanged.

`CapabilityCatalog::register` (S1) declares one family's bare heads with
their parameter rules. Anything invalid rejects the whole call: an existing
head is a duplicate error and never an overwrite (so a conflicting
parameter rule is rejected, not merged); every family and head is
shape-validated like manifest-time validation; a head carrying a
`:PARAMETER` suffix is rejected; and per-call and total bounds keep
extension input bounded.

`RoleCeilingCatalog::register_ceiling` (S5) contributes additional families
for one role. The same additive discipline applies: contributing a family
the role already admits is a duplicate error and never an overwrite, with
duplication tracked per role-and-family pair; every label is
shape-validated; membership in any catalog is deliberately not required, so
`ai`, `mcp`, and `agent` contribute even though Core catalogs do not know
them; an empty registration is rejected; and per-call and total bounds keep
extension input bounded.

## Contributed families and heads

The fixed contribution table holds three families and seven heads with
exact pre-extraction polarity. Only two heads require a parameter.

| Family  | Head                      | Requires parameter | Effect wording                                               |
| ------- | ------------------------- | ------------------ | ------------------------------------------------------------ |
| `ai`    | `ai.provider`             | No                 | Use allowlisted AI provider                                  |
| `ai`    | `ai.stream`               | No                 | Stream AI responses for this agent                           |
| `ai`    | `ai.model`                | No                 | Select AI model for this agent (bounded)                     |
| `mcp`   | `mcp.invoke`              | Yes                | Invoke allowlisted MCP tool (per-tool, bounded frame 256KiB) |
| `agent` | `agent.context.terminal`  | No                 | Observe terminal context for this agent (bounded 32KiB)      |
| `agent` | `agent.context.workspace` | No                 | Observe workspace context for this agent (bounded 32KiB)     |
| `agent` | `agent.memory`            | Yes                | Persist agent conversational memory (opt-in, 0600, <=7 days) |

In shape: `ai` carries three parameter-free heads, `mcp` carries one
parameter-required head, and `agent` carries two parameter-free heads plus
one parameter-required head. Effect wordings are verbatim from the
pre-extraction Core consent strings; presentation ownership for Core
callers stays Core-side, and the table pins the counterpart wording only so
the AI-loaded accept set stays identical to the pre-extraction set.

Registration entry points, in call order: `register_ai_capabilities`
declares every head above with its polarity; `register_ai_ceilings`
contributes every non-empty ceiling row below; `register_ai_families` runs
both sequentially. The two halves are sequential, not atomic: on a ceiling
failure the catalog keeps the registered families, so callers must discard
both catalogs, which host construction does by failing closed before any
host state exists. Against a fresh Core seed the fixed table always
registers; a rejection therefore signals catalog drift or a double
registration.

What is not contributed: no trust domains, no Core event kind, and no new
secret tier. Registration touches only the two Core contribution catalogs,
and the credential adapter reads through the Core reference mechanism only.

## Contributed role ceilings

The fixed ceiling table restores AI families per role over the AI-free Core
defaults:

| Role        | Contributed families | Effect over the Core default                   |
| ----------- | -------------------- | ---------------------------------------------- |
| Commander   | `ai`, `mcp`, `agent` | Regains all three AI families                  |
| Implementer | `agent`              | Regains the agent family only                  |
| Tester      | None (row skipped)   | Unchanged; its pre-S4 row carried no AI family |
| Reviewer    | None (row skipped)   | Unchanged; its pre-S4 row carried no AI family |

Empty rows are skipped by the registration function because the Core hook
rejects empty registrations fail-closed. Registration is four rows in the
table but two effective contributions.

Ceiling enforcement stays narrowing-only under both authorization paths.
The default `authorize_with_role` path runs the role gate (the role must
admit the enforcement point guarding the request kind) and then the
six-layer grant intersection; it performs no ceiling check and keeps its
pre-S5 gate-only behavior. The opt-in `authorize_with_role_and_ceilings`
path runs the role gate, then the ceiling gate against the effective
(Core plus contributed) ceiling, then the same intersection. Either gate
denies before the intersection runs, and on success the result is exactly
the intersection with the effective ceiling applied, so contributed
authority still narrows and never grants by itself.

## Provider credential canonical home

`ProviderCredentialConfig` is the accepted `api_key_env` and `api_key_cmd`
surface, and the slice module is its canonical home. It moved from Core in
S6 (staged at the Core provider-credential module in S5, whose deprecated
shim S7 removed), with verbatim semantics: at most one field may resolve,
and both set denies as conflict at resolution while neither set resolves
to no credential. Construction preserves the both-present shape so the
conflict stays observable and auditable instead of being rejected silently
at parse time, while each reference must still carry the matching source
kind (`Env` for `api_key_env`, `Cmd` for `api_key_cmd`) or construction
denies fail-closed.

Project scoping is narrow-only per field: an overlay may keep each
reference identical or remove it, and may never add, rename, or switch
sources. Display and diagnostic rendering quote reference names only;
values never enter this type, an error, a log, or an audit detail.

Resolution is exclusive-or first (conflict denies before anything is read),
then source-shaped: the `Env` arm resolves through an injected environment
lookup so tests stay hermetic, denying on a missing or empty variable with
a names-only error; the `Cmd` arm resolves through one bounded shell-free
command runner with direct spawn, null stdin, discarded stderr, a capped
stdout read, and empty, NUL, encoding, size, and exit checks whose errors
quote the program name only. The live wrapper supplies the standard
environment lookup; the core stays hermetic.

The adapter composes with the runtime typed secret container without
overlapping it: the configuration names where a credential comes from and
never carries a value, while the container carries values redacted
everywhere. Callers must inject resolved values into child environments
only, under the adopted credential-flow obligations.

## Extension-load wiring

The extension-load call site is `LiveBittyHost::from_binding`, and it
delegates to `LiveBittyHost::new`, which performs the full AI registration
before any grant parse, authorization, or tool registration. Every live
host therefore carries the AI extension set, and a rejected contribution
fails host construction closed: no host exists without its contributions.
The product entry point derives the client identity from the bound protocol
principal and then follows the same registration-first order, so identity
binding and capability contribution cannot be separated.

`FakeHost` is deliberately not wired. It is the deterministic scripted twin
over the `bitty-ipc` seams, and its tests keep string parity with these
heads as opaque labels rather than as registered contributions.

Layering holds by construction: only the slice takes the typed Core
dependency (`bitty-package` and `bitty-plugin-host` at the S5 merge
revision); the runtime stays std-only. The provider-credential adapter
lives in the same slice module, built on the Core credential reference,
with verbatim exclusive-or, narrow-only, and names-only semantics.

## Slice sequence and commit pointers

| Slice | Repository | Commit     | Change                                                                    |
| ----- | ---------- | ---------- | ------------------------------------------------------------------------- |
| S1    | `bitty`    | `9265baaa` | Contribution-catalog scaffold, purely additive with no behavior change    |
| S2    | `bitty`    | `22aacd3a` | Catalog-aware manifest check with the Core default                        |
| S4    | `bitty`    | `df82e350` | Breaking cutover: `agent`, `mcp`, and `ai` leave the Core seed            |
| S5    | `bitty`    | `271d662b` | Ceiling-contribution API plus the staged credential surface, all additive |
| S6    | `bitty-ai` | `4058215`  | `AI-0172`: AI families, ceilings, and credential adapter registration     |
| S7    | `bitty`    | `43749948` | Removal of the S5-staged Core credential shims                            |

Short hashes above are anchors at `origin/main` in each repository. S1, S2,
S4, and S5 are `bitty` `CTX-0916` (S4 under `CTX-0988`, S5 under
`CTX-0989`); S6 is `bitty-ai` `AI-0172`; S7 is `bitty` `CTX-0991`. The
cross-repository registration carries owner authorization recorded in the
source module (handoff `PX-5009` through `PX-5014` on `bitty` `CTX-0990`,
decisions `PX-5014` questions 1, 2, and 3 all affirmative); those
identifiers are provenance quoted from the implementation module and are
not part of this corpus's registers, so nothing here resolves them.

## Security review

- **Credentials.** Credential material remains opaque across configuration,
  Core, plugins, models, agents, diagnostics, and storage. Raw bytes are
  exposed only through the authorized host adapter edge and are never
  retained in pool keys, errors, diagnostics, traces, journals, snapshots,
  caches, child environments, discovery files, or agent-visible context. A
  raw value outside that edge is a release-blocking defect.
- **Redaction timing.** Names-only rendering applies to the configuration
  type and to every resolution and execution error. Values never enter an
  error, a log, or an audit detail; the command runner discards stderr
  rather than buffering helper diagnostics that may echo secret material.
- **No ambient authority.** Registration adds no filesystem, process, or
  network authority to the kernel or to any plugin virtual machine. Every
  contributed head stays behind the same caller, target, capability,
  consent, and budget gates as any other effect.
- **Minimization and isolation.** The contributed set is exactly the
  pre-extraction set: three families, seven heads, two ceiling rows. Nothing
  here widens a ceiling, invents a family, or grants authority; ceilings and
  grants intersect, and either gate denies first.
- **Untrusted output.** Provider output stays observation data, never an
  instruction. Nothing in the contribution path labels provider output or
  derives policy from it.
- **Fail closed.** A rejected family, head, or ceiling row rejects its whole
  call without partial mutation; a rejected extension set rejects host
  construction without a host; an unresolvable, conflicting, missing, or
  oversize credential denies rather than serving a default. If bounding,
  redaction, or consent machinery cannot start or is detected disabled, the
  host refuses to serve rather than serving unbounded or unredacted.

## Verification plan

Review verifies the transcribed tables against the owning implementation
and treats the sequence pointers as anchors, not as evidence that any
further slice exists.

- Family evidence: the source contribution table holds exactly the seven
  heads above with the stated polarity split (five parameter-free, two
  parameter-required: `mcp.invoke` and `agent.memory`), and the
  capabilities registration declares each head with its polarity.
- Polarity evidence: unknown heads fail, known heads enforce their
  parameter presence rule, and a head carrying a parameter suffix is
  rejected at registration.
- Additivity evidence: re-registering an existing head or an already
  admitted ceiling family is a duplicate error rather than an overwrite; a
  failed call leaves its catalog unchanged; bounds cap per-call and total
  extension input.
- Ceiling evidence: the source ceiling table holds the Commander and
  Implementer rows above with empty Tester and Reviewer rows; registration
  skips the empty rows; a fresh Core seed admits no AI family for any role
  while the extended catalog admits exactly the contributed rows.
- Gate evidence: the default authorization path runs the role gate without
  a ceiling gate, the opt-in path intersects the effective ceiling, and
  neither gate grants on success.
- Credential evidence: seeded tests prove the exclusive-or conflict, the
  neither-set empty resolution, the per-field source-kind construction
  check, the narrow-only overlay on each field, names-only rendering, and
  hermetic resolution through the injected lookup with the live wrapper as
  the only environment reader.
- Runner evidence: the credential command path spawns shell-free with null
  stdin and discarded stderr, retains at most the output cap plus one probe
  byte, and denies on spawn failure, non-zero exit, unreadable, empty, NUL,
  non-UTF-8, or oversize output with program-name-only errors.
- Wiring evidence: construction builds fresh Core catalogs, registers the
  full extension set before any grant parse, authorization, or tool
  registration, and fails closed on rejection; the binding entry point
  delegates to the same constructor; the scripted host carries no
  registration call.
- Layering evidence: the dependency graph shows only the slice reaching the
  typed Core crates at the pinned S5 merge revision, the runtime reaches
  neither directly nor transitively, and the default build carries no
  network backend through this path.
- Sequence evidence: each commit pointer resolves at `origin/main` in its
  named repository with the stated subject; the Core shim absence after S7
  is confirmed by the removed staged module.

## Alternatives considered

- **Keep AI authority in the Core seed.** Rejected by the S4 cutover: every
  Core install would carry AI authority by default, and the fail-closed
  Core-only posture would be lost. Contribution restores the same set
  without restoring the default.
- **Overwrite instead of duplicate-error on re-registration.** Rejected: a
  conflicting parameter rule would silently merge, and a second load would
  mask drift. Duplicate-error keeps the fixed table exact and makes double
  registration loud.
- **Require ceiling families to be catalog members.** Rejected: Core
  catalogs intentionally do not know the AI families after S4, so a
  membership requirement would make the S6 contribution unregistrable. The
  ceiling hook validates shape only, and enforcement still intersects with
  the capability catalog elsewhere.
- **Wire the scripted host too.** Rejected: the scripted twin exists for
  deterministic tests over the `bitty-ipc` seams, and registering
  contributions there would couple every scripted fixture to catalog state.
  String parity as opaque labels is sufficient.
- **Carry credential values in the configuration.** Rejected explicitly by
  the adapter design: a value-carrying config would spread secret material
  through parsing, overlays, errors, and snapshots. The config names the
  source; the typed container carries the value.

## Affected contracts

- [AI Architecture](../architecture/ai-architecture.md) (Draft): capability
  and ceiling semantics are transcribed here, not changed. Least-privilege
  dispatch, budget, consent, and containment mandates keep their current
  force.
- [Provider plugin boundary](provider-plugin-boundary.md) (Draft): the
  Core-owned surface, the transport taxonomy, the secret invariant, and the
  interface freeze are unchanged; this document is the contribution-side
  record of the families and the credential surface that boundary governs.
- [Dependency Strategy](dependency-strategy.md) (Draft): the kernel
  principle, the adapter boundary map, and the provider-and-transport
  separation are unchanged; only the slice takes the typed Core dependency
  and no dependency is adopted here.
- [Tool transport R2](../architecture/tool-transport-r2.md) (Draft): the
  unified authorization backend remains the precondition for effect entry,
  not a choice this document makes.
- [Provider transport adapter contract](transport-adapter-contract.md)
  (Draft): the host-only credential exposure edge stated there is the same
  edge this adapter serves; neither document widens it.
- [v0.1 Implementation Profile](../product/implementation-profile-v0.1.md)
  (Draft): unchanged; the zero-network posture and scope gates stand.
- [IPC and Agent RFC](../specifications/ipc-agent-rfc.md) (Accepted):
  consumed, not defined. This document creates no obligation for that
  contract and changes no value it owns.

## Open points

This document closes no register entry and proposes no new identifier:

- Authorization and isolation placement stays where the register put it.
  The contribution sits behind the unified backend for every effect it
  enables, but the backend mechanism is an unselected open choice and
  nothing here narrows it.
- Tool-transport and bridge placement stays open. This document governs the
  capability-contribution path only and decides nothing about native versus
  bridge transport placement.
- Registry split and placement for where contributions are registered and
  isolated stays with its owning decision. This document records the
  `bitty-ai` tables and the single load site, not the cross-repository
  ownership answer.
- Follow-up (code scope, not fixed here): two wrong documentation pointers
  inside `bitty-ai` code point at paths this corpus does not own. The live
  and scripted host module headers link the integration input under the
  `specifications/` tree while the page lives under `integration/`, and the
  prompt module header names `docs/specifications/` layering paths while the
  pages live under `context/`. Both live in the `bitty-ai` implementation
  repository, so a `bitty-ai` task must correct them; this task records them
  rather than editing outside docs scope.
- Open risk: a contribution table drifts from the Core seed (renamed head,
  changed polarity, widened ceiling row). Mitigation: the fixed-table
  registration that fails closed on drift, the byte-identical accept-set
  pin, and the verification plan above.
- Open risk: a caller constructs a host without the product entry point
  and skips identity binding. Mitigation: the raw seam is documented as a
  seam, product callers are directed to the binding entry point, and review
  treats a non-binding product construction as a defect.

## Acceptance criteria

This draft passes document-level review only when all of the following are
true:

- The Core seed section names the exact AI-free ceilings with no AI family
  present, and no sentence describes an AI head as Core-default.
- The family table lists exactly the seven heads above with the stated
  polarity, names `mcp.invoke` and `agent.memory` as the only
  parameter-required heads, and quotes each effect wording verbatim.
- The ceiling section records Commander with `ai`, `mcp`, and `agent`,
  Implementer with `agent`, skipped empty rows for Tester and Reviewer, the
  gate-only default path, and the opt-in ceiling path, with neither gate
  described as granting.
- Credential canonical home, exclusive-or resolution, narrow-only overlay,
  names-only rendering, and the value-free configuration versus
  value-carrying container split are all stated without claiming a new
  secret tier.
- The wiring section names the binding entry point, the registration-first
  order, the fail-closed construction rule, and the deliberately unwired
  scripted host.
- The sequence table resolves at the named revisions with the stated
  subjects, and S7 is recorded as the shim removal, not as a new surface.
- Every changed canonical file is self-contained: it contains no research
  reference, record number, or coverage ledger.
- `just check` passes, and independent architecture, docs-curator, and
  security review records no blocking finding.

## P0 Review Sign-off

No P0 sign-off is claimed by this document. Before any reliance, the
security reviewer must verify the exact head set and polarity, the
narrowing-only ceiling behavior on both authorization paths, the
host-only credential edge with names-only errors, and the fail-closed
registration and construction rules. The architecture category owner must
verify the Core-versus-extension boundary and the single load site, and the
docs curator must verify self-containment, taxonomy, metadata, and links.
Repository gate success and this draft do not constitute those sign-offs.

## References

- [Provider plugin boundary](provider-plugin-boundary.md) (Draft):
  Core-owned surface, transport taxonomy, secret invariant, and interface
  freeze that govern the contributed families and credential surface.
- [Dependency Strategy](dependency-strategy.md) (Draft): kernel principle
  and adapter boundary map behind the slice-only typed Core dependency.
- [Provider transport adapter contract](transport-adapter-contract.md)
  (Draft): host-only credential exposure edge and network delegation
  boundary served by this adapter.
- [AI Architecture](../architecture/ai-architecture.md) (Draft): capability,
  budget, consent, and containment mandates transcribed here.
- [Tool transport R2](../architecture/tool-transport-r2.md) (Draft):
  unified authorization backend as the contribution's entry precondition.
- [Bitty-side integration input](../integration/bitty-side-integration-input.md)
  (Draft): handoff direction whose rendezvous step the live host serves.
- [v0.1 Implementation Profile](../product/implementation-profile-v0.1.md)
  (Draft): scope and zero-network posture restated, not changed.
- [AI Unresolved Questions](../product/ai-unresolved-questions.md) (Draft):
  register ownership for the open placements above; no entry status changes
  here.
- [IPC and Agent RFC](../specifications/ipc-agent-rfc.md) (Accepted):
  normative framing, scope, and lifecycle contracts that override any
  statement here.
