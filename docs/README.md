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
- Read-only inspection on 2026-09-14 confirmed the initialized `bitty-ai/docs`
  submodule at `39b4c7568a8807e0940bd298660c972db0cfa92a`, introduced by
  `b6d3d6c495607f6bb2660b5442cfefcf350b486e`. The sibling's HEAD
  `3623c6b3ce33e97c1c493109ec6356219d0c9722` contains an experimental slice;
  see [inspected scope](../specifications/research/research-distillation-013-017-018.md#current-bitty-ai-evidence).
  Its mounted revision does not include these uncommitted reconciliation drafts.

## Content trees

| Tree                       | Entry points                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| -------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `specifications/`          | [AI Architecture](../specifications/ai-architecture.md) (Draft), [IPC and Agent RFC](../specifications/ipc-agent-rfc.md) (Accepted), [AI Vertical Slice Pressure Test](../specifications/ai-vertical-slice-pressure-test.md) (Draft), [v0.1 Implementation Profile](../specifications/implementation-profile-v0.1.md) (Draft), [Prefix-Cache-Friendly Context Design](../specifications/prefix-cache-context-design.md) (Draft), [Browser and Agent Panel Integration Pre-Study](../specifications/browser-agent-pre-study.md) (Draft), [Context Management Architecture](../specifications/context-management.md) (Draft), [Command and Tool Architecture](../specifications/command-tool-architecture.md) (Draft), [Agent Coordination Architecture](../specifications/agent-coordination.md) (Draft), [Code Intelligence Architecture](../specifications/code-intelligence.md) (Draft), [Persistence and Evidence Architecture](../specifications/persistence-evidence.md) (Draft), [AI Unresolved Questions](../specifications/ai-unresolved-questions.md) (Draft). |
| `specifications/research/` | Research distillation documents from CTX-0003, CTX-0004: [Research 013/017/018](../specifications/research/research-distillation-013-017-018.md), [Research 021](../specifications/research/research-distillation-021.md), [Coverage Ledger](../specifications/research/research-coverage-ledger.md).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |

New topic trees are created only as real content lands; empty placeholder pages
are not added.

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

- `ai-architecture.md` defines ModelProvider, ContextProvider, Tool Bus, Agent levels, and Rich streaming (CP-1..CP-11, MP-1..MP-11, AG-1..AG-5, TB-1..TB-7, RS-1..RS-5).
- `context-management.md` elaborates CP-5 (Budget), CP-6 (Artifacts), and CP-7 (Determinism and testability) with a consent-bounded journal and projection proposal.
- `prefix-cache-context-design.md` refines CP-5 (Budget), CP-6 (Artifacts), and CP-7 (Determinism and testability) for stable-prefix layering and deterministic serialization; Planner, epoch, and L2+ material stays beyond-v0.1 proposal.
- `command-tool-architecture.md` extends TB-1..TB-3 with Core versus Lua boundary, slash command registry, and tool runtime separation.
- `agent-coordination.md` elaborates coordination under AG-4 (Least privilege at dispatch) and AG-5 (Orchestration versus execution).
- `code-intelligence.md` extends agent-coordination service supervision with LSP sharing, stateful mediation, verification fingerprinting, and lint/build/test reuse.
- `persistence-evidence.md` compares journal representation, evidence, projection, optional indexing and replay; standalone release scope and backend selection remain undecided.

**Unresolved choices**: `ai-unresolved-questions.md` preserves 53 research identifiers, including stable aliases, rather than claiming 53 independent questions. Its feature-prerequisite classifications and proposed owner routing are local draft analysis, not accepted global OQs or assigned milestones.

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
