---
title: Wheel-config and Git-model research distillation for bitty-ai (050-051)
description: Draft bitty-ai distillation of research 050 wheel agents split config classes trust and 051 Git inspired context model storage modes
category: specifications
audience: mixed
document_type: research
status: draft
website_publish: false
sidebar_order: 62
---

# Wheel-config and Git-model research distillation for bitty-ai (050-051)

## Status and recommendation

This is a **draft discussion synthesis**, not a runtime specification, accepted
decision, dependency selection, release commitment, or implementation claim.
It distills only the `bitty-ai`-relevant parts of two workspace research
records: `050.md` (a `.wheel` versus `.agents` configuration-split
discussion: portable capabilities, eight `.wheel` function classes, the
single-entry Lua contract, no-runtime-state rule, Lua trust, capability
sandbox, and the final directory relationship) and `051.md` (a Git-inspired
context-model discussion in twenty-four numbered sections: DAG objects,
deduplication, checkpoint as commit, compaction as snapshot, HEAD, branches,
merge, cherry-pick, ref sharing, rebase, worktree binding, refs, reflog, GC,
packfile, object-hash cache, prompt-cache separation, checkout-like compiler,
multi-agent DAG, Git vocabulary, context diff, typed merge, the storage
diagram, and the closing philosophy).

The recommendation is to treat every model below as a candidate input to the
draft [AI Architecture](../ai-architecture.md) and its related draft
dispositions, never as an override of the accepted
[IPC and Agent RFC](../ipc-agent-rfc.md) or the normative security corpus.
In particular, the `.wheel` Lua API sketches, the directory layouts, the
hook and permission sketches, the trust-flow wording, the `ContextObject`
enum sketch, the CLI spellings, and the storage diagram are **not** accepted
by this distillation. Agent lifecycle, Agent events, and Agent semantics in
the sources are **discussion inputs only**: the accepted Agent contract stays
entirely with the RFC, which this draft references without restating
normatively. Nothing here is promoted to accepted status, and no
implementation is described as shipped.

The sources' strongest ideas are the What-versus-How split (portable
capabilities declared once, Harness behavior decided per project), the
reference-not-copy rule (`.wheel` filters and constrains `.agents` exposure
rather than duplicating skills or MCP), the single-entry Lua contract with
free internal organization, the no-runtime-state rule with XDG and
git-ignored placement, the no-silent-execution trust rule with hash-pinned
re-review, the capability-sandbox posture for project Lua, the immutable
content-addressed object store with cheap branches and explicit typed merges
for context and multi-agent state, and the strict separation of semantic
context storage from provider prompt cache. Their weakest claims are the
concrete Lua API spellings, which are unreviewed interface vocabulary with
no owner, versioning, or compatibility evidence; the hook-tier and
permission sketches, which name enforcement without an enforcement owner;
and the storage paths and hash choices, which are illustrations, not adopted
layout or cryptography. Those are corrected below.

This synthesis references, without duplicating or modifying, the draft
[AI Architecture](../ai-architecture.md) (candidate inputs only),
[Context Management Architecture](../context-management.md),
[Prefix-Cache-Friendly Context Design](../prefix-cache-context-design.md),
[Command and Tool Architecture](../command-tool-architecture.md),
[Agent Coordination Architecture](../agent-coordination.md),
[Execution ownership R1](../execution-ownership-r1.md),
[Context retention R3](../context-retention-r3.md),
[Task lifecycle R5](../task-lifecycle-r5.md),
[Tool transport R2](../tool-transport-r2.md),
[Provider plugin boundary](../provider-plugin-boundary.md), and
[Panel environment awareness](../panel-environment-awareness.md). The
accepted [IPC and Agent RFC](../ipc-agent-rfc.md) is unaffected by this
draft. The companion
[Quality-formula and Context-Compiler distillation (048-049)](research-distillation-048-049-bitty-ai.md)
carries the quality framing and the compiler design; this draft links to it
wherever configuration or storage touches compilation, as design input
rather than implementation. This document creates no AIQ or OQ identifier
and closes none.

## Provenance and evidence boundary

The sources are the workspace-relative `research/origin/050.md` and
`research/origin/051.md`, read read-only: `050.md` is **940 lines**,
**13,978 bytes**, SHA-256
`ce3b7f10baa003f99a2fe11824e0f4386ff55371324045edea776a19733a7d59`;
`051.md` is **1,308 lines**, **17,307 bytes**, SHA-256
`2c9ac93f653fc923be3e95bc275b88dbfa0f0dd0687fe339e571ceb338f3ef88`.
Both files are untracked in the research repository, so provenance is by path
plus fingerprint, not by commit. Neither origin file was renamed, edited, or
staged by this task; the `bitty`-side pass still needs both. The body of this
document distills the whole verified files (`050.md:1-940`,
`051.md:1-1308`); no post-task append existed at verification time, so no
head-versus-tail split applies. Any later append is uncovered and follows the
CTX-0045 pattern (distill the verified head, record the remainder as
uncovered).

