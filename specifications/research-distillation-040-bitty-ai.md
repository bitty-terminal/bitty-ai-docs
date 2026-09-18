---
title: Plugin-system research distillation for bitty-ai (040)
description: Draft bitty-ai distillation of research 040 plugin topics extension model host plugin registries manifest permissions versioning layer split
category: specifications
audience: mixed
document_type: research
status: draft
website_publish: false
sidebar_order: 52
---

# Plugin-system research distillation for bitty-ai (040)

## Status and recommendation

This is a **draft discussion synthesis**, not a runtime specification, accepted
decision, dependency selection, release commitment, or implementation claim.
It distills only the `bitty-ai`-relevant parts of workspace research record
`040.md` (a plugin-system discussion): the two-level extension model, Bitty AI
as a Host Plugin, Bitty AI as a plugin-platform plugin (registries, Agent UI,
extension catalog, slash commands as extensions), Extension Points, Manifest
relations, permission non-inheritance, and Service API versioning as they
apply to `bitty-ai`, plus the proposed two-layer split (`bitty-ai-runtime`
versus the `bitty-ai` application) and the three-layer model. Editor,
statusline, Docker, media, search, browser, and developer-runtime material is
`bitty`-side or plugin-ecosystem matter and is excluded; it is recorded as
handoff input, not as a decision.

The recommendation is to treat every layered model below as a candidate input
to the draft [AI Architecture](../architecture/ai-architecture.md) and its related draft
dispositions, never as an override of the accepted
[IPC and Agent RFC](ipc-agent-rfc.md) or the normative security corpus.
In particular, the two-layer split, the three-layer model, and the four-level
extension ladder are **not** accepted by this distillation. Nothing here is
promoted to accepted status, and no implementation is described as shipped.

The source's strongest ideas are the two-level extension model (generic host
primitives below, domain runtimes above, domain extensions on top), the
permission non-inheritance rule (a dependency never confers capabilities),
versioned service APIs per extension domain, and the minimal-base principle
(AI machinery ships only when the user opts into AI). Its weakest claims are
the concrete registry, manifest, and version-string sketches, which are
unreviewed interface proposals, and the assumed clean split between a native
runtime and a framework plugin, which has no ownership, packaging, or
compatibility evidence behind it. Those are corrected below.

This synthesis references, without duplicating or modifying, the draft
[AI Architecture](../architecture/ai-architecture.md) (candidate inputs only),
[Provider plugin boundary](../providers/provider-plugin-boundary.md),
[Panel environment awareness](../interfaces/panel-environment-awareness.md),
[Execution ownership R1](../architecture/execution-ownership-r1.md), and
[Tool transport R2](../architecture/tool-transport-r2.md). The accepted
[IPC and Agent RFC](ipc-agent-rfc.md) is unaffected by this draft. This
document creates no AIQ or OQ identifier and closes none.

## Provenance and evidence boundary

The source is research note `040` (origin), read on 2026-09-16: **2,215
lines**, **42,579 bytes**. Its SHA-256 is
`a0c714f16b2db8e48e078d61a570ab01a8c0e60edee14b118da96901fe0a234b`.
The record is untracked in the research repository, so provenance is by record
number plus fingerprint, not by commit. It was not renamed, edited, or staged by
this task; the `bitty`-side pass still needs it. The body of this document
distills the fingerprinted head (`040.md:1-1829`); the addendum at the end
distills the bitty-ai-relevant parts of the appended tail (`040.md:1830-2215`).
The fingerprinted head (`067e3c287b203ccd9a3217c1596d55cb7181b4746d07076e1023e1ddfc2cfe0e`)
keeps verifying even after further appends.

Unlike record 039, record 040 was a single pass with distinct sections at
verification time; no duplication handling applies. Adjacent boundary sections `040.md:1131-1294`
(plugin tree and the native-versus-extension axis), `040.md:1365-1505`
(non-AI native capabilities and the browser analogy), and
`040.md:1698-1829` (native ABI caution and the four-level ladder) were read
so that exclusions are accurate, but they are **not** distilled here. Only
the ranges in the coverage table were distilled. The source is a
single-author Chinese-language discussion; this document is the
English-language draft synthesis, not a translation. The fingerprint
identifies the discussion, not the truth of its claims.

