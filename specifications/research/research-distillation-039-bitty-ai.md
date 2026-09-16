---
title: Panel research distillation for bitty-ai (039)
description: Draft bitty-ai distillation of research 039 panel topics AI workspace object model panel boundary floating AI text editor agent workspace sequencing
category: specifications
audience: mixed
document_type: research
status: draft
website_publish: false
sidebar_order: 51
---

# Panel research distillation for bitty-ai (039)

## Status and recommendation

This is a **draft discussion synthesis**, not a runtime specification, accepted
decision, dependency selection, release commitment, or implementation claim.
It distills only the `bitty-ai`-relevant parts of workspace research record
`039.md` (a Panel discussion): the AI Workspace position in the object model,
the Panel identity and lifecycle boundary that future Agent UI panels inherit,
floating-mode AI, the Rust `TextEditor` primitive consumed by the AI prompt
editor, the Agent Workspace composition, and the proposed sequencing with its
unifying principle. Terminal-platform mechanics (widget layer, effects,
focus states, application services, editor application design) are
`bitty`-side matters and are excluded; they are recorded as handoff input,
not decisions.

The recommendation is to treat every Panel-side claim below as a candidate
input to the draft [AI Architecture](../ai-architecture.md) and its related
draft dispositions, never as an override of the accepted
[IPC and Agent RFC](../ipc-agent-rfc.md) or the normative security corpus.
Nothing here is promoted to accepted status, and no implementation is
described as shipped.

The source's strongest ideas are the Panel-as-host boundary (`Panel` is not a
terminal; a terminal is one activity among several), the identity separation
`PanelId != ViewId != TerminalId` with host-owned lifecycle, mode-as-property
sharing (so AI panels reuse workspace operations), and the `Document != View
!= Panel` split with a shared Rust `TextEditor` primitive. Its weakest claims
are the unverified repository and RFC citations (no revision is pinned, so
none is confirmed here), the assumed effortlessness of activity switching with
a live background session, and the sequencing proposal, which is an opinion
about build order rather than an accepted plan. Those are corrected below.

This synthesis references, without duplicating or modifying, the draft
[AI Architecture](../ai-architecture.md) (candidate inputs only),
[Provider plugin boundary](../provider-plugin-boundary.md),
[Panel environment awareness](../panel-environment-awareness.md),
[Execution ownership R1](../execution-ownership-r1.md), and
[Tool transport R2](../tool-transport-r2.md). The accepted
[IPC and Agent RFC](../ipc-agent-rfc.md) is unaffected by this draft. This
document creates no AIQ or OQ identifier and closes none.

## Provenance and evidence boundary

The source is the workspace-relative `research/origin/039.md`, read read-only
on 2026-09-16: **6,079 lines**, **123,712 bytes**. Its SHA-256 is
`d5559e19bdeb73b8a71a03b79f2ed7f8f666ac7cfeecc5d7bd43159ced28c46d`.
The file is untracked in the research repository, so provenance is by path
plus fingerprint, not by commit. The origin file was not renamed, edited, or
staged by this task; the `bitty`-side pass still needs it.

**Verification:** confirm the source file integrity with:

```bash
sha256sum $BITTY_WORKSPACE/research/origin/039.md
wc -l -c $BITTY_WORKSPACE/research/origin/039.md
```

The expected output is the SHA-256 above with 6,079 lines and 123,712 bytes.

### Duplication structure

The file is five pasted rounds of one 15-section conversation. Rounds 1
through 4 (`039.md:46-1258`, `1259-2471`, `2472-3684`, `3685-4897`) are
byte-identical (each round MD5 `1722f56ee75e9ac76eef1f90c187113b`); round 5
(`039.md:4898-6079`) equals round 1 minus the trailing 31-line conclusion
block. All source ranges below cite round 1 (`039.md:1-1258`) as canonical;
the duplication is recorded here so nobody re-extracts the later rounds.

Only the ranges in the coverage table were distilled. The source is a
single-author Chinese-language discussion; this document is the
English-language draft synthesis, not a translation. The fingerprint
identifies the discussion, not the truth of its claims. Cited repository and
RFC statements (for example the Panel Runtime RFC, Plugin API v1, and Panel
Animations RFC) carry no pinned revision in the source and were not
independently verified here; they are treated as discussion claims, not as
evidence.

## Authority and reconciliation