**Verification:** confirm the source file integrity with:

```bash
sha256sum $BITTY_WORKSPACE/research/origin/050.md $BITTY_WORKSPACE/research/origin/051.md
wc -l -c $BITTY_WORKSPACE/research/origin/050.md $BITTY_WORKSPACE/research/origin/051.md
```

The expected output is the two SHA-256 values above with 940 lines and
13,978 bytes for `050.md` and 1,308 lines and 17,307 bytes for `051.md`.
Both hashes were verified at task start and re-verified at task end with no
change, so the CTX-0045 growth pattern did not trigger.

Record 050 is a single pass: an opening rename-and-split statement, a
portable-capabilities section, a How-versus-What section with the entry
contract, eight function-class sections, a core-prompt section, a
no-runtime-state section, a Lua-trust section, a sandbox section, and a
closing layout with the boundary sentence; no duplication handling applies.
Record 051 is a single pass: a framing paragraph plus twenty-four numbered
sections and a closing philosophy; no duplication handling applies. Only the
ranges in the coverage table were distilled. Both sources are single-author
Chinese-language discussions with English code and schema sketches; this
document is the English-language draft synthesis, not a translation. The
fingerprints identify the discussions, not the truth of their claims.

## Authority and reconciliation

The draft [AI Architecture](../ai-architecture.md) layered models are
candidate inputs only; the `.wheel` class list, the Lua contract, the trust
flow, the object model, the ref namespace, and the storage diagram proposed
in the sources are **not** accepted by this distillation and must not be read
as crate, package, protocol, file-schema, or release decisions. Context
assembly, budget, and retention questions stay with
[Context Management Architecture](../context-management.md),
[Prefix-Cache-Friendly Context Design](../prefix-cache-context-design.md),
and [Context retention R3](../context-retention-r3.md); tool shape and
transport placement stay with
[Command and Tool Architecture](../command-tool-architecture.md) and
[Tool transport R2](../tool-transport-r2.md); provider questions stay with
[Provider plugin boundary](../provider-plugin-boundary.md); execution and
environment questions stay with [Execution ownership R1](../execution-ownership-r1.md)
and [Panel environment awareness](../panel-environment-awareness.md);
coordination and persistence questions stay with
[Agent Coordination Architecture](../agent-coordination.md),
[Task lifecycle R5](../task-lifecycle-r5.md), and the persistence
dispositions. Each is referenced, never duplicated or modified.

The accepted [IPC and Agent RFC](../ipc-agent-rfc.md) defines the only
accepted IPC wire, scope, and Agent vocabulary. Every sketch name in the
sources (Lua function spellings, hook names, command names, agent-profile
fields, permission names, ref spellings, CLI spellings) is a discussion
sketch: this draft records it as input and proposes no file schema, API,
command, tool, event, or wire format. Where the sources' sketches overlap
RFC-owned ground (Agent lifecycle, Agent events, Agent semantics, scopes),
the RFC wins without further argument.

Normative security obligations (least privilege, per-action scopes,
capability-based and auditable permission that fails closed, typed redaction,
consented recording, secret minimization) override every discussion example
below. The Lua trust and sandbox sketches are conceptual vocabulary, not an
adopted authorization contract. This draft creates or closes no AIQ or OQ
identifier; open questions stay with
[AI Unresolved Questions](../ai-unresolved-questions.md) and shared
governance.

## Owner rename decision (recorded verbatim)

Source: `050.md:1-17` (the opening rename statement and the two-scope
diagram), recorded here as an owner decision about naming and scope, not as
an implementation claim.

> `.bitty/` is renamed to `.wheel/` (050.md:1-17; the source states the
> directory-level rename only and fixes `init.lua` as the single `.wheel/`
> entry at 050.md:108-140 — no `.bitty.lua`-style filename mapping appears
> in the source).
> Wheel reads `.agents` for skills and MCP compatibility. `.wheel/` owns
> skill toggles, rules, custom commands, and custom tools.

The supporting scope sentence is kept with it: Bitty names the terminal and
runtime platform while Wheel names the Agent Harness, so project AI-agent
behavior must not masquerade as Bitty platform configuration. The two
directories coexist durably without overlapping duties. No file was renamed
by this documentation task, and no implementation is described as shipped.

