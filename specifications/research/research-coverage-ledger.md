---
title: Research coverage ledger
description: Coverage and disposition of AI-relevant research records 013, 017, 018, and partial 039/040
category: specifications
audience: mixed
document_type: register
status: draft
website_publish: false
sidebar_order: 21
---

# Research coverage ledger

This ledger maps specific source topics to the
[distillation](research-distillation-013-017-018.md), rather than claiming that
an entire recording or specification is covered. Line ranges are inclusive,
verified against the local recordings identified below. Repeated examples are
condensed, not adopted as API definitions.

## Source identity

Paths are workspace-relative; these are content fingerprints, not Git commits
or evidence that the recordings' factual claims are true.

| Source                                | SHA-256                                                            |
| ------------------------------------- | ------------------------------------------------------------------ |
| `recording/research/013.md.completed` | `49c9a1a03aa98857f7f679d7d7a8d1e811db36c897d0ea7cc98fe8d5badb4e8d` |
| `recording/research/017.md.completed` | `fda2144a1626d7851bb9b31ecec69a00dee1f6f726c7a5d9c6af92b102ea9070` |
| `recording/research/018.md.completed` | `101d2848325ffe281a76ad555c35deb1cbf32518a464307324c54fce59458c9e` |

## Disposition

Git and source inspection correct both the stale non-existence claim and the
later scaffold-only overcorrection. `bitty-ai` contains experimental slice code
at `3623c6b3ce33e97c1c493109ec6356219d0c9722`; mount commit
`b6d3d6c495607f6bb2660b5442cfefcf350b486e` records the docs gitlink.
The commander subsequently initialized the existing local mount; its revision
and clean parent status were verified. See
[current evidence](research-distillation-013-017-018.md#current-bitty-ai-evidence)
for the exact pointer, checked paths, and limits of the code observation.

The companion distillation is the new draft coverage. The accepted
[IPC contract](../ipc-agent-rfc.md), draft
[AI architecture](../ai-architecture.md), and experimental
[pressure-test evidence](../ai-vertical-slice-pressure-test.md) retain their
distinct status. Existing draft coverage is not accepted authority. No
proposal from the records is promoted to accepted status. Claims requiring
verification are marked as such in the distillation; source repositories under
`recording/references` remain untrusted read-only research material.

## Verification backlog

Independent review returned **APPROVE** for the distillation, including source
hashes, reference revisions/licenses, code slices, coverage, and `just check`.
The commander renamed record 018 to `.md.completed` to mark **distillation
complete only**; its content hash and source ranges are unchanged. This does
not accept the proposed architecture, certify product implementation, or
complete the delivery lifecycle. The drafts remain uncommitted/unpublished and
CTX-0003 remains in review awaiting discussion.

Before acceptance, verify library versions and licenses from upstream release
metadata, reproduce any local measurement with a documented fixture, and compare
protocol/security claims against primary specifications and advisories. Record
revision identifiers and evidence links in the relevant decision or evidence
register maintained by `bitty-docs`.

## Topic-level traceability

Section names below refer to the companion distillation unless a linked
existing document is named. Historical examples and dates remain evidence of
what was discussed, not of current implementation.

### Record 013

| Source lines | Topic                                                                 | Disposition / destination                                                                                                                                               |
| ------------ | --------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 132–160      | XDG/platform separation and credential references                     | Additional coverage / Providers, tools, and security; retain separation, not literal machine paths or new credential-store policy.                                      |
| 162–176      | Physical session restore versus plugin policy                         | Additional coverage; terminal-owned mechanism, distinct from AI event/session persistence.                                                                              |
| 230–254      | External CLI and sidecar isolation                                    | Boundary context; do not adopt native in-process loading or imply AI owns process lifecycle. Existing architecture normative-source links govern.                       |
| 256–285      | Service broker and bounded read-only ContextProvider                  | Stable boundary and Runtime/context sections; host-mediated values do not confer caller authority. Existing AI architecture covers ContextProvider.                     |
| 314–350      | Independent integration; three hand-written protocols and Lua presets | Stable boundary / Evidence disagreements; contrast with 018 library adapters, no dependency selection. Historical Core crate names are not current repository topology. |
| 352–358      | Lua agent/subagent/harness orchestration                              | Additional coverage; contrast with 018's Rust-owned state/enforcement, loop ownership remains open.                                                                     |
| 360–397      | Native panels, command events, context inspector, patch suggestions   | Additional coverage; preserve semantic/context/event consequences, exclude renderer implementation. Latency and visual-quality claims are unverified.                   |
| 401–445      | Embodied agents, panel leases, preserved state, human handoff         | Additional coverage; speculative multi-agent coordination, not an accepted identity/ownership contract.                                                                 |
| 447–469      | Terminal-to-agent platform roadmap                                    | Historical proposed ordering, not an accepted release plan; AI future scope retained under Persistence and release scope.                                               |

### Record 017

| Source lines     | Topic                                                                                                                                                | Disposition / destination                                                                                                                                                                                                                                     |
| ---------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 3–26             | M1–M8 assessment dated 2026-09-13                                                                                                                    | 017 maturity section; explicitly historical, including real early M4/M5 systems rather than treating them as empty designs.                                                                                                                                   |
| 332–395          | AI inventory; optional integration; generic-primitives pressure test                                                                                 | Stable boundary / 017 maturity; existing architecture and pressure-test specification cover context IDs, tools, Rich, and the narrow experiment.                                                                                                              |
| 340–363, 408–435 | Role/model/capability separation; task/run/execution identity; index/LSP/edits; overlays, evidence, compression, CarryCtx, growth, batching, secrets | Existing architecture research topics, not newly specified interfaces. Distillation preserves scope discipline; identity, overlay, CarryCtx-backend and agent-growth contracts remain unresolved rather than being claimed covered by the new runtime sketch. |
| 396–448          | Design expansion beyond evidence                                                                                                                     | 017 maturity / discussion admission rule; no new OQ is created here.                                                                                                                                                                                          |
| 454–466          | Stale project state, compatibility, lifecycle/property tests, plugin validation, vertical slice, OQ admission                                        | 017 maturity; governance consequences retained. Project-state freshness is not a non-AI exclusion. Concrete terminal/plugin repairs remain with their owners.                                                                                                 |
| 470–527          | Verification/dogfooding/invariant-hardening priority                                                                                                 | 017 maturity; recommendation, not current maturity certification.                                                                                                                                                                                             |

### Record 018: foundations and integration

| Source lines       | Topic                                                                                             | Disposition / destination                                                                                                                                                                                      |
| ------------------ | ------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1–82               | Independence; scaffold, docs location and toolchain drift claims; minimal contracts               | Stable boundary / Current Bitty AI evidence; docs/scaffold claims superseded by inspected commits. Historical toolchain values are not changed or adopted.                                                     |
| 83–134, 977–1065   | Crate DAG and workspace-crate deferral                                                            | Stable boundary; candidate contracts → adapters → runtime → store/protocol → CLI layering, not approved crate creation.                                                                                        |
| 135–242            | genai versus Rig/custom protocols; own public types                                               | Providers / Evidence disagreements; library/release assertions unverified; adapter boundary proposal retained.                                                                                                 |
| 243–292            | Native tool versus MCP adapter, SDK and vulnerability claims                                      | Providers / Verification backlog; common permission pipeline required, release/advisory specifics not promoted.                                                                                                |
| 293–397            | Effect analysis, approvals, unified native/MCP/remote dispatch                                    | Providers; mandatory host capability/consent controls preserved, effect-analysis interface proposed.                                                                                                           |
| 398–469            | Sans-I/O state machine and external driver                                                        | Runtime proposals; replay/pause/fork/serialization goals, not implemented guarantees.                                                                                                                          |
| 470–527            | Session/turn/event persistence, UI projection, telemetry                                          | Persistence / Additional coverage; redaction, retention, replay semantics remain open.                                                                                                                         |
| 528–601            | Typed context, semantic terminal sources, budget/dedup/redaction/compression, provider estimators | Runtime / Additional coverage; estimated and billed tokens differ; no universal tokenizer.                                                                                                                     |
| 602–654            | Curated memory versus searchable history, SQLite/FTS5, worker versus SQLx, later vectors          | Persistence / Additional coverage; storage and sequencing candidates.                                                                                                                                          |
| 655–711, 1249–1323 | ACP client boundary versus MCP tools and IPC transport, headless CLI                              | Stable boundary / Providers / discussion questions; exact transport ownership and protocol stability require review.                                                                                           |
| 712–735            | Dependency and test-library candidates                                                            | Additional coverage; includes insta, wiremock, proptest with intended test roles, no package adoption.                                                                                                         |
| 737–749            | Cancellation child-token tree                                                                     | Additional coverage; session/turn/model/tool/context propagation candidate.                                                                                                                                    |
| 750–784            | Idempotency-aware retry                                                                           | Providers; destructive effects cannot be blanket-retried; provider billing and partial-stream behavior still need contracts.                                                                                   |
| 785–810            | Secret memory, credential store, environment filtering                                            | Providers; credential references and minimization required; library behavior not audited here.                                                                                                                 |
| 811–849            | Codex, Goose, Rig, Hermes, OpenCode, ACP and MCP study leads; context-collection attack           | Providers / Primary-source ledger; only listed source slices were verified. Other projects and reported exploits remain research leads.                                                                        |
| 850–881            | cargo-deny wildcards/default features/advisory rationale                                          | Additional coverage; hardening proposal, no current vulnerability assessment or pin changes.                                                                                                                   |
| 882–976            | Narrow offline-to-provider/tool/store/replay slice, acceptance sequence, deferrals                | Persistence / Evidence disagreements; distinguishes proposed v0.1 runtime from existing experimental slice and post-1.0 architecture scope.                                                                    |
| 1066–1248          | Model/context/tools/runtime/store responsibilities                                                | Stable boundary / Runtime / Persistence; condensed responsibilities, not public API acceptance.                                                                                                                |
| 1324–1663          | Collapse/reveal, semantic IDs, panel state, PTY/shell integration, shared executor and transport  | Stable boundary / Additional coverage; terminal mechanisms excluded from AI implementation, semantic observation and host enforcement consequences retained. Shared executor/protocol crates remain proposals. |

### Record 018: code intelligence and composition

| Source lines | Topic                                                                                                           | Disposition / destination                                                                                                                                                    |
| ------------ | --------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1665–1778    | Multi-file read/write motivation, tool calls versus model requests, local measurements, execute-code/delegation | Runtime / Primary-source ledger; distinguish calls, round trips, context bytes, and subagent cost. Recording's database counts and savings were not reproduced.              |
| 1779–1896    | Compact tool family and project-level inspection                                                                | Runtime / Additional coverage; bounded project/symbol map proposal.                                                                                                          |
| 1897–1972    | Aider-style graph ranking; syntax versus semantic/search/index layers                                           | Additional coverage / Aider evidence limitation; secondary description only, not verified Aider behavior.                                                                    |
| 1973–2063    | LSP operations and OpenCode comparison                                                                          | Runtime / Additional coverage; proposed semantic operations retained; claim that OpenCode exposes mainly diagnostics is not established by the permission inspection.        |
| 2064–2175    | Multi-target code.read and four search modes                                                                    | Additional coverage; deduplication, sorting, compression and budgeting inside the tool.                                                                                      |
| 2176–2248    | Dependency/reference graph, impact, optional SCIP                                                               | Additional coverage; future semantic index, no complete call-graph guarantee from syntax alone.                                                                              |
| 2249–2330    | User tool discovery/configuration, deferred installer, async-lsp evaluation                                     | Additional coverage; no toolchain installer or SDK API accepted.                                                                                                             |
| 2331–2386    | Separate syntax/semantic/diagnostic/formatter/build providers                                                   | Additional coverage; capability interfaces rather than assuming all checks are LSP.                                                                                          |
| 2387–2523    | Multi-file transactions, expected hashes, overlap, WorkspaceEdit conversion                                     | Additional coverage; permissions and stale-write checks before application. Crash-atomic filesystem writes remain an unresolved contract, not proven by in-memory atomicity. |
| 2524–2567    | Parse/format/diagnostics/lint/build/test verification                                                           | Additional coverage; compact results must retain failures and evidence, not only a success summary.                                                                          |
| 2568–2661    | Reuse ctx-symbol/ctx-exec, proposed bitty-ai-code                                                               | Additional coverage; library reuse candidate, not dependency or new crate authorization.                                                                                     |
| 2662–2747    | Independent batch versus dependent typed pipeline                                                               | Runtime / Additional coverage; local dependency execution does not remove per-node permission/budget gates.                                                                  |
| 2748–2832    | Worked read/edit flow and staged roadmap                                                                        | Additional coverage; initial project map/edit/check, then LSP, then pipelines/persistent index/SCIP/semantic impact.                                                         |
| 2834–3290    | Rust runtime plus Lua agent/harness/workflow/context/tool composition                                           | Additional coverage; configuration versus engine boundary proposed; arbitrary composition cannot bypass host enforcement.                                                    |
| 3291–3499    | Rust-owned persistence, secrets, resources and enforcement; no unique mandatory harness                         | Additional coverage; Lua proposes, Rust enforces; this does not move all mechanisms into the terminal repository.                                                            |
| 3500–3802    | Project intent directory, init, agents, harness, knowledge                                                      | Additional coverage; versionable declarations, not accepted file schema or automatic execution.                                                                              |
| 3803–3973    | Project trust, restricted Lua, runtime-state separation                                                         | Additional coverage; mandatory trust/capability boundary, XDG-aware machine-local state.                                                                                     |
| 3974–4024    | Explicit configuration precedence                                                                               | Additional coverage; built-in → user → project → session/CLI for ordinary settings, bounded by non-overridable security policy.                                              |
| 4026–4105    | Repository-local AI interface                                                                                   | Additional coverage; project knowledge, tools, workflows and intent, not granted authority.                                                                                  |
| 4106–4318    | Project skills, MCP declarations versus implementation, external service placement                              | Additional coverage; skill/tool/protocol distinction, declaration never starts a server by itself.                                                                           |
| 4319–4374    | Skill versus Tool versus MCP vocabulary                                                                         | Additional coverage; knowledge/workflow versus capability versus protocol.                                                                                                   |
| 4376–4471    | Versionable skills and explicit MCP trust                                                                       | Additional coverage; reviewable project material, server spawn passes permission engine.                                                                                     |
| 4473–4607    | Global/project skills and MCP, explicit namespaces, harness composition, Project Agent Manifest                 | Additional coverage; namespace/shadowing semantics and manifest schema remain proposals.                                                                                     |

## Explicit exclusions and unresolved coverage

- 013:47–131,179–229 and 288–313: theme adaptation, Markdown/BiDi/media
  rendering, generic Lua reuse, music visualization, bundled-plugin categories,
  and editor philosophy are terminal/plugin-owned. Their AI-facing semantic
  observation and composition consequences are retained above; no renderer or
  plugin implementation is distilled into an AI requirement.
- 017:28–331: detailed terminal, workspace and plugin implementation survey is
  outside this AI distillation. Its historical M1–M8 summary and the lifecycle,
  verification and optional-integration recommendations are retained, not
  excluded wholesale.
- 018's terminal mechanism discussion at 1324–1663 is boundary evidence, not
  an excluded source block. Likewise, embodied workspaces and governance
  freshness have AI consequences and remain documented proposals/assessments.
- Named crates, protocol versions, security advisories, upstream comparative
  claims, performance measurements and release dates are not validated by
  repetition. Only the primary-source slices in the companion ledger are
  verified. This task does not close identity, workspace-overlay, CarryCtx
  backend, agent-growth, dependency, protocol or release-scope decisions.

## Provenance limitation

The recordings have no Git provenance established by this task; their SHA-256
fingerprints make the checked line ranges identifiable. The three available
reference clones do have verified immutable revisions and inspected MIT license
files, recorded with concrete source paths in the companion
[primary-source ledger](research-distillation-013-017-018.md#primary-source-inspection-ledger).
Aider's exact missing paths are recorded there. Neither clean source nor a
license file proves runtime correctness, security completeness, dependency
compatibility, or a production-ready Bitty feature.

## Source identity: records 039 and 040 (partial)

Paths are workspace-relative; these are content fingerprints, not Git commits
or evidence that the records' factual claims are true. Both files are
untracked in the research repository, so provenance is path plus SHA-256, and
neither file was renamed, edited, or staged by the distilling task: the
`bitty`-side pass still needs both.

| Source                   | Lines | Bytes   | SHA-256                                                            |
| ------------------------ | ----- | ------- | ------------------------------------------------------------------ |
| `research/origin/039.md` | 6,079 | 123,712 | `d5559e19bdeb73b8a71a03b79f2ed7f8f666ac7cfeecc5d7bd43159ced28c46d` |
| `research/origin/040.md` | 2,215 | 42,579  | `a0c714f16b2db8e48e078d61a570ab01a8c0e60edee14b118da96901fe0a234b` |

Record 039 is five pasted rounds of one 15-section conversation: rounds 1-4
(`039.md:46-1258`, `1259-2471`, `2472-3684`, `3685-4897`) are byte-identical
(each round MD5 `1722f56ee75e9ac76eef1f90c187113b`); round 5
(`039.md:4898-6079`) equals round 1 minus the trailing 31-line conclusion
block. Round-1 ranges (`039.md:1-1258`) are canonical; later rounds were not
re-extracted. Record 040 is a single pass with distinct sections; no
duplication handling applies.

Verify with `sha256sum "$BITTY_WORKSPACE/research/origin/039.md"` and
`sha256sum "$BITTY_WORKSPACE/research/origin/040.md"` plus
`wc -l -c` on both paths. The record-040 fingerprinted head keeps verifying
even after appends: `head -n 1829 "$BITTY_WORKSPACE/research/origin/040.md" |
sha256sum` and `head -c 31779 "$BITTY_WORKSPACE/research/origin/040.md" |
sha256sum` must both print
`067e3c287b203ccd9a3217c1596d55cb7181b4746d07076e1023e1ddfc2cfe0e`. Post-verification note: record 040 grew by a pure
append after the distilling task verified its head (first 1,829 lines and 31,779
bytes still hash to the head fingerprint above; appended tail
`040.md:1830-2215`, 386 lines of comparable-programs discussion, distilled in
the 040 addendum). Every 040 head range
cited in this ledger still verifies against the head fingerprint; the
whole-file fingerprint in the table above matches the current file, which has
not grown since the append was recorded.

## Disposition: 039/040 partial distillations

The companion drafts are
[Panel research distillation for bitty-ai (039)](research-distillation-039-bitty-ai.md)
and
[Plugin-system research distillation for bitty-ai (040)](research-distillation-040-bitty-ai.md).
Each carries its own provenance block, topic-traceability table, and explicit
exclusions. Both are draft discussion syntheses: the layered models they
record (Panel object model, two-level extension model, Host Plugin,
two-layer `bitty-ai-runtime` split, three-layer model) are candidate inputs
to the draft AI architecture and its related draft dispositions, not accepted
contracts. The accepted [IPC contract](../ipc-agent-rfc.md) is unaffected.
Neither draft creates or closes an AIQ or OQ identifier, duplicates or
modifies an existing canonical document, or describes implementation as
shipped. Source repositories under `recording/references` remain untrusted
read-only research material.

## Topic-level traceability: record 039 (partial)

Section names below refer to the companion 039 distillation unless a linked
existing document is named. Round-1 ranges are canonical.

| Source lines | Topic                                                                                        | Disposition / destination                                                                                                     |
| ------------ | -------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| 1-14         | Opening Panel question                                                                       | Boundary context in the 039 distillation; motivates the synthesis, not a distilled claim.                                     |
| 15-45        | Head conclusion, AI Workspace in object model                                                | 039 AI Workspace section; retain hosting direction with the R1 projection-only rule; analogy is motivation, not architecture. |
| 46-83        | Panel identity, lifecycle, `PanelContent` enum, `Panel != Pty`                               | 039 identity and lifecycle section; retain separation and host-owned lifecycle; vocabulary is discussion-only, not registry.  |
| 84-145       | Modes, floating-mode AI, shared workspace operations                                         | 039 floating-mode section; retain mode-as-property and uniform operations; reject agent-driven windowing; defer key bindings. |
| 407-489      | Rust `TextEditor` primitive, AI prompt editor as consumer, `Document != View != Panel`       | 039 TextEditor section; retain consumer-only direction with R2 gating on editing effects; defer API shape.                    |
| 1067-1153    | IDE question, Agent Workspace composition (Agent, Terminal, Diff, Task Board, Logs, Browser) | 039 Agent Workspace section; retain composition as illustration with per-panel authorization; reject composition as plan.     |
| 1154-1258    | Six-step sequencing, closing object model, Panel-as-host principle sentence                  | 039 sequencing section; sequencing retained as author opinion; principle sentence retained as candidate.                      |
| 1259-6079    | Rounds 2-5 duplicates                                                                        | Not re-extracted; byte-identity recorded in the 039 provenance section.                                                       |

## Topic-level traceability: record 040 (partial)

Section names below refer to the companion 040 distillation unless a linked
existing document is named. Boundary sections read but not distilled are
marked as such.

| Source lines  | Topic                                                                                        | Disposition / destination                                                                                                       |
| ------------- | -------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| 138           | AI Activity among panel activities                                                           | 040 boundary context; AI as hosted content, consistent with R1 projection-only panels.                                          |
| 229           | AI in development-environment composition                                                    | 040 boundary context; composition illustration, not a required panel set.                                                       |
| 244-254       | AI workspace composition (Agent, Tasks, Terminal, Diff, Browser, Logs)                       | 040 boundary context.                                                                                                           |
| 309, 324, 377 | Bitty AI as a grown application, first-class runtime peer                                    | 040 boundary context; application-not-Core positioning retained as objective.                                                   |
| 402           | Motivating question on agent-plugin extension                                                | 040 two-level model section; retained question framing the model.                                                               |
| 437           | Two-level extension model (primitives, AI runtime, AI extensions)                            | 040 two-level model section; retain mediation direction with R2 gating; defer topology.                                         |
| 460-477       | Bitty AI as Host Plugin tree                                                                 | 040 Host Plugin section; retain dual role and flat runtime; example children are illustration only.                             |
| 706-778       | Bitty AI as plugin-platform plugin (registries, Agent UI, extension catalog, slash commands) | 040 plugin-platform section; retain decomposition principle; registry and command names are proposals; transport stays with R2. |
| 779-851       | Extension Points, `bitty-ai.*` declarations                                                  | 040 Extension Points section; retain typed attachment; point names are proposals needing capability review.                     |
| 852-911       | Manifest relations and dependency resolution                                                 | 040 Manifest section; retain dependency honesty; sketches are illustration; declaration never grants permission.                |
| 912-998       | Permission non-inheritance, Bitty AI emphasis                                                | 040 permissions section; retained as the load-bearing draft principle with default-deny for extensions behind R2.               |
| 999-1062      | Service API versioning (`bitty-ai.tools`, `bitty-ai.context` style)                          | 040 versioning section; retain decoupling objective; version strings are illustration; defer policy.                            |
| 1295-1364     | `bitty-ai-runtime` versus `bitty-ai` split, three-layer model                                | 040 layer-split section; retained as candidate input only; weakest structural claim; not a crate, package, or release decision. |
| 1131-1294     | Plugin tree, native-versus-extension axis, small base                                        | Read, not distilled; axis and base-minimality inform the layer split; tree mechanics are `bitty`-side.                          |
| 1365-1505     | Non-AI native capabilities, browser analogy                                                  | Read, not distilled; runtime-selection criteria are `bitty`-side handoff input.                                                 |
| 1698-1829     | Native ABI caution, four-level ladder                                                        | Read, not distilled; ABI caution is `bitty`-side engineering; ladder not accepted here.                                         |
| 1830-2215     | Appended comparable-programs round (Emacs, VS Code, JupyterLab, Eclipse RCP, Neovim, Zed)    | 040 addendum; bitty-ai-relevant parts retained as candidate input, remainder excluded as `bitty`-side.                          |

## Explicit exclusions: records 039 and 040

- 039 widget-layer, terminal-as-widget, effects, focus and idle visual-state,
  application-services, editor-application, and startup-performance sections
  are `bitty`-side terminal or plugin-ecosystem design. Their AI-facing
  consequences (host-owned lifecycle, consumer-only editing, composed
  workspace with per-panel authorization) are retained in the 039
  distillation; no renderer, widget, service, or editor implementation is
  distilled into an AI requirement.
- 039 repository and RFC citations carry no pinned revision in the source and
  were not independently verified; they are discussion claims, not evidence,
  and cannot ground scope.
- 040 editor extension API, syntax highlighting, LSP layering, statusline,
  Docker, media, search, browser, and developer-runtime sections are
  `bitty`-side or plugin-ecosystem design. Their AI-facing consequences
  (mediated extension, typed points, non-inheritance, versioned surfaces) are
  retained in the 040 distillation; no editor, registry, or runtime
  implementation is distilled into an AI requirement.
- 040 registry names, extension-point names, manifest fields, version
  strings, key bindings, and composition lists are unreviewed sketches, not
  adopted interfaces, defaults, or plans. Performance and effortlessness
  assertions in both records are unmeasured and no conclusion is drawn from
  them here.
- This task does not close identity, workspace-overlay, CarryCtx backend,
  agent-growth, dependency, protocol, registry, manifest, versioning, or
  release-scope decisions, and it changes no normative contract.