### Post-verification append (distilled in the addendum below)

After the head verification above, the source file grew by a pure
append: at 2026-09-16 the file held 2,215 lines and 42,579 bytes, while the
first 1,829 lines (31,779 bytes) still hash to exactly the head SHA-256
above, and spot-checked anchors (`040.md:138`, `229`, `402`, `437`) are
unchanged. Line 1830 still opens the appended comparable-programs round
(Emacs, VS Code, JupyterLab, Eclipse RCP, Neovim, Zed, and adjacent
references), which now runs to the current end-of-file at line 2215: 386
lines in total. The file has not grown since that append was recorded, so
the addendum below distills the whole tail and no newer remainder is left
uncovered. The whole-file hash in the provenance block above therefore
matches the current file, while every head range cited in the body still
verifies against the head fingerprint.

## Authority and reconciliation

The draft [AI Architecture](../architecture/ai-architecture.md) layered models are
candidate inputs only; the two-layer split, three-layer model, and four-level
ladder proposed in the source are **not** accepted by this distillation and
must not be read as crate, package, or release decisions. Provider contract
and transport questions stay with the draft
[Provider plugin boundary](../providers/provider-plugin-boundary.md); execution and
environment questions stay with [Execution ownership R1](../architecture/execution-ownership-r1.md)
and [Panel environment awareness](../interfaces/panel-environment-awareness.md); tool
authorization and transport placement stay with
[Tool transport R2](../architecture/tool-transport-r2.md). Each is referenced, never
duplicated or modified.

Normative security obligations (least privilege, per-action scopes,
capability-based and auditable permission that fails closed, typed redaction,
consented recording, secret minimization) override every discussion example
below. The source's registry names (`Tool Registry`, `Context Registry`,
`Model Registry`, and similar), extension-point names, manifest fields, and
version strings are conceptual vocabulary, not additions to any accepted
registry or schema. This draft creates or closes no AIQ or OQ identifier;
open questions stay with
[AI Unresolved Questions](../product/ai-unresolved-questions.md) and shared
governance.

## Boundary context: where AI sits in the discussed models

Source: `040.md:138` (AI Activity among panel activities), `040.md:229`
(AI in the development-environment composition), `040.md:244-254` (AI
workspace composition: Agent, Tasks, Terminal, Diff, Browser, Logs),
`040.md:309` (Bitty AI as an application grown on the platform),
`040.md:324` (Bitty AI in the applications grid), `040.md:377` (AI as a
first-class runtime child beside Terminal, Editor, Docker, Git, and Mail).

These placement lines are boundary context only. Their retained consequence
for `bitty-ai` is positional: in every model the source draws, AI appears as
an application or runtime peer, never as Core. Core is credited with knowing
only generic primitives (panel, widget, command, service, event, task,
storage, capability) while remaining ignorant of AI specifics. That matches
the standing direction that terminal, panel, process, rendering, and
transport mechanisms stay terminal-side while model, context policy, agent
state, tools, and permissions stay `bitty-ai`-side.

**Critical judgment:** none of these lines specifies an interface. The
"Core knows primitives, not applications" principle is retained as a design
objective; the concrete primitive set is undecided and stays with the owning
repositories.

## Motivating question and the two-level extension model

Source: `040.md:402` (motivating question: whether agent-related plugins can
themselves be extended by further plugins for non-core functionality) and
`040.md:437` (two-level extension model: Bitty provides generic primitives
plus service, capability, and event mechanisms; `bitty-ai` builds an
AI-specific runtime on top; further AI plugins extend `bitty-ai`).

The retained model is:

```text
Bitty Core (generic primitives, services, capabilities, events)
  -> bitty-ai (AI-specific runtime)
    -> AI extensions (providers, context sources, memory, commands)
```

Extensions of `bitty-ai` extend AI-domain capability, never Bitty Core
capability. A provider, memory backend, or review command plugs into the AI
runtime's registries; it does not gain terminal, panel, or process authority
by virtue of participating in the AI ecosystem. This is the same pattern the
source draws for the editor domain, and the symmetry is deliberate: each
domain runtime mediates its own extensions.