The draft [AI Architecture](../ai-architecture.md) layered models discussed
in the source are candidate inputs only and are **not** accepted by this
distillation. Provider placement questions stay with the draft
[Provider plugin boundary](../provider-plugin-boundary.md); panel environment
semantics stay with [Panel environment awareness](../panel-environment-awareness.md);
single-agent execution ownership stays with
[Execution ownership R1](../execution-ownership-r1.md); tool authorization and
transport placement stay with [Tool transport R2](../tool-transport-r2.md).
Each is referenced, never duplicated or modified.

Normative security obligations (authenticated local IPC, per-action scopes,
read-only agent defaults, typed redaction, consented recording, secret
minimization) override every discussion example below. The source's `PanelId`,
`ViewId`, `TerminalId`, `PanelContent`, and mode names are conceptual
vocabulary from the discussion, not additions to any accepted registry.
This draft creates or closes no AIQ or OQ identifier; open questions stay
with [AI Unresolved Questions](../ai-unresolved-questions.md) and shared
governance.

## AI Workspace in the object model

Source: `039.md:15-45` (head conclusion: Panel as host of the Native
Application Runtime; `Bitty` tree with `AI Workspace` as a first-class
branch; Emacs plus window-manager plus terminal plus native toolkit analogy).

The source proposes placing an `AI Workspace` beside the terminal emulator,
window and workspace manager, native UI runtime, Lua application platform,
plugin runtime, and automation runtime, with `Panel` as the place where they
meet. For `bitty-ai`, the retained candidate direction is narrow: AI surfaces
are guests of a host-owned workspace, not owners of it. `bitty-ai` supplies
agent state, model and context policy, tools, and permissions; the terminal
side supplies panels, lifecycle, rendering, and transport. The product
analogy (Emacs-like composability over a Rust plus GPU base with a Lua
extension layer) is motivation, not an adopted architecture.

**Critical judgment:** the object-model tree is a proposal. The stable claim
is only the hosting direction: AI content lives inside host-managed
containers under host lifecycle, consistent with the R1
`ExecutionContext`-primary model. The source's repository and RFC citations
are unverified here and cannot ground scope claims.

## Panel identity and lifecycle boundary

Source: `039.md:46-83` (section 1: `PanelId != ViewId != TerminalId`;
`PanelRuntime` create, mount, suspend, resume, dispose; PTY, GPU object, and
OS window handle not owned by Panel; `PanelContent` enum with terminal, rich,
browser, helper, and canvas variants; `Panel != Pty`).

The retained boundary for future Agent UI panels is:

- Identity separation: a panel, its view, and its terminal session are
  distinct identities with distinct lifetimes. An agent panel can outlive,
  hide, or rebind its visible surface without moving or duplicating the
  underlying execution.
- Host-owned lifecycle: creation, mounting, suspension, resumption, and
  disposal belong to the terminal-side runtime. `bitty-ai` refers to panels
  and observations; it never owns PTY descriptors, GPU objects, or window
  handles.
- Content polymorphism: a panel may host a terminal, rich content, a browser
  surface, a helper process, or a canvas. Agent UI is one content kind among
  several, so agent panels inherit generic workspace behavior instead of
  needing a parallel windowing system.

**Critical judgment:** the enum variants and lifecycle verb names are
discussion vocabulary, not an accepted interface. The stable claim is the
ownership direction, which agrees with R1 (projection-only panels) and with
[Panel environment awareness](../panel-environment-awareness.md)
(host-mediated environment, no ambient authority).

## Floating-mode AI and shared workspace operations

Source: `039.md:84-145` (section 2: mode table with `tiled`, `floating`
covering help, settings, AI, pet, and tool uses, `overlay`, `fullscreen`,
`scratchpad`, `pinned`, `popover`; mode as runtime property with identity
preserved across transitions; shared key operations across all panel kinds).

The retained direction is that AI panels need no bespoke windowing: because
mode is a runtime property rather than a panel type, an AI panel can move
between tiled, floating, fullscreen, and scratchpad states while keeping its
identity, lifecycle, input routing, and surface. The source's example
operation set (toggle fullscreen, toggle floating, scratchpad recall, focus
and move navigation) applies uniformly to AI, editor, Docker, mail, and
terminal panels alike.

**Critical judgment:** floating presentation is a host decision, not an agent
capability. An agent must never open, focus, or hide panels by itself;
showing a decision surface is a request to the host, and background work
stays inspectable under user authority. The key bindings are illustrations,
not adopted defaults.

