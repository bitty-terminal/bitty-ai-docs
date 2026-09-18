---
title: Architecture Diagrams
description: Small glossary-driven AI architecture diagram set for overview agent graph and context compile
category: architecture
audience: contributor
document_type: overview
status: draft
website_publish: false
sidebar_order: 10
---

# Architecture Diagrams

Small-scale AI architecture diagram set for `bitty-ai-docs`, mirroring the
terminal-docs glossary-driven model: one canonical
[glossary](glossary.yaml), D2 text sources, and vector SVG exports. Every
node traces to accepted or draft code or RFC; no diagram invents nodes.

## Status and authority

- **Accepted floor:** [IPC and Agent RFC](../specifications/ipc-agent-rfc.md)
  defines the only accepted IPC wire, scope, and Agent vocabulary. Solid
  edges to `IPC Channel` and `AgentMessage` rest on that contract.
- **Draft dispositions:** R1 single-agent execution ownership
  ([R1](../specifications/execution-ownership-r1.md)), draft mailbox
  directions ([Agent coordination](../specifications/agent-coordination.md)),
  draft single-authority Task model ([R5](../specifications/task-lifecycle-r5.md)),
  draft session-versus-context invariant
  ([Context management](../specifications/context-management.md)), draft
  stable-prefix ordering
  ([Prefix-cache design](../specifications/prefix-cache-context-design.md)),
  and draft structural compaction (same design, eight invariants).
- **Candidate discussion input:** the 055 research distillation
  ([055](../specifications/research-distillation-055-bitty-ai.md))
  plus companions
  [048-049](../specifications/research-distillation-048-049-bitty-ai.md),
  [050-051](../specifications/research-distillation-050-051-bitty-ai.md),
  [052](../specifications/research-distillation-052-bitty-ai.md),
  [044](../specifications/research-distillation-044-bitty-ai.md),
  [046/053/054](../specifications/research-distillation-046-053-054-bitty-ai.md),
  [039](../specifications/research-distillation-039-bitty-ai.md),
  [040](../specifications/research-distillation-040-bitty-ai.md),
  [041](../specifications/research-distillation-041-bitty-ai.md),
  and [047](../specifications/caller-attribution-design.md).
  Dashed nodes and edges are vocabulary only: no schema, wire, command,
  threshold, or release decision follows.
- **Implemented anchors:** observed in `bitty-ai` at `3623c6b3`
  (inspected 2026-09-14, experimental slice, not the complete runtime):
  `crates/bitty-ai-runtime/src/{agent,context,session,tool,bridge,prompt}.rs`
  and `crates/bitty-ai-slice/src/journal_prototype.rs`. Anchors mirror the
  draft pipeline at small scale; the single-agent, L0-plus-L1, and
  deny-by-default limits stay in force.

## Diagram inventory

| ID                   | Title                                | Level | Sources                                              | Export                                                       |
| -------------------- | ------------------------------------ | ----- | ---------------------------------------------------- | ------------------------------------------------------------ |
| `00-overview`        | AI overview and six-object split     | L0    | [d2/00-overview.d2](d2/00-overview.d2)               | [final/00-overview.svg](final/00-overview.svg)               |
| `01-agent-graph`     | Agent Graph vs Task DAG with mailbox | L1    | [d2/01-agent-graph.d2](d2/01-agent-graph.d2)         | [final/01-agent-graph.svg](final/01-agent-graph.svg)         |
| `02-context-compile` | Context compile Cold Warm Hot        | L1-L2 | [d2/02-context-compile.d2](d2/02-context-compile.d2) | [final/02-context-compile.svg](final/02-context-compile.svg) |

## Single source of truth

- Canonical inventory: [glossary.yaml](glossary.yaml) defines every node,
  edge, status, and trace. No diagram may invent nodes not defined there.
- Text sources: `d2/*.d2` are the editable graph definitions.
- Static vectors: `final/*.svg` are D2 exports committed alongside the
  sources; regenerate with `d2 architecture/d2/<id>.d2
architecture/final/<id>.svg`, then run `just svg`.

## Directory layout

```text
architecture/
├── README.md               # this file — small-scale diagram index
├── glossary.yaml           # single node and edge data dictionary
├── d2/                     # editable D2 text graph sources
│   ├── 00-overview.d2
│   ├── 01-agent-graph.d2
│   └── 02-context-compile.d2
└── final/                  # vector SVG exports
    ├── 00-overview.svg
    ├── 01-agent-graph.svg
    └── 02-context-compile.svg
```

## Maintenance

When a diagram changes, update its D2 source, its SVG export, and the
glossary trace in the same change, then run `just svg` to confirm every
committed SVG is well-formed XML. Promotion of any candidate node to draft
or accepted status happens in the owning specification first, never here.
