---
title: Wheel project configuration (candidate)
description: Candidate single-entry project configuration contract for the Wheel harness directory
category: specifications
audience: mixed
document_type: specification
status: draft
website_publish: false
sidebar_order: 71
---

# Wheel project configuration (candidate)

## Purpose and scope

This is a **draft candidate design record**, not a runtime specification,
accepted decision, or implementation claim. It records the candidate
contract for Wheel project configuration: the single-entry shape of the
project harness directory, the eight configuration function classes, the
direnv-like trust posture, the portable-versus-native directory split, the
no-runtime-state rule, and the implementation reality that bounds all of
it. Labels below separate observed behavior, scaffold illustration,
candidate direction, and unimplemented discussion, verified against the
owning implementation repositories.

The project harness directory is the per-repository directory that carries
Wheel-native behavior. The candidate direction fixes one entry file at its
root while remaining organization stays free. Only the singularity (one
entry, not a mandatory tree) is the stable claim; every filename, field
name, and spelling attached to it is unreviewed vocabulary proposing no
API. This document creates no AIQ or OQ identifier and closes none.

## Status vocabulary

| Status        | Meaning in this document                                                                                            |
| ------------- | ------------------------------------------------------------------------------------------------------------------- |
| Real          | Observed in the owning implementation repository. Experimental-slice evidence only, never a shipped-behavior claim. |
| Scaffold      | Present only in the pre-implementation plugin scaffold with no host wiring; illustration, never behavior evidence.  |
| Candidate     | Proposed by a design direction only; no review has accepted it.                                                     |
| Unimplemented | Named in discussion but with no mechanism anywhere; explicitly not claimed.                                         |

## Single-entry contract

Status: **candidate; the stable claim is singularity only.**

The retained contract fixes exactly one entry point at the project
directory root while remaining directories stay ordinary modules the entry
requires; no prescribed subdirectory is mandatory. The organization
principle follows the configuration-style reference: one stable contract at
the root, user freedom inside. A survey of the portable project-level
directory design constrains this contract from the ecosystem side, and that
survey stays with its owning task: this candidate decides no schema on
either side.

**Critical judgment:** the entry filename and the module organization are
the candidate proposal, not an adopted schema. No per-concern module
filename is attested anywhere, so none is stated here: the interior is
always described as ordinary modules the entry requires, never enumerated.
The stable claim is the singularity (one entry, not a mandatory tree).

## Eight function classes

Status: **candidate coverage; every spelling unreviewed.**

The retained classes, with the labels this document guarantees:

1. Capability filtering over discovered skills and integrations: enable,
   disable, allow, prefer, and permission scoping, flowing from a
   discovered registry through Wheel policy into a resolved set for the
   compiler.
2. Context policy as high-level declarations: budgets, code preferences,
   log handling, pinning, and exclusion, with the explicit rule that users
   describe policy rather than hand-roll per-prompt compiler algorithms.
3. Structured rules and constraints that a policy engine enforces
   deterministically: path-scoped denials, confirmation gates, and
   change-gated verification, instead of prompt reminders.
4. Project-defined custom commands built in Lua.
5. Project-defined custom tools with strict provenance separation: native,
   Lua, integration, and skill-script origins stay distinguishable under
   one agent-visible capability surface.
6. Structured agent definitions: model reference, tool and skill selection,
   and permission bounds that let the compiler trim per profile.
7. Verification declarations: change-gated checks, test discovery with a
   targeted-then-full shape, and completion gates, so done-ness is decided
   by evidence.
8. Hooks across task, tool, compile, and spawn points with explicit
   grading, so ordinary project configuration cannot silently break
   compiler invariants.

The shared rationale is kept: every rule the runtimes enforce
deterministically is a rule the core prompt no longer has to carry, which
shortens the prompt and stabilizes the cached prefix.

**Critical judgment:** every Lua spelling, field name, command name, tool
shape, and hook name is unreviewed vocabulary proposing no API. The stable
claims are the class list as candidate coverage (filtering, policy, rules,
commands, tools, agents, verification, hooks), the policy-over-code rule
for context configuration, the provenance-separation rule for tools, and
the hook-grading requirement.

## Trust posture

Status: **candidate shape; no protocol adopted.**

The retained trust rule is absolute at first contact: cloning a project and
entering it must never silently execute project Lua, because configuration
files are executable code with supply-chain consequences. The direnv-like
flow is kept as candidate shape: first sighting surfaces an untrusted
notice with no execution, an explicit trust action records the project path
plus a config hash, any config change invalidates trust pending re-review,
and inspection shows requested capabilities before granting. The sandbox
direction keeps project Lua off ambient authority (no direct process,
filesystem, or network reach) behind mediated functions, with declared
project permissions (filesystem scoping with outside-project denial, plus
process and network flags), so a project configuration is itself an
auditable dependency.

