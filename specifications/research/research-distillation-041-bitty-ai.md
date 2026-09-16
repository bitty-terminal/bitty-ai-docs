---
title: IPC-value research distillation for bitty-ai (041)
description: Draft bitty-ai distillation of research 041 IPC second extension boundary out-of-process agent runtime panel capability permissions layer model
category: specifications
audience: mixed
document_type: research
status: draft
website_publish: false
sidebar_order: 53
---

# IPC-value research distillation for bitty-ai (041)

## Status and recommendation

This is a **draft discussion synthesis**, not a runtime specification, accepted
decision, dependency selection, release commitment, or implementation claim.
It distills only the `bitty-ai`-relevant parts of workspace research record
`041.md` (a discussion of what IPC design brings to the plugin system): IPC
as the second extension boundary, the out-of-process `bitty-ai` shape (agent
runtime, MCP, memory, context management), the `bitty-ai` daemon with
network-dependency independence, the `agent.spawn` / `agent.send` /
`agent.cancel` sketches, Agent/Panel lifecycle separation, `agent.*` events
as discussion inputs, unified Lua/IPC semantics with the Capability API
proposal, IPC-native permissions, and the closing three-layer placement.
Crash isolation, debugging and DevTools, controlling a running Bitty,
multi-instance addressing, and the Lua-versus-IPC split were read for
exclusion accuracy and stay `bitty`-side, except for the direct Agent
consequence of the split recorded below.

The recommendation is to treat every model below as a candidate input to the
draft [AI Architecture](../ai-architecture.md) and its related draft
dispositions, never as an override of the accepted
[IPC and Agent RFC](../ipc-agent-rfc.md) or the normative security corpus.
In particular, the `agent.*` method and event names, the Capability API, the
permission-manifest sketch, and the three-layer model are **not** accepted by
this distillation. Agent lifecycle, Agent events, and Agent semantics in the
source are **discussion inputs only**: the accepted Agent contract stays
entirely with the RFC, which this draft references without restating
normatively. Nothing here is promoted to accepted status, and no
implementation is described as shipped.

The source's strongest ideas are the out-of-process `bitty-ai` daemon with
Core ignorant of providers (which agrees with the network-dependency
independence direction), the Agent/Panel lifecycle separation carried over
IPC (which agrees with the projection-only panel direction), the Capability
API unity (Lua, IPC, and CLI as frontends of one capability model, consistent
with the mediation direction kept from record 040), and IPC-native
capability permissions (consistent with the unified authorization backend
direction and the non-inheritance principle kept from 040). Its weakest
claims are the concrete method and event names (`agent.spawn`,
`panel.create`, `agent.status.changed`, and similar), which are unreviewed
interface sketches with no ownership, versioning, or compatibility evidence;
the three-layer model, which is product positioning without packaging or
process-boundary evidence; and the "terminal platform" reframing, which is
motivation, not architecture. Those are corrected below.

This synthesis references, without duplicating or modifying, the draft
[AI Architecture](../ai-architecture.md) (candidate inputs only),
[Provider plugin boundary](../provider-plugin-boundary.md),
[Panel environment awareness](../panel-environment-awareness.md),
[Execution ownership R1](../execution-ownership-r1.md), and
[Tool transport R2](../tool-transport-r2.md). The accepted
[IPC and Agent RFC](../ipc-agent-rfc.md) is unaffected by this draft. This
document creates no AIQ or OQ identifier and closes none.

## Provenance and evidence boundary

The source is the workspace-relative `research/origin/041.md`, read read-only
on 2026-09-16: **821 lines**, **13,140 bytes**. Its SHA-256 is
`15182dc1d8b709a8d6a7f18de57f387e2db83f476fed12c8e5b19087387d9754`.
The file is untracked in the research repository, so provenance is by path
plus fingerprint, not by commit. The origin file was not renamed, edited, or
staged by this task; the `bitty`-side pass still needs it. The body of this
document distills the whole verified file (`041.md:1-821`); no post-task
append existed at verification time, so no head-versus-tail split applies.
Any later append is uncovered and follows the CTX-0045 pattern (distill the
verified head, record the remainder as uncovered).

