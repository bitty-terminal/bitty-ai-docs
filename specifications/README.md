---
title: Specifications
description: Index of AI-core technical contracts draft dispositions and integration registers
category: specifications
audience: mixed
document_type: index
status: accepted
website_publish: true
sidebar_order: 10
---

# Specifications

Index of the AI-core technical contracts. Normative detail lives in the
linked pages; this index carries no duplicate normative prose. Draft text
does not authorize shipped, stable, or compatibility-guaranteed behavior.

## Accepted contract

| Document                              | Status   | Purpose                                                                 |
| ------------------------------------- | -------- | ----------------------------------------------------------------------- |
| [IPC and Agent RFC](ipc-agent-rfc.md) | Accepted | Bounded IPC framing, wire, auth, scopes, and agent messages for OQ-018. |

## Draft architecture and core design

| Document                                                                    | Status | Purpose                                                                                    |
| --------------------------------------------------------------------------- | ------ | ------------------------------------------------------------------------------------------ |
| [AI Architecture](ai-architecture.md)                                       | Draft  | ModelProvider, ContextProvider, Tool bus, and Agent layers direction, post 1.0.            |
| [Context management architecture](context-management.md)                    | Draft  | Session journal, context view model, and multi-level compression pipeline.                 |
| [Command and tool architecture](command-tool-architecture.md)               | Draft  | Slash command registry, tool runtime separation, and Core versus Lua boundary.             |
| [Agent coordination architecture](agent-coordination.md)                    | Draft  | Workspace services, supervision, delegation, teams, and panel lifecycle.                   |
| [Code intelligence architecture](code-intelligence.md)                      | Draft  | LSP sharing, stateful mediation, lint/build/test reuse, verification fingerprinting.       |
| [Persistence and evidence architecture](persistence-evidence.md)            | Draft  | Journal, evidence, projection, indexing, and replay tradeoffs under privacy controls.      |
| [Prefix-Cache-Friendly Context Design](prefix-cache-context-design.md)      | Draft  | Stable-prefix layering and deterministic serialization for prefix-cache reuse.             |
| [Prompt Layering Design](prompt-layering-design.md)                         | Draft  | Five-layer prompt contract with capability separation and stable assembly order.           |
| [Dependency Strategy](dependency-strategy.md)                               | Draft  | Std-only runtime kernel with post-v0.1 adapter dependency boundaries.                      |
| [AI Vertical Slice Pressure Test](ai-vertical-slice-pressure-test.md)       | Draft  | Experimental vertical slice on generic Core primitives with a mapped abstraction gap list. |
| [v0.1 Implementation Profile](implementation-profile-v0.1.md)               | Draft  | Minimal experimental scope for the first runtime increment.                                |
| [Browser and Agent Panel Integration Pre-Study](browser-agent-pre-study.md) | Draft  | Browser WebView and Agent panel integration via Panel Runtime, MCP, and memory isolation.  |

## R1 to R6 draft dispositions

| Document                                                        | Status | Purpose                                                                        |
| --------------------------------------------------------------- | ------ | ------------------------------------------------------------------------------ |
| [Execution ownership R1](execution-ownership-r1.md)             | Draft  | Single-agent execution ownership via ExecutionContext and Panel projection.    |
| [Tool transport R2](tool-transport-r2.md)                       | Draft  | Unified authorization backend with native and MCP path selection.              |
| [Context retention R3](context-retention-r3.md)                 | Draft  | Consent-bounded retention, deletion propagation, and recovery limits.          |
| [Code intelligence sharing R4](code-intelligence-sharing-r4.md) | Draft  | Domain-keyed language-service sharing with lease and generation fencing.       |
| [Task lifecycle R5](task-lifecycle-r5.md)                       | Draft  | Product Task authority versus CarryCtx backend-or-handoff with review rules.   |
| [Persistence profile R6](persistence-profile-r6.md)             | Draft  | Append-ordered journal backend profile with projection as a derived operation. |

## Boundary, integration, and process drafts