**Critical judgment:** the command spellings, notice wording, hash
mechanics, and permission fields are unreviewed sketches needing security
review; no trust protocol is adopted. The stable claims are the
no-silent-execution rule, the hash-pinned re-review trigger, and the
mediated-access posture. Enforcement placement stays with the security
corpus and the task-lifecycle and tool-transport dispositions.

## Portable versus native split

Status: **candidate; coexistence and non-pollution are the stable claims.**

The ecosystem compatibility layer declares what capabilities a project has:
skills and integration material live there in portable form, Wheel supports
and reads that layer, and Wheel never stuffs its own advanced harness
behavior into it. A project carrying its capabilities in the portable layer
keeps migration value across harness implementations, while the project
harness directory adds strictly Wheel-native behavior on top for projects
that opt into Wheel. The defining questions are kept as the boundary
slogan: the portable layer answers what capabilities exist; the harness
directory answers how Wheel should behave. The closing rule is
reference-not-copy: the harness directory references, filters, constrains,
and composes what the portable layer exposes instead of re-storing skills
or integration material.

The supporting scope sentence is kept with it: Bitty names the terminal
and runtime platform while Wheel names the Agent Harness, so project
agent behavior must not masquerade as platform configuration. The two
directories coexist durably without overlapping duties.

**Critical judgment:** no portable-layer layout is decided here. The stable
claims are the coexistence rule and the non-pollution direction
(harness-native extensions never land in the portable layer).

## No runtime state

Status: **candidate rule; separation itself is the stable claim.**

The retained rule keeps the project harness directory commit-worthy:
configuration only, never history databases, sessions, caches, or
checkpoints. Runtime data belongs in platform state, cache, and data homes,
or in explicitly ignored project paths that never enter the repository, so
versioned intent and mutable execution never mix. Everything under the
project directory must be safe to commit and review; that reviewability is
part of the rule, not a side effect.

**Critical judgment:** directory names and environment variables shown
anywhere in the direction are portable placeholders, not adopted paths. The
stable claim is the separation itself. Layering keeps three scopes
distinct: a global home carries personal defaults, the project directory
overrides them per repository, and the portable layer stays portable
across harness implementations. Only the
global-overridden-by-project precedence sentence is candidate direction;
every placement beyond it is design input for the owning task.

## Implementation reality

Status: **real; bounds every candidate above.**

The kernel Lua client exposes only task, slot, checkpoint, action, and
context-compile commands over its dispatch boundary. It loads no project
configuration: reading or evaluating a project entry appears nowhere in its
command surface.

No code in the Rust crates loads the project entry either. The only project
directory input the runtime accepts is a declarative data-file loader: the
host performs every filesystem read and passes bytes in, the module itself
performs no filesystem I/O, loaded bytes are validated exactly like
programmatic input, they can only narrow (never widen an upper layer), and
path escapes fail closed before any byte is parsed. That loader reads
structured declaration files, never executable configuration.

The scaffold configuration loader lives only inside the pre-implementation
plugin tree: layered defaults, sandbox-evaluated chunks, and a trust gate
with a content hash and an explicit approval action. It has no host wiring,
no caller, and no shipped behavior; it is illustration of the candidate
shape, never evidence that project configuration loads today.

Stated plainly: project Lua configuration does not execute anywhere in the
current implementation. The candidate contract above describes what a
future scoped task could adopt, not what exists.

## Security review

- No silent execution: entering or cloning a project must never execute
  its configuration without an explicit trust action, and any configuration
  change invalidates trust pending re-review.
- Mediated access only: project configuration reaches process, filesystem,
  or network solely through mediated functions with declared, auditable
  permissions; ambient reach is denied.
- Provenance separation: tools of different origins stay distinguishable
  under one capability surface so a permission decision never confuses
  where a capability came from.
- Capability-first inspection: requested capabilities are shown before
  granting, and grants stay narrow, reviewable, and revocable.
- Commit-worthy configuration: everything versioned under the project
  directory is safe to commit and review; secrets, sessions, caches, and
  checkpoints never belong there.
- Normative security obligations (least privilege, per-action scopes,
  capability-based and auditable permission that fails closed, typed
  redaction, consented recording, secret minimization) override every
  discussion example throughout this document.