**Verification:** confirm the source file integrity with:

```bash
sha256sum $BITTY_WORKSPACE/research/origin/041.md
wc -l -c $BITTY_WORKSPACE/research/origin/041.md
```

The expected output is the SHA-256 above with 821 lines and 13,140 bytes.

Unlike record 039, record 041 is a single pass with twelve numbered sections
plus a closing three-layer model at verification time; no duplication
handling applies. Sections 8-12 (`041.md:511-764`: crash isolation,
debugging, controlling a running Bitty, multi-instance support, and the
Lua-versus-IPC split) were read so that exclusions are accurate, but only
the direct Agent consequence of the split is distilled here. Only the ranges
in the coverage table were distilled. The source is a single-author
Chinese-language discussion; this document is the English-language draft
synthesis, not a translation. The fingerprint identifies the discussion, not
the truth of its claims.

## Authority and reconciliation

The draft [AI Architecture](../ai-architecture.md) layered models are
candidate inputs only; the three-layer model and the Capability Layer
proposed in the source are **not** accepted by this distillation and must
not be read as crate, package, protocol, or release decisions. Provider
contract and transport questions stay with the draft
[Provider plugin boundary](../provider-plugin-boundary.md); execution and
environment questions stay with [Execution ownership R1](../execution-ownership-r1.md)
and [Panel environment awareness](../panel-environment-awareness.md); tool
authorization and transport placement stay with
[Tool transport R2](../tool-transport-r2.md). Each is referenced, never
duplicated or modified.

The accepted [IPC and Agent RFC](../ipc-agent-rfc.md) defines the only
accepted IPC wire, scope, and Agent vocabulary. Every `agent.*` and
`panel.*` name in the source (`agent.spawn`, `agent.send`, `agent.cancel`,
`panel.create`, `agent.status.changed`, `agent.started`, `agent.finished`,
and similar) is a discussion sketch: this draft records it as input and
proposes no wire method, event name, or CLI surface. Where the source's
sketches overlap RFC-owned ground (Agent lifecycle, Agent events, Agent
semantics, scopes, CLI control), the RFC wins without further argument.

Normative security obligations (least privilege, per-action scopes,
capability-based and auditable permission that fails closed, typed redaction,
consented recording, secret minimization) override every discussion example
below. The source's method names, event names, manifest fields, permission
display, and capability list are conceptual vocabulary, not additions to any
accepted registry, schema, or protocol. This draft creates or closes no AIQ
or OQ identifier; open questions stay with
[AI Unresolved Questions](../ai-unresolved-questions.md) and shared
governance.

## IPC as the second extension boundary

Source: `041.md:3-78` (IPC as the second extension boundary beside the
in-process Lua API; the `lightweight / bitty-ai / external / other bitty`
table; `bitty-ai` in the out-of-process plugin tree; language freedom;
"terminal platform" reframing).

The retained model places two extension boundaries under Core: the
in-process Lua API on one side and the IPC boundary on the other, with
`bitty-ai`, external plugins, and other Bitty instances on the IPC side
while lightweight plugins stay in-process. Out-of-process plugins may use
any language (the source lists Rust, Python, Go, TypeScript, Java, and
Shell) and need not follow the Lua runtime. The source frames this as moving
from an extensible terminal to a terminal platform with a unified control
protocol.

**Critical judgment:** the boundary drawing is retained as a candidate
objective; the "terminal platform" reframing is product motivation, not
architecture. The stable claim is only the separation itself (in-process Lua
for lightweight extension, IPC for out-of-process participants), with the
exact capability set on each side undecided and owned by the terminal,
plugin, and AI repositories jointly. The language list is illustrative
freedom, not a supported-language promise.

## Heavyweight plugin shape: the bitty-ai daemon