## Portable capabilities in `.agents`

Source: `050.md:19-73` (the interoperable-layer section with the skills and
MCP tree sketches and the portability examples).

The retained direction keeps `.agents/` as the ecosystem compatibility layer
that declares what capabilities a project has: skills and MCP material live
there in portable form, Wheel supports and reads that layer, and Wheel never
stuffs its own advanced Harness behavior into it. The portability argument
is kept: a project carrying its capabilities in `.agents/` keeps migration
value across Harness implementations, while `.wheel/` adds strictly
Wheel-native behavior on top for projects that opt into Wheel. The defining
questions are kept as the boundary slogan: `.agents` answers what
capabilities exist; `.wheel` answers how Wheel should behave.

**Critical judgment:** the tree sketches are illustrations, not an adopted
directory schema; no `.agents` layout is decided here. The stable claims are
the coexistence rule and the non-pollution direction (Harness-native
extensions never land in the portable layer).

## Open survey item: `.agents` project-level design

Owner direction, recorded as an open survey item rather than a claim: Wheel
must survey the `.agents` project-level directory design, taking the
Neovim-like configuration style as a reference, because that design
constrains `.wheel`. The source's own Neovim-style point (a single fixed
entry file with free internal organization, `050.md:108-140`) currently
addresses `.wheel` entry shape, not `.agents` layout; whether the same
reference transfers to the `.agents` project-level design, and exactly which
constraints it imposes on `.wheel` (discovery, filtering, precedence,
namespacing), stays with the owning survey task. This distillation decides
no schema on either side.

## Eight `.wheel` function classes

Source: `050.md:142-353` (capability filtering, context policy, rules) and
`050.md:401-644` (custom commands, custom tools, agent definitions,
verification, hooks), with `050.md:355-399` (the core-prompt shortening
argument) read as the shared rationale.

The retained classes are: capability filtering over discovered skills and
MCP (enable, disable, allow, prefer, and permission scoping, flowing from a
discovered registry through Wheel policy into a resolved set for the
compiler); context policy as high-level declarations (budgets, code
preferences, log handling, pinning, exclusion) with the explicit rule that
users describe policy rather than hand-roll per-prompt compiler algorithms;
structured rules and constraints that a policy engine enforces
deterministically (path-scoped denials, confirmation gates, change-gated
verification) instead of prompt reminders; project-defined custom commands
built in Lua; project-defined custom tools with strict provenance separation
(native, Lua, MCP, and skill-script origins stay distinguishable under one
agent-visible capability surface); structured agent definitions (model
reference, tool and skill selection, permission bounds) that let the
compiler trim per profile; verification declarations (change-gated checks,
test discovery with targeted-then-full shape, completion gates) so
done-ness is decided by evidence; and hooks across task, tool, compile, and
spawn points with explicit grading (safe, advanced, unsafe or
capability-gated) so ordinary project configuration cannot silently break
compiler invariants. The shared rationale is kept: every rule the runtimes
enforce deterministically is a rule the core prompt no longer has to carry,
which shortens the prompt and stabilizes the cached prefix.

**Critical judgment:** every Lua spelling, field name, command name, tool
URI shape, and hook name is unreviewed vocabulary proposing no API. The
stable claims are the class list as candidate coverage (filtering, policy,
rules, commands, tools, agents, verification, hooks), the policy-over-code
rule for context configuration, the provenance-separation rule for tools,
and the hook-grading requirement.

## Single-entry contract and free organization

Source: `050.md:77-140` (the How-versus-What section with the entry sketch
and the Neovim-style sentence).

The retained contract fixes exactly one entry point (an init file at the
`.wheel` root) while remaining directories stay ordinary modules the entry
requires; no prescribed subdirectory is mandatory. The Neovim-style
reference is kept as the organization principle: one stable contract at the
root, user freedom inside.

**Critical judgment:** the entry filename and module organization are the
source's proposal, not an adopted schema; the open survey item above owns
any constraint flowing from the `.agents` side. The stable claim is only the
singularity (one entry, not a mandatory tree).

## No runtime state in `.wheel`

Source: `050.md:646-709` (the versioned-configuration versus mutable-data
split with XDG and git-ignored placements).

The retained rule keeps `.wheel/` commit-worthy: configuration only, never
history databases, sessions, caches, or checkpoints. Runtime data belongs in
platform state, cache, and data homes (or explicitly git-ignored project
paths that never enter the repository), so versioned intent and mutable
execution never mix.

**Critical judgment:** the directory names shown are illustrations, not an
adopted layout. The stable claim is the separation itself, which is also a
reviewability property: everything under `.wheel/` must be safe to commit
and review.