## TextEditor primitive and the AI prompt editor

Source: `039.md:407-489` (section 6: Rust Core versus Lua application split;
buffer, rope, cursor, selection, IME, grapheme segmentation, shaping,
undo and redo engine, viewport, virtualized rendering, and clipboard
primitive on the Rust side; modes, keymaps, commands, LSP integration, syntax
configuration, UI, and workflows on the Lua side; `bitty-ai` prompt editor
listed as a `TextEditor` consumer; Emacs buffer analogy; `Document != View
!= Panel`).

The retained `bitty-ai` direction is consumer-only:

- The AI prompt editor is a consumer of a host-provided text-editing
  primitive, alongside the mail composer, commit editor, configuration
  editor, and the reference editor plugin. `bitty-ai` declares editing needs
  (multiline input, history, completion hooks); it does not implement buffer,
  cursor, shaping, virtualization, or undo machinery.
- `Document != View != Panel` separates the editable object, its display
  state, and its host container. One document may be shown in zero, one, or
  several views; a view carries display state and history; a panel hosts
  views. Agent edits therefore target documents through reviewed operations,
  never panel pixels.

**Critical judgment:** the primitive's exact API, the reference editor
plugin, and the Emacs analogy are proposals, not accepted contracts. The
stable claim is the layering: editing machinery stays host-side (or in a
reviewed native capability), while `bitty-ai` stays a client. Editing effects
still pass the R2 gate order and the expected-revision discipline; the
primitive grants no ambient file authority.

## Agent Workspace composition

Source: `039.md:1067-1153` (section 14: terminal-first programmable
workspace framing; IDE, DevOps, personal, and Agent workspace compositions;
`Bitty` as a runtime that can construct an IDE rather than being one).

The retained Agent Workspace composition is `Agent, Terminal, Diff, Task
Board, Logs, Browser` as co-hosted panels over one runtime. For `bitty-ai`,
the consequences are:

- The agent panel is one member of a composed workspace, not the workspace
  itself. Diff display, task tracking, log observation, terminal execution,
  and browser surfaces are sibling capabilities coordinated by the host, with
  `bitty-ai` contributing agent state and tool effects behind scoped IPC.
- The "runtime that can construct an IDE" framing keeps `bitty-ai` out of
  the composition business: workspace layout, panel kinds, and cross-panel
  data flow are host and plugin-ecosystem decisions. `bitty-ai` must work in
  any composition the host assembles, including headless execution with no
  agent panel at all.

**Critical judgment:** the composition lists are illustrative configurations,
not a required panel set and not a release plan. No composition implies
co-located authority: observing a sibling panel's output is a separately
authorized, redacted, bounded read, consistent with R2 and R3.

## Sequencing and unifying principle