Source: `041.md:79-151` (lightweight Lua topics versus heavyweight topics;
the `bitty-ai` component sketch: provider, agent runtime, MCP, embedding,
memory, SQLite, network, context management; the `bitty-ai` daemon split;
"Core need not know providers"; the `agent.spawn / agent.send /
agent.cancel` and `panel.attach / panel.output / panel.close` sketches;
network-dependency independence agreement).

The retained direction is that `bitty-ai` is the canonical heavyweight
plugin: provider handling, agent runtime, MCP, embedding, memory, storage,
networking, and context management do not belong in Core, and Core stays
ignorant of providers (OpenAI, Anthropic, MCP, and similar are named only as
examples of what Core must not know). The source's agreement point is kept:
the earlier direction that `bitty-ai` network dependencies stay independent
fits the IPC split exactly, because the daemon carries HTTP, TLS, SDK, and
storage dependencies that Core never takes on.

**Critical judgment:** the component list is discussion vocabulary for what
makes a plugin "heavyweight," not an accepted `bitty-ai` decomposition; the
authoritative decomposition stays with the draft AI Architecture and its
dispositions. The `agent.*` and `panel.*` method sketches are unreviewed
interface proposals recorded as discussion inputs only: they propose no wire
method and change nothing in the accepted RFC. The stable claims are the
daemon direction and Core's provider ignorance, which reinforce the
minimal-base principle kept from record 040 (a bare install carries no AI
machinery until the user opts in).

## Panel as a public abstraction

Source: `041.md:152-223` (Panel abstracted beyond terminal tabs: terminal,
floating, overlay, headless, AI, dashboard, custom UI; the
`create / write / focus / move / resize / close` sketches; the `panel.create`
message sketch; plugins operating a public protocol instead of internal
structs; the Plugin-to-IPC-to-Panel-Manager flow).

The retained direction is that Panel becomes a public abstraction external
programs operate through protocol rather than internal structs, with tiled,
floating, and headless panels behind one Panel Manager. The AI entry is
positional only: AI appears as one panel kind among others, which is
consistent with the boundary context kept from record 040 (AI as hosted
content under projection-only panels).

**Critical judgment:** every operation and message shape in this range is a
proposal, not a protocol. The stable claim is the abstraction direction
(external parties address panels through capability-checked protocol, never
through Core internals); the operation set, message envelope, and
authorization checks are undecided and, where they touch IPC ground, already
owned by the accepted RFC.

## Agent/Panel lifecycle separation over IPC

Source: `041.md:224-285` (Agent/Panel lifecycle independence; the Agent A
diagram across headless panels; what an Agent does not need; the
`panel.list / inspect / attach / detach / send_input / read_output`
sketches; "cleaner than embedding the Agent in terminal core" as author
judgment).

The retained direction is that an Agent is not bound to one panel: Agent A
can work through one headless panel, leave it, and enter another while both
panels continue to exist, addressing panels by list, inspect, attach,
detach, input, and output operations rather than by holding Core panel
references. The source's cleanness judgment (this model is cleaner than
embedding the Agent in terminal core) is recorded as author opinion
supporting the out-of-process direction.

**Critical judgment:** Agent lifecycle is RFC-owned ground, and this section
is read strictly as a discussion input: it proposes no lifecycle state
machine, no ownership contract, and no wire method. The operation sketches
are unreviewed vocabulary. The stable claim is only the separation
direction, which agrees with the R1 projection-only consequence (panels as
host-owned views an agent works through, not objects an agent owns) without
restating it.

## Plugin-to-plugin communication and agent events

Source: `041.md:286-370` (the IPC bus: plugin A through Core to plugin B;
`git.branch.changed`, `command.completed`, and `agent.status.changed`
examples; the event-bus list including `agent.started` and
`agent.finished`; shared Lua `bitty.on` and external `subscribe` semantics).

The retained direction is that IPC is a bus, not just a plugin-to-Core
channel: plugins publish and subscribe to one shared event semantics from
both Lua (`bitty.on`) and external processes (`subscribe`). The
`bitty-ai`-relevant example is `agent.status.changed` flowing to a
dashboard-style consumer; the surrounding examples (branch changes,
command completion, panel and workspace events) are `bitty`-side bus
content recorded here only as context for the shared-semantics proposal.