## Lua trust and the capability sandbox

Source: `050.md:711-843` (the no-silent-execution argument with the
direnv-like trust flow, plus the sandbox section with the permission
sketch).

The retained trust rule is absolute at first contact: cloning a project and
entering it must never silently execute project Lua, because configuration
files are executable code with supply-chain consequences. The direnv-like
flow is kept as candidate shape: first sighting surfaces an untrusted notice
with no execution, an explicit trust action records project path plus config
hash, any config change invalidates trust pending re-review, and inspection
shows requested capabilities before granting. The sandbox direction keeps
project Lua off ambient authority (no direct process, filesystem, or network
reach) behind Wheel-mediated functions, with declared project permissions
(filesystem scoping with outside-project denial, process and network flags)
so a project configuration is itself an auditable dependency.

**Critical judgment:** the command spellings, notice wording, hash mechanics,
and permission fields are unreviewed sketches needing security review; no
trust protocol is adopted. The stable claims are the no-silent-execution
rule, the hash-pinned re-review trigger, and the mediated-access posture.
Enforcement placement stays with the security corpus and the R1/R2
dispositions.

## Final layout and the load pipeline

Source: `050.md:845-940` (the closing directory relationship, the
discovery-to-runtime pipeline, and the boundary sentence).

The retained relationship loads both project layers and resolves them into
runtime behavior: `.agents` feeds capability discovery into a registry,
`.wheel` Lua configuration feeds a policy engine, a resolver combines them
into enabled skills and tools, the Context Compiler shapes the active view,
and the Agent Runtime acts on it. The closing boundary sentence is kept as
the slogan: `.agents` holds portable agent capabilities while `.wheel`
holds Wheel-native Harness behavior, so `.wheel` references, filters,
constrains, and composes what `.agents` exposes instead of re-storing skills
or MCP material.

**Critical judgment:** the layout and pipeline are candidate topology, not
adopted packaging or control flow. The stable claim is the
reference-not-copy rule, which is the configuration analogue of the
compiler's superset inequality in the companion distillation.

## Git-inspired context model

Source: `051.md:1-1308` as grouped below. Cross-cutting lens (owner
direction): everywhere this section touches compression, context management,
or multi-agent design, the Git design (object model, content addressing,
GC and packfile discipline, DAG merge) is noted explicitly as design input,
never as implementation.

### DAG over transcript, objects over strings

Source: `051.md:9-121` (sections 1-2: the DAG sketch with checkpoint,
decision, and result nodes; the artifact, tree, checkpoint, summary,
decision, task, and result object sketch with content hashing and sharded
storage).

The retained shift replaces the append-only message list with a context
graph whose nodes are typed objects (artifacts, trees, checkpoints,
summaries, decisions, tasks, results), each immutable, hashed, deduplicated,
integrity-checked, and addressable. Storage and presentation separate from
the start: the graph persists everything while per-turn views are compiled.

**Critical judgment:** the enum shape, hash choice, and storage paths are
illustrations. The stable claims are immutability, content addressing, and
the graph-not-list topology. Git-design reference: object model and content
addressing are explicit design inputs for compression and context
management.

### Deduplication by hash

Source: `051.md:123-168` (section 3: repeated file reads collapsing to one
artifact with ref sharing).

The retained rule stores one artifact per content hash no matter how often
an agent reads it; distinct contexts hold references, mirroring multiple
commits sharing one blob. This bounds both storage and the temptation to
re-bill identical bytes against the active view.

**Critical judgment:** no hash function or retention rule is adopted. The
stable claim is ref-sharing over re-storage. Git-design reference: content
addressing is an explicit design input for context management.

### Checkpoint as commit

Source: `051.md:170-231` (section 4: the checkpoint sketch with parent,
goal, tree, modified paths, decisions, verification) and `051.md:303-351`
(section 6: HEAD pointing at a checkpoint, per-phase checkpointing, a
context-log sketch).

The retained model makes a checkpoint the commit analogue: it binds a
context tree, a parent, task state, decisions, evidence, and the acting
agent, so a session reads as a checkpoint chain rather than a transcript.
HEAD names the current checkpoint, the Active Context compiles from HEAD,
phase completion advances HEAD, and a log-like view recovers the session
narrative without replaying raw history.

**Critical judgment:** the checkpoint fields and log syntax are vocabulary;
no checkpoint schema or command is adopted. The stable claim is the
chain-of-states topology with HEAD as the single current pointer.
Git-design reference: commit chaining and HEAD are explicit design inputs
for context management.

### Compaction as snapshot, not deletion