**Critical judgment:** the model is a candidate layering, not an accepted
crate or process topology. Whether the AI runtime is a library, a process, a
plugin, or a combination remains open; the stable claim is only the
mediation direction (extensions attach to the domain runtime, not to Core).

## Bitty AI as a Host Plugin

Source: `040.md:460-477` (Host Plugin tree: Bitty AI hosting provider,
memory, protocol, and review extensions; editor, statusline, and Docker hosts
as parallels; "Host Plugin" definition: itself a Bitty plugin and
simultaneously a host for other plugins).

The retained direction is that Bitty AI is both a plugin of the terminal
platform and a platform for AI extensions. The source's example tree names
provider extensions (OpenAI, Anthropic), memory, a device or protocol
extension, and review tooling as children of the Bitty AI host. Runtime
management stays flat: all plugins remain peer components under one plugin
runtime even when their extension relationships form a tree, so the "plugin
tree" is a dependency and extension graph, not a process hierarchy.

**Critical judgment:** the example child list is illustrative, not a
roadmap. The stable claims are the dual role (plugin and host) and the
flat-runtime reading, which keeps extension relationships from becoming
ambient authority paths. Concretely, a review or memory extension of
`bitty-ai` must pass the same R2 gate order as any tool or provider effect;
hosting confers no bypass.

## Bitty AI as a plugin-platform plugin

Source: `040.md:706-778` (Agent Runtime plus conversation; Tool, Context,
Model, Memory, and Compaction registries plus Agent UI; extension catalog of
providers, context sources, memory backends, and agent workflows; slash
commands such as review, plan, task, compact, and loop contributed as
extensions; a deliberately small `bitty-ai` core).

The retained candidate decomposition is:

- **Agent Runtime and conversation state** as the domain core: scheduling,
  turn structure, and conversation records owned by the AI runtime.
- **Registries** as the extension seams: separate registration points for
  tools, context sources, models, memory backends, and compaction
  strategies, so each category evolves and reviews independently.
- **Agent UI** as a consumer surface: chat, panels, commands, settings, and
  workflows rendered through host panels, not through `bitty-ai`-owned
  windowing.
- **Slash commands as extensions**: review, plan, task, compact, and loop
  style commands contributed by extensions rather than built into the core,
  keeping the core small and making each command separately reviewable,
  permissioned, and removable.

**Critical judgment:** every registry name, catalog entry, and command name
is a proposal. The stable claim is the decomposition principle (small core,
typed seams, contributed surface behavior) together with its review
consequence: each extension is authorized, budgeted, and revocable on its
own, and removing an extension removes its commands without touching the
core. Transport placement of these extensions (native versus MCP) stays with
R2; provider contract shape stays with the provider boundary draft.

## Extension Points as they apply to bitty-ai

Source: `040.md:779-851` (Extension Point concept beyond commands, events,
services, UI, and keymaps; Bitty AI declarations including model, tool,
context, memory, compactor, agent, command, and UI points; editor,
statusline, and Docker points as parallels; composition of extension graphs
rather than mere file loading).

The retained direction is that `bitty-ai` declares a fixed set of typed
extension points, one per capability category, and extensions attach only
through declared points. The source's declaration sketch names model, tool,
context, memory, compactor, agent, command, and UI points for the AI domain.
The consequence is compositional: installing AI extensions composes an
extension graph (which points are filled, by which reviewed extensions, at
which versions) rather than merely loading files, and the plugin manager can
reason about that graph for dependency resolution, conflict detection, and
removal.

**Critical judgment:** the point names and their exact arity are proposals,
not an accepted schema. The stable claim is typed attachment: no extension
contributes capability outside a declared point, and undeclared integration
(such as a memory backend reaching directly into provider transport) is out
of scope for this model. Point definitions, once drafted, need security
review as capability-bearing interfaces.

## Manifest relations

Source: `040.md:852-911` (manifest sketches: plugin name and version,
dependency on the host plugin with a version predicate, contributed
extension points; manager-level dependency-tree resolution on install).

The retained direction is dependency honesty in metadata: an AI extension
declares its host (`bitty-ai` with a version predicate) and the extension
points it contributes, so the plugin manager knows the extension cannot work
without its host and can resolve, verify, or refuse the dependency tree at
install time. The source's TOML sketches are illustrative shapes, not a
schema.