**Critical judgment:** every event name in this range, including all
`agent.*` names, is a discussion input, not an accepted event. The accepted
Agent observation and event vocabulary stays with the RFC, which this draft
does not restate. The stable claim is the bus direction (one shared event
semantics across Lua and IPC frontends); the event catalog, delivery
guarantees, filtering, and authorization are entirely open and, for
agent-bearing events, require security review as observation-data interfaces
before any adoption.

## Unified Lua/IPC semantics and the Capability API

Source: `041.md:371-449` (the "do not build two systems" recommendation;
the Bitty Capability API with Lua-binding and IPC-binding frontends; the
`panel.list / create / focus / close`, workspace, terminal, command, and
notification capability sketches; the Lua / IPC / CLI triple-frontend
example for `panel.create`; "three frontends of one capability model" as
the maintenance-cost argument).

The retained direction is semantic unity: Lua plugins, IPC clients, and the
CLI control surface should be frontends of one capability model rather than
three independent systems, so a capability is defined once and bound per
frontend. The source's capability sketches (panel, workspace, terminal,
command, notification operations) are illustrative vocabulary for that
model, and the `panel.create` triple example (Lua call, IPC message, CLI
invocation) states the intent, not a schema.

**Critical judgment:** the Capability API is a candidate model, not an
accepted contract: it names no owner, no versioning policy, and no
enforcement mechanism, and it must not be read as the accepted scope or
method registry, which stays with the RFC. The stable claim is the unity
objective, which is compatible in direction with the mediation rule kept
from record 040 (extensions attach through declared, reviewable seams);
whether the 041 capability model and the 040 extension-point model are the
same mechanism is undecided and stays with the owning tasks.

## IPC as a natural permission system