| Document                                                                  | Status | Purpose                                                                            |
| ------------------------------------------------------------------------- | ------ | ---------------------------------------------------------------------------------- |
| [Provider plugin boundary](provider-plugin-boundary.md)                   | Draft  | Core versus provider-plugin boundary for ModelProvider contract and secrets.       |
| [Multimodal inference boundary](multimodal-inference-boundary.md)         | Draft  | Core multimodal extension for capability vocabulary, task envelope, and assets.    |
| [Panel environment awareness](panel-environment-awareness.md)             | Draft  | Bitty-ai-facing awareness note for sanitized Agent View and env-handle semantics.  |
| [Storage memory and export design](storage-memory-export-design.md)       | Draft  | Per-session storage direction with catalog control plane and export scopes.        |
| [History consumption boundary](history-consumption-boundary.md)           | Draft  | Consume-not-own history boundary with scoped panel-history-style reads.            |
| [Cross-repo integration risk register](integration-risk-register.md)      | Draft  | Handoff register of bitty-ai and bitty cross-repo integration risks.               |
| [Bitty-side integration input](bitty-side-integration-input.md)           | Draft  | Draft bitty-side handoff input assembling dispatch, execution, transport needs.    |
| [Bitty-side delivery verification](bitty-side-delivery-verification.md)   | Draft  | Read-only verification mapping bitty deliveries to integration inputs.             |
| [RFC-split readiness evaluation](rfc-split-readiness.md)                  | Draft  | Evaluation of which drafts are ready to split into narrow RFCs with evidence bars. |
| [Fragment pre-split and reassembly rule](fragment-pre-split-rule.md)      | Draft  | Implemented slice-layer fragment pre-split and reassembly mapping rule.            |
| [Caller attribution design](caller-attribution-design.md)                 | Draft  | Candidate caller-attribution field and LLM-plugin boundary from research 047.      |
| [Audited git wrapper API design](git-wrapper-api-design.md)               | Draft  | Candidate API shape for audited version-control access behind a scoped wrapper.    |
| [HostBoundary trait and lint-gate design](host-boundary-trait-design.md)  | Draft  | Core versus Lua enforcement sketch through a trait and a fail-closed lint gate.    |
| [Prototype-to-Core promotion checklist](prototype-promotion-checklist.md) | Draft  | Hard-gate checklist for promoting workflow prototypes into the AI Core.            |

## Registers

| Document                                              | Status | Purpose                                                               |
| ----------------------------------------------------- | ------ | --------------------------------------------------------------------- |
| [AI Unresolved Questions](ai-unresolved-questions.md) | Draft  | Local research choices with feature prerequisites and review routing. |

## Research distillations

Provenance-preserving research syntheses. A distillation is a research record,
not a contract: it preserves provenance and observations and never becomes a
decision or an implementation claim by implication. The coverage register and
AIQ triage live under [`docs/sources/`](../docs/sources/README.md).

| Distillation                                                                                                                             | Status | Purpose                                                                                   |
| ---------------------------------------------------------------------------------------------------------------------------------------- | ------ | ----------------------------------------------------------------------------------------- |
| [AI research distillation from 013, 017, and 018](research-distillation-013-017-018.md)                                                  | Draft  | Traceable synthesis of AI-relevant research findings and unresolved questions.            |
| [Shared workspace services and agent coordination research](research-distillation-021.md)                                                | Draft  | Tooling reuse, teams, context messaging, and panel lifecycle synthesis.                   |
| [Panel research distillation for bitty-ai (039)](research-distillation-039-bitty-ai.md)                                                  | Draft  | Panel topics, AI workspace object model, and agent workspace sequencing.                  |
| [Plugin-system research distillation for bitty-ai (040)](research-distillation-040-bitty-ai.md)                                          | Draft  | Extension model, host plugin registries, manifest, permissions, and layer split.          |
| [IPC-value research distillation for bitty-ai (041)](research-distillation-041-bitty-ai.md)                                              | Draft  | IPC second extension boundary, out-of-process agent runtime, and capability permissions.  |
| [Execution-supervisor research distillation for bitty-ai (044)](research-distillation-044-bitty-ai.md)                                   | Draft  | Execution supervisor, job service object model, lifetime, timeout, and outcome boundary.  |
| [Lua-versus-Core safety-boundary research distillation for bitty-ai (045)](research-distillation-045-bitty-ai.md)                        | Draft  | Four-layer architecture, budgets, attenuation, commander, and secret-handle policy stack. |
| [Quality-formula and Context-Compiler research distillation for bitty-ai (048-049)](research-distillation-048-049-bitty-ai.md)           | Draft  | Quality formula, Wheel architecture, Context Compiler zones, cache, and budgets.          |
| [Wheel-config and Git-model research distillation for bitty-ai (050-051)](research-distillation-050-051-bitty-ai.md)                     | Draft  | Wheel agents split, config classes, trust, and Git-inspired context storage modes.        |
| [Wheel decoupling and Core-Plugin boundary research distillation for bitty-ai (052)](research-distillation-052-bitty-ai.md)              | Draft  | Wheel Core versus Plugin boundary, Lua-to-Lua composition, and layering.                  |
| [Wheel scope and framework-illustration research distillation for bitty-ai (046/053/054)](research-distillation-046-053-054-bitty-ai.md) | Draft  | Wheel Coding scope, provider and streaming illustrations, and packaging notes.            |
| [Event-Sourced Agent Workspace research distillation for bitty-ai (055)](research-distillation-055-bitty-ai.md)                          | Draft  | Six-object split, agent versus task graphs, mailbox, typed merges, and Task-as-Issue.     |

## Authority and status

The four projected AI-core topic trees (`architecture/`, `context/`,
`providers/`, `reference/`) are plans, not directories; landing order lives
in the [documentation map](../docs/README.md#planned-trees). Shared
cross-project governance stays in
[bitty-docs](https://github.com/bitty-terminal/bitty-docs) and is linked,
never copied. Implementation claims require evidence from the owning code
repository.