**Critical judgment:** no manifest field, version predicate syntax, or
resolution algorithm is adopted here. The stable claim is only that
host-dependence is declared and machine-checkable before execution, and that
declaration never doubles as a permission grant (see the next section).

## Permission non-inheritance, especially for Bitty AI

Source: `040.md:912-998` (non-inheritance principle with editor-domain
examples: theme extensions need only theme capability, language extensions
need scoped process and filesystem rights; dependency must not become
capability escalation; Bitty AI emphasis at `040.md:977-995`: the AI host may
hold filesystem, terminal, process, network, and agent capabilities while its
extensions default to none of them).

This is the load-bearing rule of the whole distillation and is retained
verbatim as a draft principle: **dependency is not authority**. A plugin that
depends on, extends, or is hosted by `bitty-ai` inherits no capability from
that relationship. Each extension is granted exactly the capabilities its own
reviewed function needs (a theme-like extension needs only its presentation
point; a context provider needs only its scoped reads; a tool extension
needs only its declared effect class), attenuated from and never exceeding
the host's own grants. A malicious or compromised extension that merely
declares a dependency on a powerful host gains nothing.

For Bitty AI the rule bites hardest because the host is drawn with broad
powers (filesystem, terminal, process, network, agent execution). The draft
consequence is a default-deny posture for AI extensions: no inherited
filesystem, process, network, or agent-execution rights; every such right is
an explicit, separately consented grant behind the R2 unified authorization
backend, with typed redaction before context entry per the standing privacy
baseline.

**Critical judgment:** this principle aligns with, and does not relax, the
normative least-privilege and fail-closed obligations. It still needs a
capability vocabulary, a grant and attenuation mechanism, and enforcement
evidence before any extension exercises a sensitive effect.

## Service API versioning for bitty-ai surfaces

Source: `040.md:999-1062` (versioned service APIs per extension domain, with
`bitty-ai.tools@1` and `bitty-ai.context@2` style sketches; host-internal
refactoring decoupled from ecosystem stability; operating-system library
layering analogy).

The retained direction is per-domain versioned contracts: once `bitty-ai`
exposes a tools or context surface to extensions, that surface carries a
version independent of both the host implementation and sibling surfaces, so
internal refactoring (buffers, documents, views, cursors, or their AI-side
equivalents) does not break the extension ecosystem. The version strings in
the source are illustrative, not an adopted scheme.

**Critical judgment:** versioning policy (compatibility windows, deprecation,
migration, coexistence of major versions) is entirely open. The stable claim
is only the decoupling objective, plus its process consequence: introducing a
versioned surface is a cross-ecosystem promise that needs owner review, not a
unilateral host decision, and experimental surfaces should stay unversioned
until that review happens.

## Two-layer split and the three-layer model

Source: `040.md:1295-1364` (proposed split into a native `bitty-ai-runtime`
for provider transport, streaming, tool execution runtime, agent scheduling,
context engine, token accounting, MCP transport, process integration, and
high-performance storage primitives, versus a `bitty-ai` application or
plugin for chat UI, agent panel, commands, settings, workflows, and prompt
configuration; upward extension tree; `Native Primitive -> Framework Plugin
-> Extension Plugin` layering).

The retained candidate allocation is:

| Layer            | Candidate contents from the source                                                                                                                          |
| ---------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Native primitive | Provider transport, streaming, tool execution runtime, scheduling, context engine, token accounting, MCP transport, process integration, storage primitives |
| Framework plugin | Chat UI, agent panel, commands, settings, workflows, prompt configuration                                                                                   |
| Extension plugin | Providers, context sources, memory backends, review and workflow extensions                                                                                 |

The names `bitty-ai-runtime` and `bitty-ai` for the lower two layers are the
source's suggestion; the task-level caution stands that the two concepts
should not share one name. The minimal-base consequence is preserved: a bare
install carries no HTTP clients, protocol parsers, agent runtimes, or
conversation stores until the user opts into AI capability.

