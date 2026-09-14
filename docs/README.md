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
- The repository is designed to be mounted at `bitty-ai/docs` as a Git
  submodule once the `bitty-ai` implementation repository exists.

## Content trees

| Tree              | Entry points                                                                                                                                                                                                                                                                                                                                 |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `specifications/` | [AI Architecture](../specifications/ai-architecture.md) (Draft), [IPC and Agent RFC](../specifications/ipc-agent-rfc.md) (Accepted), [AI Vertical Slice Pressure Test](../specifications/ai-vertical-slice-pressure-test.md) (Draft), [Browser and Agent Panel Integration Pre-Study](../specifications/browser-agent-pre-study.md) (Draft). |

New topic trees are created only as real content lands; empty placeholder pages
are not added.

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