Source: `051.md:233-301` (section 5: phase-end snapshots preserving goal,
decisions, tree, artifacts, open problems, and verification while the raw
episode leaves the active view).

The retained rule changes what compaction means: creating a higher-level
snapshot that HEAD now sees, with history retained rather than destroyed,
mirroring that checking out a commit never deletes prior commits. This is
the storage analogue of the compiler's traceable-compaction rule in the
companion distillation.

**Critical judgment:** no snapshot schema or retention bound is adopted. The
stable claim is view-change-without-history-loss. Git-design reference:
snapshot discipline is an explicit design input for compression.

### Branches for hypotheses, merges for synthesis

Source: `051.md:352-482` (sections 7-8: hypothesis branches sharing one
parent with delta-only records; multi-agent merge across code, decisions,
task state, evidence, and context with explicit conflicts) and
`051.md:484-520` (section 9: cherry-picking one finding with its evidence
instead of merging a whole branch).

The retained model treats subagent exploration as cheap branches (shared
parent, no transcript copying, per-branch deltas) and multi-agent synthesis
as merge with first-class conflicts: contradictory hypotheses surface as
structured pending conflicts rather than being quietly summarized into a
vague consensus. Cherry-picking carries a single finding plus evidence into
another context, which is the mechanism behind subagent isolation: parents
absorb conclusions, never transcripts.

**Critical judgment:** no branch, merge, or conflict schema is adopted. The
stable claims are delta-only branching, explicit-merge synthesis, and
finding-level cherry-picking. Git-design reference: DAG merge is an explicit
design input for multi-agent design.

### Sharing refs, rebasing stale work

Source: `051.md:522-611` (sections 10-11: agents exchanging parent and
artifact references with on-demand expansion; rebasing a branch onto a newer
checkpoint with staleness checks, revalidation, replay, and conflict
marking).

The retained rules are that agents share references rather than histories
(expand only what is needed, so sharing costs refs instead of tokens) and
that long-running branches rebase semantically: extract post-divergence
decisions and artifacts, check each against the new base, revalidate, replay
what still holds, and mark stale-artifact conflicts where hashes moved.

**Critical judgment:** no ref-exchange protocol or rebase procedure is
adopted. The stable claims are ref-cost sharing and validate-before-replay.
Git-design reference: ref sharing and rebase semantics are explicit design
inputs for multi-agent design.

### Worktree binding and the ref namespace

Source: `051.md:613-695` (sections 12-13: per-agent context, git worktree,
and panel binding; task, agent, checkpoint, session, and user ref
spellings).

The retained direction binds the three isolations together (context branch,
filesystem worktree, execution panel per agent) so parallel agents never
share a working tree, and names task and agent state through a ref-like
namespace instead of scattering it across relational tables over joins. The
object store plus refs is kept as sufficient expressive machinery for much
of agent state.

**Critical judgment:** the ref spellings and binding shape are vocabulary;
no namespace or lifecycle machine is adopted, and lifecycle authority stays
with the RFC and R1/R5. Git-design reference: refs and worktree binding are
explicit design inputs for multi-agent design.

### Reflog recovery, GC, and packfile

Source: `051.md:697-842` (sections 14-16: reflog over checkpoints with
restore; mark-and-sweep GC from refs, pins, sessions, and saved memory with
a grace period; loose objects packed with delta compression in background)
and `051.md:844-894` (section 17: hash-keyed artifact, retrieval, and
compaction caches with clean invalidation).

The retained mechanisms are: every HEAD move stays recoverable through a
reflog (bad compacts, switches, merges, and task updates are inspectable and
restorable); unreachable artifacts, abandoned branches, and temporary logs
collect after a grace period by traversing from live roots instead of
growing a store without bound; small objects start loose and pack in
background with delta compression for near-duplicate snapshots; and all
three compiler caches key off content hashes (plus parser, compiler, prompt,
and model versions where relevant) so reuse and invalidation stay exact.

**Critical judgment:** no GC policy, pack format, key shape, or command
spelling is adopted. The stable claims are recoverability-by-default,
rooted collection with a grace period, and hash-exact cache identity.
Git-design reference: GC and packfile discipline are explicit design inputs
for compression and context management.

### Store-cache separation and the checkout compiler

Source: `051.md:896-1012` (sections 18-19: the DAG-to-compiler-to-prompt
chain; checkout-HEAD producing an Active Context View through authority,
dedup, and budget passes).

The retained separation keeps the content-addressed context DAG (semantic
storage) strictly apart from the provider prompt cache (serialized prefix):
store hashes stabilize the compiler's input, which makes provider caching
easier without ever equating the two. The compiler itself is kept as the
checkout analogue: from objects plus refs plus DAG it produces the Active
Context View, which is the operational form of the per-turn compiled view in
the companion distillation.