**Critical judgment:** this split is the weakest structural claim in the
distilled ranges. It has no ownership, packaging, versioning-compatibility,
or process-boundary evidence behind it, and it must not be read as an
accepted crate, package, or release decision. It is recorded as a candidate
input to [AI Architecture](../architecture/ai-architecture.md) and the related R-series
dispositions, where the std-only runtime direction, provider boundary, and
transport placement are decided on their own evidence. The neighboring
four-level ladder (`040.md:1779-1829`) and the native-versus-extension axis
(`040.md:1131-1294`) were read for exclusion accuracy and are not distilled
here.

## Source coverage and critical disposition

Retain means retain as a **proposal**, not accept as normative. Improve means
retain the objective with the correction above. Reject means reject that
mechanism or absolute claim; defer means no commitment pending named
evidence. Boundary sections read but not distilled are marked as such.

| Source lines | Topic                                                                                     | Disposition, rationale, and alternative                                                                                  |
| ------------ | ----------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| 1-137        | Panel and Activity framing, OS boundary                                                   | Boundary context; no new `bitty-ai` claim beyond host-guest positioning                                                  |
| 138          | AI Activity among panel activities                                                        | Boundary context; AI as hosted content, consistent with R1 projection-only panels                                        |
| 229          | AI in development-environment composition                                                 | Boundary context; composition illustration, not a required panel set                                                     |
| 244-254      | AI workspace composition                                                                  | Boundary context; Agent, Tasks, Terminal, Diff, Browser, Logs as illustration                                            |
| 309, 324     | Bitty AI as a grown application                                                           | Boundary context; application-not-Core positioning retained as objective                                                 |
| 377          | AI as first-class runtime child                                                           | Boundary context; peer-of-Terminal positioning, no interface implied                                                     |
| 402          | Motivating question on agent-plugin extension                                             | Retain question; frames the two-level model below                                                                        |
| 437          | Two-level extension model                                                                 | Retain mediation direction; improve with R2 gating; defer topology (library, process, plugin)                            |
| 438-459      | Editor-domain two-level illustration                                                      | Excluded (editor application design); symmetry noted, not distilled                                                      |
| 460-477      | Bitty AI as Host Plugin tree                                                              | Retain dual role and flat runtime; improve example children as illustration only                                         |
| 478-705      | Editor extension API, syntax, LSP layering                                                | Excluded (editor domain design); `bitty-ai` analogue covered via its own registries below                                |
| 706-778      | Bitty AI as plugin-platform plugin                                                        | Retain decomposition principle; improve registry and command names as proposals; defer schemas                           |
| 779-851      | Extension Points, `bitty-ai.*` declarations                                               | Retain typed attachment; improve point names as proposals needing capability review                                      |
| 852-911      | Manifest relations and dependency resolution                                              | Retain dependency honesty; improve TOML sketches as illustration; defer schema and resolution algorithm                  |
| 912-998      | Permission non-inheritance, Bitty AI emphasis                                             | Retain as load-bearing draft principle; default-deny for extensions behind R2                                            |
| 999-1062     | Service API versioning                                                                    | Retain decoupling objective; improve version strings as illustration; defer policy                                       |
| 1063-1130    | Editor-platform positioning and minimal install                                           | Excluded (editor application design); minimal-base consequence for AI retained via the layer split                       |
| 1131-1294    | Plugin tree, native-versus-extension axis, small base                                     | Read, not distilled; axis and base-minimality inform the layer split; tree mechanics are `bitty`-side                    |
| 1295-1364    | `bitty-ai-runtime` versus `bitty-ai` split, three-layer model                             | Retain as candidate input; weakest structural claim; not accepted as crate, package, or release decision                 |
| 1365-1505    | Non-AI native capabilities, browser analogy                                               | Read, not distilled; runtime-selection criteria are `bitty`-side handoff input                                           |
| 1506-1697    | Browser engine analogy, component upgrade sketch                                          | Excluded (analogy and packaging sketch); no `bitty-ai` contract follows                                                  |
| 1698-1829    | Native ABI caution, four-level ladder                                                     | Read, not distilled; ABI caution is `bitty`-side engineering; ladder not accepted here                                   |
| 1830-2215    | Appended comparable-programs round (Emacs, VS Code, JupyterLab, Eclipse RCP, Neovim, Zed) | Distilled in the addendum below; bitty-ai-relevant parts retained as candidate input, remainder excluded as `bitty`-side |

