---
title: Quality-formula and Context-Compiler research distillation for bitty-ai (048-049)
description: Draft bitty-ai distillation of research 048 quality formula Wheel architecture and 049 Context Compiler zones cache budgets observability
category: specifications
audience: mixed
document_type: research
status: draft
website_publish: false
sidebar_order: 61
---

# Quality-formula and Context-Compiler research distillation for bitty-ai (048-049)

## Status and recommendation

This is a **draft discussion synthesis**, not a runtime specification, accepted
decision, dependency selection, release commitment, or implementation claim.
It distills only the `bitty-ai`-relevant parts of two workspace research
records: `048.md` (a coding-agent quality-formula discussion with a Wheel
architecture proposal: quality factors, pollution, code reading, tool
ergonomics, Skill Hell, trust, prompt size, verification, state separation,
compaction, lifecycle splits, scheduling, provider separation, eval, and the
closing Wheel architecture with its top principle) and `049.md` (a Context
Compiler design discussion: Context IR, stability zones, cache tree, stable
serialization, admission scoring, authority, content addressing, tool-result
reduction, compaction modes, traceability, Context Pointer, pass pipeline,
budgets and quotas, cache kinds, stable prefix, tool activation, provider
cache capability, versioning, observability, trace, worked example, and the
five-subsystem split).

The recommendation is to treat every model below as a candidate input to the
draft [AI Architecture](../ai-architecture.md) and its related draft
dispositions, never as an override of the accepted
[IPC and Agent RFC](../ipc-agent-rfc.md) or the normative security corpus.
In particular, the quality-formula symbols, the `ContextUnit` and
`CapabilityMeta` struct sketches, the zone labels, the admission-score
formula, the authority ladder, the pass-pipeline shape, the budget numbers,
the cache-namespace fields, the `/context` display sketches, and the Wheel
module diagram are **not** accepted by this distillation. Agent lifecycle,
Agent events, and Agent semantics in the sources are **discussion inputs
only**: the accepted Agent contract stays entirely with the RFC, which this
draft references without restating normatively. Nothing here is promoted to
accepted status, and no implementation is described as shipped.

The sources' strongest ideas are the multiplicative quality framing (a weak
factor zeroes the product, so verification, relevance, and ergonomics deserve
engineering before window size), the pollution-first diagnosis with Context
Efficiency as the objective, the per-turn compiled-view compiler over
append-until-compact, the Cold/Warm/Hot stratification with stable-first
ordering, the correctness-over-cache-hit priority, the authority ladder that
resolves conflicts before the model sees them, content-addressed code with
staleness eviction, deterministic-then-structural reduction before any model
summarization, traceable compaction with rehydration pointers, and the
eval-from-day-one loop. Its weakest claims are the concrete numeric sketches
(55K-token tool-set anecdote, 30-to-50-tool threshold, 80-percent cache-ratio
hope, 40K-to-250K budget bands, relevance scores such as 0.93), which are
unmeasured illustrations with no fixture, replay, or product evidence; the
struct and enum sketches, which name no owner, versioning, or compatibility
contract; and the provider-behavior descriptions, which report third-party
cache semantics the task did not independently verify. Those are corrected
below.

This synthesis references, without duplicating or modifying, the draft
[AI Architecture](../ai-architecture.md) (candidate inputs only),
[Context Management Architecture](../context-management.md),
[Prefix-Cache-Friendly Context Design](../prefix-cache-context-design.md),
[Prompt Layering Design](../prompt-layering-design.md),
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
[Wheel-config and Git-model distillation (050-051)](research-distillation-050-051-bitty-ai.md)
carries the `.wheel` configuration split and the Git-inspired storage model;
this draft links to it wherever compression, context management, or
multi-agent design touches that model, as design input rather than
implementation. This document creates no AIQ or OQ identifier and closes
none.

## Provenance and evidence boundary

The sources are the workspace-relative `research/origin/048.md` and
`research/origin/049.md`, read read-only: `048.md` is **2,351 lines**,
**57,159 bytes**, SHA-256
`ddfd88b2eaa66e983d1bd9dc659c03ee7615db546d3eb92510c8e4d4444d0455`;
`049.md` is **2,135 lines**, **32,648 bytes**, SHA-256
`30b6c86d8c61b692d95527d487ac1ab523c59da324eadde77e6469f8920594aa`.
Both files are untracked in the research repository, so provenance is by path
plus fingerprint, not by commit. Neither origin file was renamed, edited, or
staged by this task; the `bitty`-side pass still needs both. The body of this
document distills the whole verified files (`048.md:1-2351`,
`049.md:1-2135`); no post-task append existed at verification time, so no
head-versus-tail split applies. Any later append is uncovered and follows the
CTX-0045 pattern (distill the verified head, record the remainder as
uncovered).

**Verification:** confirm the source file integrity with:

```bash
sha256sum $BITTY_WORKSPACE/research/origin/048.md $BITTY_WORKSPACE/research/origin/049.md
wc -l -c $BITTY_WORKSPACE/research/origin/048.md $BITTY_WORKSPACE/research/origin/049.md
```

The expected output is the two SHA-256 values above with 2,351 lines and
57,159 bytes for `048.md` and 2,135 lines and 32,648 bytes for `049.md`.
Both hashes were verified at task start and re-verified at task end with no
change, so the CTX-0045 growth pattern did not trigger.

Record 048 is a single pass: an owner question block, a conclusion block, a
quality-formula section, eleven numbered quality sections, a prioritized
optimization order, a Harness-philosophy section, a Wheel-goal restatement,
seventeen Wheel design sections, a closing architecture, and a top
principle; no duplication handling applies. Record 049 is a single pass: a
framing answer plus thirty numbered Context Compiler sections with a closing
subsystem split and reference list; no duplication handling applies. Only
the ranges in the coverage table were distilled. Both sources are
single-author Chinese-language discussions with English code and formula
sketches; this document is the English-language draft synthesis, not a
translation. The fingerprints identify the discussions, not the truth of
their claims. External citations inside the sources (SWE-agent ACI
experiments, lost-in-the-middle and context-rot studies, Anthropic context
engineering and tool-writing guidance, OpenAI harness-engineering notes,
provider cache documentation, ToxicSkills scan figures, tool-poisoning
reports) are unverified discussion citations, not findings reproduced here.

