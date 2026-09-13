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
the independent AI-core sub-platform of Bitty. The repository is in its
bootstrap state (CTX-0187 Phase 1): the docs-quality toolchain and this
skeleton exist, while the AI-core documents are migrated from `bitty-docs` in a
later, separately tracked phase. Nothing on this page claims migrated content.

## Authority and composition

- This repository owns AI-core architecture, runtime, provider, context,
  specification, and reference documents.
- Shared cross-project governance lives in
  [bitty-docs](https://github.com/bitty-terminal/bitty-docs): decisions, the
  security corpus, sources, findings, reviews, handoff, project state, roadmap,
  and releases. This repository links to those documents instead of copying
  them.
- Sibling documentation repositories:
  [bitty-terminal-docs](https://github.com/bitty-terminal/bitty-terminal-docs)
  (terminal platform) and
  [bitty-plugins-docs](https://github.com/bitty-terminal/bitty-plugins-docs)
  (plugin ecosystem).
- The repository is designed to be mounted at `bitty-ai/docs` as a Git
  submodule once the `bitty-ai` implementation repository exists.

## Current tree

| Document                                                        | Purpose                                                   |
| --------------------------------------------------------------- | --------------------------------------------------------- |
| [Development](development/README.md)                            | Contributor entry point and local gates.                  |
| [Documentation workflow](development/documentation-workflow.md) | Normative authoring, metadata, status, and review policy. |

## Planned structure

The AI-core documents migrate into topic trees appropriate to an independent
sub-platform:

| Planned tree      | Content                                                       |
| ----------------- | ------------------------------------------------------------- |
| `architecture/`   | AI-core system context, boundaries, and component model.      |
| `providers/`      | Model and tool provider contracts, capability boundaries.     |
| `context/`        | Context assembly, provenance, and budget semantics.           |
| `specifications/` | Versioned technical contracts with verification obligations.  |
| `reference/`      | Factual lookup material derived from implementation evidence. |

Trees are created only as real content lands; empty placeholder pages are not
added.

## Maintaining the corpus

1. Update the canonical topic document first.
2. Keep status labels honest: `draft`, `accepted`, `normative`, `stable`,
   `deprecated`, `archived`.
3. Cross-link one authoritative definition instead of copying divergent
   wording.
4. Update this index and the root `README.md` when navigation changes.
5. Run `just check` before every push; documentation synchronization is part of
   delivery completion.