## Proposed validation and promotion path

These are future evidence requirements, not tests executed by this
documentation task. They keep the design falsifiable before any plugin-system
proposal constrains `bitty-ai`.

| Campaign               | Required observation                                                                                                                                      |
| ---------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Mediation direction    | An AI extension exercises capability only through its declared point; removing the host or the point disables it without touching Core                    |
| Small core             | A minimal `bitty-ai` install carries no provider, memory, or workflow machinery; each arrives as a separately reviewable, removable extension             |
| Commands as extensions | Review, plan, task, compact, and loop style commands install, update, and uninstall independently with per-command permission and no core change          |
| Non-inheritance        | An extension declaring a dependency on a powerful host gains no capability; each sensitive effect needs its own explicit grant behind the unified backend |
| Versioned surfaces     | Host-internal refactoring ships without breaking extensions pinned to a prior surface version; experimental surfaces stay unversioned until owner review  |
| Layer-split evidence   | Any future native-versus-framework split names an owner, a packaging and compatibility contract, and a process boundary before code is authorized         |

Promotion needs independent AI architecture, provider-boundary, terminal and
plugin-owner, docs-curator, and security review. Route registry schemas,
extension-point capability review, manifest format, versioning policy, and
the layer-split ownership and compatibility contract to scoped owner tasks.
This draft changes no normative contract and authorizes no product code.

## Addendum: appended comparable-programs round (`040.md:1830-2215`)

This addendum distills ONLY the bitty-ai-relevant parts of the appended
tail (`040.md:1830-2215`, 386 lines, verified unchanged since the append
was recorded: the whole file still holds 2,215 lines and 42,579 bytes). It
carries the same standing as the body above: a draft discussion synthesis
and candidate input to the draft
[AI Architecture](../architecture/ai-architecture.md) and its related draft
dispositions, never an override of the accepted
[IPC and Agent RFC](ipc-agent-rfc.md) or the normative security corpus.
The layered models below stay candidate-only. This addendum creates no AIQ
or OQ identifier and closes none. Relations are referenced, not modified.

The addendum's strongest ideas are the composition equation (`Bitty Base`
plus AI Runtime plus `Bitty AI` yields an AI Agent Workspace), which states
the candidate layering's intent more clearly than any head range, and the
JupyterLab token-mediated service pattern as adjacent evidence for typed
extension seams. Its weakest claims are the unverified external references
(`040.md:2207-2215`) and the product-positioning material (the intersection
diagram and the programmable-workspace definition), which are unreviewed
discussion claims and stay excluded.

### AI Panel placement in the workbench analogy

Source: `040.md:1914-1922` (Bitty Workspace panel tree inside the VS Code
workbench comparison, `040.md:1894-1956`).

The retained consequence is positional only: AI appears as a peer panel
(AI Panel beside Terminal, Bitter, Git, Docker, and Custom App panels),
consistent with the boundary-context section above (AI as hosted content
under R1 projection-only panels) and with the plugin-platform section's
Agent UI as a consumer surface rendered through host panels. The
surrounding workbench mechanics (Workbench regions, the WebView route, the
native widget-tree sketch) are `bitty`-side rendering and workbench design
and stay out.

**Critical judgment:** illustration, not a required panel set. No
interface follows from this placement.

### Base plus AI Runtime plus Bitty AI yields an AI Agent Workspace

Source: `040.md:2035-2043` (inside the Eclipse RCP comparison,
`040.md:2015-2066`).

The retained candidate equation is:

```text
Bitty Base + AI Runtime + Bitty AI = AI Agent Workspace
```

set beside the source's sibling equations (Base plus Editor Runtime plus
Bitter yields an editing environment; bare Base is a terminal/workspace).
This reinforces the two-layer/three-layer model recorded in the body as a
CANDIDATE input: it restates the layering intent, and it adds no
ownership, packaging, versioning-compatibility, or process-boundary
evidence. The weakness recorded in the layer-split section therefore
stands unchanged, and nothing here is accepted as a crate, package, or
release decision.

**Critical judgment:** reinforcement of candidacy, not new evidence.

### Study recommendation for bitty-ai extension composition

