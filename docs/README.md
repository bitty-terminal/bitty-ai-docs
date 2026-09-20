---
title: Documentation map
description: Canonical navigation and authority rules for the bitty-ai AI core documentation corpus
category: project
audience: mixed
document_type: index
status: accepted
website_publish: true
sidebar_order: 1
---

# Documentation map

This index is the entry point for the canonical documentation of `bitty-ai`,
the independent AI-core sub-platform of Bitty. The AI-core corpus was migrated
from `bitty-docs` (`docs/projects/bitty/specifications/` at revision `c664214`)
with history preserved in CTX-0001. Canonical content lives in topic trees at
the repository root; this repository's own process documents live under
`docs/`.

## Authority and composition

- This repository owns AI-core architecture, runtime, provider, context,
  specification, and reference documents.
- Shared cross-project governance lives in
  [bitty-docs](https://github.com/bitty-terminal/bitty-docs): decisions, the
  security corpus, sources, findings, reviews, handoff, project state, roadmap,
  and releases. This repository links to those documents with absolute URLs
  instead of copying them.
- Sibling documentation repositories:
  [bitty-terminal-docs](https://github.com/bitty-terminal/bitty-terminal-docs)
  (terminal platform) and
  [bitty-plugins-docs](https://github.com/bitty-terminal/bitty-plugins-docs)
  (plugin ecosystem). Documents migrated to those repositories are referenced
  from surviving pages by absolute cross-repository URL.
- `bitty-ai/docs` is a pinned submodule of this repository. Pinned revisions
  move with integration needs; point-in-time inspection details live with the
  candidate design records (see
  [AI runtime boundaries](../specifications/ai-runtime-boundaries-candidate.md)),
  not in this map.

## Content trees

Every canonical AI-core document lives in exactly one root topic tree. A tree's
route-only index lists its documents; normative detail stays in the linked
pages.

| Tree              | Entry points                                                                                                                                                                                                                         |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `architecture/`   | [Architecture index](../architecture/README.md) — the umbrella [AI Architecture](../architecture/ai-architecture.md) specification, the command/tool and host-boundary designs, the R1-R6 draft dispositions, and the diagram suite. |
| `context/`        | [Context index](../context/README.md) — context management, prefix-cache context design, prompt layering, the Wheel context runtime candidate, and the project continuity candidate.                                                 |
| `providers/`      | [Providers index](../providers/README.md) — provider plugin boundary, multimodal inference boundary, and dependency strategy.                                                                                                        |
| `agent/`          | [Agent index](../agent/README.md) — agent coordination, code intelligence, caller attribution, git wrapper API, and fragment pre-split rule.                                                                                         |
| `persistence/`    | [Persistence index](../persistence/README.md) — persistence evidence, storage memory and export, and history consumption boundary.                                                                                                   |
| `interfaces/`     | [Interfaces index](../interfaces/README.md) — panel environment awareness and browser/agent panel pre-study.                                                                                                                         |
| `integration/`    | [Integration index](../integration/README.md) — bitty-side integration input, delivery verification, integration risk register, and RFC-split readiness.                                                                             |
| `product/`        | [Product index](../product/README.md) — v0.1 implementation profile, vertical-slice pressure test, and unresolved-questions register.                                                                                                |
| `specifications/` | [Specification register](../specifications/README.md) — the accepted IPC and Agent RFC plus the candidate design records.                                                                                                            |

New topic trees are created only as real content lands; empty placeholder pages
are not added.

The AI Architecture Family relationship below traces how the topic documents
elaborate the [AI Architecture](../architecture/ai-architecture.md) draft. It is
orientation only and promotes no mechanism to accepted status.

## Planned trees

The repository owns the scope below, but a topic tree is created only when its
first reviewed content lands. **No page means not started: a tree appears here
only as a plan, and no empty placeholder page is created.**

| Tree         | Purpose                                                                                             | Owning open questions                                                      | Prerequisite / source specifications                                                      | Landing order |
| ------------ | --------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- | ------------- |
| `reference/` | Reference material derived from verified implementation evidence in the `bitty-ai` code repository. | None yet; evidence-gated, lands only with reviewed implementation evidence | Verified `bitty-ai` implementation evidence (no in-repository specification prerequisite) | 1             |

## AI Architecture Family

The AI architecture is a draft with related draft subsystem proposals. Topic relationships below do not promote any mechanism to accepted status:

```text
ai-architecture.md (Main specification)
├─→ context-management.md (Session journal, compression, context budget)
├─→ command-tool-architecture.md (Core/Lua boundary, tool dispatch)
├─→ agent-coordination.md (Multi-agent teams, delegation, supervision)
│   └─→ code-intelligence.md (LSP sharing, verification reuse)
└─→ persistence-evidence.md (Session journal, evidence store, event log)
```

**Dependency graph**:

- `architecture/ai-architecture.md` defines ModelProvider, ContextProvider, Tool Bus, Agent levels, and Rich streaming (CP-1..CP-11, MP-1..MP-11, AG-1..AG-5, TB-1..TB-7, RS-1..RS-5).
- `context/context-management.md` elaborates CP-5 (Budget), CP-6 (Artifacts), and CP-7 (Determinism and testability) with a consent-bounded journal and projection proposal.
- `context/prefix-cache-context-design.md` refines CP-5 (Budget), CP-6 (Artifacts), and CP-7 (Determinism and testability) for stable-prefix layering and deterministic serialization; Planner, epoch, and L2+ material stays beyond-v0.1 proposal.
- `context/prompt-layering-design.md` layers prompt text (Core Contract, User, Project `.wheel`, Skills/Agent profile, Runtime/Turn) over the context pipeline with stable-before-dynamic assembly; prompt text never grants capability (AG-4) and profiles stay single-agent for v0.1.
- `providers/dependency-strategy.md` keeps `bitty-ai-runtime` std-only with dependency inversion and maps provider, tool-bus, MCP, code, and store adapters to post-v0.1 proposals; v0.1 adds zero dependencies and MSRV bumps stay open decisions.
- `architecture/command-tool-architecture.md` extends TB-1..TB-3 with Core versus Lua boundary, slash command registry, and tool runtime separation.
- `agent/agent-coordination.md` elaborates coordination under AG-4 (Least privilege at dispatch) and AG-5 (Orchestration versus execution).
- `agent/code-intelligence.md` extends agent-coordination service supervision with LSP sharing, stateful mediation, verification fingerprinting, and lint/build/test reuse.
- `persistence/persistence-evidence.md` compares journal representation, evidence, projection, optional indexing and replay; standalone release scope and backend selection remain undecided.
- `architecture/execution-ownership-r1.md` records the R1 draft disposition for single-agent execution ownership (`ExecutionContext` primary, optional Panel projection) without closing AIQ-29, AIQ-2A, or AIQ-38.
- `architecture/context-retention-r3.md` records the R3 draft disposition for consent-bounded retention, deletion propagation, and recovery limits without closing AIQ-01 through AIQ-11 or AIQ-57.
- `architecture/task-lifecycle-r5.md` records the R5 draft disposition for product Task lifecycle authority versus CarryCtx backend-or-handoff, with independent-review and critical-message rules, without closing AIQ-10 (with the AIQ-56 alias), AIQ-26, or AIQ-28.
- `architecture/tool-transport-r2.md` records the R2 draft disposition for unified authorization backend with native and MCP path selection, declared per-tool placement, and fail-closed denial, without closing AIQ-33, AIQ-36, AIQ-37, AIQ-38, or AIQ-08.
- `architecture/code-intelligence-sharing-r4.md` records the R4 draft disposition for domain-keyed language-service sharing with lease and generation fencing, complete input fingerprints, per-reader authorization, cached-PASS separation from independent approval, and disabled-by-default effectful coalescing, without closing AIQ-21, AIQ-22 (with the AIQ-42 alias), or AIQ-41 through AIQ-48.
- `architecture/persistence-profile-r6.md` records the R6 draft disposition for append-ordered journal backend profile with file-held artifact bytes, projection and optional index as derived operations, state-only replay with Unknown reconciliation and no background scheduler, and an ephemeral v0.1 with gate-conditioned durability deferral, without closing AIQ-51 through AIQ-5C.
- `integration/bitty-side-integration-input.md` assembles the draft bitty-side handoff input (ten 031 blockers plus 023 G-2/G-3 confirmation with R1 registry-split framing, R2 transport open surface, and R5 lifecycle boundary) without closing any AIQ entry; every requirement names bitty-side ownership with a suggested priority.
- `integration/bitty-side-delivery-verification.md` records draft read-only verification mapping `bitty` `main` deliveries (#702/#703/#705/#707/#709/#711) to BII-01 through BII-10 with G-2/G-3 confirmation without closing any AIQ entry or granting `bitty`-side acceptance.
- `integration/rfc-split-readiness.md` records the draft 028 P2-3 readiness evaluation for the five CTX-0023 split candidates with per-candidate evidence bars and future boundary drafts without accepting any Request For Comments or closing any Artificial Intelligence Question entry.
- `providers/provider-plugin-boundary.md` records the draft 032 Core versus provider-plugin boundary for ModelProvider contract, transport taxonomy, two-level plugins, model aliases, routing inputs, and the secret invariant with bitty-side handoff without closing any Artificial Intelligence Question entry or proposing any new identifier.
- `providers/multimodal-inference-boundary.md` records the draft 033 Core multimodal extension for capability vocabulary, task envelope, generation-job lifecycle with Unknown reconciliation, asset outputs, agent events, Lua adapter placement, and generic-provider direction as beyond-v0.1 proposal without closing any Artificial Intelligence Question entry or proposing any new identifier.
- `interfaces/panel-environment-awareness.md` records the draft 036 bitty-ai-facing awareness note for sanitized Agent View, Use-versus-Read boundary, env-handle semantics, no-persistence default, and exported-only scope with bitty-side handoff without closing any Artificial Intelligence Question entry or proposing any new identifier.
- `persistence/storage-memory-export-design.md` records the draft 037 bitty-ai-side storage direction for per-session SQLite with catalog control plane, content-addressed objects, event model, context recipes, memory tiers, export scopes, Core storage API, interchange format, lifecycle, collection, and project identity with bitty-side handoff without closing any Artificial Intelligence Question entry or proposing any new identifier.
- `persistence/history-consumption-boundary.md` records the draft 038 bitty-ai-side history consumption boundary for consume-not-own principle, backend-agnostic HistoryProvider direction, scoped panel.history-style reads, CommandRecord attribution with external references, agent-command sink direction, Atuin MCP tool reference shape, and command-history versus panel-output boundary with bitty-side handoff without closing any Artificial Intelligence Question entry or proposing any new identifier.
- `integration/integration-risk-register.md` records the draft review-07 cross-repo integration-risk register (eight bitty-ai and bitty contract questions with evidence anchors, decision owners, required contracts, and acceptance evidence) with bitty-side handoff without closing any Artificial Intelligence Question entry or proposing any new identifier.
- `agent/fragment-pre-split-rule.md` transcribes the implemented AI-0066/AI-0070 slice-layer fragment pre-split and reassembly mapping rule (64 KiB runtime fragments to at most 16 KiB transport parts at code-point boundaries with a continuation marker, dense cursor-assigned `seq`, and a caller-bindable reassembly identity) with revision-pinned evidence and explicit non-claims (mapping layer only; no `rich.*` wire method registered) without closing any Artificial Intelligence Question entry or proposing any new identifier.

**Unresolved choices**: `product/ai-unresolved-questions.md` preserves 53 unresolved-question identifiers, including stable aliases, rather than claiming 53 independent questions. Its feature-prerequisite classifications and proposed owner routing are local draft analysis, not accepted global OQs or assigned milestones.

## Process documents

| Document                                                        | Purpose                                                   |
| --------------------------------------------------------------- | --------------------------------------------------------- |
| [Development](development/README.md)                            | Contributor entry point and local gates.                  |
| [Documentation workflow](development/documentation-workflow.md) | Normative authoring, metadata, status, and review policy. |

## Maintaining the corpus

1. Update the canonical topic document first.
2. Keep status labels honest: `draft`, `accepted`, `normative`, `stable`,
   `deprecated`, `archived`.
3. Cross-link one authoritative definition instead of copying divergent
   wording.
4. Update this index and the root `README.md` when navigation changes.
5. Run `just check` before every push; documentation synchronization is part of
   delivery completion.