Source: `041.md:450-510` (the plugin-manifest permission sketch with
`panel.read`, `panel.create`, `terminal.input`, `filesystem.read`,
`network`, and `agent.control`; authenticate-to-capability-token flow;
per-capability allow/deny checks; the permission-display sketch; "easier
security boundary than the whole Lua Core API" as author judgment).

The retained direction is that the IPC boundary carries permissions
naturally: a plugin authenticates, receives capability-scoped rights, and
Core admits or denies each operation (including `agent.control`) against
those rights, with a human-readable permission display for review. The
source's judgment that this boundary is easier to hold than exposing a
whole in-process API is recorded as supporting opinion for capability
attenuation at the IPC layer.

**Critical judgment:** the manifest fields, token shape, check semantics,
and display format are unreviewed sketches, not a permission schema. They
align with, and do not relax, the normative least-privilege and fail-closed
obligations and the non-inheritance principle kept from record 040
(relationship to a powerful host confers no capability; each sensitive right
is an explicit grant). The stable claim is the placement direction
(permissions enforced at the IPC boundary behind a unified authorization
backend per R2); the vocabulary, grant mechanics, attenuation rules, and
audit consequences are open and need security review before any enforcement
claim.

## Sections read but excluded (bitty-side)

Source: `041.md:511-764`, read in full for exclusion accuracy. Section 8
(`041.md:511-558`, crash isolation: plugin crash without Core crash,
restart/disable/log handling) is `bitty`-side process supervision and stays
out; its AI-facing consequence is already covered by the daemon direction
above. Section 9 (`041.md:560-609`, debugging: panel listing, tree, and
event-stream sketches for future DevTools) is `bitty`-side tooling design
and stays out. Section 10 (`041.md:611-658`, controlling a running Bitty:
the `hyprctl` analogy, workspace and command control sketches, the `agent
ask` sketch, CLI-as-IPC-client) is `bitty`-side control-surface design and
stays out; the `agent ask` sketch in particular is a discussion input that
proposes no accepted CLI surface. Section 11 (`041.md:660-697`,
multi-instance sockets, instance listing, and the
`bitty://instance/.../panel/...` address sketch) is `bitty`-side addressing
and stays out; instance selection for IPC clients is already RFC-owned
ground.

Section 12 (`041.md:699-764`, the Lua-versus-IPC split) is the one range in
this block with a direct Agent consequence, distilled below; its latency and
UI-tightness rationale (per-frame IPC as unreasonable, Lua for frequent
low-latency UI-coupled work) is `bitty`-side engineering and stays out.

### The split's direct Agent consequence

Source: `041.md:736-761` (the IPC-plugin fit list: AI, Git daemon, language
tooling, indexer, sync, database, network service, large computation,
external integration; the fit characteristics: independent lifecycle,
crashable, complex dependencies, other languages, network or database use,
coarse call granularity).

Retained as the placement rule for `bitty-ai`: AI sits on the IPC side
because it has an independent lifecycle, may crash, carries complex
dependencies, may use other languages, needs network or database access, and
is invoked at coarse granularity. This is consistent with the daemon
direction and the heavyweight shape above; it adds the explicit negative
(that AI does not belong in-process) to the positive daemon claim.

**Critical judgment:** a placement heuristic, not a decision procedure. New
AI-adjacent work that is fine-grained, latency-sensitive, or UI-coupled
would need its own placement review rather than inheriting the IPC side by
default.

## Closing three-layer model and bitty-ai placement

Source: `041.md:765-821` (the `bitty-core` with Lua Plugin API and IPC API
diagram placing `bitty-ai` and MCP/Agent beside themes, UI plugins, and
services; the Capability Layer with Lua, IPC, and CLI frontends; "not just
messages" and the Bitty Capability Protocol framing; the Panel, Workspace,
Terminal, Command, Agent, Plugin, Notification, Event, Clipboard, and
History capability list; Lua, `bittyctl`, `bitty-ai`, and third-party
daemons as consumers; downstream motivation for `bitty-ai`, headless
panels, the plugin marketplace, DevTools, and remote control).

The retained candidate placement puts `bitty-ai` (with MCP and Agent) on
the IPC side of Core, beside services in other languages, while themes and
UI plugins stay on the Lua side; above both sits the Capability Layer
consumed by Lua, IPC, and CLI frontends alike. The source's punchline is
kept as a modeling claim: IPC should be designed as a capability protocol
stating what the outside world may do to Bitty, so that later `bitty-ai`,
headless-panel, marketplace, DevTools, and remote-control work shares one
infrastructure instead of inventing a channel per feature.

**Critical judgment:** this is candidate-only relative to the 040 layering,
and the two must not be merged silently. Record 040's candidate says
`Native Primitive -> Framework Plugin -> Extension Plugin` with a proposed
`bitty-ai-runtime` versus `bitty-ai` split; record 041's candidate says
Core with Lua and IPC APIs under a Capability Layer of three frontends.
Both are unaccepted, both lack ownership, packaging, versioning, and
process-boundary evidence, and neither overrides the other or the accepted
RFC. Their compatible direction (domain capability mediated through
declared, reviewable seams, never ambient) is retained; the choice between,
or synthesis of, the two layerings stays with the draft AI Architecture and
its related dispositions. The capability list is discussion vocabulary, not
an accepted scope catalog, and the downstream motivations are scheduling
context, not a roadmap.

## Source coverage and critical disposition

Retain means retain as a **proposal**, not accept as normative. Improve means
retain the objective with the correction above. Reject means reject that
mechanism or absolute claim; defer means no commitment pending named
evidence. Boundary sections read but not distilled are marked as such.

| Source lines | Topic                                                                                      | Disposition, rationale, and alternative                                                                                   |
| ------------ | ------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------- |
| 1-2          | Opening IPC question                                                                       | Boundary context in this distillation; motivates the synthesis, not a distilled claim.                                    |
| 3-78         | IPC as second extension boundary; plugin-kind table; out-of-process tree; language freedom | Retain boundary separation as candidate objective; improve platform reframing as motivation; defer capability split.      |
| 79-151       | Heavyweight shape; `bitty-ai` daemon; Core provider ignorance; `agent.*` sketches          | Retain daemon direction and provider ignorance; `agent.*` sketches are discussion inputs; defer decomposition to AI Arch. |
| 152-223      | Panel as public abstraction; operation sketches; Agent panel entry                         | Retain abstraction direction; improve operations as proposals; Agent entry is positional context under R1.                |
| 224-285      | Agent/Panel lifecycle separation; Agent A diagram; operation sketches                      | Retain separation direction; lifecycle and operations are RFC-owned discussion inputs only; defer contracts.              |
| 286-370      | IPC bus; plugin-to-plugin flow; `agent.*` event examples; shared Lua/IPC semantics         | Retain bus direction; all event names are discussion inputs; delivery and auth entirely open.                             |
| 371-449      | Capability API unity; triple-frontend example; capability sketches                         | Retain unity objective as candidate; improve names as vocabulary; defer owner, versioning, enforcement.                   |
| 450-510      | Manifest sketch; capability token; per-capability checks; permission display               | Retain boundary-enforcement placement; sketches unreviewed; align with R2 backend and 040 non-inheritance.                |
| 511-558      | Crash isolation (section 8)                                                                | Read, not distilled; process supervision is `bitty`-side; consequence covered by the daemon direction.                    |
| 560-609      | Debugging and DevTools sketches (section 9)                                                | Excluded (`bitty`-side tooling design); no `bitty-ai` contract follows.                                                   |
| 611-658      | Controlling a running Bitty; `agent ask` sketch (section 10)                               | Excluded (`bitty`-side control design); `agent ask` proposes no accepted CLI surface.                                     |
| 660-697      | Multi-instance sockets and addressing (section 11)                                         | Read, not distilled; addressing is `bitty`-side and partly RFC-owned; no `bitty-ai` contract follows.                     |
| 699-735      | Lua-plugin fit and rationale (section 12, first half)                                      | Read, not distilled; latency and UI-coupling rationale is `bitty`-side engineering.                                       |
| 736-761      | IPC-plugin fit list and characteristics (section 12, second half)                          | Retain as AI placement rule; improve as heuristic needing per-case review, not a decision procedure.                      |
| 765-821      | Closing three-layer model; Capability Layer; capability list; downstream motivation        | Retain as candidate input only; weakest structural claim; not accepted; do not merge silently with the 040 layering.      |

## Proposed validation and promotion path

These are future evidence requirements, not tests executed by this
documentation task. They keep the design falsifiable before any IPC-value
proposal constrains `bitty-ai`.

| Campaign             | Required observation                                                                                                                                  |
| -------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| Daemon direction     | A `bitty-ai` capability ships out-of-process with Core carrying no provider, network, or storage dependency for it; bare installs stay AI-free.       |
| Provider ignorance   | Core handles an Agent-class request knowing only generic capability checks, with no provider-named branch in Core code or configuration.              |
| Lifecycle separation | An Agent works through successive panels without owning any panel object, and panel lifetime is independent of Agent attachment in reviewable tests.  |
| Bus semantics        | A Lua consumer and an external subscriber observe the same event with one shared semantics description; neither receives capability by subscribing.   |
| Capability unity     | One capability definition serves Lua, IPC, and CLI frontends with per-frontend bindings; no capability exists in only one frontend by accident.       |
| Boundary permission  | An IPC plugin exercises exactly its granted capabilities with explicit `agent.control`-class denial by default; grants attenuate and audit.           |
| Layer-split evidence | Any future Capability Layer or three-layer adoption names an owner, a packaging and compatibility contract, and a process boundary before code ships. |

Promotion needs independent AI architecture, provider-boundary, terminal and
plugin-owner, docs-curator, and security review. Route capability vocabulary, event catalog, manifest format, permission mechanics, and the
layer-model ownership and compatibility contract to scoped owner tasks.
This draft changes no normative contract and authorizes no product code.