## Authority and reconciliation

The draft [AI Architecture](../ai-architecture.md) layered models are
candidate inputs only; the quality formula, the seven-module Wheel sketch,
the Context IR shape, the zone model, the pipeline, and the five-subsystem
split proposed in the sources are **not** accepted by this distillation and
must not be read as crate, package, protocol, or release decisions. Context
assembly, budget, and retention questions stay with
[Context Management Architecture](../context-management.md),
[Prefix-Cache-Friendly Context Design](../prefix-cache-context-design.md),
and [Context retention R3](../context-retention-r3.md); prompt-text layering
stays with [Prompt Layering Design](../prompt-layering-design.md); tool shape
and transport placement stay with
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
sources (tool names such as `code.search` or `capability.call`, hook names,
command names, struct and enum names, metric names, CLI spellings such as
`wheel eval` or `wheel context log`) is a discussion sketch: this draft
records it as input and proposes no command, tool, event, wire format, or
CLI surface. Where the sources' sketches overlap RFC-owned ground (Agent
lifecycle, Agent events, Agent semantics, scopes), the RFC wins without
further argument.

Normative security obligations (least privilege, per-action scopes,
capability-based and auditable permission that fails closed, typed redaction,
consented recording, secret minimization) override every discussion example
below. Capability metadata, trust levels, permission sets, and verification
gates in the sources are conceptual vocabulary, not additions to any accepted
registry, schema, or protocol. This draft creates or closes no AIQ or OQ
identifier; open questions stay with
[AI Unresolved Questions](../ai-unresolved-questions.md) and shared
governance.

## Quality formula and optimization order

Source: `048.md:31-77` (the multiplicative conclusion) and `048.md:925-963`
(the ten-item optimization order with the Harness-philosophy framing).

The retained model writes agent quality as a product of Model, Context, ACI
and Tools, Verification, and Harness: a near-zero factor zeroes the whole,
so a strong model with no verification, unusable tools, polluted context, or
a coercive Harness still fails. The worked consequences are kept: no tests
invite confident-but-wrong implementations; brittle tools burn turns on
schema errors; garbage context hides the real constraints; a rigid system
prompt forces capable models around long detours. The ten-item engineering
order is retained as a resourcing opinion under a strong-model assumption:
verification loop first, then context relevance, ACI and tool ergonomics,
Harness policy and prompt posture, repository legibility, state and
compaction, skill and MCP discovery quality, orchestration, security and
provenance, and finally the model ceiling, which reasserts itself on hard
reasoning and novel architecture.

**Critical judgment:** the multiplication is a framing device, not a measured
model; the order is an author opinion that shifts by task, not a benchmark
ranking. The stable claims are the bottleneck argument (Harness-side factors
routinely dominate once models are capable) and the falsifiability hook
below: any adoption must name which factor it moves and how the Eval section
measures it. The Harness-philosophy corollary (small core prompt, small
active tool and skill sets, small high-signal context, rich discoverable
environment, strong verification, durable structured state) is retained as
the design slogan that the rest of the distillation cashes out.

## Context pollution and the efficiency objective

Source: `048.md:78-207` (window-versus-attention, the pollution catalog, the
cargo-test reduction example) and `049.md:605-661` (the quality-over-hit
priority with its utility and ordering statements).

The retained diagnosis is that low signal-to-noise ratio, not raw length, is
the first-order context problem: stale logs, full build outputs, resolved
stacks, unrelated file dumps, abandoned diffs and hypotheses, full MCP tool
schemas, overlapping skill descriptions, duplicated instructions, and whole
subagent transcripts are all formerly-useful content billed against every
future inference. The objective is therefore Context Efficiency
(task-relevant information over total active tokens): a smaller high-signal
context routinely beats a larger diluted one on cost, latency, and
goal-adherence. The cargo-test illustration is kept as the canonical
reduction (thousands of raw lines compile to counts, named failures, and
relevant diagnostics, with the raw log retained in storage and re-fetchable
on demand). Record 049 adds the priority guardrail that this distillation
adopts: correctness outranks relevance, relevance outranks cache hit, and
cache hit outranks token cost, so stale content is never retained merely to
protect a hit rate.

**Critical judgment:** the efficiency ratio is a design compass, not a
metered claim; no measurement in the sources establishes any threshold. The
stable claims are the lifecycle sentence (once-useful does not mean
next-turn-needed) and the reduction pattern (structured result in Hot
context, raw artifact behind a pointer). Cross-cutting Git-design reference
(owner direction): where this reduction and eviction design touches
compression, it should take the companion Git-model material (content
addressing, immutable objects, GC and packfile discipline) as design input;
see the companion
[Wheel-config and Git-model distillation](research-distillation-050-051-bitty-ai.md),
not an implementation.

## Code reading as a Code Intelligence Layer

Source: `048.md:208-271` (the layered-reading section with the viewer-size
and repo-map citations and the ctxctl level sketch).

The retained direction replaces whole-file dumping with staged cognition:
repository tree, module map, symbol map, references and callers, relevant
region, exact source, and whole-file reads only rarely. The source's
supporting citations (a viewer showing on the order of one hundred lines,
concise search matches over match-plus-wide-context, repo-map topology
before deep reads) are kept as discussion-reported observations, not as
verified findings. The ctxctl-style level sketch (tree, symbol index,
signature and docs, relevant function, local surroundings, whole file) is
kept as compatible prior art already inside this project's practice.