## Relation to existing systems

The draft [AI Architecture](../architecture/ai-architecture.md) layered
models are candidate inputs only; the class list, the entry contract, the
trust flow, and the layering sketched here are **not** accepted by this
candidate design and must not be read as crate, package, protocol,
file-schema, or release decisions. Context assembly, budget, and retention
questions stay with
[Context Management Architecture](../context/context-management.md),
[Prefix-Cache-Friendly Context Design](../context/prefix-cache-context-design.md),
and [Context retention R3](../architecture/context-retention-r3.md); tool
shape and transport placement stay with
[Command and Tool Architecture](../architecture/command-tool-architecture.md)
and [Tool transport R2](../architecture/tool-transport-r2.md); provider
questions stay with
[Provider plugin boundary](../providers/provider-plugin-boundary.md);
execution and environment questions stay with
[Execution ownership R1](../architecture/execution-ownership-r1.md) and
[Panel environment awareness](../interfaces/panel-environment-awareness.md);
coordination and persistence questions stay with
[Agent Coordination Architecture](../agent/agent-coordination.md) and
[Task lifecycle R5](../architecture/task-lifecycle-r5.md). Extension
paradigms, composition layers, and distribution stay with the companion
[Wheel-extension-paradigms candidate design](wheel-extension-paradigms-candidate.md);
core-versus-plugin layering stays with the companion
[Wheel-core and Plugin-boundary candidate design](wheel-core-plugin-boundary-candidate.md);
compiler observability stays with the companion
[Quality-formula and Context-Compiler candidate design](quality-and-context-compiler-candidate.md).
Each is referenced, never duplicated or modified.

The accepted [IPC and Agent RFC](ipc-agent-rfc.md) defines the only
accepted IPC wire, scope, and Agent vocabulary. Every sketch name in the
candidate direction (entry and module spellings, class field names, hook
names, command names, permission names, notice wording) is a discussion
sketch: this draft records it as input and proposes no file schema, API,
command, tool, event, or wire format.

This draft creates or closes no AIQ or OQ identifier; open questions stay
with [AI Unresolved Questions](../product/ai-unresolved-questions.md) and
shared governance.

## Open items

These are future evidence requirements, not tests executed by this
documentation task. They keep the configuration contract falsifiable before
it constrains `bitty-ai`.

| Campaign     | Required observation                                                                                                                                               |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Single entry | One entry resolves project behavior while the interior organizes freely, with no mandatory subtree required, in reviewable tests.                                  |
| Filtering    | A disabled skill or integration stays out of discovery, activation, and context while an enabled one resolves identically, in reviewable tests.                    |
| Policy       | A declared context policy changes compiled output identically to its hand-rolled equivalent without per-prompt user code, in reviewable tests.                     |
| Rules        | A denied path, a gated confirmation, and a change-triggered check all hold without any prompt reminder present, in reviewable tests.                               |
| Tools        | A same-named tool from two provenances resolves with distinguishable identity and correctly scoped permission, in reviewable tests.                                |
| Verification | A completion gate rejects a plausible-but-wrong change the model self-reported as done, in reviewable tests.                                                       |
| Trust        | A changed project configuration re-prompts for trust before any execution, and an untrusted clone executes nothing silently, in reviewable tests.                  |
| Sandbox      | Project configuration attempting ambient process, filesystem, or network reach is denied through mediation, in reviewable tests.                                   |
| No-state     | Versioned intent and mutable execution never mix: the project directory holds reviewable configuration while runtime data lands elsewhere, in reviewable fixtures. |
| Split        | A harness-native extension ships without touching the portable layer, and a portable skill migrates harness implementations without edits, in reviewable tests.    |

Promotion needs independent AI architecture, context-management, terminal
and plugin-owner, docs-curator, and security review. Route entry spellings,
module organization, class fields, hook tiers, permission fields, trust
wording, and layering placements to scoped owner tasks. This draft changes
no normative contract and authorizes no product code.

## References

- [Wheel-extension-paradigms candidate design](wheel-extension-paradigms-candidate.md) (Draft): the four extension paradigms with real, candidate, and unimplemented labels; referenced, never duplicated.
- [Wheel-config and Git-model candidate design](wheel-config-and-context-git-model-candidate.md) (Draft): the fuller configuration, trust, and storage synthesis this record draws its candidate direction from; referenced, never duplicated.
- [Wheel-core and Plugin-boundary candidate design](wheel-core-plugin-boundary-candidate.md) (Draft): composition role and global, project, and portable layering; referenced, never duplicated.