Source: `040.md:1958-2011` (JupyterLab token-mediated services, core
capabilities shipped as plugins, shell regions accepting widgets, and the
Bitty tree pairing `Bitty AI` with `bitty-ai-memory` at `040.md:2003-2004`),
`040.md:2055-2066` (Eclipse RCP research checklist: component dependency,
extension points, services, plugin lifecycle, optional components,
application composition), and `040.md:2205` (the closing recommendation to
study Emacs for the programmable environment, JupyterLab for modern
Service/Widget/Plugin composition, and Eclipse RCP for
Core/Component/Extension Point; VS Code and Neovim for UX and developer
ecosystem only).

Retained insofar as it applies to `bitty-ai` extension composition:

- JupyterLab's token-mediated services and core-as-plugins pattern as
  adjacent evidence for the registries and extension-points decomposition
  (typed seams, small core, contributed surface behavior).
- The Eclipse RCP checklist as review dimensions for any future manifest,
  lifecycle, or composition proposal; the minimal-platform analogy itself
  is motivation, not architecture.
- The Emacs host-provides-primitives packages-decide-product principle as
  philosophical support for the two-level model; the Emacs runtime and
  Lisp sections themselves (`040.md:1846-1891`) stay excluded as
  `bitty`-side philosophy.

**Critical judgment:** study leads, not findings. The external reference
links (`040.md:2207-2215`) are unverified discussion citations, with the
same standing as the repository citations already excluded in the body.

### Zed agent-server extension category as adjacent boundary context

Source: `040.md:2129` (Zed extensions providing language, grammar,
language-server, debug-adapter, MCP-server, agent-server, theme, and
snippet capabilities, with the marketplace organized around these
extension categories).

Retained as adjacent boundary context only: an existing native editor
packages agent capability as an extension category, which is consistent
with the small-core and contributed-surface direction but is a product
fact about Zed, not a `bitty-ai` contract. The remainder of the Zed
section (`040.md:2127-2165`, including the Zed-as-Bitty-plus-Bitter
equation) is `bitty`-side editor comparison and stays out.

### Tail coverage and disposition

Retain means retain as a **proposal**, not accept as normative. Boundary
sections read but not distilled are marked as such.

| Source lines         | Topic                                                                                                 | Disposition, rationale, and alternative                                                                           |
| -------------------- | ----------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| 1830-1844            | Comparable-programs question and comparison table                                                     | Excluded (product positioning); AI-bearing cells covered via their sections below                                 |
| 1846-1891            | Emacs philosophy (runtime, Lisp, packages on packages)                                                | Read, not distilled; host-primitives principle noted as consistent with the two-level model                       |
| 1894-1913, 1924-1956 | VS Code workbench mechanics, WebView route, native widget tree                                        | Excluded (`bitty`-side workbench and rendering design)                                                            |
| 1914-1922            | Bitty Workspace panel tree including AI Panel                                                         | Boundary context; AI as peer hosted panel, consistent with R1                                                     |
| 1958-2011            | JupyterLab services, tokens, core-as-plugins; Bitty tree with `Bitty AI` and `bitty-ai-memory`        | Retain as study lead for registries and extension points; names are discussion vocabulary                         |
| 2015-2034, 2045-2066 | Eclipse RCP minimal-platform analogy and research checklist                                           | Retain checklist as review dimensions; analogy is motivation, not architecture                                    |
| 2035-2043            | Base plus AI Runtime plus Bitty AI yields AI Agent Workspace                                          | Retain as candidate input reinforcing the layer split; no new evidence; not accepted                              |
| 2072-2124            | Neovim early-stage analogy (Buffer/Window/Tabpage versus Window/Workspace/Panel/Activity/Application) | Excluded (editor UI-model comparison; `bitty`-side)                                                               |
| 2127-2165            | Zed native-primitive extension API and Zed equation                                                   | Excluded except the `040.md:2129` agent-server category as adjacent boundary context                              |
| 2162-2205            | Intersection diagram, workspace definition, study recommendation                                      | Study recommendation (`040.md:2205`) retained as lead; diagram and definition excluded (`bitty`-side positioning) |
| 2207-2215            | Reference links                                                                                       | Unverified discussion citations, not evidence                                                                     |