**Critical judgment:** the level list is vocabulary, not an adopted API; the
cited viewer sizes are anecdote without a reproduced fixture. The stable
claim is the ordering principle (topology before locality, locality before
totality) plus on-demand expansion behind references, which the Context
Pointer section below makes addressable.

## Tool ergonomics: expose intent, hide plumbing

Source: `048.md:272-426` (tool-parameter failures, MCP count effects, the
Tool Search direction) and `049.md:1506-1639` (tool activation, deferred
loading strategies, the no-universal-call rule).

The retained rule is that agent tools are redesigned for model cognition,
not wrapped one-to-one from internal APIs: narrow schemas with clear
optionality, defaults, and descriptions beat wide plumbing-exposing surfaces
whose parameter interactions invite repeated schema-error retries. The
MCP-count material is kept with its reported shape (tens of tools cited as
the region where selection accuracy degrades; one vendor combination cited
near 55K tokens of definitions) strictly as unverified discussion figures,
with the architectural consequence retained: registry plus discovery plus
ranking plus temporary activation, not connect-equals-expose-everything. The
049 activation material adds the cache-aware loading strategies (static,
deferred-native, harness dispatch, cache branch) with a small static core
(such as search, inspect, edit, run, context inspection, and task update)
and deferred capabilities for rare integrations, plus the explicit refusal
of a single universal JSON dispatcher: one static schema would cache well
but cost call accuracy, so correctness still outranks cache.

**Critical judgment:** every tool name, count, and threshold in these ranges
is illustration; no schema, catalog, or loading strategy is adopted. The
stable claims are the design rule (expose intent, hide plumbing), the
activation shape (small static core, need-driven remainder), and the
tiebreak role of cache (prefer the cached candidate only among
near-equal-value options).

## Capability discipline: Skill Hell, Resolver, and trust

Source: `048.md:427-581` (progressive disclosure, the four collision modes,
the Skill Hell sum) and `048.md:582-622` plus `048.md:1704-1776` (supply
chain, capability metadata, trust tiers).

The retained analysis accepts progressive disclosure (name plus description
always loaded, body on trigger, references and scripts on demand) as correct
but insufficient: it bounds per-skill residence without bounding a large
overlapping discovery space. The four collision modes are kept as the
working definition of Skill Hell: discovery collision (near-identical
triggers force guess-or-load-all), instruction duplication (the same
guidance enters context repeatedly), instruction conflict (contradictory
directives with no arbiter), and trigger amplification (one activation
cascades into more loads). The proposed engineering answer is a Skill
Resolver: registry-time overlap, trigger, instruction, trust, footprint, and
provenance analysis feeding task-time deduplication, ranking, and activation
of one or two candidates with overlap warnings. The trust material adds the
metadata direction (source, trust tier, permissions, token cost,
side-effects, network and executability flags), the tier sketch (built-in,
signed official, user-local, third-party and bundle downloads as
untrusted), and the per-part policy inside a skill (reading text allowed,
executing scripts and network gated, sensitive paths denied or prompted).

**Critical judgment:** the resolver is a candidate subsystem, not an adopted
algorithm; overlap scores and tier assignments are vocabulary. The scan
figures cited in the source (fractions of a public skill corpus with issues,
counts of confirmed payloads) describe that study's corpus only and are not
ecosystem prevalence claims. The stable claims are the Hell decomposition
(useful for measurement), the resolver placement (deduplicate before the
model, not through the model), and the no-implicit-trust rule (installation
never implies executability). Enforcement placement and schema stay with
[Tool transport R2](../tool-transport-r2.md) and the normative security
corpus.

## Small prompt and need-driven discovery

Source: `048.md:623-779` (over-prescriptive prompts, the map-not-manual
turn, must-use versus discoverable capabilities, thin global layers).

The retained posture keeps the core prompt to environment facts, principles,
constraints, and invariants while leaving tactics to the model: no mandatory
skill-inspection, MCP-consideration, or fixed step sequences per task. The
reported vendor turn (a large instruction file judged a failure for crowding
context, going stale, and hiding priorities, replaced by a short index over
progressively disclosed docs) is kept as discussion-reported experience,
not verified evidence. The discovery rule is kept with its slogan:
capabilities are discoverable on need (native reasoning first, discovery
only on insufficiency), never obligatory per turn; skills cover
project-specific unknowns, not general coding ability the model already
has. The thin-global plus project-delta layering (few global skills,
project-scoped additions only) is kept as compatible practice.

**Critical judgment:** the prompt sketch lines are illustration, not adopted
text; the vendor anecdote proves nothing about any threshold. The stable
claims are the ceiling argument (prompts should shrink as models strengthen,
bounded above by the brittle-workflow versus vague-constraint middle) and
the obligation inversion (usage follows demonstrated need, never inventory).

## Verification Runtime as a first-class citizen

Source: `048.md:1817-1903` (the action-to-evidence chain, language-specific
chains, project-adapter inference).

The retained model makes verification Harness capability rather than prompt
reminder: every action carries an expected outcome, a verification step, and
evidence that gates the state transition. The language-chain illustrations
(edit to format to diagnostics to targeted to broader tests to diff review)
and the project-adapter direction (manifest files implying the right
verifier family, so agents never rediscover lint, test, or typecheck
commands) are kept as candidate behavior. The primacy claim (verification
first in the optimization order) is kept as the resourcing opinion that
justifies building this runtime before further capability inventory.

**Critical judgment:** no verifier, chain, or inference rule is adopted; the
stable claim is the gating shape (done-ness is decided by evidence, not by
model self-report) plus the adapter placement (project facts imply
verification, prompts do not re-derive it).

## Structured state, lifecycle hygiene, and advanced compaction

Source: `048.md:1904-2007` (thought versus state, the structured-state
sketch, lifecycle cleanup over threshold summarize) and `049.md:992-1165`
(the four compaction modes, traceable compaction, Context Pointer).

