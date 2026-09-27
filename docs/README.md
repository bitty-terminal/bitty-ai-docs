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
the independent AI-core sub-platform of Bitty. Canonical content lives in topic
trees at the repository root; this repository's own process documents live under
`docs/`.

## Admission criteria

A canonical page is admitted to an active route only when:

- Real, reviewed content exists; an empty placeholder page is not created.
- The page has the required metadata, an H1 matching its title, an explicit
  status, and the document-type spine defined by the
  [documentation workflow](development/documentation-workflow.md).
- The page belongs to exactly one root topic tree and is reachable from that
  tree's route-only index without copying normative detail into the index.
- Cross-repository links name the owning document, and dependent pages link to
  one authoritative definition instead of restating a divergent contract.
- The page is self-contained, passes the repository quality gates, and keeps
  unresolved questions, risks, and implementation status explicit.

A planned tree remains a plan until its first qualifying page lands. Admission
to this map establishes routing and corpus ownership, not acceptance,
normative status, ownership assignment, or implementation authorization.

## Authority and status

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
  (plugin ecosystem). Documents owned by those repositories are referenced
  from this corpus by absolute cross-repository URL.
- `bitty-ai/docs` is a pinned submodule of this repository. It supplies
  implementation-facing material when a topic document needs it; this map
  routes to the canonical document rather than copying that material.

## Content trees

Every canonical AI-core document lives in exactly one root topic tree. A tree's
route-only index lists its documents; normative detail stays in the linked
pages.

| Tree              | Entry points                                                                                                                                                                                                                         |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `architecture/`   | [Architecture index](../architecture/README.md) — the umbrella [AI Architecture](../architecture/ai-architecture.md) specification, the command/tool and host-boundary designs, the R1-R6 draft dispositions, and the diagram suite. |
| `context/`        | [Context index](../context/README.md) — context management, prefix-cache context design, prompt layering, the Wheel context runtime candidate, and the project continuity candidate.                                                 |
| `providers/`      | [Providers index](../providers/README.md) — provider plugin boundary, multimodal inference boundary, dependency strategy, and the provider transport adapter contract.                                                               |
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
- `providers/transport-adapter-contract.md` separates the frozen v0.1
  `TurnRequest` and `ModelDescriptor` surface from a separately labeled
  post-v0.1 adapter-envelope proposal, states the adapter guarantees and the
  network-layer delegation boundary, and keeps AIQ-33, AIQ-36, and AIQ-38
  open.
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
- `integration/bitty-side-integration-input.md` assembles cross-repository handoff
  requirements for registry ownership, transport, and lifecycle, with each
  requirement identifying its owning side and a suggested priority; it closes no
  AIQ entry.
- `integration/bitty-side-delivery-verification.md` maps current `bitty`-side
  deliveries to read-only verification items and acceptance evidence without
  closing any AIQ entry or granting `bitty`-side acceptance.
- `integration/rfc-split-readiness.md` evaluates five split candidates against
  their evidence bars and future boundary work without accepting a Request For
  Comments or closing any Artificial Intelligence Question entry.
- `providers/provider-plugin-boundary.md` defines the Core versus
  provider-plugin boundary for the `ModelProvider` contract, transport
  taxonomy, plugin levels, model aliases, routing inputs, and the secret
  invariant without closing any Artificial Intelligence Question entry or
  proposing a new identifier.
- `providers/multimodal-inference-boundary.md` defines the Core multimodal
  extension for capabilities, task envelopes, generation-job lifecycle,
  `Unknown` reconciliation, asset outputs, agent events, Lua adapter placement,
  and generic-provider direction as a beyond-v0.1 proposal without closing any
  Artificial Intelligence Question entry or proposing a new identifier.
- `interfaces/panel-environment-awareness.md` defines the bitty-ai-facing
  awareness boundary for sanitized Agent View, Use-versus-Read behavior,
  environment-handle semantics, no-persistence defaults, and exported-only scope
  without closing any Artificial Intelligence Question entry or proposing a new
  identifier.
- `persistence/storage-memory-export-design.md` defines the bitty-ai-side
  storage direction for per-session SQLite, catalogs, content-addressed
  objects, events, context recipes, memory tiers, export scopes, lifecycle, and
  project identity without closing any Artificial Intelligence Question entry
  or proposing a new identifier.
- `persistence/history-consumption-boundary.md` defines the bitty-ai-side
  history-consumption boundary, including the consume-not-own principle,
  `HistoryProvider` direction, scoped reads, command attribution, agent-command
  sinks, and command-history versus panel-output separation without closing any
  Artificial Intelligence Question entry or proposing a new identifier.
- `integration/integration-risk-register.md` records eight cross-repository
  integration risks with required contracts, decision ownership, and acceptance
  evidence without closing any Artificial Intelligence Question entry or
  proposing a new identifier.
- `agent/fragment-pre-split-rule.md` defines the implemented slice-layer
  fragment pre-split and reassembly mapping from 64 KiB runtime fragments to at
  most 16 KiB transport parts, including code-point boundaries, a continuation
  marker, dense cursor-assigned `seq`, and a caller-bindable reassembly identity.
  It limits the claim to the mapping layer and records that no `rich.*` wire
  method is registered.

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
