---
title: Specifications
description: Index of the accepted AI-core contract and research distillations
category: specifications
audience: mixed
document_type: index
status: accepted
website_publish: true
sidebar_order: 10
---

# Specifications

Index of the AI-core specifications tree. This tree retains the accepted
contract and the research distillations; the draft technical contracts now live
in topic trees. Normative detail lives in the linked pages; this index carries
no duplicate normative prose. Draft text does not authorize shipped, stable, or
compatibility-guaranteed behavior.

## Admission criteria

A document belongs here when it is an accepted versioned contract or a
provenance-preserving research distillation. Draft technical contracts and
dispositions live in the topic trees routed below.

## Authority and status

The accepted [IPC and Agent RFC](ipc-agent-rfc.md) is the overriding authority
for IPC framing, wire, scopes, and agent messages. A distillation is a research
record, not a contract: it preserves provenance and observations and never
becomes a decision or an implementation claim by implication. Shared
cross-project governance stays in
[bitty-docs](https://github.com/bitty-terminal/bitty-docs) and is linked, never
copied. Implementation claims require evidence from the owning code repository.

## Accepted contract

| Document                              | Status   | Purpose                                                                 |
| ------------------------------------- | -------- | ----------------------------------------------------------------------- |
| [IPC and Agent RFC](ipc-agent-rfc.md) | Accepted | Bounded IPC framing, wire, auth, scopes, and agent messages for OQ-018. |

## Draft topic trees

Draft technical contracts and dispositions were distributed from this tree into
topic trees; each tree's index routes its documents and restates no status.

| Tree            | Index                                                                                                                                                             |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `architecture/` | [Architecture diagrams](../architecture/README.md) plus the [AI Architecture](../architecture/ai-architecture.md) specification and the R1-R6 draft dispositions. |
| `context/`      | [Context index](../context/README.md).                                                                                                                            |
| `providers/`    | [Providers index](../providers/README.md).                                                                                                                        |
| `agent/`        | [Agent index](../agent/README.md).                                                                                                                                |
| `persistence/`  | [Persistence index](../persistence/README.md).                                                                                                                    |
| `interfaces/`   | [Interfaces index](../interfaces/README.md).                                                                                                                      |
| `integration/`  | [Integration index](../integration/README.md).                                                                                                                    |
| `product/`      | [Product index](../product/README.md).                                                                                                                            |

## Research distillations

Provenance-preserving research syntheses. The coverage register and AIQ triage
live under [`docs/sources/`](../docs/sources/README.md).

| Distillation                                                                                                                             | Status | Purpose                                                                                          |
| ---------------------------------------------------------------------------------------------------------------------------------------- | ------ | ------------------------------------------------------------------------------------------------ |
| [AI research distillation from 013, 017, and 018](research-distillation-013-017-018.md)                                                  | Draft  | Traceable synthesis of AI-relevant research findings and unresolved questions.                   |
| [Shared workspace services and agent coordination research](research-distillation-021.md)                                                | Draft  | Tooling reuse, teams, context messaging, and panel lifecycle synthesis.                          |
| [Panel research distillation for bitty-ai (039)](research-distillation-039-bitty-ai.md)                                                  | Draft  | Panel topics, AI workspace object model, and agent workspace sequencing.                         |
| [Plugin-system research distillation for bitty-ai (040)](research-distillation-040-bitty-ai.md)                                          | Draft  | Extension model, host plugin registries, manifest, permissions, and layer split.                 |
| [IPC-value research distillation for bitty-ai (041)](research-distillation-041-bitty-ai.md)                                              | Draft  | IPC second extension boundary, out-of-process agent runtime, and capability permissions.         |
| [Execution-supervisor research distillation for bitty-ai (044)](research-distillation-044-bitty-ai.md)                                   | Draft  | Execution supervisor, job service object model, lifetime, timeout, and outcome boundary.         |
| [Lua-versus-Core safety-boundary research distillation for bitty-ai (045)](research-distillation-045-bitty-ai.md)                        | Draft  | Four-layer architecture, budgets, attenuation, commander, and secret-handle policy stack.        |
| [Quality-formula and Context-Compiler research distillation for bitty-ai (048-049)](research-distillation-048-049-bitty-ai.md)           | Draft  | Quality formula, Wheel architecture, Context Compiler zones, cache, and budgets.                 |
| [Wheel-config and Git-model research distillation for bitty-ai (050-051)](research-distillation-050-051-bitty-ai.md)                     | Draft  | Wheel agents split, config classes, trust, and Git-inspired context storage modes.               |
| [Wheel decoupling and Core-Plugin boundary research distillation for bitty-ai (052)](research-distillation-052-bitty-ai.md)              | Draft  | Wheel Core versus Plugin boundary, Lua-to-Lua composition, and layering.                         |
| [Wheel scope and framework-illustration research distillation for bitty-ai (046/053/054)](research-distillation-046-053-054-bitty-ai.md) | Draft  | Wheel Coding scope, provider and streaming illustrations, and packaging notes.                   |
| [Event-Sourced Agent Workspace research distillation for bitty-ai (055)](research-distillation-055-bitty-ai.md)                          | Draft  | Six-object split, agent versus task graphs, mailbox, typed merges, and Task-as-Issue.            |
| [Wheel Context Storage and Reasoning Management research distillation for bitty-ai (056)](research-distillation-056-bitty-ai.md)         | Draft  | Stored history versus compiled context, Reasoning Record, Context Compiler, and cache economics. |