The retained split refuses to treat the transcript as task state: goals,
status, inspected and modified files, decisions, open questions, and
verification state persist structurally while conversation stays
discardable. Compaction therefore becomes lifecycle hygiene (decisions and
evidence promoted to state and artifacts, ephemera dropped) rather than a
single threshold-triggered summarization, with repeated summarization
explicitly distrusted for information loss. Record 049 contributes the four
operating modes (continuous reduction at ingestion, continuous eviction of
re-fetchable content, checkpoint compaction at phase boundaries, emergency
compaction only near budget exhaustion), the traceability rule (a summary is
a compressed view plus evidence pointers with rehydration, never a
replacement), and the Context Pointer mechanism (addressable artifacts the
model inspects on demand instead of receiving bulk payloads up front).

**Critical judgment:** the state sketch fields are vocabulary; no retention,
schema, or compaction trigger is adopted. The stable claims are the
separation (state survives, transcript need not), the hygiene cadence
(cleanup at phase completion, not only at budget pressure), and the
provenance rule (every surviving summary names its evidence). Cross-cutting
Git-design reference (owner direction): checkpoint and reflog-style recovery
for this state should take the companion Git-model material as design input;
see the companion
[Wheel-config and Git-model distillation](research-distillation-050-051-bitty-ai.md).

## Wheel decomposition and top principle

Source: `048.md:1054-1226` (goal restatement, seven-module sketch,
Core-versus-Wheel boundary, compiler-as-view) and `048.md:2223-2351`
(closing architecture, Core substrate, top principle).

The retained decomposition places a deterministic Bitty Core (workspace,
panel, PTY, process, filesystem, environment, IPC, events, permissions,
resource accounting) under a stable API, with Wheel above composing agent
loop, planner, model routing, memory, skills and MCP, context compilation,
capability resolution, verification, multi-agent scheduling, policy, and
evaluation. Core knows neither model brands nor agent roles; Wheel owns all
of that composition, so a rewritten agent architecture need not disturb
Core. The compiler sketch is kept as the central metaphor: raw sources enter
(normalize, classify, resolve, retrieve, compress, budget, cache-plan,
provider-lower) and a per-turn Active Context leaves, with Wheel Knowledge
always a strict superset of what the model sees. Of the seven proposed
modules, the source nominates Context Compiler and Verification Runtime as
the two worth the deepest investment. The closing top principle is kept as
the author-proposed architecture sentence: at every step the agent should
see only the minimum high-quality information needed for the current
decision, with reliable feedback on every consequential action.

**Critical judgment:** the module boxes and arrows are a candidate topology,
not an adopted decomposition; no module boundary here implies any crate,
package, or team split. The stable claims are the ignorance rule (Core
never branches on providers or roles), the superset inequality (stored
knowledge exceeds shown context by design), and the investment ordering
(compiler and verification before inventory).

## Panel and Agent separation, scheduling, and provider routing

Source: `048.md:1297-1384` (terminal-substrate advantage),
`048.md:2008-2121` (lifecycle split, subagent scheduler with structured
returns), and `048.md:2122-2222` (provider abstraction, Eval table).

The retained directions are: Wheel reads structured terminal truth (panel,
process, environment, command lifecycle) instead of re-deriving it through
shell text parsing; Agent, Panel, Process, Workspace, Task, and Session are
independent entities joined by association, so agents attach, spawn
headless, release, and reattach while panels outlive any single agent; and
spawning is a scheduled, budgeted primitive weighed against coordination
cost rather than a model impulse, with subagents returning structured
conclusions plus evidence and confidence while transcripts stay in storage.
Providers are kept out of the agent architecture behind a capability profile
(window, reasoning levels, tool calling, structured output, vision, caching,
pricing, latency) with agents holding only a model reference. The Eval
direction (task success, unnecessary and failed tool calls, tokens per turn,
duplication, skill precision, retries, compactions, spawns, wall time, and
cost, compared across versions by a repeatable command) is kept as the
long-term competitiveness claim: Harness changes must move meters, not
impressions.