**Critical judgment:** no store schema or compiler control flow is adopted;
the pass list here defers to the companion twelve-pass pipeline. The stable
claims are the two-layer split (semantic store below, provider cache above)
and the checkout metaphor (HEAD in, view out).

### Multi-agent DAG and the vocabulary decision

Source: `051.md:1014-1125` (sections 20-21: DAG over manager-tree with
parent lists carrying parallelism, merge, reuse, and provenance; user-facing
checkpoint, artifact, and context terms over commit, blob, and tree with
object kept as the implementation word and a CLI sketch).

The retained topology models agent work as a DAG (explore branches fanning
out and rejoining through review into implementation, each node naming
parents) rather than a strict manager tree, because real agent dependencies
cross. The vocabulary decision is kept: user-facing terms stay
domain-familiar (checkpoint, branch, merge, rebase, gc, ref, HEAD,
worktree) while storage-level Git terms are not cargo-culted onto users
(checkpoint not commit, artifact not blob, context not tree), with the
programmer familiarity argument recorded as motivation, not evidence.

**Critical judgment:** the CLI spellings are illustration proposing no
surface; the DAG shape is candidate topology. The stable claims are
parent-listed multi-agent history and the two-level naming rule.

### Context diff and typed merge

Source: `051.md:1127-1203` (sections 22-23: a checkpoint-to-checkpoint diff
sketch across task state, decisions, code, diagnostics, skills, and tools;
per-type merge rules with conflict surfacing) and `051.md:1205-1308`
(section 24 with the storage diagram and the closing philosophy).

The retained instruments are a context diff that shows what changed between
checkpoints per content type (added root cause, new decisions, code-context
swaps, diagnostic flips, unchanged skills and tools) so compaction loss is
inspectable, and a typed merge that never concatenates text: decisions merge
with semantic conflict detection, task state with structured merge,
artifacts with union plus version resolution, code through version control
itself, diagnostics latest-valid-first, and summaries with provenance
preserved. The storage diagram (refs over task, agent, and checkpoint
names; HEAD over the DAG; DAG over content-addressed loose and packed
objects; store feeding compiler feeding provider) and the closing
philosophy (full history plus current snapshot plus efficient references;
full session history kept distinct from Active Model Context) are kept as
the candidate architecture and its slogan.

**Critical judgment:** the diff format, merge rules, diagram boxes, and
slogan are proposals, not contracts; nodes stay typed so the architecture
keeps its advantage over markdown-string merging. Git-design reference:
typed DAG merge is an explicit design input for multi-agent design and
compression.

## Wheel modes

Owner direction, recorded as design-only with no implementation claim:
Wheel operates in three modes. Headless mode runs agents without visible
panels (grounded in the lifecycle-independence material: `048.md:2008-2062`
and the per-agent headless binding sketch at `051.md:613-655`). The
Panel-versus-Agent lifecycle split keeps panels and agents as independent
entities joined by association (attach, spawn-headless, release, reattach),
grounded in the same ranges with lifecycle authority staying RFC-owned.
One-shot chat, the single-question-and-answer non-Agent form, is recorded
here as an owner-directed Wheel mode alongside the two source-grounded
modes; its details stay with the owning task and it proposes no interface,
no lifecycle machine, and no shipped behavior.

## Provider observability as a design pointer

