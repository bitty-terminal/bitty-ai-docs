---
title: Research coverage ledger
description: Coverage and disposition of AI-relevant research records 013, 017, 018, 025, and partial 039/040/041/044/045/046/048/049/050/051/052/053/054/055
category: specifications
audience: mixed
document_type: register
status: draft
website_publish: false
sidebar_order: 21
---

# Research coverage ledger

This ledger maps specific source topics to the
[distillation](research-distillation-013-017-018.md), to the
[prefix-cache context design](../prefix-cache-context-design.md) for record
025, and to the 039/040/041/044/045/046/048/049/050/051/052/053/054/055 companion distillations, rather than claiming
that an entire recording or specification is covered. Line ranges are
inclusive, verified against the local recordings identified below. Repeated
examples are condensed, not adopted as API definitions.

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
complete the delivery lifecycle.

Historical snapshot (2026-09-14): the companion distillation and this ledger
were captured locally in
`fa18390586babc2c63555c72e691ffb8463a7d76` (PR #7, merged 2026-09-14,
closing issue #6). That capture records local committed history only: it is
not architecture acceptance, not product verification, and not evidence of
current remote publication, task, or PR status. The delivery state of later
drafts is tracked separately and is not inferred here.

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

## Source identity: record 025 (partial, already distilled)

Paths are workspace-relative; these are content fingerprints, not Git commits
or evidence that the record's factual claims are true. The origin file is
tracked in the research repository; the distilling task neither renamed,
edited, nor staged it.

| Source                   | Lines | Bytes  | SHA-256                                                            |
| ------------------------ | ----- | ------ | ------------------------------------------------------------------ |
| `research/origin/025.md` | 1,404 | 17,288 | `e2adbbcb22a6e4e84b2e4eef5ad955c5dbb89d292ad2091d021fefb873d85b6a` |

Record 025 is a single pass with one framing principle, twenty-three numbered
sections, and a closing pipeline summary; no duplication handling applies.
The fingerprint above matches `origin/025.md.completed`: the
same content carries the `.completed` suffix because the distillation
([Prefix-Cache-Friendly Context Design](../prefix-cache-context-design.md),
`sidebar_order: 26`, CTX-0017) recorded its conclusions in this repository.

Verify with `sha256sum "$BITTY_WORKSPACE/research/origin/025.md.completed"`
plus `wc -l -c` on the same path. The distillation's provenance block cites
the pre-rename `recording/research/025.md` copy (1,404 lines, same SHA-256);
that staging path no longer exists at the workspace root, so verify against
the current `research/origin/025.md.completed` path.

## Disposition: 025 partial distillation (Captured)

The companion draft is
[Prefix-Cache-Friendly Context Design](../prefix-cache-context-design.md)
(`sidebar_order: 26`): stable-before-dynamic layering, deterministic
serialization, session-pinned registry snapshots, append-only epochs with
structural compaction, provider qualifications, and privacy controls, with a
v0.1 scope boundary marking the planner, epoch, snapshot, content-addressed
block, routing, multi-agent, and observability material as later proposals.
It is a draft discussion synthesis: the layering, epoch, snapshot, planner,
content-addressed block, and routing models it records are candidate inputs
to the draft AI architecture and its related draft dispositions, not accepted
contracts. The accepted [IPC contract](../ipc-agent-rfc.md) is unaffected.
The draft creates no AIQ or OQ identifier and closes none; it proposes
AIQ-12 (canonical serialization, Closed adopted-draft) and AIQ-13
(provider-scoped cache key and routing scope), whose register state stays
with [AI Unresolved Questions](../ai-unresolved-questions.md). Source
repositories under `recording/references` remain untrusted read-only
research material.

The 025 design covers context assembly only (ordering, canonicalization,
snapshots, epochs, observability). Provider-side cache behavior discussed in
025 sections 19-20 (routing, stickiness) is qualified, not adopted: no
cross-provider key-value reuse, no breakpoint or pricing standard, and no
v0.1 routing or affinity contract. The companion `CacheKey`/`CacheScope`
mechanism in the sibling `bitty-ai` runtime (read-only evidence, never
modified here) is the code-side traceability for the key-scope question,
not an implementation of the 025 design: main at `a8d3422` carries
AI-0082 (`fdb37c5`, tests `crates/bitty-ai-runtime/tests/cache_key.rs`)
with a provider-scoped `(provider_id, model_id, scope,
stable_prefix_hash, prefix_len)` key and FNV-1a-64 digest over the stable
prefix. The marker-collision fix (AI-0084, length-aware stable-prefix
boundary, no naive marker scan) merged in `bitty-ai` `2b984c4` (AI-0084) at
the time of writing: **record 025 is Captured — the AI-0084 merge resolves
the open collision cited at CTX-0053.**

## Topic-level traceability: record 025 (partial)

Section names below refer to the companion 025 design unless a linked
existing document is named. Source section numbers are the record's own
`# N.` headings; source ranges below map each section to its body (the
`---` separators at 31, 96, 198, 262, 342, 427, 473, 521, 585, 682, 738,
798, 838, 868, 936, 986, 1027, 1077, 1122, 1174, 1223, 1291, 1318, and 1351
are excluded from every range).

| Source lines | Topic                                                                             | Disposition / destination                                                                                                                                              |
| ------------ | --------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1-29         | Framing principle: longest-common-prefix stability; stable-to-dynamic layer order | 025 stable-prefix layering section; adopt stable-before-dynamic ordering, dynamic values in trailing layers or behind on-demand tools; seven-layer split is proposal.  |
| 33-94        | 1. Stable System Prompt; build once per epoch, per-turn values trailing           | 025 layering and invariant 2; adopt static system prompt with trailing turn-context block; defer serializer ownership.                                                 |
| 98-196       | 2. Deterministic Tool Schema; fixed ordering and field shape                      | 025 invariant 3 and deterministic-serialization section; adopt determinism as requirement on any future cache claim; canonical type and encoding open.                 |
| 200-260      | 3. Fully deterministic Serialization; canonical encoding proposal                 | 025 deterministic-serialization section; adopt canonical ordering prerequisite; block-hash sketch is optimization proposal, never a consent or budget substitute.      |
| 264-340      | 4. Stable Skill ordering; fixed core plus versioned session set                   | 025 invariant 6; adopt snapshot direction with explicit versioning; snapshot type, digest, and placement open; overlaps AIQ-08.                                        |
| 344-425      | 5. MCP/plugin Session Snapshot; pinned listings, no per-turn re-enumeration       | 025 invariants 5-6 and registry-snapshot section; adopt versioned session-pinned snapshots; representation and owner open.                                             |
| 429-471      | 6. Runtime State placement; full state trailing, never leading                    | 025 layering and invariant 4; adopt references-over-embedding consistent with CP-6; drill-down never widens authority.                                                 |
| 475-519      | 7. On-demand state; references over inline dumps                                  | 025 invariant 4; adopt addressable references with typed unavailability; panel state stays outside the agent prefix unless requested.                                  |
| 523-583      | 8. Append-only history; no silent mid-history rewrites                            | 025 invariant 1 and invariant 7; adopt append-only within one epoch with governed deletion propagation; tombstones retain no sensitive payload.                        |
| 587-680      | 9. Low-frequency structural `/compact`; newest-segment compression                | 025 invariant 7; adopt structural compaction over background rewriting; undo and disclosure tracked as AIQ-06 with its AIQ-57 facet.                                   |
| 684-736      | 10. Context Epoch; explicit invalidation boundary                                 | 025 invariant 5 and epoch section; adopt explicit epoch boundary; epoch schema beyond-v0.1, overlaps AIQ-06 without a new identifier.                                  |
| 740-796      | 11. Agent-loop hit rate; append-only consecutive turns                            | 025 invariant 1; retained as working hypothesis; no measured hit-rate claim; measurement requires replayed-trace evidence.                                             |
| 800-836      | 12. No per-turn System Prompt regeneration                                        | 025 invariant 2; adopt build-once-per-epoch static prompt; token-budget counters trailing.                                                                             |
| 840-866      | 13. Token Budget placement; budget text trailing or on demand                     | 025 invariant 2; adopt trailing budget placement; sharpest invalidation example retained as illustration.                                                              |
| 870-934      | 14. Repository Context snapshots plus deltas; explicit refresh                    | 025 registry-snapshot section; adopt immutable project snapshot with trailing deltas; refresh authorization is an AIQ-03/AIQ-04 facet, no new identifier.              |
| 938-984      | 15. Git-like base plus append-only commits; computable stability                  | 025 registry-snapshot section; retained as analogy and motivation, not a snapshot-versioning contract.                                                                 |
| 988-1025     | 16. Content-addressed Context Blocks; per-section hashes                          | 025 deterministic-serialization section; record block-hash sketch as serialization-work optimization; hashes are not cache keys and never substitute for policy.       |
| 1029-1075    | 17. Multi-agent shared prefixes within identical scope                            | 025 epoch section; retained as future hypothesis; permissible only within identical provider, model, tokenizer, and consent scope; not a v0.1 goal.                    |
| 1079-1120    | 18. Panel state outside the agent prefix; on-demand reads                         | 025 invariant 4 and provider qualifications; adopt separation; panel addressing creates no agent-side authority.                                                       |
| 1124-1172    | 19. Provider Routing scope; key includes provider and model                       | 025 provider qualifications; adopt provider identity in key scope; cross-provider reuse rejected as correctness boundary.                                              |
| 1176-1221    | 20. Sticky routing for self-hosted fleets only                                    | 025 invariant 8 and provider qualifications; qualified to self-hosted inference clusters; out of scope for v0.1, no client-visible contract.                           |
| 1225-1289    | 21. Context Planner proposal; ordering, canonicalization, budget                  | 025 epoch section; record as beyond-v0.1 structural proposal; overlaps ContextBuilder direction and AIQ-10/AIQ-29, no new identifier.                                  |
| 1293-1316    | 22. `/context` cache-friendliness observability                                   | 025 observability section; accept debugging direction with estimates labeled as estimates; interactive operations stay proposals under AIQ-04.                         |
| 1320-1349    | 23. Eight design invariants; adopt/qualify/reject/open judgment                   | 025 invariants section; adopt append-only, stable-first, deterministic serialization, reference-over-embed, versioned change, structural compaction; qualify locality. |
| 1353-1404    | Closing pipeline: size, stability, and hit rate as independent metrics            | 025 problem statement and privacy controls; shorter context is not faster when it rewrites early segments; cache never overrides budget, minimization, or consent.     |

## Explicit exclusions: record 025

- 025 provider-routing numbers, breakpoint syntax, minimum lengths, pricing
  behavior, and upstream harness claims are illustrative, never adopted
  defaults; provider and harness behavior cited by the source was not
  independently verified.
- 025 `ContextSerializer`, `Canonical Context Encoding`, `ContextEpoch`,
  `ToolRegistrySnapshot`, `ContextBlock`, `ContextPlan`, and Context Planner
  type names and schemas are unreviewed sketches, not adopted types,
  contracts, or module paths.
- 025 key-scope, breakpoint-placement, cache-retention, serializer-ownership,
  epoch-boundary, planner-placement, snapshot-refresh, and compaction-undo
  choices stay open (AIQ-03, AIQ-04, AIQ-06 with its AIQ-57 facet, AIQ-08,
  AIQ-10, AIQ-12, AIQ-13, AIQ-29); this task closes none of them and changes
  no normative contract.
- This task does not close identity, workspace-overlay, CarryCtx backend,
  agent-growth, dependency, protocol, registry, manifest, versioning, or
  release-scope decisions.

## Source identity: records 039, 040, 041, 044, 045, 048, 049, 050, 051, and 052 (partial), plus summaries 046, 053, 054, and 055 (AI slice)

Paths are workspace-relative; these are content fingerprints, not Git commits
or evidence that the records' factual claims are true. All ten origin files are
untracked in the research repository, so provenance is path plus SHA-256, and
none of the files was renamed, edited, or staged by the distilling tasks: the
`bitty`-side pass still needs all ten. The four summaries below are the
CTX-0074 and CTX-0076 AI-slice sources (read-only; origins unread and untouched by those
tasks); the `bitty`-side, plugin-side, Wheel-side, terminal-side, packaging-side, and
storage-side passes still need their halves.

| Source                    | Lines | Bytes   | SHA-256                                                            |
| ------------------------- | ----- | ------- | ------------------------------------------------------------------ |
| `research/origin/039.md`  | 6,079 | 123,712 | `d5559e19bdeb73b8a71a03b79f2ed7f8f666ac7cfeecc5d7bd43159ced28c46d` |
| `research/origin/040.md`  | 2,215 | 42,579  | `a0c714f16b2db8e48e078d61a570ab01a8c0e60edee14b118da96901fe0a234b` |
| `research/origin/041.md`  | 821   | 13,140  | `15182dc1d8b709a8d6a7f18de57f387e2db83f476fed12c8e5b19087387d9754` |
| `research/origin/044.md`  | 2,630 | 46,063  | `d00d7f6c5845d9964cd09bcf447c71f2759cf1322fec03a5da4d0e0d8d49c5d0` |
| `research/origin/045.md`  | 914   | 14,827  | `6d0954320cedfa43bc5c6fe0d6216f8a6f15974af87bff2765b629c35c6c6dd0` |
| `research/origin/048.md`  | 2,351 | 57,159  | `ddfd88b2eaa66e983d1bd9dc659c03ee7615db546d3eb92510c8e4d4444d0455` |
| `research/origin/049.md`  | 2,135 | 32,648  | `30b6c86d8c61b692d95527d487ac1ab523c59da324eadde77e6469f8920594aa` |
| `research/origin/050.md`  | 940   | 13,978  | `ce3b7f10baa003f99a2fe11824e0f4386ff55371324045edea776a19733a7d59` |
| `research/origin/051.md`  | 1,308 | 17,307  | `2c9ac93f653fc923be3e95bc275b88dbfa0f0dd0687fe339e571ceb338f3ef88` |
| `research/origin/052.md`  | 561   | 12,836  | `999b04e9a12f30ce68102999c027727fb2a144fa96c443e8363e87276d9d72bd` |
| `research/summary/046.md` | 16    | 4,694   | `381397ac14fdba77cd030436c7202f192430a7678eacfe6666f404d0c28f8954` |
| `research/summary/053.md` | 24    | 6,796   | `82dfe75628d7c960bfad7d94ad07c8fdcceb06970049c9f8b1fc5ceb42dfece5` |
| `research/summary/054.md` | 23    | 4,191   | `8b5d8b218e5bb3f8b4f32048e5bca99fcd96262a276a5ad4e189b58b2584088d` |
| `research/summary/055.md` | 28    | 7,251   | `a23dfd8fa93adbb530a4cf4ba881eaa0f9135eb5d00143254d84e6cdd81a09f8` |

Record 055 scope note: the fingerprint above covers the summary distilled by
CTX-0076 (AI slice only; panel, event-log, tool-runtime, pipeline, storage,
and dashboard halves as owner-pending pointers in the companion). The origin
`055.md` was unread and untouched by this task; it stays unrenamed until its
owners' captures land, and split-owner capture stays Partial until all owned
conclusions are accounted for. The summary landed on the research `main`
branch as `e7a18d3`, which satisfies the CTX-0076 dependency on the landing
task; CTX-0032 is completed and this record confirms the landed revision
instead.

Record 046/053/054 scope note: the fingerprints above cover the summaries
distilled by CTX-0074 (AI slice only; plugin and governance halves as
owner-pending pointers in the companion). Origins `046.md` (643 lines,
12,530 bytes), `053.md` (1,328 lines, 20,316 bytes), and `054.md` (741
lines, 12,298 bytes) were unread and untouched by that task; split-owner
capture stays Partial until all owned conclusions are accounted for.

Record 052 is a single pass: an opening plugin-between-plugins question, a
distribution-framing paragraph, a recommended-boundary table with the
mechanism rule, a UI-separation section, a model-management section, a tool
pluginization section, a Lua-to-Lua layering section, a dependency-graph
section, a `.wheel/` composition section, a meta-package section, and a
closing thesis with a summary formula; no duplication handling applies. The
fingerprint above was verified at task start and re-verified at task end
with no change, so the CTX-0045 growth pattern did not trigger: the 052
body distills the whole verified file (`052.md:1-561`), and any later append
is uncovered.

Record 048 is a single pass: an owner question block, a conclusion block, a
quality-formula section, eleven numbered quality sections, a prioritized
optimization order, a Harness-philosophy section, a Wheel-goal restatement,
seventeen Wheel design sections, a closing architecture, and a top
principle; no duplication handling applies. The fingerprint above was
verified at task start and re-verified at task end with no change, so the
CTX-0045 growth pattern did not trigger: the 048-049 body distills the whole
verified files (`048.md:1-2351`, `049.md:1-2135`), and any later append is
uncovered.

Record 049 is a single pass: a framing answer plus thirty numbered Context
Compiler sections with a closing five-subsystem split and reference list; no
duplication handling applies. The fingerprint above was verified at task
start and re-verified at task end with no change, so the CTX-0045 growth
pattern did not trigger: the 048-049 body distills the whole verified file
(`049.md:1-2135`), and any later append is uncovered.

Record 050 is a single pass: an opening rename-and-split statement, a
portable-capabilities section, a How-versus-What section with the entry
contract, eight function-class sections, a core-prompt section, a
no-runtime-state section, a Lua-trust section, a sandbox section, and a
closing layout with the boundary sentence; no duplication handling applies.
The fingerprint above was verified at task start and re-verified at task end
with no change, so the CTX-0045 growth pattern did not trigger: the 050-051
body distills the whole verified file (`050.md:1-940`), and any later append
is uncovered.

Record 051 is a single pass: a framing paragraph plus twenty-four numbered
sections and a closing philosophy; no duplication handling applies. The
fingerprint above was verified at task start and re-verified at task end
with no change, so the CTX-0045 growth pattern did not trigger: the 050-051
body distills the whole verified file (`051.md:1-1308`), and any later
append is uncovered.

Record 045 is a single pass with one framing diagram, one core principle,
twelve numbered sections plus a closing strengths section; no duplication
handling applies. The fingerprint above was verified at task start and
re-verified at task end with no change, so the CTX-0045 growth pattern did
not trigger: the 045 body distills the whole verified file
(`045.md:1-914`), and any later append is uncovered.

Record 044 is a single pass with twenty-eight numbered sections plus a
four-point follow-up with its own mechanism-versus-semantics split, boundary
table, repository placement, IPC contract, and worked example; no duplication
handling applies. The fingerprint above was verified at task start and
re-verified at task end with no change, so the CTX-0045 growth pattern did
not trigger: the 044 body distills the whole verified file
(`044.md:1-2630`), and any later append is uncovered.

Record 039 is five pasted rounds of one 15-section conversation: rounds 1-4
(`039.md:46-1258`, `1259-2471`, `2472-3684`, `3685-4897`) are byte-identical
(each round MD5 `1722f56ee75e9ac76eef1f90c187113b`); round 5
(`039.md:4898-6079`) equals round 1 minus the trailing 31-line conclusion
block. Round-1 ranges (`039.md:1-1258`) are canonical; later rounds were not
re-extracted. Record 040 is a single pass with distinct sections; no
duplication handling applies. Record 041 is a single pass with twelve
numbered sections plus a closing three-layer model; no duplication handling
applies.

Verify with `sha256sum "$BITTY_WORKSPACE/research/origin/039.md"`,
`sha256sum "$BITTY_WORKSPACE/research/origin/040.md"`, and
`sha256sum "$BITTY_WORKSPACE/research/origin/041.md"` plus
`wc -l -c` on all three paths. The record-040 fingerprinted head keeps verifying
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

## Disposition: 039/040/041/044/045/046/048/049/050/051/052/053/054/055 partial distillations

The companion drafts are
[Panel research distillation for bitty-ai (039)](research-distillation-039-bitty-ai.md),
[Plugin-system research distillation for bitty-ai (040)](research-distillation-040-bitty-ai.md),
[IPC-value research distillation for bitty-ai (041)](research-distillation-041-bitty-ai.md),
[Execution-supervisor research distillation for bitty-ai (044)](research-distillation-044-bitty-ai.md),
[Lua-versus-Core safety-boundary research distillation for bitty-ai (045)](research-distillation-045-bitty-ai.md),
[Wheel scope and framework-illustration research distillation for bitty-ai (046/053/054)](research-distillation-046-053-054-bitty-ai.md),
[Quality-formula and Context-Compiler research distillation for bitty-ai (048-049)](research-distillation-048-049-bitty-ai.md),
and
[Wheel-config and Git-model research distillation for bitty-ai (050-051)](research-distillation-050-051-bitty-ai.md),
and
[Wheel decoupling and Core-Plugin boundary research distillation for bitty-ai (052)](research-distillation-052-bitty-ai.md),
and
[Event-Sourced Agent Workspace research distillation for bitty-ai (055)](research-distillation-055-bitty-ai.md).
Each carries its own provenance block, topic-traceability table, and explicit
exclusions. All ten are draft discussion syntheses: the layered models they
record (Panel object model, two-level extension model, Host Plugin,
two-layer `bitty-ai-runtime` split, three-layer model, Capability Layer,
Execution Supervisor with the mechanism-versus-semantics split, four-layer
safety boundary with the intersection-only authority rule, fixed-Coding-domain
Wheel scope with open Roles reconciled against the architecture candidates,
multiplicative
quality formula with the Context Compiler and Verification Runtime, Cold,
Warm, and Hot stratification with stability zones and the pass pipeline,
`.wheel` configuration classes with the portable-capability split,
Git-inspired content-addressed context DAG with checkpoint, branch, merge,
and GC discipline, Wheel decoupling with the mechanism-versus-policy rule,
two-layer Lua composition, distribution-as-composition, and the 053
owner-pending provider, tool-schema, and streaming illustrations with the
054 host-services and trust-boundary notes, and the 055 six-object split with
founding inequalities, agent-graph versus task-graph split, mailbox IPC with
first-class context share, typed merges with diff inventory, Context GC
versus compaction, Task-as-Issue with reason-as-commit-message, and the
four-clause core principle) are candidate inputs
to the draft AI architecture and its related draft dispositions, not accepted
contracts. The accepted [IPC contract](../ipc-agent-rfc.md) is unaffected.
No draft creates or closes an AIQ or OQ identifier, duplicates or
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

## Topic-level traceability: record 041 (partial)

Section names below refer to the companion 041 distillation unless a linked
existing document is named. All `agent.*` and `panel.*` names are discussion
inputs; the accepted Agent and IPC vocabulary stays with the IPC and Agent
RFC, which the 041 draft references without restating.

| Source lines | Topic                                                                                      | Disposition / destination                                                                                                     |
| ------------ | ------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------- |
| 1-2          | Opening IPC question                                                                       | 041 boundary context; motivates the synthesis, not a distilled claim.                                                         |
| 3-78         | IPC as second extension boundary; plugin-kind table; out-of-process tree; language freedom | 041 boundary section; retain separation as candidate objective; platform reframing is motivation; defer capability split.     |
| 79-151       | Heavyweight shape; `bitty-ai` daemon; Core provider ignorance; `agent.*` sketches          | 041 daemon section; retain daemon direction and provider ignorance; sketches are discussion inputs; defer to AI architecture. |
| 152-223      | Panel as public abstraction; operation sketches; Agent panel entry                         | 041 panel section; retain abstraction direction; operations are proposals; Agent entry is positional context under R1.        |
| 224-285      | Agent/Panel lifecycle separation; Agent A diagram; operation sketches                      | 041 lifecycle section; retain separation direction; lifecycle and operations are RFC-owned discussion inputs only.            |
| 286-370      | IPC bus; plugin-to-plugin flow; `agent.*` event examples; shared Lua/IPC semantics         | 041 bus section; retain bus direction; all event names are discussion inputs; delivery and auth entirely open.                |
| 371-449      | Capability API unity; triple-frontend example; capability sketches                         | 041 capability section; retain unity objective as candidate; names are vocabulary; defer owner, versioning, enforcement.      |
| 450-510      | Manifest sketch; capability token; per-capability checks; permission display               | 041 permissions section; retain boundary-enforcement placement; align with R2 backend and 040 non-inheritance.                |
| 511-558      | Crash isolation (section 8)                                                                | Read, not distilled; process supervision is `bitty`-side; consequence covered by the daemon direction.                        |
| 560-609      | Debugging and DevTools sketches (section 9)                                                | Excluded (`bitty`-side tooling design); no `bitty-ai` contract follows.                                                       |
| 611-658      | Controlling a running Bitty; `agent ask` sketch (section 10)                               | Excluded (`bitty`-side control design); `agent ask` proposes no accepted CLI surface.                                         |
| 660-697      | Multi-instance sockets and addressing (section 11)                                         | Read, not distilled; addressing is `bitty`-side and partly RFC-owned; no `bitty-ai` contract follows.                         |
| 699-735      | Lua-plugin fit and rationale (section 12, first half)                                      | Read, not distilled; latency and UI-coupling rationale is `bitty`-side engineering.                                           |
| 736-761      | IPC-plugin fit list and characteristics (section 12, second half)                          | 041 split section; retain as AI placement rule and heuristic needing per-case review.                                         |
| 765-821      | Closing three-layer model; Capability Layer; capability list; downstream motivation        | 041 layer section; candidate input only; weakest structural claim; do not merge silently with the 040 layering.               |

## Topic-level traceability: record 044 (partial)

Section names below refer to the companion 044 distillation unless a linked
existing document is named. All tool, method, event, struct, and enum names
are discussion inputs; the accepted Agent and IPC vocabulary stays with the
IPC and Agent RFC, which the 044 draft references without restating. The
whole verified file (`044.md:1-2630`) was distilled; no head-versus-tail
split applies.

| Source lines | Topic                                                       | Disposition / destination                                                                         |
| ------------ | ----------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| 1-36         | Harness anecdote and opening questions                      | 044 boundary context; motivates the synthesis, not a distilled claim.                             |
| 37-60        | Core thesis: supervisor and Job Service, not stronger spawn | 044 thesis section; retain layering as candidate objective; defer ownership, packaging, schedule. |
| 61-148       | Harness table; pueue detachment, groups, dependencies, logs | 044 survey section; retain waiting and subscription principles; borrow-design-not-backend.        |
| 149-224      | OpenCode event-driven notification                          | 044 survey section; retain event-over-polling direction; PTY-session shape is backend detail.     |
| 225-268      | Codex process protocol; polling critique                    | 044 survey section; retain no-model-polling rule; deficiency claims are author opinion.           |
| 269-308      | Claude background tasks, Monitor, session separation        | 044 survey section; retain process-job versus agent-task separation.                              |
| 309-356      | Cursor Agent/Run split, event stream, subscriptions         | 044 survey section; retain subscription direction as target shape.                                |
| 357-411      | Four-object model and invariance claims                     | 044 four-object section; retain separation as candidate; invariance conditioned on Lifetime.      |
| 412-476      | Job not belonging to Panel; mailbox routing                 | 044 routing section; retain routing direction; mailbox mechanics undecided.                       |
| 477-523      | Owner plus Subscriber; observe versus control               | 044 ownership section; permission names unreviewed; enforcement stays host-side.                  |
| 524-593      | Agent-killed disposition; Lifetime sketch                   | 044 lifetime section; explicit-policy direction; Detached durability deferred.                    |
| 594-654      | Pre-exit quiescence gate as mechanism                       | 044 gate section; invariant direction; gate mechanics need owning-task design.                    |
| 655-723      | Hard, idle, and retention clocks; anti-default rule         | 044 timeout section; retain clock separation; uniform kill rule rejected.                         |
| 724-765      | Job kind taxonomy                                           | 044 kind section; heuristic needing per-case review, not a decision procedure.                    |
| 766-808      | Closed-stdin default; PTY only for interactive              | 044 stdin section; retain default-closed direction; spawn-options shape open.                     |
| 809-849      | Supervisor-held output; bounded completion                  | 044 output section; retain retention-and-reference direction; bounds open.                        |
| 850-918      | Completion as Critical Message; Unknown rule                | 044 completion section; consistency note; critical semantics stay R5-owned.                       |
| 919-997      | Three partition classes and resume cursor                   | 044 partition section; retain taxonomy; no supervisor inference.                                  |
| 998-1052     | Structured outcomes; OOM rule; tree kill                    | 044 outcome section; enum unreviewed; OOM backends unevidenced.                                   |
| 1053-1110    | No-default retry; Unknown-inspect-first                     | 044 retry section; consistency note; vocab and backoff open.                                      |
| 1111-1188    | `exec` and `job_*` tool sketches                            | Excluded; unreviewed tool surface; proposes no accepted command or tool.                          |
| 1189-1233    | Argument vectors versus shell default                       | Read, not distilled; `bitty`-side detail with R2-owned consequences.                              |
| 1234-1286    | Visibility console mockup                                   | Excluded; illustration without data contract or authorization analysis.                           |
| 1287-1344    | Owner handoff with generation                               | Read; restates R5 fencing for Jobs; no new mechanism.                                             |
| 1345-1410    | Observation versus Critical events; no-wake rule            | 044 progress section; retain split and no-wake direction.                                         |
| 1411-1455    | Input-needed event; WaitingInput; secret boundary           | 044 input section; candidate direction; needs security review.                                    |
| 1456-1499    | SQLite metadata with filesystem logs and artifact refs      | 044 persistence section; all post-v0.1 under R6; schema open.                                     |
| 1500-1554    | v0.1 boundary; three-phase sketch                           | 044 boundary section; scope carried; phases are staging opinion.                                  |
| 1555-1653    | Borrow-design synthesis and architecture diagram            | 044 survey close; retain composition direction; proposes no backend.                              |
| 1654-1730    | Dedicated-supervisor-document proposal                      | Recorded as author proposal; no document created or adopted here.                                 |
| 1731-1755    | Mechanism versus semantics framing                          | 044 framing section; retain framing as candidate.                                                 |
| 1756-1851    | Job-to-Task binding cut with workspace refs                 | 044 binding section; raw-path execution rejected.                                                 |
| 1852-1931    | Authoritative `ExecutionResult` layer                       | 044 result section; retain anti-conflation and credibility ordering.                              |
| 1932-2011    | Semantic `JobResult` and artifact layers                    | 044 result section; interpretation-above-fact layering.                                           |
| 2012-2094    | Host capability enforcement                                 | 044 security section; consistent with the CTX-0047 comparison.                                    |
| 2095-2123    | AI-side claim and ownership semantics                       | 044 security section; retain coordination ownership.                                              |
| 2124-2181    | Dual staleness with dual generations                        | 044 security section; retain distinction as load-bearing.                                         |
| 2182-2263    | Cancel mechanism versus policy                              | 044 cancel section; outcomes reported, never assumed.                                             |
| 2264-2316    | Supervisor-held deadlines; policy selection                 | 044 timeout section; retain deadline-holding direction.                                           |
| 2317-2366    | Timeout-versus-Failed analysis; outcome split               | 044 timeout section; strongest timing claim; interpretation stays policy.                         |
| 2367-2396    | Full boundary table                                         | 044 boundary section; candidate input; interface cells are vocabulary.                            |
| 2397-2499    | Repository placement; wrapper principle                     | 044 placement section; retain agnosticism rule and principle.                                     |
| 2500-2526    | Shared generic IPC direction                                | 044 placement section; operation names propose no wire method.                                    |
| 2527-2630    | T128, J31, and E77 worked example                           | 044 example section; binding illustration, not registry.                                          |

## Topic-level traceability: record 045 (partial)

Section names below refer to the companion 045 distillation unless a linked
existing document is named. All budget, capability, call, error, and scope
names are discussion inputs; the accepted Agent and IPC vocabulary stays with
the IPC and Agent RFC, which the 045 draft references without restating. The
whole verified file (`045.md:1-914`) was distilled; no head-versus-tail
split applies.

| Source lines | Topic                                                           | Disposition / destination                                                                          |
| ------------ | --------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| 1-25         | Four-layer diagram; Lua-decides-how principle                   | 045 architecture section; retain layering and principle as candidate; Lua never self-authorizes.   |
| 27-52        | Review-workflow agent counts; Host-User-Project effective value | 045 ceiling section; retain intersection rule; numbers are illustration; strictest-wins.           |
| 54-89        | Commander budget tree; fan-out counterexample; budget sketch    | 045 ceiling section; retain attenuation invariant; struct unreviewed; Lua schedules inside budget. |
| 93-168       | Resource enforcement; cgroup and Job Objects backends           | Excluded; `bitty`-side host design; backends and limit names are host evidence.                    |
| 172-241      | Panel lease sketch; handoff; Lua scheduling discipline          | Read; lease mechanism is `bitty`-side; discipline split already carried above.                     |
| 245-320      | `.env` and credential authorization model                       | Excluded; `bitty`-side filesystem authorization; no-bypass kept as boundary context.               |
| 324-381      | Secret-handle pattern; host injection; four never-hold places   | 045 secret section; retain handle direction; URI and store mechanics open.                         |
| 385-437      | Root and sudo escalation; approval flow                         | Excluded; `bitty`-side privilege authorization; envelope unreviewed.                               |
| 441-479      | Hard versus Policy versus Strategy table                        | 045 classification section; retain `bitty-ai`-owned rows only; no mechanism adopted.               |
| 483-546      | Self-grant counterexample; six-way intersection; shrink-only    | 045 intersection section; retain shrink-only rule; `bitty-ai` computes three factors.              |
| 550-601      | Subagent no-copy rule; narrowing illustration; subset invariant | 045 subagent section; retain invariant as candidate; profiles and scopes are illustration.         |
| 605-667      | Commander as Lua concept; eight primitives; team shapes         | 045 commander section; retain vocabulary cut; primitive list is vocabulary; roles stay in Lua.     |
| 670-727      | User-Hygiene split; preventable versus user-only                | 045 hygiene section; retain split; example bindings are illustration; system first.                |
| 731-787      | Four-layer stack; intersection priority; path narrowing         | 045 intersection section; retain composition order and strictest-wins.                             |
| 791-850      | Controlled-request Lua API; raw-escape-hatch note; error names  | 045 API section; retain request direction; sandbox restriction needs its own task.                 |
| 854-914      | Primitives list; official harness; unbreakable boundaries       | 045 API section; retain placement; behavior-versus-boundary principle kept.                        |

## Topic-level traceability: records 048 and 049 (partial)

Section names below refer to the companion 048-049 distillation unless a
linked existing document is named. All formula symbols, struct and enum
names, tool names, hook names, metric names, and CLI spellings are
discussion inputs; the accepted Agent and IPC vocabulary stays with the IPC
and Agent RFC, which the 048-049 draft references without restating. The
whole verified files (`048.md:1-2351`, `049.md:1-2135`) were distilled; no
head-versus-tail split applies.

| Source lines  | Topic                                                                    | Disposition / destination                                                                                       |
| ------------- | ------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------- |
| 048:1-30      | Owner questions; Codex behavior; skill-minimal habit                     | 048-049 boundary context; motivates the synthesis; directory habit is compatible context for the 050-051 split. |
| 048:31-77     | Multiplicative quality formula; factor table; verification primacy       | 048-049 formula section; retain framing and bottleneck argument; formula is a device, not a model.              |
| 048:78-145    | Window versus attention; efficiency ratio; cited studies                 | 048-049 pollution section; retain efficiency objective; citations unverified; no threshold adopted.             |
| 048:146-207   | Pollution catalog; clearing direction; cargo reduction                   | 048-049 pollution section; retain lifecycle sentence and reduction pattern; counts are illustration.            |
| 048:208-271   | Layered code reading; viewer and map citations; level sketch             | 048-049 code-reading section; retain topology-first order; sizes are anecdote; levels are vocabulary.           |
| 048:272-361   | Tool-parameter failures; intent-hiding rule                              | 048-049 ergonomics section; retain expose-intent rule; sketches are illustration; transport stays with R2.      |
| 048:362-426   | MCP counts; selection degradation; Tool Search direction                 | 048-049 ergonomics section; retain activation shape; counts are unverified figures.                             |
| 048:427-581   | Progressive disclosure limits; four collisions; Skill Hell sum; Resolver | 048-049 capability section; retain Hell decomposition and resolver placement; scores are illustration.          |
| 048:582-622   | Supply-chain figures; poisoning reports; metadata and tiers              | 048-049 capability section; retain no-implicit-trust rule; figures describe one corpus only.                    |
| 048:623-779   | Prompt over-specification; map-not-manual; need-driven discovery         | 048-049 prompt section; retain middle-height and need-driven rules; no prompt text adopted.                     |
| 048:780-866   | Threshold-summarize critique; lifecycle classes; hygiene                 | 048-049 state section; retain hygiene cadence; classes are vocabulary.                                          |
| 048:867-924   | Subagent costs; parallel-gain inequality                                 | 048-049 scheduling paragraph; retain cost inequality; numbers are illustration.                                 |
| 048:925-963   | Ten-item order; Harness slogan and diagram                               | 048-049 order section; retain resourcing opinion and slogan; order shifts by task.                              |
| 048:1035-1051 | Reference list [1]-[16]                                                  | Provenance only; no cited claim reproduced as a finding.                                                        |
| 048:1054-1078 | Wheel goal; seven-module sketch; compiler-plus-verification priority     | 048-049 decomposition section; retain goal and investment order; candidate topology.                            |
| 048:1079-1124 | Core-versus-Wheel boundary diagram                                       | 048-049 decomposition section; retain ignorance rule; no crate or team split implied.                           |
| 048:1125-1226 | Compiler-as-view; append anti-pattern; knowledge superset                | 048-049 decomposition section; retain per-turn-view metaphor and superset inequality.                           |
| 048:1227-1296 | Lifecycle state model; cargo-state example                               | 048-049 state section; retain separation and hygiene direction; fields are vocabulary.                          |
| 048:1297-1384 | Terminal-substrate advantage; structured reads                           | 048-049 substrate paragraph; retain association reads; mechanisms stay `bitty`-side.                            |
| 048:1385-1474 | Code Intelligence Layer proposal                                         | 048-049 code-reading section; retain staged-cognition direction; API shape open.                                |
| 048:1475-1559 | Tools versus APIs; narrow surfaces                                       | 048-049 ergonomics section; retain intent-hiding rule; operation lists are illustration.                        |
| 048:1560-1638 | Capability Registry versus Active Set                                    | 048-049 capability section; retain small-active-set shape; counts are illustration.                             |
| 048:1639-1703 | Skill Resolver with overlap and ranking                                  | 048-049 capability section; retain resolver as candidate subsystem; algorithm open.                             |
| 048:1704-1776 | Trust metadata and tiers                                                 | 048-049 capability section; retain tiered no-implicit-trust direction; needs security review.                   |
| 048:1777-1816 | Tiny core prompt sketch                                                  | 048-049 prompt section; retain shrink-with-strength posture; sketch lines are illustration.                     |
| 048:1817-1903 | Verification Runtime chains; adapter inference                           | 048-049 verification section; retain evidence-gating shape; chains and rules open.                              |
| 048:1904-1959 | Thought-state split; state sketch                                        | 048-049 state section; retain discardable-transcript rule; fields are vocabulary.                               |
| 048:1960-2007 | Advanced compaction over threshold summarize                             | 048-049 state section; retain promotion-to-state direction; triggers open.                                      |
| 048:2008-2062 | Panel and Agent lifecycle split; headless panels                         | 048-049 modes paragraph; retain independence direction; lifecycle machine is RFC-owned input.                   |
| 048:2063-2121 | Subagent scheduler; structured returns                                   | 048-049 scheduling paragraph; retain budgeted-scheduling direction; policies open.                              |
| 048:2122-2167 | Provider abstraction; capability profile                                 | 048-049 provider paragraph; retain brand-indifference rule; profile fields are vocabulary.                      |
| 048:2168-2222 | Eval metrics; version comparison                                         | 048-049 scheduling paragraph; retain meter-not-impression rule; metrics are vocabulary.                         |
| 048:2223-2351 | Closing architecture; Core substrate; top principle; refs                | 048-049 decomposition section; retain candidate claims; topology not packaging.                                 |
| 049:1-47      | Compiler framing; Storage versus Active versus Cache                     | 048-049 compiler framing; retain three-way split as the design basis.                                           |
| 049:48-127    | Cold, Warm, Hot worlds; subset relation                                  | 048-049 stratification section; retain Hot-selects-from-Universe framing.                                       |
| 049:128-196   | Context IR struct; worked unit examples                                  | 048-049 IR section; retain IR-first direction; struct and values are sketches.                                  |
| 049:197-256   | Content-layout split; per-vendor backends                                | 048-049 IR section; retain backend seam; backend names are vocabulary.                                          |
| 049:257-373   | Six stability zones with examples and lifetimes                          | 048-049 zones section; retain volatility ordering; zone cuts are candidate.                                     |
| 049:374-450   | Cache tree; chained-hash sketch                                          | 048-049 zones section; retain layered-invalidation direction; hash mechanics open.                              |
| 049:451-525   | Canonical serializer; ordering hazards                                   | 048-049 serialization section; retain single-serializer ownership.                                              |
| 049:526-604   | Dynamic-out-of-prefix; position rule                                     | 048-049 serialization section; retain inverse-volatility placement.                                             |
| 049:605-661   | Utility formula; correctness-first ordering                              | 048-049 priority guardrail; retain correctness-first order; formula is a device.                                |
| 049:662-715   | Admission score; small cache tiebreak                                    | 048-049 admission section; retain tiebreak ceiling; formula is a thinking tool.                                 |
| 049:716-772   | Authority ladder; conflict eviction                                      | 048-049 authority section; retain pre-resolution rule; ladder order is candidate.                               |
| 049:773-839   | Content-addressed code; STALE bar                                        | 048-049 code section; retain hash-mismatch exclusion; artifact shape open.                                      |
| 049:840-892   | Dedup merge with provenance                                              | 048-049 code section; retain value-plus-sources shape; merge behavior is candidate.                             |
| 049:893-991   | Three-level tool-result reduction                                        | 048-049 reduction section; retain model-last level order; parsers are illustration.                             |
| 049:992-1071  | Four compaction modes                                                    | 048-049 compaction section; retain mode split with emergency-last; triggers open.                               |
| 049:1072-1123 | Traceable compaction; rehydration                                        | 048-049 compaction section; retain view-plus-pointers rule; syntax is illustration.                             |
| 049:1124-1165 | Context Pointer addressability                                           | 048-049 pointer paragraph; retain on-demand inspection direction; call shapes open.                             |
| 049:1166-1242 | Twelve-pass pipeline; telemetry feedback                                 | 048-049 pipeline section; retain feedback closure; pass list is candidate flow.                                 |
| 049:1243-1294 | Budgets decoupled from window; small-start                               | 048-049 budget section; retain decoupling; bands are illustration needing eval.                                 |
| 049:1295-1349 | Pinned, Flexible, Reserve quotas                                         | 048-049 budget section; retain reserve invariant; quota contents are illustration.                              |
| 049:1350-1461 | Four cache kinds with keys                                               | 048-049 cache section; retain four-way split; key shapes are illustration.                                      |
| 049:1462-1505 | Stable-prefix illustration; 80-percent hope                              | 048-049 cache section; retain stable-first discipline; ratio is narrative.                                      |
| 049:1506-1574 | Tool activation; deferred strategies                                     | 048-049 ergonomics section; retain deferred-loading direction; names are vocabulary.                            |
| 049:1575-1639 | No universal dispatcher; static core plus deferred                       | 048-049 ergonomics section; retain correctness-over-cache rule; tool lists open.                                |
| 049:1640-1694 | Provider cache capability record                                         | 048-049 cache section; retain capability seam; fields are vocabulary.                                           |
| 049:1695-1748 | Reasoning and tool-choice namespace                                      | 048-049 cache section; retain namespace inclusion; field list is illustration.                                  |
| 049:1749-1777 | Compiler versioning                                                      | 048-049 cache section; retain self-explaining-invalidation direction.                                           |
| 049:1778-1856 | Observability displays; statusline and views                             | 048-049 observability section; retain debuggability bar; provider pointer is owner direction.                   |
| 049:1858-1898 | Context Trace syntax                                                     | 048-049 observability section; retain attribution direction; syntax is illustration.                            |
| 049:1899-2032 | Worked leak-task walkthrough                                             | 048-049 example section; retain steady-state inequality; numbers are narrative.                                 |
| 049:2034-2135 | Five-subsystem split; dataflow; refs                                     | 048-049 subsystem section; retain never-direct dataflow; paths are candidate layout.                            |

## Topic-level traceability: records 050 and 051 (partial)

Section names below refer to the companion 050-051 distillation unless a
linked existing document is named. All Lua spellings, hook names, command
names, profile fields, permission names, ref spellings, and CLI spellings
are discussion inputs; the accepted Agent and IPC vocabulary stays with the
IPC and Agent RFC, which the 050-051 draft references without restating. The
whole verified files (`050.md:1-940`, `051.md:1-1308`) were distilled; no
head-versus-tail split applies.

| Source lines  | Topic                                                               | Disposition / destination                                                                      |
| ------------- | ------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| 050:1-17      | Rename `.bitty` to `.wheel`; two scopes; coexistence                | 050-051 rename section; record owner decision verbatim; naming and scope only.                 |
| 050:19-73     | `.agents` as portable capabilities; skills and MCP; migration value | 050-051 capabilities section; retain coexistence and non-pollution; sketches are illustration. |
| 050:77-140    | How-versus-What; single entry; Neovim-style organization            | 050-051 entry section; retain one-entry contract as candidate; survey item owns constraints.   |
| 050:142-220   | Capability filtering; registry to resolved set                      | 050-051 classes section; retain class as candidate; spellings are vocabulary.                  |
| 050:222-282   | Context policy declarations; policy-over-code rule                  | 050-051 classes section; retain declarative direction; budget sketches are illustration.       |
| 050:284-353   | Structured rules; deterministic enforcement                         | 050-051 classes section; retain engine-over-reminder direction; rule shapes open.              |
| 050:355-399   | Core-prompt shortening through runtimes                             | 050-051 classes rationale; retain rationale; prompt lines are illustration.                    |
| 050:401-435   | Custom commands in Lua                                              | 050-051 classes section; retain class as candidate; command shapes open.                       |
| 050:437-486   | Custom tools with provenance separation                             | 050-051 classes section; retain provenance rule; URIs and fields are illustration.             |
| 050:488-548   | Agent definitions with per-profile trimming                         | 050-051 classes section; retain structured-profile direction; fields are vocabulary.           |
| 050:550-600   | Verification declarations; completion gates                         | 050-051 classes section; retain evidence-gating direction; check lists open.                   |
| 050:602-644   | Hooks with safe, advanced, unsafe grading                           | 050-051 classes section; retain grading requirement; hook names are vocabulary.                |
| 050:646-709   | No runtime state; XDG and git-ignored placement                     | 050-051 state section; retain separation rule; directory names are illustration.               |
| 050:711-803   | Lua trust; no silent execution; direnv-like flow                    | 050-051 trust section; retain no-silent-execution and re-review rules; flow is candidate.      |
| 050:805-843   | Capability sandbox; mediated access; permission sketch              | 050-051 sandbox section; retain mediated-access posture; permission fields open.               |
| 050:845-940   | Final layout; discovery-to-runtime pipeline; boundary slogan        | 050-051 layout section; retain reference-not-copy rule; layout is candidate topology.          |
| 051:1-8       | Git borrowing thesis and principle list                             | 050-051 model framing; retain as author motivation; principles are design inputs.              |
| 051:9-60      | Context DAG over message list                                       | 050-051 DAG section; retain graph topology as candidate; node list open.                       |
| 051:62-121    | Object sketch; hashing; sharded storage                             | 050-051 DAG section; retain immutability and addressing; enum, hash, paths open.               |
| 051:123-168   | Deduplication by hash; ref sharing                                  | 050-051 dedup paragraph; retain ref-sharing rule; hash choice open.                            |
| 051:170-231   | Checkpoint as commit with parents and evidence                      | 050-051 checkpoint paragraph; retain chain topology; fields are vocabulary.                    |
| 051:233-301   | Compaction as snapshot without deletion                             | 050-051 compaction paragraph; retain view-change rule; schema open.                            |
| 051:303-351   | HEAD pointer; per-phase advance; log sketch                         | 050-051 checkpoint paragraph; retain single-pointer rule; lifecycle RFC-owned.                 |
| 051:352-403   | Hypothesis branches with shared parents                             | 050-051 branches paragraph; retain delta-only branching; mechanics open.                       |
| 051:405-482   | Multi-agent merge with explicit conflicts                           | 050-051 merge paragraph; retain explicit-merge rule; conflict schema open.                     |
| 051:484-520   | Cherry-picking findings with evidence                               | 050-051 merge paragraph; retain finding-level rule; procedure open.                            |
| 051:522-557   | Ref sharing between agents                                          | 050-051 sharing paragraph; retain ref-cost rule; protocol open.                                |
| 051:559-611   | Semantic rebase with staleness checks                               | 050-051 sharing paragraph; retain validate-before-replay; procedure open.                      |
| 051:613-655   | Worktree binding per agent                                          | 050-051 worktree paragraph; retain isolation direction; binding shape open.                    |
| 051:657-695   | Ref namespace for task and agent state                              | 050-051 refs paragraph; retain namespace direction; spellings open.                            |
| 051:697-734   | Reflog recovery and restore                                         | 050-051 recovery paragraph; retain recoverability rule; syntax open.                           |
| 051:736-788   | GC from live roots with grace period                                | 050-051 GC paragraph; retain rooted-collection rule; policy open.                              |
| 051:790-842   | Packfile packing with delta compression                             | 050-051 packfile paragraph; retain pack discipline; format open.                               |
| 051:844-894   | Hash-keyed compiler caches                                          | 050-051 cache paragraph; retain hash-exact identity; key shapes open.                          |
| 051:896-942   | Store-cache separation chain                                        | 050-051 separation paragraph; retain two-layer split; no store schema adopted.                 |
| 051:944-1012  | Checkout-like compiler to Active View                               | 050-051 compiler paragraph; retain checkout metaphor; passes defer to companion.               |
| 051:1014-1074 | Multi-agent DAG with parent lists                                   | 050-051 multi-agent paragraph; retain DAG topology as candidate; procedures open.              |
| 051:1076-1125 | Vocabulary decision; CLI sketch                                     | 050-051 vocabulary paragraph; retain two-level naming; spellings propose no surface.           |
| 051:1127-1165 | Context diff across checkpoints                                     | 050-051 diff paragraph; retain inspectability rule; diff format open.                          |
| 051:1167-1203 | Typed merge with per-type rules                                     | 050-051 merge paragraph; retain never-concatenate rule; merge rules open.                      |
| 051:1205-1262 | Storage diagram; refs over DAG over objects                         | 050-051 storage paragraph; retain as candidate architecture; boxes are not packaging.          |
| 051:1264-1308 | Closing philosophy; history versus active view                      | 050-051 philosophy paragraph; retain slogan as author opinion; inequality is load-bearing.     |

Section names below refer to the companion 052 distillation unless a linked
existing document is named. All Lua spellings, service names, plugin names,
file names, field names, command spellings, and flavor names are discussion
inputs; the accepted Agent and IPC vocabulary stays with the IPC and Agent
RFC, which the 052 draft references without restating. The whole verified
file (`052.md:1-561`) was distilled; no head-versus-tail split applies.

| Source lines | Topic                                                                     | Disposition / destination                                                                |
| ------------ | ------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| 052:1-12     | Plugin-between-plugins question; installable UI, models, dashboard, tools | 052 installable-plugin direction; retain direction; headless note is compatible context. |
| 052:13-16    | `bitty-ai` as distribution; Wheel as Harness kernel; minimal mechanism    | 052 framing section; retain framing as candidate; implies no crate or package split.     |
| 052:17-43    | Core-versus-Plugin boundary table                                         | 052 boundary section; retain placement as candidate; cell contents are vocabulary.       |
| 052:45-51    | Mechanism-versus-policy rule with three examples                          | 052 boundary section; retain rule as load-bearing; examples are canonical illustrations. |
| 052:55-71    | UI separation; bus operation sketch                                       | 052 UI section; retain independence direction; operation spellings are vocabulary.       |
| 052:73-84    | UI variant list                                                           | 052 UI section; retain variety objective; variant names are illustration.                |
| 052:86-117   | Headless and multi-panel UI layouts                                       | 052 UI section; retain placement independence; diagrams are illustration.                |
| 052:120-149  | Model gateway reframing; provider and subscription list                   | 052 model section; retain gateway direction; names are unverified illustration.          |
| 052:150-160  | Unified generation-interface sketch                                       | 052 model section; retain ignorance rule; call shape is vocabulary.                      |
| 052:162-177  | Auth ignorance; ecosystem-churn rationale                                 | 052 model section; retain churn-containment direction; credentials stay provider-side.   |
| 052:180-201  | Tool Provider contribution pattern                                        | 052 tools section; retain provider shape as candidate; contribution fields open.         |
| 052:203-229  | Tool-pack family sketches                                                 | 052 tools section; retain per-domain direction; family names are illustration.           |
| 052:231-244  | Minimal Core capabilities; Unix analogy                                   | 052 tools section; retain minimal-default posture; capability labels are vocabulary.     |
| 052:248-296  | Same-process service composition correction                               | 052 composition section; retain two-layer split; registry and bus names are vocabulary.  |
| 052:298-341  | Cross-process IPC layer; unified-call sketch and diagram                  | 052 composition section; retain transparency objective; routing mechanics open.          |
| 052:345-380  | Dependency declaration sketches                                           | 052 dependency section; retain dependency honesty; spellings propose no resolver.        |
| 052:382-403  | Provider and tool trees; package-graph analogy                            | 052 dependency section; retain graph direction; trees are illustration.                  |
| 052:407-438  | `.wheel/` composition claim; file sketch; selection sketch                | 052 composition-layer section; retain role; file and field sketches are illustration.    |
| 052:440-455  | Global defaults overridden by project composition                         | 052 layering section; retain precedence sentence; extended layout is owner direction.    |
| 052:457-471  | `.agents/` portable interop duties; non-confusion rule                    | 052 layering section; retain non-confusion rule; defers to the 050-051 direction.        |
| 052:475-512  | Distribution tree; official-composition reframing                         | 052 meta-package section; retain distribution direction; tree is candidate topology.     |
| 052:514-540  | Minimal versus recommended installs; flavor list                          | 052 meta-package section; retain composition direction; spellings propose no surface.    |
| 052:544-560  | Closing thesis; Runtime-Kernel-Distribution formula                       | 052 meta-package section; retain thesis as author opinion; formula is a slogan.          |

## Topic-level traceability: record 055 (AI slice)

Section names below refer to the companion 055 distillation unless a linked
existing document is named. All object names, event names, mailbox and
bundle fields, manifest fields, operation spellings, and primitive
spellings are discussion inputs; the accepted Agent and IPC vocabulary
stays with the IPC and Agent RFC, which the 055 draft references without
restating. The whole verified summary (`055.md:1-28`) was distilled; no
head-versus-tail split applies. The origin (`research/origin/055.md`) was
not read by this task.

| Source lines | Topic                                                                                                | Disposition / destination                                                                                    |
| ------------ | ---------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| 055:1-5      | Status, date, one-liner; Open capture state with owner-pending routing                               | 055 provenance section; retain routing; no capture claim beyond the AI slice.                                |
| 055:6-8      | Owner context-management question; flat-shape limits; log volume; Git-style storage proposal         | 055 boundary context; retain workspace framing as candidate; terminal halves route to pointers.              |
| 055:10       | Six-object split with four founding inequalities                                                     | 055 split section; retain vocabulary and inequalities as candidate; reconciled per the section.              |
| 055:11       | Headless-only agent rule; headed, shared, and detached panel kinds; fork-snapshot touch rule         | Read for boundary accuracy; agent-side consequence retained; panel mechanics route to terminal owner.        |
| 055:12       | Immutable event-log vocabulary                                                                       | Read for boundary accuracy; shape routes to terminal owner; no event name adopted here.                      |
| 055:13       | Per-call reason as durable rationale (commit-message shape); thinking stays ephemeral                | 055 Task section; retain rationale convention as candidate; no field adopted.                                |
| 055:14       | Tool-call detachment from terminal input; Wheel tool runtime under policy and control                | Read for boundary accuracy; placement routes to terminal and runtime owners; no contract adopted.            |
| 055:15       | Batch plus DAG multi-tool shape                                                                      | 055 principle section; compatible context for tool dispositions; mechanics stay open.                        |
| 055:16       | Running-task status discipline; UI collapse decoupled from materialization                           | Read for boundary accuracy; status direction retained; views route to terminal owner.                        |
| 055:17       | Output pipeline (blob store, parser, summary, compiler) with hashed stable blocks                    | 055 principle section; compatible context for the 048-049 compiler; stage contracts stay open.               |
| 055:18       | Context Manifest with parent lineage as a selectable tree; prompt as budgeted materialized view      | 055 merge section; retain manifest direction as candidate; no schema adopted.                                |
| 055:19       | Git analogues on versionable context only; squash keeps provenance and stays expandable              | 055 merge section; retain non-rewriting rule as load-bearing; defer to the 051 companion.                    |
| 055:20       | Agent Graph (cyclic, uniform Agents, dynamic roles) distinct from Task DAG; fork without reseeding   | 055 graph section; retain split as candidate; reconciled per the section.                                    |
| 055:21       | Mailbox IPC with context base and refs; lazy ContextBundles; typed merges; six-way context diff      | 055 mailbox and merge sections; retain directions as candidate; no wire adopted.                             |
| 055:22       | Hybrid storage (SQLite index plus content-addressed store; JSON and JSONL interchange); GC lifetimes | Read for boundary accuracy; storage routes to owning repos; GC direction retained per the section.           |
| 055:23       | Task-as-Issue durable objects; company-console dashboard; first-priority primitives; core principle  | 055 Task and principle sections; retain Task and principle as candidate; dashboard routes to terminal owner. |
| 055:24       | Destination routing to terminal and AI owners                                                        | 055 pointers section; retain routing; no canonical page claimed beyond this draft.                           |
| 055:25-28    | Open items; origin stays unrenamed; split-owner capture stays Partial                                | 055 coverage section; retain; origin stays unrenamed and split-owner capture stays Partial.                  |

## Explicit exclusions: records 039, 040, 041, 044, 045, 048, 049, 050, 051, 052, and 055

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
  assertions in these records are unmeasured and no conclusion is drawn from
  them here.
- 041 crash isolation, debugging and DevTools, running-instance control, and
  multi-instance addressing sections are `bitty`-side terminal or
  plugin-ecosystem design. Their AI-facing consequences (out-of-process
  daemon placement, protocol-operated panels) are retained in the 041
  distillation; no supervision, tooling, control-surface, or addressing
  implementation is distilled into an AI requirement.
- 041 `agent.*` and `panel.*` method names, event names, manifest fields,
  capability-list entries, and the `agent ask` sketch are unreviewed
  discussion inputs, not adopted wire, event, CLI, or permission interfaces.
  Agent lifecycle, events, and semantics stay with the accepted IPC and
  Agent RFC, which the 041 draft references without restating.
- 044 `exec` and `job_*` tool sketches, the visibility console mockup, the
  `execution.*` IPC sketches, the `ExecutionResult` and `JobResult` struct
  sketches, the `Lifetime`, `JobKind`, outcome, and cancel enums, the
  shell-versus-argv attribution detail, and the three-phase rollout are
  unreviewed discussion inputs, not adopted commands, tools, events, wire
  formats, types, defaults, or schedules. Shell-versus-argv mechanics are
  `bitty`-side execution design with R2-owned consequences; the console
  mockup proposes no accepted view. Durability, crash adoption, detached
  long-lived jobs, and any supervisor daemon stay post-v0.1 under R6.
- 045 host resource enforcement (CPU, RAM, GPU, disk, cgroup, RLIMIT,
  process-group, GPU-backend, filesystem-quota, and Job Objects backends),
  the panel-lease mechanism with generation-fenced handoff, `.env`, `.ssh`,
  and credentials authorization, and root and sudo escalation are
  `bitty`-side terminal or host design with config-owned and Lua-workflow
  rows. Their AI-facing consequences (intersection-only authority,
  runtime-enforced spawn budgets, subagent attenuation, Commander-as-Lua
  vocabulary cut, secret handles, delegation semantics, and the
  User-Hygiene split) are retained in the 045 distillation; no host
  enforcement, lease, authorization, or escalation implementation is
  distilled into an AI requirement.
- 045 `DelegationBudget` fields, `agent.spawn`, `execution.spawn`,
  `execution.run`, `panel.acquire`, and `fs.read` call shapes, the
  `PrivilegeRequest` envelope, the `BudgetExceeded`,
  `ResourceLimitExceeded`, `ExecutionBudgetExhausted`, and
  `ConcurrencyLimitReached` error names, capability and scope names, profile
  names, path patterns, numeric ceilings, the four-layer stack, and the
  raw-escape-hatch (`os.execute`, `io.open`) restriction note are unreviewed
  discussion inputs, not adopted budgets, calls, envelopes, errors,
  interfaces, defaults, or plans. Agent lifecycle, events, and semantics
  stay with the accepted IPC and Agent RFC, which the 045 draft references
  without restating.
- 048 quality-formula symbols, window-versus-attention figures, viewer and
  repo-map size anecdotes, tool and MCP count and threshold figures, Skill
  Hell overlap scores, supply-chain scan figures, prompt sketch lines,
  `ContextUnit` and `CapabilityMeta` struct sketches, admission-score
  formula, authority-ladder order, budget bands, cache-namespace fields,
  Eval metric names, and the seven-module Wheel diagram are unreviewed
  discussion inputs, not adopted formulas, thresholds, schemas, metrics,
  commands, or packaging. Terminal-substrate mechanisms stay `bitty`-side.
- 049 Context IR fields and unit values, zone cuts and lifetimes, chained-hash
  mechanics, serializer and section lists, utility and admission formulas,
  artifact shapes and hash choices, dedup merge behavior, tool-result parser
  schemas and token counts, compaction triggers, pointer call shapes, pass
  list, budget bands, quota contents, cache key shapes and hit-rate figures,
  deferred-loading strategy names, provider cache capability fields,
  namespace fields, version strings, observability display contents, trace
  syntax, walkthrough numbers, module paths, and the five-subsystem layout
  are unreviewed discussion inputs, not adopted types, contracts, keys,
  views, commands, or packaging. Per-vendor cache behavior notes are
  reported vendor semantics the task did not verify; only the capability
  seam is distilled. Provider cost, cache-hit, input, context, and timing
  exposure stays a design pointer for a future adapter task with no
  `bitty-ai` code change here.
- 050 `.wheel` Lua API spellings, field and entry names, directory layouts,
  command and tool shapes, tool URI forms, agent-profile fields, hook names
  and tiers, trust-flow wording, permission fields, pipeline boxes, and the
  boundary slogan are unreviewed discussion inputs, not adopted schemas,
  APIs, commands, tools, permissions, or packaging. The `.agents`
  project-level survey stays open with the owning task; no schema on either
  side is decided here. The `.bitty` to `.wheel` rename is a recorded owner
  naming decision with no implementation claim.
- 051 `ContextObject` enum sketch, checkpoint fields, ref spellings, log and
  diff formats, merge rules, storage paths, pack filenames, hash algorithm
  names, CLI spellings, diagram boxes, and the closing slogan are unreviewed
  discussion inputs, not adopted schemas, paths, formats, commands, or
  packaging. Lifecycle authority stays with the accepted IPC and Agent RFC
  and the R1/R5 dispositions. Wheel modes (headless, Panel and Agent split,
  one-shot single-question non-Agent chat) are design-only with no
  implementation claim.
- 052 boundary-table cell contents, Lua operation and call spellings, service
  names, plugin, variant, family, and capability labels, `.wheel/` file and
  field sketches, selection and disable sketches, dependency-declaration
  sketches, install and scaffolding spellings (`wheel init`, `/init`),
  `.wheel/` constraint-surface fields and flags, per-prompt versus
  unattended (yolo) permission-mode vocabulary, global and runtime path
  illustrations, flavor names, the distribution tree, provider and product
  names, the third-party harness comparison, and the closing formula are
  unreviewed discussion inputs (with the constraint surface, scaffolding,
  layout extension, permission modes, and provider observability recorded as
  owner direction), not adopted schemas, APIs, commands, tools, permissions,
  packages, paths, or releases. The `.agents` project-level survey stays
  open with the owning task; no schema on either side is decided here.
  Lifecycle authority stays with the accepted IPC and Agent RFC and the
  R1/R5 dispositions. Decoupling modes (headless operation) are design-only
  with no implementation claim.
- 055 six-object vocabulary, event names, mailbox and bundle shapes, manifest
  fields, Git-operation mapping, storage split, dashboard mechanics, and
  primitive spellings are unreviewed discussion inputs, not adopted types,
  schemas, events, wire formats, commands, views, or packaging. Panel,
  event-log, tool-runtime, pipeline-stage, storage, and dashboard halves
  route to the terminal, tool, compiler, and storage owners as pointers in
  the 055 distillation; lifecycle authority stays with the accepted IPC and
  Agent RFC and the R1/R5 dispositions. The headless-only agent rule and the
  dashboard mechanics are design-only with no implementation claim.
- This task does not close identity, workspace-overlay, CarryCtx backend,
  agent-growth, dependency, protocol, registry, manifest, versioning, or
  release-scope decisions, and it changes no normative contract.