Source: `039.md:1154-1258` (section 15: six-step order from Panel Runtime
completion through activity and surface layer, widget tree, `TextEditor`
primitive with a small reference plugin, application services under
capability sandbox, and deferred advanced effects; closing object model with
presentation versus activity split; the principle sentence developing
"Panel is not Terminal" into "Panel is a host; Terminal is only one
Activity").

The retained unifying principle is **Panel is a host; Terminal is only one
Activity**. It explains, in one sentence, why agent panels, editor panels,
mail panels, shell panels, and native-app panels belong to the same system:
each is an activity hosted by a generic workspace-managed container with a
presentation state (tiled, floating, fullscreen, scratchpad, pinned) and an
activity kind (terminal, native application, rich content, canvas, helper).

The six-step build order is recorded as the source author's opinion, not as
an accepted plan: finish the Panel Runtime first, then the
activity and surface layer, then the widget tree, then the `TextEditor`
primitive validated by a small reference plugin, then application services,
and only later advanced panel effects under a bounded core animation
discipline. `bitty-ai` depends on the first two steps (stable panel identity
and lifecycle, activity switching with a surviving background session) and is
otherwise decoupled from terminal-side sequencing.

**Critical judgment:** lossless activity switching with a live background
session is asserted, not demonstrated, in the source. Session survival,
state reconciliation after switching, and the cost of hidden activities need
host-side evidence before `bitty-ai` relies on them. The final object-model
tree is a proposal; the accepted lifecycle vocabulary is unchanged by this
draft.

## Source coverage and critical disposition

Retain means retain as a **proposal**, not accept as normative. Improve means
retain the objective with the correction above. Reject means reject that
mechanism or absolute claim; defer means no commitment pending named
evidence. Round-1 ranges are canonical; rounds 2 through 5 duplicate them as
documented in the provenance section.

| Source lines | Topic                                         | Disposition, rationale, and alternative                                                                   |
| ------------ | --------------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| 1-14         | Opening Panel question                        | Boundary context; motivates the distillation, not a distilled claim                                       |
| 15-45        | Head conclusion, AI Workspace in object model | Retain hosting direction; improve with R1 projection-only rule; reject analogy as architecture            |
| 46-83        | Panel identity, lifecycle, content enum       | Retain identity separation and host-owned lifecycle; improve vocabulary as discussion-only, not registry  |
| 84-145       | Modes, floating AI, shared operations         | Retain mode-as-property and uniform operations; reject agent-driven windowing; defer key bindings         |
| 146-237      | Panel versus Activity layer                   | Excluded (`bitty`-side container design); AI consequence covered via identity and activity sections above |
| 238-328      | Native UI versus traditional TUI              | Excluded (`bitty`-side UI direction); no `bitty-ai` contract follows from the comparison                  |
| 329-406      | Reference editor plugin existence             | Excluded (editor application design); `bitty-ai` consequence is consumer-only, covered above              |
| 407-489      | TextEditor primitive, AI prompt editor        | Retain consumer-only direction; improve with R2 gating on editing effects; defer API shape                |
| 490-620      | Native widget layer proposal                  | Excluded (`bitty`-side UI runtime); `bitty-ai` declares needs, never widgets                              |
| 621-698      | Terminal as a native widget                   | Excluded (`bitty`-side rendering direction); execution ownership stays with R1                            |
| 699-809      | Panel effects abstraction                     | Excluded (`bitty`-side effects design); advanced effects deferred by the source itself                    |
| 810-895      | Focus and idle visual states                  | Excluded (`bitty`-side presentation state); agent authority unaffected by presentation                    |
| 896-954      | Docker, mail, chat plugins as applications    | Excluded (application designs); composition consequence covered via Agent Workspace above                 |
| 955-1018     | Application services half                     | Excluded (`bitty`-side services); capability-sandbox note is handoff input, not a `bitty-ai` decision     |
| 1019-1066    | Editor startup performance comparison         | Excluded (unmeasured claim); no performance conclusion is drawn here                                      |
| 1067-1153    | IDE question, Agent Workspace composition     | Retain composition as illustration; improve with per-panel authorization; reject composition as plan      |
| 1154-1227    | Six-step sequencing                           | Retain as author opinion; defer as plan pending host-side acceptance                                      |
| 1228-1258    | Closing object model and principle sentence   | Retain principle sentence as candidate; improve closing tree as proposal, not accepted vocabulary         |
| 1259-6079    | Rounds 2-5 duplicates                         | Not re-extracted; byte-identity documented in provenance                                                  |

## Proposed validation and promotion path

These are future evidence requirements, not tests executed by this
documentation task. They keep the design falsifiable before any Panel-side
proposal constrains `bitty-ai`.

| Campaign                  | Required observation                                                                                                                                               |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Identity separation       | Hiding, moving, or rebinding an agent panel never moves its execution target, duplicates its session, or widens its authority                                      |
| Lifecycle ownership       | Suspend, resume, and disposal of agent panels are host operations; `bitty-ai` holds references, and stale handles fail closed                                      |
| Mode transitions          | Tiled, floating, fullscreen, and scratchpad transitions preserve panel identity and input routing while background sessions survive with reconciled state          |
| Prompt-editor consumption | The AI prompt editor performs multiline editing, history, and completion through the host primitive without `bitty-ai`-owned buffer machinery                      |
| Workspace composition     | Agent, terminal, diff, task, log, and browser panels compose and decompose without co-located authority; cross-panel reads stay separately authorized and redacted |
| Host-requested surfacing  | Agent requests to show a decision surface are host-mediated; the agent cannot open, focus, or close panels directly                                                |

Promotion needs independent AI architecture, terminal and panel-owner,
docs-curator, and security review. Route missing Panel Runtime revisions,
accepted lifecycle conflicts, activity-switching evidence, primitive API
shape, and composition authorization to scoped owner tasks. This draft
changes no normative contract and authorizes no product code.