Owner direction, recorded as a design pointer with explicitly no
`bitty-ai` code change in this task: the provider side should expose cost,
cache-hit, input, context, and timing signals for future plugins, with
git-style traceability (`git log --stat` and `git blame` as the UX
reference) as the interaction model for attributing cost and cache behavior
to decisions over time. A future `AI-XXXX` task implements this against a
real HTTP or Router adapter; that task owns the endpoint, field, metric,
and view contract. No name or format sketched anywhere in the sources is
adopted here. This pointer complements, and does not duplicate, the
compiler-side observability direction (statusline, context, cache, why, and
trace views) carried in the companion
[Quality-formula and Context-Compiler distillation](research-distillation-048-049-bitty-ai.md#provider-observability-as-a-design-pointer).

## Sections read for boundary accuracy

Both verified files were distilled in full; the ranges below were read so
that non-`bitty-ai` content is not silently absorbed.

- `050.md` Bitty-platform passages (the terminal-versus-Harness scope
  sentences): kept only as the naming rationale for the rename decision;
  terminal mechanisms propose no Core API.
- `051.md:1-8` framing (`Git as the most borrow-worthy system for context,
compaction, multi-agent`): kept as the author's borrowing thesis with its
  principle list (immutable objects, content addressing, DAG, cheap
  branches, explicit merge, refs, collection); no Git behavior is claimed
  for any implementation.
- `051.md` storage-path and hash-choice illustrations (object directories,
  pack filenames, hash algorithm names): read as illustrations; no path,
  format, or algorithm is adopted.
- Reference lists and vendor-behavior notes in either record: read as
  citation provenance only; no cited claim is reproduced as a finding.

## Source coverage and critical disposition

Retain means retain as a **proposal**, not accept as normative. Improve means
retain the objective with the correction above. Reject means reject that
mechanism or absolute claim; defer means no commitment pending named
evidence.

| Source lines  | Topic                                                               | Disposition, rationale, and alternative                                                                   |
| ------------- | ------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| 050:1-17      | Rename `.bitty` to `.wheel`; two scopes; coexistence                | Record owner decision verbatim; naming and scope only; no implementation claim.                           |
| 050:19-73     | `.agents` as portable capabilities; skills and MCP; migration value | Retain coexistence and non-pollution; tree sketches are illustration, not schema.                         |
| 050:77-140    | How-versus-What; single entry; Neovim-style organization            | Retain one-entry contract as candidate; organization is proposal; survey item owns `.agents` constraints. |
| 050:142-220   | Capability filtering; registry to resolved set                      | Retain class as candidate; API spellings are vocabulary; enforcement needs review.                        |
| 050:222-282   | Context policy declarations; policy-over-code rule                  | Retain declarative direction; budget and field sketches are illustration.                                 |
| 050:284-353   | Structured rules; deterministic enforcement                         | Retain engine-over-reminder direction; rule shapes are vocabulary.                                        |
| 050:355-399   | Core-prompt shortening through runtimes                             | Retain rationale; prompt lines are illustration, not adopted text.                                        |
| 050:401-435   | Custom commands in Lua                                              | Retain class as candidate; command shapes are vocabulary, not API.                                        |
| 050:437-486   | Custom tools with provenance separation                             | Retain provenance rule; tool URIs and fields are illustration.                                            |
| 050:488-548   | Agent definitions with per-profile trimming                         | Retain structured-profile direction; profile fields are vocabulary.                                       |
| 050:550-600   | Verification declarations; completion gates                         | Retain evidence-gating direction; check lists and gates are illustration.                                 |
| 050:602-644   | Hooks with safe, advanced, unsafe grading                           | Retain grading requirement; hook names are vocabulary; owner open.                                        |
| 050:646-709   | No runtime state; XDG and git-ignored placement                     | Retain separation rule; directory names are illustration.                                                 |
| 050:711-803   | Lua trust; no silent execution; direnv-like flow                    | Retain no-silent-execution and re-review rules; flow is candidate needing review.                         |
| 050:805-843   | Capability sandbox; mediated access; permission sketch              | Retain mediated-access posture; permission fields are vocabulary.                                         |
| 050:845-940   | Final layout; discovery-to-runtime pipeline; boundary slogan        | Retain reference-not-copy rule; layout and pipeline are candidate topology.                               |
| 051:1-8       | Git borrowing thesis and principle list                             | Retain as author motivation; principles become design inputs, not claims.                                 |
| 051:9-60      | Context DAG over message list                                       | Retain graph topology as candidate; node list is vocabulary.                                              |
| 051:62-121    | Object sketch; hashing; sharded storage                             | Retain immutability and addressing; enum, hash, paths are illustration.                                   |
| 051:123-168   | Deduplication by hash; ref sharing                                  | Retain ref-sharing rule; hash choice open.                                                                |
| 051:170-231   | Checkpoint as commit with parents and evidence                      | Retain chain topology; checkpoint fields are vocabulary.                                                  |
| 051:233-301   | Compaction as snapshot without deletion                             | Retain view-change rule; schema and bounds open.                                                          |
| 051:303-351   | HEAD pointer; per-phase advance; log sketch                         | Retain single-pointer rule; lifecycle stays RFC-owned; syntax open.                                       |
| 051:352-403   | Hypothesis branches with shared parents                             | Retain delta-only branching; branch mechanics open.                                                       |
| 051:405-482   | Multi-agent merge with explicit conflicts                           | Retain explicit-merge rule; conflict schema open.                                                         |
| 051:484-520   | Cherry-picking findings with evidence                               | Retain finding-level rule; procedure open.                                                                |
| 051:522-557   | Ref sharing between agents                                          | Retain ref-cost rule; exchange protocol open.                                                             |
| 051:559-611   | Semantic rebase with staleness checks                               | Retain validate-before-replay; procedure open.                                                            |
| 051:613-655   | Worktree binding per agent                                          | Retain isolation direction; binding shape and lifecycle open.                                             |
| 051:657-695   | Ref namespace for task and agent state                              | Retain namespace direction; spellings are vocabulary.                                                     |
| 051:697-734   | Reflog recovery and restore                                         | Retain recoverability rule; syntax open.                                                                  |
| 051:736-788   | GC from live roots with grace period                                | Retain rooted-collection rule; policy open.                                                               |
| 051:790-842   | Packfile packing with delta compression                             | Retain pack discipline; format open.                                                                      |
| 051:844-894   | Hash-keyed compiler caches                                          | Retain hash-exact identity; key shapes open.                                                              |
| 051:896-942   | Store-cache separation chain                                        | Retain two-layer split; no store schema adopted.                                                          |
| 051:944-1012  | Checkout-like compiler to Active View                               | Retain checkout metaphor; passes defer to the companion pipeline.                                         |
| 051:1014-1074 | Multi-agent DAG with parent lists                                   | Retain DAG topology as candidate; procedures open.                                                        |
| 051:1076-1125 | Vocabulary decision; CLI sketch                                     | Retain two-level naming; spellings propose no surface.                                                    |
| 051:1127-1165 | Context diff across checkpoints                                     | Retain inspectability rule; diff format open.                                                             |
| 051:1167-1203 | Typed merge with per-type rules                                     | Retain never-concatenate rule; merge rules open.                                                          |
| 051:1205-1262 | Storage diagram; refs over DAG over objects                         | Retain as candidate architecture; boxes are not packaging.                                                |
| 051:1264-1308 | Closing philosophy; history versus active view                      | Retain slogan as author opinion; inequality is the load-bearing claim.                                    |

## Proposed validation and promotion path

These are future evidence requirements, not tests executed by this
documentation task. They keep the design falsifiable before any
configuration or storage proposal constrains `bitty-ai`.

| Campaign         | Required observation                                                                                                                                                    |
| ---------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Rename           | Project configuration under the new name resolves while portable capabilities keep working across Harness implementations, with no behavior change in reviewable tests. |
| Split            | A Harness-native extension ships without touching the portable layer, and a portable skill migrates Harness implementations without edits, in reviewable tests.         |
| Survey           | The `.agents` project-level survey names the constraints on `.wheel` with the configuration reference evaluated and a schema decision recorded by the owning task.      |
| Filtering        | A disabled skill or MCP stays out of discovery, activation, and context while an enabled one resolves identically, in reviewable tests.                                 |
| Policy           | A declared context policy changes compiled output identically to its hand-rolled equivalent without per-prompt user code, in reviewable tests.                          |
| Rules            | A denied path, a gated confirmation, and a change-triggered check all hold without any prompt reminder present, in reviewable tests.                                    |
| Tools            | A same-named tool from two provenances resolves with distinguishable identity and correctly scoped permission, in reviewable tests.                                     |
| Verification     | A completion gate rejects a plausible-but-wrong change the model self-reported as done, in reviewable tests.                                                            |
| Trust            | A changed project configuration re-prompts for trust before any execution, and an untrusted clone executes nothing silently, in reviewable tests.                       |
| Sandbox          | Project Lua attempting ambient process, filesystem, or network reach is denied through mediation, in reviewable tests.                                                  |
| Objects          | Repeated identical reads store once and serve by reference with integrity intact, in reviewable fixtures.                                                               |
| Checkpoint       | Phase completion advances HEAD with parent linkage intact, and history replays from the chain without transcript dependence, in reviewable tests.                       |
| Compaction       | A compacted episode leaves the active view while remaining restorable from history, with diff showing exactly what left, in reviewable tests.                           |
| Merge            | Contradictory agent hypotheses surface as structured pending conflicts rather than silent consensus, in reviewable tests.                                               |
| GC               | Abandoned branches and temporary artifacts collect after the grace period while pinned roots survive, in reviewable storage fixtures.                                   |
| Modes            | Headless, split-lifecycle, and one-shot forms each complete a scoped task without implying the others' machinery, in reviewable tests.                                  |
| Provider pointer | A future adapter task demonstrates cost, cache-hit, input, context, and timing signals flowing to a plugin surface with git-style attribution, with no Core change.     |

Promotion needs independent AI architecture, context-management, terminal
and plugin-owner, docs-curator, and security review. Route Lua spellings,
schemas, hook tiers, permission fields, trust wording, ref spellings, CLI
spellings, storage paths, hash choices, merge rules, diff formats, mode
details, and the provider-signal contract to scoped owner tasks. This draft
changes no normative contract and authorizes no product code.