**Critical judgment:** every entity, field, metric, and command spelling is
vocabulary; no lifecycle machine, scheduler policy, profile schema, or eval
harness is adopted. The stable claims are the negative identities (Agent is
not Process, Panel, or Workspace), the cost inequality (parallel gain must
exceed coordination cost), and the routing indifference (agent code never
branches on provider brand). Wheel modes pointer (owner direction): headless
operation and the Panel-versus-Agent lifecycle split are recorded as Wheel
modes alongside one-shot single-question non-Agent chat; the headless and
split material is grounded in the ranges above and in record 051, while the
one-shot form is owner direction recorded as design-only in the companion
[Wheel-config and Git-model distillation](research-distillation-050-051-bitty-ai.md#wheel-modes),
which carries no implementation claim.

## Context IR and content-layout separation

Source: `049.md:128-256` (the unit struct with worked examples, the
IR-versus-string argument, provider backends).

The retained design makes the compiler emit a Context Intermediate
Representation rather than a prompt string: each unit carries identity,
kind, source, content, priority, relevance, authority, trust, freshness,
lifetime, token estimate, cacheability, stability, content hash,
dependencies, and provenance. The worked sketches (a symbol-scoped code unit
with stability-until-file-changes, a test-result unit with task-phase
lifetime bound to a panel event source) are kept as illustrations of the
field semantics. Content is separated from arrangement so that provider
backends (one per vendor family) own prefix order, cache markers, and key
semantics while the agent runtime stays brand-agnostic.

**Critical judgment:** the struct is an unreviewed sketch proposing no type,
module, or contract; every field value shown is illustration. The stable
claims are the IR-first pipeline (rank, budget, and plan over units, never
over raw strings) and the backend seam (provider specifics live behind one
interface the compiler programs, not inside agent logic).

## Stability zones and the cache tree

Source: `049.md:257-450` (Z0 through Z5 with per-zone examples, the layered
cache tree with its chained-hash sketch).

The retained stratification orders context by volatility: provider and model
contract as the most stable layer, then Wheel Core, project context, task
state, working set, and current turn as the most dynamic. Each zone's
examples are kept (core prompt and invariants; project instructions and
conventions; goal, constraints, decisions, modified and open items; symbols,
diagnostics, and diffs; latest user message and fresh tool results) with
their stated lifetimes (version-scale, checkout-scale, tens-of-turns,
per-turn). The cache tree follows the same layering (core, then project,
then task checkpoint, then dynamic suffix) with a chained-hash sketch so a
working-set change invalidates only the suffix while a project change
invalidates everything below it, mirroring prefix-cache behavior.

**Critical judgment:** zone boundaries and hash mechanics are candidate
design, not adopted versioning; the source's zone count wording and the
sketch differ cosmetically and neither is normative. The stable claims are
the ordering (volatility increases down the stack) and the invalidation
direction (a change invalidates its layer and below, never above).
Cross-cutting Git-design reference (owner direction): the chained,
content-derived layering should take the companion object-model material
(content addressing, immutable layers) as design input.

## Serialization stability and dynamic placement

Source: `049.md:451-604` (canonical serializer, timestamp and ordering
hazards, stable-prefix layering with the position-volatility relation).

The retained rule is that the compiler owns a canonical serializer: stable
tool ordering, stable property order and whitespace and enum forms, and
fixed section order, because nondeterminism as small as key order or a
leading timestamp can collapse prefix reuse. Dynamic content (time, request
identifiers, temporary paths, live status, budgets-as-text) is banished from
the stable prefix into trailing snapshots, task state, and runtime sections,
following the position rule that volatility belongs late and stability
belongs early.

**Critical judgment:** no serializer, ordering, or section list is adopted;
the hazard examples are illustrations of a real failure class. The stable
claims are the ownership (one canonical serializer gates every prompt byte)
and the placement inequality (prompt position tracks inverse volatility).

## Admission scoring and authority resolution

Source: `049.md:662-772` (the multiplicative score with additive noise and
cost penalties, the small cache tiebreak, the authority ladder with
conflict eviction).

The retained scoring treats relevance, importance, freshness, and authority
multiplicatively against noise and token-cost penalties, with cache affinity
as a deliberately small additive tiebreak: among near-equal candidates the
cached one wins, but cache never selects content. The authority ladder
(system policy, current user instruction, explicit project rules, task
decisions, retrieved documentation, old summaries, model-generated notes) is
kept with its conflict rule: on contradiction the higher-authority item
survives and the lower is evicted before the model ever arbitrates.

**Critical judgment:** the formula is a thinking tool, not a calibrated
model; weights, scales, and the tiebreak constant are all open. The stable
claims are the tiebreak ceiling (cache affinity stays epsilon-small) and the
pre-resolution rule (the compiler settles authority conflicts; the model
does not receive both sides as a judgment call).

## Content-addressed code and deduplication

Source: `049.md:773-892` (code artifacts with hash and revision, the STALE
rule, the four-source fmt-rule merge with provenance).

The retained rule binds every code unit to path, symbol, range, file hash,
and optional revision: on mismatch the unit flips STALE and is barred from
Hot context, defeating the classic failure of reasoning from a
three-turns-ago file version. Duplicated guidance across instructions,
skills, docs, and notes is merged into one derived unit (such as a single
project formatting invariant) that keeps its full source list, cutting
semantic duplication while preserving auditability through a
value-plus-sources shape.

**Critical judgment:** the artifact shape and merge behavior are candidate
design; no hash function, key, or store is adopted. The stable claims are
the staleness bar (hash mismatch excludes, never warns-and-keeps) and the
provenance retention (merges name every source). Cross-cutting Git-design
reference (owner direction): artifact identity and dedupe should take the
companion content-addressed object material as design input.

## Tool-result reduction pipeline

Source: `049.md:893-991` (deterministic, structural, and semantic levels
with the cargo, typecheck, and diff illustrations).

The retained pipeline reduces tool output in three levels before the model
pays attention: deterministic scrubbing without any model call (escape
sequences, progress rendering, repetition, timestamps), structural parsing
without any model call (build and test output to counts plus named failures
plus diagnostics, typecheck output to located errors, diffs to file counts
before hunks), and model-backed semantic compaction reserved for genuinely
unstructured material (long discussions, exploration, large documents,
research results).

**Critical judgment:** no parser, schema, or threshold is adopted; the token
counts shown are illustrations. The stable claim is the level order
(deterministic first, structural second, model-backed last) with model calls
as the scarce last resort.

## Pass pipeline, budgets, and quotas

Source: `049.md:1166-1349` (the twelve-pass pipeline with telemetry
feedback, the soft-hard-emergency budget triple, the Pinned-Flexible-Reserve
quota split).

The retained pipeline runs ingestion, normalization, classification,
staleness check, candidate retrieval, authority resolution, deduplication,
reduction, relevance ranking, token budgeting, cache planning, and provider
lowering, with post-request telemetry (effectiveness, hit outcome, tool
success) feeding the next turn. Budgets are deliberately decoupled from the
model window (soft active target, hard active cap, emergency limit, with
small-start growth tuned by evaluation rather than hardcoded bands), and the
active budget is split into Pinned (instructions, intent, invariants,
safety), Flexible (code, docs, diagnostics, conversation, tools competing
for the remainder), and Reserve (room for the next tool result, model
output, and reasoning, so reads can never starve the turn's own work).

**Critical judgment:** the pass list is a candidate pipeline, not an adopted
control flow; every number attached to budgets is illustration. The stable
claims are the feedback closure (telemetry revises future compiles) and the
reserve invariant (a turn always holds room for its own output).

## Cache kinds, stable prefix, and provider capability

Source: `049.md:1350-1777` (four cache kinds, the prefix-hit illustration,
tool activation, the capability struct, reasoning namespace, versioning).

The retained taxonomy keeps four distinct caches: provider prompt and KV
cache (latency and cost), artifact cache (parses, indexes, maps, summaries
keyed by content hash), retrieval cache (query results keyed by revision,
query semantics, and index version), and compaction cache (event ranges
already compacted to a checkpoint, reused while inputs are unchanged).
Prefix stability is kept as the provider-cache lever (stable core, tools,
and project layers up front so later dynamic layers miss narrowly rather
than globally), with tool activation handled by deferred strategies so a
newly discovered tool does not rewrite the cached prefix. Provider cache
differences are abstracted behind a capability record (mode, prefix basis,
breakpoint count, explicit and implicit support, prewarm, TTLs,
invalidation rules) maintained by each provider plugin, and cache identity
further includes provider, model, protocol, reasoning profile, toolset, and
prompt versions, with explicit compiler versioning so a one-line prompt
change explains its own invalidation instead of surfacing as a mystery hit
drop.

**Critical judgment:** every struct field, key shape, and hit-rate figure is
unreviewed illustration; no cache contract or namespace is adopted. The
stable claims are the four-way split (one cache story per concern, never a
single boolean), the prefix discipline (stable-first ordering), and the
capability seam (the compiler knows capabilities, never brands).

## Provider observability as a design pointer

Source: `049.md:1778-1898` (the observability section with its statusline
and three display sketches, plus the Context Trace section).

The retained direction gives the compiler strong observability: a compact
statusline (active tokens against budget, cache share, hot, stable, and
dynamic splits), an inspectable context breakdown (per-layer token
attribution from core prompt through current turn), a cache view
(attributable reusable prefix, cached bytes, hit rate, miss causes), a
why-view (per-unit relevance, referencing diagnostic, containing symbol,
source revision), and a per-compile trace (added, reduced, evicted, and
hit-marked units with reasons) so a bad turn is debuggable as what the model
saw, why each unit was chosen, what was compacted away, and why the cache
missed.

Provider observability (owner direction): the provider side should expose
cost, cache-hit, input, context, and timing signals for future plugins, with
git-style traceability (`git log --stat` and `git blame` as the UX
reference) as the interaction model for attributing context cost and cache
behavior to decisions over time. This is explicitly a design pointer
recorded here: **no `bitty-ai` code change happens in this task**, and a
future `AI-XXXX` task implements it against a real HTTP or Router adapter.
No endpoint, field, metric name, or display in the source sketches is
adopted; the sketches stay discussion vocabulary for that future task.

**Critical judgment:** the display contents, field names, and trace syntax
are illustrations, not accepted views; the stable claim is the debuggability
bar (every admission, eviction, reduction, and miss is attributable after
the fact).

## Worked example and subsystem split

Source: `049.md:1899-2135` (the panel-process-leak walkthrough, the
store-index-compiler-cache-backend module sketch, the closing dataflow).

The retained walkthrough shows the intended steady state: a bug task
compiles to a small stable-plus-task-plus-just-in-time first view; search
returns symbols rather than transcripts; edits flip old code versions STALE
on hash change; a multi-thousand-token test log persists in the event store
while Hot context keeps on the order of a hundred structured tokens behind
a pointer; raw generated information grows into the hundreds of thousands
while Hot context holds steady in the tens of thousands with most of the
provider request served from cached prefix. The five-subsystem split (store
with events, artifacts, and checkpoints; index with code, docs, and
semantic; compiler with retrieval, ranking, dedup, reduction, budgeting, and
authority; cache with artifact, retrieval, compaction, and provider; backend
with one vendor module each) is kept as the candidate module map, with the
load-bearing dataflow that full session state never reaches the model
directly but only through the compiler's retrieve, reduce, and cache-plan
passes into IR and then a provider backend.

**Critical judgment:** the walkthrough numbers are narrative illustration,
not measured evidence; the module paths are a candidate layout, not adopted
packaging. The stable claims are the steady-state inequality (stored
knowledge grows while Active Context stays small and high-signal) and the
anti-decay thesis (session length must enrich storage, never dirty the
view). The closing architectural sentence is kept as author opinion: the
Harness manages a growing session knowledge graph and compiles per-turn
views, with threshold compaction demoted to one optimization pass among
many.

## Sections read for boundary accuracy

Both verified files were distilled in full; the ranges below were read so
that non-`bitty-ai` content is not silently absorbed.

- `048.md:1-30` (owner question block: Codex must-check behavior, 1M-window
  skepticism, whole-file reading, tool-argument failures, skill minimalism):
  boundary context; motivates the synthesis, and its `.agents` directory
  habit is compatible with, but not normative for, the companion 050-051
  split.
- `048.md:1035-1051` and `049.md:2129-2135` (reference lists): read as
  citation provenance only; no cited claim is reproduced as a finding.
- `048.md:1297-1384` terminal-substrate specifics (panel, PTY, process,
  environment reads): the structural advantage is distilled above; any
  terminal-side mechanism stays `bitty`-side and proposes no Core API.
- `049.md` provider-behavior passages (per-vendor cache, breakpoint, and
  reasoning-configuration notes): read as reported vendor semantics the task
  did not verify; only the capability-abstraction seam is distilled.

## Source coverage and critical disposition

Retain means retain as a **proposal**, not accept as normative. Improve means
retain the objective with the correction above. Reject means reject that
mechanism or absolute claim; defer means no commitment pending named
evidence.

| Source lines  | Topic                                                                    | Disposition, rationale, and alternative                                                                           |
| ------------- | ------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------- |
| 048:1-30      | Owner questions; Codex behavior; skill-minimal habit                     | Boundary context; motivates the synthesis; directory habit is compatible context for the 050-051 split.           |
| 048:31-77     | Multiplicative quality formula; factor table; verification primacy       | Retain framing and bottleneck argument; formula is a device, not a model; factors need Eval grounding.            |
| 048:78-145    | Window versus attention; efficiency ratio; cited studies                 | Retain efficiency objective; citations are unverified discussion reports; no threshold adopted.                   |
| 048:146-207   | Pollution catalog; clearing direction; cargo reduction                   | Retain lifecycle sentence and reduction pattern; counts are illustration; Git lens is design input via companion. |
| 048:208-271   | Layered code reading; viewer and map citations; level sketch             | Retain topology-first order; sizes are anecdote; levels are vocabulary, not API.                                  |
| 048:272-361   | Tool-parameter failures; intent-hiding rule                              | Retain expose-intent rule; schema sketches are illustration; transport stays with R2.                             |
| 048:362-426   | MCP counts; selection degradation; Tool Search direction                 | Retain activation shape; counts and thresholds are unverified figures; defer catalog design.                      |
| 048:427-581   | Progressive disclosure limits; four collisions; Skill Hell sum; Resolver | Retain Hell decomposition and resolver placement; scores and overlap figures are illustration.                    |
| 048:582-622   | Supply-chain figures; poisoning reports; metadata and tiers              | Retain no-implicit-trust rule; figures describe one corpus only; enforcement needs security review.               |
| 048:623-715   | Prompt over-specification; map-not-manual; vendor anecdote               | Retain middle-height posture; anecdote is discussion report; no prompt text adopted.                              |
| 048:716-779   | Must-use versus discoverable; thin layers                                | Retain need-driven rule; sketches are illustration; layering is compatible practice.                              |
| 048:780-866   | Threshold-summarize critique; lifecycle classes; hygiene                 | Retain hygiene cadence; classes are vocabulary; repeated summarization distrusted.                                |
| 048:867-924   | Subagent costs; parallel-gain inequality                                 | Retain cost inequality; numbers are illustration; scheduling stays with coordination dispositions.                |
| 048:925-963   | Ten-item order; Harness slogan and diagram                               | Retain resourcing opinion and slogan; order shifts by task; needs Eval grounding.                                 |
| 048:1035-1051 | Reference list [1]-[16]                                                  | Provenance only; no cited claim reproduced as a finding.                                                          |
| 048:1054-1078 | Wheel goal; seven-module sketch; compiler-plus-verification priority     | Retain goal and investment order; modules are candidate topology, not packaging.                                  |
| 048:1079-1124 | Core-versus-Wheel boundary diagram                                       | Retain ignorance rule; boundary implies no crate or team split.                                                   |
| 048:1125-1226 | Compiler-as-view; append anti-pattern; knowledge superset                | Retain per-turn-view metaphor and superset inequality; passes live in the 049 sections.                           |
| 048:1227-1296 | Lifecycle state model; cargo-state example                               | Retain separation and hygiene direction; sketch fields are vocabulary.                                            |
| 048:1297-1384 | Terminal-substrate advantage; structured reads                           | Retain association reads; mechanisms stay `bitty`-side; no Core API proposed.                                     |
| 048:1385-1474 | Code Intelligence Layer proposal                                         | Retain staged-cognition direction; layer names are vocabulary; API shape open.                                    |
| 048:1475-1559 | Tools versus APIs; narrow surfaces                                       | Retain intent-hiding rule; operation lists are illustration.                                                      |
| 048:1560-1638 | Capability Registry versus Active Set                                    | Retain small-active-set shape; counts are illustration; discovery stays open.                                     |
| 048:1639-1703 | Skill Resolver with overlap and ranking                                  | Retain resolver as candidate subsystem; algorithm and scores open.                                                |
| 048:1704-1776 | Trust metadata and tiers                                                 | Retain tiered no-implicit-trust direction; metadata shape needs security review.                                  |
| 048:1777-1816 | Tiny core prompt sketch                                                  | Retain shrink-with-strength posture; sketch lines are illustration, not adopted text.                             |
| 048:1817-1903 | Verification Runtime chains; adapter inference                           | Retain evidence-gating shape; chains and inference rules open.                                                    |
| 048:1904-1959 | Thought-state split; state sketch                                        | Retain discardable-transcript rule; fields are vocabulary; retention open.                                        |
| 048:1960-2007 | Advanced compaction over threshold summarize                             | Retain promotion-to-state direction; triggers stay open.                                                          |
| 048:2008-2062 | Panel and Agent lifecycle split; headless panels                         | Retain independence direction; lifecycle machine is RFC-owned discussion input; modes in companion.               |
| 048:2063-2121 | Subagent scheduler; structured returns                                   | Retain budgeted-scheduling direction; policies and schemas open.                                                  |
| 048:2122-2167 | Provider abstraction; capability profile                                 | Retain brand-indifference rule; profile fields are vocabulary; transport stays with R2.                           |
| 048:2168-2222 | Eval metrics; version comparison                                         | Retain meter-not-impression rule; metrics and command spelling are vocabulary.                                    |
| 048:2223-2351 | Closing architecture; Core substrate; top principle; refs                | Retain ignorance, superset, and minimum-information claims as candidate; topology not packaging.                  |
| 049:1-47      | Compiler framing; Storage versus Active versus Cache                     | Retain three-way split as the design basis; compiler-over-concatenation metaphor kept.                            |
| 049:48-127    | Cold, Warm, Hot worlds; subset relation                                  | Retain stratification and Hot-selects-from-Universe framing; examples are illustration.                           |
| 049:128-196   | Context IR struct; worked unit examples                                  | Retain IR-first direction; struct and values are unreviewed sketches.                                             |
| 049:197-256   | Content-layout split; per-vendor backends                                | Retain backend seam; backend names are vocabulary; brands stay behind the seam.                                   |
| 049:257-373   | Six stability zones with examples and lifetimes                          | Retain volatility ordering; zone cuts and lifetimes are candidate, not normative.                                 |
| 049:374-450   | Cache tree; chained-hash sketch                                          | Retain layered-invalidation direction; hash mechanics are illustration.                                           |
| 049:451-525   | Canonical serializer; ordering hazards                                   | Retain single-serializer ownership; lists are illustration of a failure class.                                    |
| 049:526-604   | Dynamic-out-of-prefix; position rule                                     | Retain inverse-volatility placement; section lists are illustration.                                              |
| 049:605-661   | Utility formula; correctness-first ordering                              | Retain priority guardrail; formula is a device; stale retention for hits rejected.                                |
| 049:662-715   | Admission score; small cache tiebreak                                    | Retain tiebreak ceiling; formula is a thinking tool; weights open.                                                |
| 049:716-772   | Authority ladder; conflict eviction                                      | Retain pre-resolution rule; ladder order is candidate; eviction before model arbitration.                         |
| 049:773-839   | Content-addressed code; STALE bar                                        | Retain hash-mismatch exclusion; artifact shape and hash choice open; Git lens via companion.                      |
| 049:840-892   | Dedup merge with provenance                                              | Retain value-plus-sources shape; merge behavior is candidate.                                                     |
| 049:893-991   | Three-level tool-result reduction                                        | Retain level order with model-last; parsers and counts are illustration.                                          |
| 049:992-1071  | Four compaction modes                                                    | Retain mode split with emergency-last; triggers stay open.                                                        |
| 049:1072-1123 | Traceable compaction; rehydration                                        | Retain view-plus-pointers rule; syntax is illustration.                                                           |
| 049:1124-1165 | Context Pointer addressability                                           | Retain on-demand inspection direction; call shapes are vocabulary.                                                |
| 049:1166-1242 | Twelve-pass pipeline; telemetry feedback                                 | Retain feedback closure; pass list is candidate control flow.                                                     |
| 049:1243-1294 | Budgets decoupled from window; small-start                               | Retain decoupling and reserve reasoning; bands are illustration needing eval.                                     |
| 049:1295-1349 | Pinned, Flexible, Reserve quotas                                         | Retain reserve invariant; quota contents are illustration.                                                        |
| 049:1350-1461 | Four cache kinds with keys                                               | Retain four-way split; key shapes and hit figures are illustration.                                               |
| 049:1462-1505 | Stable-prefix illustration; 80-percent hope                              | Retain stable-first discipline; ratio is narrative, not evidence.                                                 |
| 049:1506-1574 | Tool activation; deferred strategies                                     | Retain deferred-loading direction; strategy names are vocabulary.                                                 |
| 049:1575-1639 | No universal dispatcher; static core plus deferred                       | Retain correctness-over-cache rule; tool lists are illustration.                                                  |
| 049:1640-1694 | Provider cache capability record                                         | Retain capability seam; fields are unreviewed vocabulary.                                                         |
| 049:1695-1748 | Reasoning and tool-choice namespace                                      | Retain namespace inclusion; field list is illustration.                                                           |
| 049:1749-1777 | Compiler versioning                                                      | Retain self-explaining-invalidation direction; version strings are illustration.                                  |
| 049:1778-1856 | Observability displays; statusline and views                             | Retain debuggability bar; displays are illustration; provider pointer is owner direction.                         |
| 049:1858-1898 | Context Trace syntax                                                     | Retain attribution direction; syntax is illustration.                                                             |
| 049:1899-2032 | Worked leak-task walkthrough                                             | Retain steady-state inequality; numbers are narrative illustration.                                               |
| 049:2034-2135 | Five-subsystem split; dataflow; refs                                     | Retain never-direct dataflow; module paths are candidate layout, not packaging.                                   |

## Proposed validation and promotion path

These are future evidence requirements, not tests executed by this
documentation task. They keep the design falsifiable before any
quality-formula or compiler proposal constrains `bitty-ai`.

| Campaign         | Required observation                                                                                                                                                |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Quality product  | A Harness change that zeroes one factor (verification, relevance, ergonomics) is shown to move task success in replayed-trace evaluation before any window claim.   |
| Efficiency       | A reduced high-signal context beats a larger diluted one on success, latency, and cost for the same task family in reviewable fixtures.                             |
| Pollution        | A named pollution class is removed from Hot context with its raw artifact retained and re-fetchable, and success does not regress in reviewable tests.              |
| Tool ergonomics  | A narrowed schema or renamed parameter reduces call-failure and retry rates in logged tool telemetry before any catalog change.                                     |
| Discovery        | A Resolver change improves skill precision (fewer wrong activations, fewer total loads) without regressing success in reviewable evaluation.                        |
| Trust            | An untrusted skill or MCP exercises exactly its grant, with script execution and sensitive reads gated or denied in reviewable tests.                               |
| Verification     | A completion gate rejects a plausible-but-wrong change that the model self-reported as done, in reviewable tests.                                                   |
| State hygiene    | Phase-boundary cleanup preserves decisions and evidence while dropping ephemera, with rehydration succeeding from pointers in reviewable tests.                     |
| Cache discipline | A stale unit is excluded on hash mismatch, and a relevance eviction is never vetoed to protect a hit rate, in reviewable tests.                                     |
| Budgets          | Turns hold Pinned guarantees and Reserve room under adversarial reads, with soft, hard, and emergency behavior observable in reviewable tests.                      |
| Observability    | A bad turn is attributable to admissions, evictions, reductions, and miss causes from trace and views alone, without model self-report.                             |
| Provider pointer | A future adapter task demonstrates cost, cache-hit, input, context, and timing signals flowing to a plugin surface with git-style attribution, with no Core change. |

Promotion needs independent AI architecture, context-management, terminal
and plugin-owner, docs-curator, and security review. Route struct shapes,
formulas, zone cuts, pipeline order, budget bands, namespace fields, metric
names, display formats, and the provider-signal contract to scoped owner
tasks. This draft changes no normative contract and authorizes no product
code.
