---
title: Provider configuration source and helper binary/mode contract
description: Draft decision input comparing provider catalog sources and helper binary modes that block strict provider publication
category: specifications
audience: contributor
document_type: specification
status: draft
website_publish: false
sidebar_order: 48
---

# Provider configuration source and helper binary/mode contract

> Status: **decision input (AI-0349), draft. Not an accepted contract, not a
> design approval, and not implementation evidence.** Nothing here authorizes
> code, a schema, a new dependency, or a packaging change. Strict
> `ai.provider_list` stays disabled and
> [bitty-ai#548](https://github.com/bitty-terminal/bitty-ai/issues/548) stays
> blocked until the owners named in [Open points](#open-points) accept a
> contract.

Task: `AI-0349` (bitty-ai). Blocked consumers: `AI-0329`
([bitty-ai#521](https://github.com/bitty-terminal/bitty-ai/issues/521), strict
`ai.provider_list`) and
[bitty-ai#548](https://github.com/bitty-terminal/bitty-ai/issues/548) (packaging
`ai` to R2).

## Purpose and scope

Two independent product gaps block strict provider publication and packaging:

1. **No production provider/model configuration source.** No loader populates
   `ProviderRegistry` outside tests; the helper entrypoint uses
   `ControlOnlyRegistry`, a deliberately empty placeholder. MPR-4 records that
   no provider configuration path is implemented (quoted second-hand in the
   2026-10-21 Wheel nightly; the packet itself is not present in this
   workspace).
2. **No accepted binary/mode contract.** Core's `ResidentHelper` expects the
   DIR-030 component `ai` with executable `bitty-ai` and a stdio data path,
   while this repository builds only `bitty-ai-helper`, a control-only fd 3/4
   entrypoint that exits when publication authority closes.

This document compares the options, recommends one contract for each gap, and
lists the exact decisions Core and bitty-ai owners must make. It deliberately
does not choose an implementation crate, schema finalization, dependency, or
release step.

Out of scope: implementing a loader, adding or renaming binaries, changing
Core, enabling strict dispatch, publishing artifacts, editing canonical
documentation corpora.

**Recommendation at a glance (proposal, not accepted):** C6 — a helper-owned,
metadata-only provider catalog whose absolute path Core supplies — plus M2 —
one `bitty-ai` artifact with a required closed `--mode` value, where the
`resident-publication` role serves stdio data RPC and the fd 3/4 control
session in the same process so the publication proof compares the registry the
`ai.provider_list` handler actually reads.

## Normative sources this specification must not weaken

- DIR-030 Native Component Boundary
  ([bitty-docs](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/development/native-component-boundary.md)):
  component `ai`, executable `bitty-ai`, install-root resolution, no `PATH`,
  cleared environment, digest-before-spawn, one process per instance, stdio
  coprocess.
- R1 provider/routing contract
  ([Execution ownership R1](../architecture/execution-ownership-r1.md)): the
  helper owns the authoritative populated `ProviderRegistry` and proves
  equality against the registry the `ai.provider_list` handler consumes; Core
  neither populates the registry nor holds credentials.
- The bitty-docs security corpus, especially MP-10 and the provider-plugin
  secret invariant: credentials are referenced, never embedded; a raw value
  crossing a model, agent, plugin, diagnostic, trace, snapshot, journal,
  discovery file, **child environment**, or `BITTY_*` variable is release
  blocking.
- Draft ingress designs
  ([Authenticated provider publication](../architecture/authenticated-provider-publication.md)
  and
  [bitty-docs authenticated provider publication ingress](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/authenticated-provider-publication-ingress.md));
  where they overlap with this document, the stricter requirement applies and
  a divergence is a defect to resolve before acceptance.

## Terminology

| Term                      | Meaning here                                                                                                                                                                   |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Provider catalog          | Helper-owned, metadata-only document listing providers and their models. v1 carries no endpoints, headers, transport kinds, aliases, or credentials.                           |
| Catalog path              | Absolute path Core derives from its user configuration root and passes as a fixed argv value. The helper validates it and never resolves a relative path.                      |
| Provider registry         | `ProviderRegistry` in `bitty-ai-runtime`, holding validated `ModelRegistration` values, aliases, and a local mutation `revision` that never crosses IPC.                       |
| Feed revision             | View-owned revision of the published projection. It starts at `1` and is distinct from the registry's local mutation revision.                                                 |
| Tolerant path             | Existing `ai.provider_list` and resident data-RPC behavior that serves without a strict publication proof.                                                                     |
| Strict publication        | Target `ai.provider_list` path that admits only after comparing retained publication material with the registry projection the handler reads. It stays `Unbound` and disabled. |
| Resident mode             | `--mode=resident`: stdio data RPC, resident, no idle stop.                                                                                                                     |
| Resident-publication mode | `--mode=resident-publication`: stdio data RPC plus fd 3 bootstrap and fd 4 control in one process, with launch-scoped TLS.                                                     |
| Control-only mode         | `--mode=control-only`: fd 3/4 only, no stdio, empty placeholder registry, exits on authority close; fixture and bring-up only.                                                 |
| Publication proof         | Equality check between retained publication bytes and the current registry projection; a restart or configuration change revokes it.                                           |

The authoritative registry, selection, secret, and descriptor definitions stay
with the owning implementation modules this document cites. This section only
fixes vocabulary for the proposal.

## Pinned baseline

Facts below are current-state observations from the checkouts used for this
task. They describe what exists, not what the proposed contract accepts.

| Surface           | Pinned evidence                                                                                                                                  | Limit of claim                                                                                                                                                                                                      |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| bitty-ai          | `03e7d3b` (2026-10-23)                                                                                                                           | Only `bitty-ai-helper` is built; no `bitty-ai` binary, no stdio server entrypoint, no production registry source.                                                                                                   |
| Core (`bitty`)    | `9b5d117d` (2026-10-23)                                                                                                                          | `ResidentHelper` resolves the `ai` component, verifies the digest before every spawn, clears the environment, and passes **no argv**. `ai_publication_launch` prepares fd 3/4 but is not wired to the helper spawn. |
| Descriptor rule   | `bitty` `crates/bitty-runtime/src/component/descriptor.rs`                                                                                       | `executable` must equal `bitty-<name>` exactly; the descriptor has no argv, mode, or second-executable field.                                                                                                       |
| Helper entrypoint | `bitty-ai` `crates/bitty-ai-control/src/bin/bitty-ai-helper.rs`                                                                                  | Adopts fd 3/4 unconditionally on Unix, fails closed on non-Unix, runs the control state machine, and exits on authority close.                                                                                      |
| Registry view     | `bitty-ai` `crates/bitty-ai-control/src/control_registry.rs`                                                                                     | Fixed empty `provider-routing` projection at feed revision `1`; explicitly not a provider registry.                                                                                                                 |
| Publication state | `bitty-ai-docs` `architecture/authenticated-provider-publication.md`, `bitty-docs` `docs/security/authenticated-provider-publication-ingress.md` | Both are drafts. The security draft proposes "keep existing stdio for bounded data RPC and add a dedicated control stream" (D1), and one active control session per supervised helper (CB-9).                       |
| Provider registry | `bitty-ai` `crates/bitty-ai-runtime/src/selection.rs`                                                                                            | `ProviderRegistry` holds validated `ModelRegistration` values, aliases, and a local mutation `revision` that never crosses IPC.                                                                                     |
| Secret container  | `bitty-ai` `crates/bitty-ai-runtime/src/secret.rs`                                                                                               | `SecretField` redacts unconditionally; the only raw accessor is `expose_for_adapter`.                                                                                                                               |

## Decision area 1: provider/model configuration source

### Requirements

- The helper stays the sole registry, selection, provider-I/O, and credential
  authority; Core must not parse provider config, populate the registry, or
  hold credentials.
- The source must work both when Core spawns the helper and when the AI
  component runs standalone/headless.
- It must carry the exact fields the accepted projection needs: `provider_id`,
  model `name`, closed capability labels, `context_window_tokens`,
  `input_cost_weight`, `output_cost_weight`.
- It must fail closed on unknown fields, duplicates, conflicts, out-of-bounds
  values, and malformed input, and must never invent providers.
- Changes must not silently invalidate a live proof: either the source is
  immutable for the process lifetime, or a mutation channel exists that
  revokes the proof and republishes. No such channel exists today.

### Provider source option matrix

| ID  | Source                                                     | Authority / owner                                 | Secret handling                                                                                      | Reload                                                                      | Cross-platform                                                       | Standalone parity              | Verdict                                                 |
| --- | ---------------------------------------------------------- | ------------------------------------------------- | ---------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------- | -------------------------------------------------------------------- | ------------------------------ | ------------------------------------------------------- |
| C1  | Core config file (Core parses, passes entries)             | Core                                              | Core would hold/forward values or refs; violates Core-holds-no-credentials and the proof-owner split | Core-owned reload semantics; helper must ingest a new feed                  | Core platform code only                                              | None (standalone has no Core)  | Reject                                                  |
| C2  | Shared user file parsed by Core, delivered to helper       | Split: Core locates/parses, helper registers      | Core sees credential refs and could be pressured into resolving them                                 | Core reload + IPC delivery needed                                           | Core locates; helper trusts Core                                     | None                           | Reject                                                  |
| C3  | Helper-owned user file (helper parses)                     | Helper                                            | Refs only; values stay in helper-side storage                                                        | Load once per process; restart to change                                    | Path must be derived or supplied (see C6)                            | Yes                            | Viable                                                  |
| C4  | Environment only                                           | Ambient                                           | Forbidden: child env is cleared and secret passthrough is release blocking                           | Requires restart                                                            | Env names differ per OS; no structured catalog                       | Partial                        | Reject as primary                                       |
| C5  | Host secret store / credential broker as the config source | Core or OS                                        | Values never cross to helper; handles only                                                           | Out of scope for metadata                                                   | Keyring availability varies                                          | No                             | Reject as source; keep as credential-resolution adjunct |
| C6  | Helper-owned file with Core-supplied absolute path         | Helper owns content and schema; Core only locates | Same as C3                                                                                           | Same as C3                                                                  | Core derives the path from its user config root; helper validates it | Yes, with env-derived fallback | **Recommended**                                         |
| C7  | Lua preset registration (`ai.register_provider`)           | Lua plugin                                        | Presets must not carry values                                                                        | Lua reload semantics; helper registry would need an IPC registration method | Lua host required                                                    | No                             | Defer; open under OQ-080                                |
| C8  | No source (status quo)                                     | None                                              | No secrets because no entries                                                                        | None                                                                        | N/A                                                                  | N/A                            | Safe interim only; strict stays off                     |

### Provider source option analysis

**C1/C2 Core-owned config.** Core has a user configuration surface (Lua
`init.lua`, `bitty-plugins.toml`), so storing AI provider entries there looks
convenient. It is rejected for three independent reasons: (a) the accepted R1
split makes the helper the authoritative registry and Core a read-only
publication view, so Core-authored entries invert the source; (b) Core would
become an AI-specific configuration API, which BA-6 classifies as a plugin API
gap, not a Core feature; (c) Core would see credential references and become
the natural place to resolve them, contradicting "Core holds no credential".
Standalone mode has no Core at all.

**C3 helper-owned file.** This is the only option that keeps one authority and
works identically for Core-spawned and standalone runs. The unsolved part is
path discovery: Core's spawn clears the environment, so the helper cannot read
`XDG_CONFIG_HOME`, `HOME`, or `APPDATA`. Options are Core passing a path (C6),
extending the component environment allowlist with layout variables (a DIR-030
refinement that affects `net` too), or platform account APIs (unsafe and
platform-specific). C6 is the smallest change.

**C4 environment.** Core clears the environment to the DIR-030 allowlist
(proxy, TLS trust, locale; `PATH` never). An env-only provider catalog would
need a new allowlist entry per variable and a structured encoding. More
importantly, the security invariant makes a credential in the child
environment a release-blocking defect, so `api_key_env` cannot be implemented
by forwarding the value to the helper. Environment is rejected as a primary
source. A single non-secret test-only override (for example a config path) is
acceptable only if Core owners approve it and it is documented as test-only.

**C5 host secret store.** This is a credential-resolution mechanism, not a
provider catalog. It cannot supply model metadata. Keep it out of v1; the
ownership question (Core-side vs helper-side store) is open (OD-J1).

**C6 recommended hybrid.** The helper owns the file content, schema, parsing,
validation, registry population, and error vocabulary. Core owns only the
absolute path: it derives `<core-config-dir>/ai/<catalog-file>` (where
`<core-config-dir>` is Core's existing per-user configuration directory) and
passes it as a fixed argv value at spawn.
Standalone `bitty-ai` derives the same path from its inherited environment or
accepts the same flag. This keeps Core free of AI schema knowledge while
making the file reachable under `env_clear`.

**C7 Lua presets.** MPC-5/OQ-080 is still open and no registration API exists.
Lua runs in Core; the registry lives in the helper. Bridging presets would
require a new IPC registration method, a second mutation path, and a consent
model for preset-supplied metadata. Not viable for v1; keep as a later
catalog/override layer if OQ-080 resolves that way.

**C8 status quo.** Correct and safe today. It is not a production contract:
the only published projection would be the empty placeholder, which is exactly
the "fake registry" the security constraints forbid presenting as
configured.

### Recommended C6 shape (proposal only)

- **Content:** metadata-only catalog. No `base_url`, headers, transport kind,
  `api_key_env`, `api_key_cmd`, tokens, or aliases in v1.
- **Schema sketch:** a bounded document with a schema version and a
  `providers` array; each provider has an `id` and a non-empty `models` array;
  each model has `name`, `capabilities`, `context_window_tokens`,
  `input_cost_weight`, `output_cost_weight`. Field names are illustrative;
  the schema is an open decision (OD-A2).
- **Bounds:** reuse the registry constants exactly (`MAX_PROVIDER_ID_LEN`,
  `MAX_MODEL_NAME_LEN`, `MAX_REGISTERED_MODELS`, closed capability set) plus a
  file-size bound and a nesting-depth bound.
- **Validation:** construct `ModelRegistration::new` for every entry and call
  `ProviderRegistry::register`; any refusal aborts the load with the registry
  unchanged. Duplicate and conflict checks run before capacity, matching the
  registry.
- **Feed revision:** view-owned, starts at `1` (the publication contract's
  floor), and is not the registry's local `revision`. Load-once means it stays
  `1` for the process lifetime.
- **Secret-shaped keys:** refuse any key named `api_key`, `api_key_env`,
  `api_key_cmd`, `credential`, `secret`, `token`, `password`, or
  `authorization` anywhere in the document. A user who pastes a key into the
  catalog gets a fixed refusal instead of silent storage.
- **File checks:** absolute path only; regular file; no symlink; user-only
  permissions on Unix (owner match, no group/other bits); reject FIFOs and
  devices before reading; size and UTF-8 bounds. Windows ACL verification is a
  separate platform decision (OD-J5).
- **Absent vs invalid:** an absent file is a normal empty configuration: start
  with zero providers and a fixed diagnostic; strict stays off. An invalid or
  unreadable file fails startup so Core's crash path surfaces the bounded
  stderr tail (Core does not log stderr otherwise under D4). Never fall back to
  a built-in provider set.
- **Reload:** load once, immutable for the process lifetime. Changing the
  catalog requires restarting the helper; a restart revokes publication
  authority and requires a fresh launch and republish. No file watching, no
  SIGHUP, no IPC mutation channel in v1.

### Provider source test/evidence requirements

| Option | Minimum evidence                                                                                                                                                                                                                                                                                                                          |
| ------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| C1/C2  | A written disposition rejecting Core-side parsing, plus a proof that no Core path reads provider entries or credentials. No code evidence is expected because the option is rejected.                                                                                                                                                     |
| C3     | Loader unit tests for bounds, duplicates, conflicts, unknown fields, capability vocabulary, and zero entries; integration test proving the registry projection equals the parsed catalog; standalone startup test. Path discovery remains unproven until C6/allowlist decision.                                                           |
| C4     | Test that the helper process environment contains only the DIR-030 allowlist entries; a sentinel variable is absent; an env-configured run is refused with a fixed diagnostic.                                                                                                                                                            |
| C5     | Threat-model entry and security review of handle ownership; no implementation before the store owner accepts.                                                                                                                                                                                                                             |
| C6     | All C3 evidence plus: Core spawn test proving the exact absolute path is passed; helper refusal for relative, symlinked, oversized, non-regular, group-writable, and wrong-owner files; absent-file empty-registry test; invalid-file fail-fast test; diagnostic scan proving no catalog values, resolved paths, or file contents appear. |
| C7     | Requires an accepted OQ-080 registration API, an IPC mutation design, proof invalidation on preset mutation, and a consent review. Not planned in v1.                                                                                                                                                                                     |
| C8     | Existing tolerant `ai.provider_list` tests stay green; strict remains `Unbound`; no publication of the placeholder beyond fixtures.                                                                                                                                                                                                       |

## Decision area 2: helper binary and mode contract

### Hard constraints

- **Descriptor v1:** `executable` must be exactly `bitty-<name>`. For component
  `ai`, the only valid executable is `bitty-ai`. A second shipped executable
  needs either a descriptor change or a second component.
- **No argv today:** Core's `ResidentHelper` spawn sets no arguments; adding a
  mode flag is a Core change, not only a bitty-ai change.
- **Cleared environment:** mode selection cannot use environment variables
  without a DIR-030 allowlist refinement.
- **One registry, one proof:** the accepted R1 contract compares retained
  publication bytes with "the current `ProviderRegistry` projection consumed by
  `ai.provider_list`". A publication control session in one process cannot
  prove equality for a handler in another process unless both share registry
  state, which would add a second authority path.
- **Packaging:** #548 requires that the current `bitty-ai-helper` artifact is
  never published as `ai`.

### Helper mode option matrix

| ID  | Shape                                                               | Core selection                                             | Core authentication                          | One-registry/proof                                     | Lifecycle                                                                           | Packaging                                                 | Verdict                                     |
| --- | ------------------------------------------------------------------- | ---------------------------------------------------------- | -------------------------------------------- | ------------------------------------------------------ | ----------------------------------------------------------------------------------- | --------------------------------------------------------- | ------------------------------------------- |
| M1  | One `bitty-ai`, infer role from fd/stdio presence                   | Descriptor `ai` / `bitty-ai`                               | Digest + pipe/fd possession                  | Possible                                               | Ambiguous startup; implicit behavior                                                | One artifact                                              | Reject (implicit, untestable failure modes) |
| M2  | One `bitty-ai`, explicit closed `--mode` values                     | Descriptor `ai` / `bitty-ai` + fixed argv                  | Digest for stdio; launch TLS for publication | Yes, if control and data RPC share the process         | Explicit per role; control loss does not have to kill data RPC                      | One artifact                                              | **Recommended**                             |
| M3  | One `bitty-ai`, both channels always; publication optional at spawn | Descriptor + argv or fd presence                           | Digest + launch TLS                          | Yes                                                    | Publication lifetime coupled to resident spawn; restart/expiry semantics unresolved | One artifact                                              | Viable later; larger Core integration       |
| M4  | Two executables in one version dir (`bitty-ai`, `bitty-ai-helper`)  | Descriptor v1 names only `bitty-ai`; sibling is unverified | Digest covers one executable only            | No (two processes, two registries)                     | Two lifetimes; duplicate cleanup                                                    | Installer copies one executable                           | Reject (descriptor v2 + unverified sibling) |
| M5  | Two native components (`ai`, `ai-helper`)                           | Both descriptor-valid (`bitty-ai`, `bitty-ai-helper`)      | Digest per component                         | No, unless the helper serves `ai.provider_list` itself | Two supervisors, two digests, two versions                                          | Two install units; #548 forbids `bitty-ai-helper` as `ai` | Reject for v0.1                             |
| M6  | Keep only `bitty-ai-helper`; no `bitty-ai`                          | Fails DIR-030 component `ai`                               | N/A (wrong identity)                         | No data RPC                                            | Control-only                                                                        | Unpublishable as `ai`                                     | Reject (status quo blocker)                 |

### Helper mode option analysis

**M1 (implicit inference).** Tempting because it needs no Core argv change: the
binary could check whether stdin is a pipe (data mode) or fd 3/4 are open
(control mode). This is rejected. `env_clear` does not remove inherited
descriptors, so fd presence is ambient state; a future Core change, a test
harness, or a same-user wrapper can produce a valid-looking but unintended
combination. The Wheel decision input explicitly requires "explicit mode flag
or fd-presence detection, never a silent fallback" as an either/or, and the
fallback part is the risk: inference cannot distinguish "no control channel"
from "control channel expected but missing". The helper would either start in
the wrong role or exit for an unrelated reason, and the failure would be
diagnosed late, on the wire.

**M2 (recommended).** One artifact, one descriptor, one digest, one component
identity, three explicit roles:

- `--mode=resident` — stdio data RPC, resident, no idle stop.
- `--mode=resident-publication` — stdio data RPC plus fd 3 bootstrap and fd 4
  control; TLS establishment; control expiry or closure disables strict
  publication but must not kill tolerant data RPC.
- `--mode=control-only` — fd 3/4 only, no stdio, exits on authority close.
  This preserves today's bring-up and fixture behavior. It must never be the
  production component role, and it must not serve strict `ai.provider_list`
  (its registry is the empty placeholder).

Mode parsing is hand-written and strict: exactly one `--mode=<value>`, no
default, no aliases, no positional arguments, no duplicate flags, unknown or
malformed values exit with a dedicated usage code before any fd adoption,
config read, allocation beyond argv parsing, or TLS setup. `--provider-config`
is accepted by both resident modes (Core always passes it; standalone may omit
it and use the default path resolution) and refused for `control-only`.

Why this satisfies the constraints: the component executable stays `bitty-ai`
(M2 is the only explicit multi-role option that does); the same process can
own the registry, serve the data RPC, and hold the control session, so the
proof equality is against the state the handler actually reads; and the role is
explicit, so every failure mode is a fixed exit code rather than an inference.

Core-side change is small but real: `ResidentHelper::spawn` adds the fixed
`--mode=resident` argument (and `--provider-config` when configured), and the
publication launch path adds `--mode=resident-publication` plus
`install_child_descriptors`. Neither value is user-controlled. This is the
main decision Core owners must accept (OD-C1/OD-C3).

**M3 (single process, always capable).** This is the cleanest long-term
authority model: one process, stdio and control always available, publication
started when Core publishes. It avoids a mode switch entirely. The cost is
that publication lifetime becomes coupled to the resident helper lifetime:
`ai_publication_launch::prepare` would have to run before the first helper
spawn, the launch manifest would be created even when publication is never
used, and a control failure or 60-second session expiry would either strand
the resident process (no fresh launch without a restart) or force a restart
that kills tolerant data RPC. The draft's CB-2 ("a broker stop, crash restart,
backoff, or replacement must revoke publication authority and require a fresh
launch") makes the coupling explicit. Keep M3 as the target if the owners
decide the resident and publication lifetimes must be identical; do not choose
it as the first integration step.

**M4 (two executables, one artifact).** Descriptor v1 verifies exactly one
executable name and one digest. Shipping a sibling `bitty-ai-helper` in the
same version directory leaves it outside the descriptor and outside the digest
check that every spawn relies on. Fixing that requires descriptor v2 (a
multi-executable map, install-tool changes, and a verification rule for every
entry). Larger than M2 with no authority benefit, and it still has two
registries.

**M5 (two components).** On paper this is legal: component `ai-helper` maps to
executable `bitty-ai-helper` under the `bitty-<name>` rule. It is rejected for
v0.1 because the strict proof and the `ai.provider_list` handler must read the
same registry; a control-only component with its own registry cannot prove
equality for the resident component's handler. Serving `ai.provider_list` from
the control component instead would split the AI data path and add a second
supervision/version/digest lifecycle, plus a second install unit that #548
explicitly forbids publishing as `ai`. Revisit only if Core rejects argv-mode
selection and the architecture owner accepts two AI components.

**M6 (status quo).** No `bitty-ai`, so Core's `ResidentHelper` has nothing to
spawn. Correctly blocked.

### Core selection and authentication

Common to every viable option:

- **Resolution:** DIR-030 user-over-system search under the component install
  root; `PATH` never read; version directory as working directory; descriptor
  name/version/protocol/executable/digest validated before every spawn.
- **Integrity:** SHA-256 of the executable compared with `bitty-component.toml`
  on every spawn; mismatch fails closed before process creation.
- **Environment:** cleared, then the fixed allowlist only; no mode, path, or
  secret through the environment.
- **Stdio role:** no cryptographic peer authentication. Integrity is
  digest-before-spawn plus pipe ownership; records are validated helper-side
  under the accepted IPC framing.
- **Publication role:** launch-scoped mutual TLS (TLS 1.3, fixed ALPN, pinned
  leaf digests, launch and channel nonces) plus the bootstrap manifest binding
  component name, descriptor version, executable name, and executable digest.
  Digest verification is necessary but not sufficient (ingress CB-6).
- **No cross-role fallback:** a failed publication handshake never falls back
  to the stdio data path, and a stdio failure never falls back to another
  binary or a direct provider path.
- **Restart:** a crash, replacement, or backoff revokes publication authority
  and requires a fresh launch; the broker must never restart a helper into an
  old lifetime (CB-2).

M2-specific: Core passes the mode literal it owns (`resident` or
`resident-publication`); the helper never reads `argv[0]`, the executable
name, or its own path to infer the role. The manifest's executable name is
`bitty-ai` for both roles, so a publication process and a data process from
the same version are indistinguishable on the wire except by channel and
role, which is what the manifest already binds.

### Helper mode test/evidence requirements

| Option | Minimum evidence                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| M1     | Tests proving fd/stdio inference rejects ambiguous combinations; a fuzz corpus of descriptor layouts. The option is rejected; evidence is only needed if an owner reopens it.                                                                                                                                                                                                                                                                                                                                 |
| M2     | Helper: mode parser table (missing, unknown, duplicate, extra args, wrong config for control-only), exit codes before fd adoption, control-only parity with today's entrypoint, resident stdio frame round trip, resident-publication handshake plus data RPC concurrently, control closure/expiry leaves data RPC alive, non-Unix fail-closed. Core: spawn tests asserting the exact argv, digest mismatch, missing mode rejection, publication launch with fd3/4, and no silent restart into an old launch. |
| M3     | All M2 evidence plus launch-before-first-spawn integration, control expiry with a live resident, and a documented rule for when the resident is replaced.                                                                                                                                                                                                                                                                                                                                                     |
| M4     | Descriptor v2 design plus installer tests proving every shipped executable is digest-verified; two-process registry consistency evidence.                                                                                                                                                                                                                                                                                                                                                                     |
| M5     | Two component descriptors, install/list/remove tests, two supervisors, and proof that the strict handler reads the authoritative registry. Expected to fail the last item.                                                                                                                                                                                                                                                                                                                                    |
| M6     | None; it is the current blocked state.                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |

## Recommended contract (proposal, not accepted)

1. **Provider catalog (C6).** Helper-owned, metadata-only, load-once file.
   Core supplies only an absolute path under its user config root; the helper
   owns schema, parsing, validation, registry population, diagnostics, and
   feed revision. No credentials in v1.
2. **Helper artifact (M2).** One shipped `bitty-ai` (component `ai`,
   descriptor executable `bitty-ai`) with a required closed `--mode` value:
   `resident`, `resident-publication`, or `control-only`. `control-only` is
   bring-up/fixture-only; it keeps the empty registry and must not serve
   strict provider publication. `resident-publication` is the only role that
   adopts fd 3/4 and establishes the control session; it also serves the stdio
   data RPC in the same process, because the proof must compare the registry
   the handler reads.
3. **Core selection.** Fixed literals, never user input:
   `--mode=resident` (or `--mode=resident-publication` on the publication
   path) plus `--provider-config=<absolute-path>` when a catalog is
   configured. The descriptor stays v1; no argv field is added to it.
4. **Authentication.** Digest-before-spawn for both roles; launch-scoped
   mutual TLS for publication; no cryptographic claim for stdio beyond pipe
   ownership and digest integrity.
5. **Strict remains off.** No `Admit`, no strict `ai.provider_list`, no
   publication of the placeholder registry. AI-0329 stays blocked until its
   own gates and these decisions are accepted.

What does not change: component resolution, digest verification, environment
clearing, supervision/backoff, IPC framing, publication control records, the
accepted projection schema, and the tolerant standalone behavior.

## Migration and compatibility path

| Phase     | Content                                                                                                                                                       | Gate                                                                 |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------- |
| 0 (today) | `bitty-ai-helper` control-only; empty registry; strict off; no catalog                                                                                        | Existing tests only                                                  |
| 1         | Rename the binary to `bitty-ai`, add strict mode parsing, keep `control-only` for fixtures, add a `resident` stdio skeleton with an empty registry            | Mode/exit-code tests; no Core change                                 |
| 2         | Add the metadata-only catalog loader and a real registry view; feed revision 1; still no production publication                                               | Loader and projection-equality tests; security review of file checks |
| 3         | Core integration behind a default-off feature: fixed argv, catalog path, fd3/4 on the publication role, establishment/projection/publish/ack/bind; no `Admit` | Cross-repo integration tests; independent review                     |
| 4         | Strict `ai.provider_list` for the single domain after AI-0329 acceptance                                                                                      | Published negative/refusal matrix                                    |
| 5         | Packaging (#548) only after the platform matrix and owner approval; publish `bitty-ai`, never `bitty-ai-helper`                                               | Per-platform install/start/stop evidence                             |

Compatibility notes:

- There is no published artifact and no stable external contract, so no
  dual-name shim is justified. Keeping `bitty-ai-helper` as an alias would
  ship an unverified sibling executable; remove or rename it in the same
  change that introduces `bitty-ai`.
- In-repo consumers to update: control crate tests and fixtures, the control
  crate README, and any developer scripts that name `bitty-ai-helper`.
- Documentation that currently describes the experimental helper (for example
  the bitty-manual AI extension page and the security draft's current-shapes
  table) becomes a docs-sync item after acceptance; this task does not edit
  those corpora.
- If Core rejects argv-based mode selection, the fallback order is M3 (single
  process, always capable) first, then M5 (two components) only with an
  explicit architecture decision. M4 (descriptor v2) is the largest change and
  has no authority benefit.

## Failure modes

| Failure                                                                          | Detection                               | Required behavior                                                                             | Evidence               |
| -------------------------------------------------------------------------------- | --------------------------------------- | --------------------------------------------------------------------------------------------- | ---------------------- |
| Missing/unknown/duplicate mode                                                   | argv parse before any fd or file access | Exit with a dedicated usage code; fixed diagnostic                                            | Exit-code table tests  |
| Relative or non-absolute catalog path                                            | argv validation                         | Exit with a usage code; no path-resolution fallback                                           | Spawn tests            |
| Catalog absent                                                                   | File open                               | Start with zero providers; fixed diagnostic; strict off                                       | Empty-registry tests   |
| Catalog invalid, oversized, wrong permissions, symlink, non-regular, wrong owner | Pre-read checks and parser              | Fail startup with a fixed diagnostic; no fallback source                                      | Refusal matrix         |
| Catalog contains a secret-shaped key                                             | Parser key check                        | Refuse the whole document; never store or log the value                                       | Sentinel tests         |
| Registry entry refused (duplicate/conflict/bounds)                               | `ProviderRegistry::register`            | Abort the load; registry unchanged                                                            | Loader tests           |
| Descriptor/digest mismatch                                                       | Core before spawn                       | No process; no partial state                                                                  | Core spawn tests       |
| fd3/4 missing or wrong type in `resident-publication`                            | Adoption/type check                     | Exit with a fixed code; never downgrade to `resident`                                         | Helper tests           |
| fd3/4 present in `resident`                                                      | Mode contract                           | Do not adopt; keep the data path                                                              | Helper tests           |
| Control handshake failure                                                        | TLS/establishment                       | Publication disabled; tolerant data RPC may continue; strict never enabled                    | Integration tests      |
| Control session expiry                                                           | Deadline                                | Strict disabled; no automatic respawn; tolerant path unaffected                               | Integration tests      |
| Helper crash/restart                                                             | Supervisor                              | Revoke publication; fresh launch with fresh material; no replay                               | Core supervision tests |
| Non-Unix publication                                                             | Platform gate                           | Fail closed before fd adoption; resident stdio may still work                                 | Platform tests         |
| Provider catalog change                                                          | None in v1                              | Requires restart; no live mutation                                                            | Documented behavior    |
| Zero providers                                                                   | Registry load                           | Empty truthful registry; publication of the empty projection stays refused by default (OD-A5) | Zero-provider tests    |

## Acceptance evidence this task does not produce

- No loader, mode parser, binary rename, or Core change.
- No catalog schema file, fixture, or dependency.
- No strict publication, `Admit`, or `provider_list` change.
- No packaging, R2 upload, or platform matrix.
- No canonical docs change beyond publishing this decision input; the
  affected corpora are listed for a later sync.

## Security review

The proposed contract carries these hard constraints. They are candidate
requirements for owner acceptance, not accepted policy, and they change no
security corpus.

1. **No secret logging.** Diagnostics are fixed strings or bounded structural
   error codes. They never contain catalog values, credential values or command
   lines, resolved paths, environment contents, file contents, certificate or
   key material, or model/provider strings from the wire. `SecretField`
   redaction stays the only raw-value path and remains adapter-only.
2. **No implicit fallback.** No built-in provider list, no default model, no
   alternate config source, no environment tier, no `PATH`, no different
   binary, no second component, no downgrade from `resident-publication` to
   `resident`, and no tolerant path for the strict target.
3. **No fake provider registry.** `ControlOnlyRegistry` stays fixture-only and
   must not be presented as configured state. No placeholder entries, no
   synthetic models, and no entry that is not backed by a configured catalog
   record.
4. **No strict `provider_list` before acceptance.** `dispatch_with_strict`
   stays `Unbound`; the publication control session may be exercised offline
   but no production strict handler is enabled; tests assert the refusal.
5. **No secrets on argv, in the environment, or in the catalog.** v1 carries
   no credentials at all; a later schema may carry references only, and a
   literal value is always refused. A future credential tier needs its own
   accepted decision.
6. **Bounded parsing.** File size, depth, entries, string lengths, and
   capability labels are bounded before allocation; unknown fields fail
   closed; no duplicate keys.
7. **No side effects at load.** Reading one user-owned file is the only I/O;
   no network, no command execution, no keyring access in v1.
8. **No project-level widening.** Only the user-level path is accepted;
   project files cannot add providers, models, endpoints, or credentials.
9. **Proof/lifetime discipline.** A restart, config change, or control failure
   revokes authority; the old proof cannot be replayed into a new state.
10. **Platform honesty.** Non-Unix publication stays unsupported until native
    evidence exists; Windows resident stdio is not evidence for publication.

## Verification plan

The per-option evidence tables in each decision area list the minimum evidence
for whichever option an owner accepts. This section states the review-level
gates for this document and for any later implementation of the recommended
C6/M2 contract:

- **Document evidence.** Every current-state claim resolves to the pinned
  baseline above, the recommendation stays labeled a proposal, and every open
  decision retains its owner and default.
- **Catalog evidence (C6).** Loader tests cover bounds, duplicates, conflicts,
  unknown fields, capability vocabulary, and zero entries; the projection
  equals the parsed catalog; absent and invalid files behave as specified;
  file checks reject relative, symlinked, non-regular, oversized, and
  wrong-owner inputs before reading.
- **Mode evidence (M2).** The mode parser rejects missing, unknown, duplicate,
  extra, and cross-role arguments before fd adoption or config read; each
  resident mode serves its data path; `control-only` matches today's
  entrypoint without serving strict publication.
- **Proof evidence.** `resident-publication` establishes the control session
  and serves data RPC in one process; control closure or expiry disables
  strict publication without killing tolerant data RPC; a restart requires a
  fresh launch and republish.
- **Diagnostics evidence.** A scan of emitted diagnostics finds no catalog
  values, resolved paths, file contents, or credential material, including
  the failure paths of every row in the failure-mode table.
- **Platform evidence.** Non-Unix builds fail closed before fd adoption; no
  claim of Windows publication support is made without native evidence.
- **Repository gates.** `just check` passes, and independent architecture,
  security, and docs-curator review records no blocking finding.

## Alternatives considered

The option matrices above carry the full analysis. This section records the
disposition of each alternative:

- **C1/C2 Core-owned provider configuration.** Rejected: it inverts the
  accepted R1 authority split, turns Core into an AI-specific configuration
  API, and makes Core the natural credential resolver. Standalone mode has no
  Core at all.
- **C3 helper-owned file without a supplied path.** Viable authority model,
  but path discovery under `env_clear` is unresolved; C6 adds only the
  Core-supplied absolute path.
- **C4 environment-only configuration.** Rejected as a primary source:
  credential passthrough is release blocking, the allowlist would grow per
  variable, and the encoding is unstructured.
- **C5 host secret store as the catalog source.** Rejected as a source: it is
  a credential-resolution mechanism with no model metadata. Kept as a
  possible later adjunct under OD-J1.
- **C7 Lua preset registration.** Deferred: OQ-080 is open, no registration
  API exists, and bridging presets would add a second mutation path.
- **C8 no source.** Safe interim only; the empty placeholder is not a
  production contract.
- **M1 implicit role inference.** Rejected: ambient fd/stdio state cannot
  distinguish a missing channel from an expected one, and failures surface
  late on the wire.
- **M3 always-capable single process.** Viable later: it is the cleanest
  authority model, but it couples publication lifetime to the resident spawn
  before the first integration step.
- **M4 two executables in one artifact.** Rejected: descriptor v1 verifies one
  executable, so the sibling would be unverified; fixing that needs descriptor
  v2 and still leaves two registries.
- **M5 two native components.** Rejected for v0.1: a control-only component
  cannot prove equality for the resident component's handler, and #548 forbids
  publishing `bitty-ai-helper` as `ai`.
- **M6 keep only `bitty-ai-helper`.** Rejected: DIR-030 component `ai` has
  nothing valid to spawn.

## Affected contracts

- [DIR-030 Native Component Boundary](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/development/native-component-boundary.md)
  (Normative): component `ai`, executable `bitty-ai`, install-root resolution,
  no `PATH`, cleared environment, digest-before-spawn, one process per
  instance, stdio coprocess. M2 adds fixed argv to the spawn; that refinement
  requires the Core decisions OD-C1 and OD-C3 before acceptance.
- [Execution ownership R1](../architecture/execution-ownership-r1.md) (Draft):
  the helper stays the authoritative registry and Core stays a read-only
  publication view. C6 keeps the source and proof on the helper side.
- [Authenticated provider publication](../architecture/authenticated-provider-publication.md)
  (Draft) and
  [bitty-docs authenticated provider publication ingress](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/authenticated-provider-publication-ingress.md)
  (Draft): D1 keeps stdio for bounded data RPC and adds a control stream;
  CB-9 allows one active control session per supervised helper; CB-2 requires
  a fresh launch after a restart. This document supplies the catalog source
  and the mode contract those requirements assume.
- [Provider plugin boundary](provider-plugin-boundary.md) (Draft): the secret
  invariant, transport taxonomy, and frozen v0.1 provider surface are
  unchanged; the catalog adds metadata only.
- [Dependency Strategy](dependency-strategy.md) (Draft): the std-only kernel
  rule stands; a TOML parser stays refused unless OD-A3 accepts a reviewed
  dependency and supply-chain gate.
- [Provider transport adapter contract](transport-adapter-contract.md)
  (Draft): the adapter input envelope and network delegation boundary are
  unchanged; this document adds no endpoint or transport field to the
  catalog.
- [v0.1 Implementation Profile](../product/implementation-profile-v0.1.md)
  (Draft): implementation scope and no-network posture stay as stated.
- [bitty-ai#521](https://github.com/bitty-terminal/bitty-ai/issues/521)
  (AI-0329, strict `ai.provider_list`) and
  [bitty-ai#548](https://github.com/bitty-terminal/bitty-ai/issues/548)
  (packaging): both stay blocked until the open decisions below are answered
  and an accepted contract exists.

## Open points

Each item needs an explicit answer before implementation. "Default if
unresolved" is the safe interim, not an acceptance.

| ID    | Owner                      | Decision                                                                                                                                                                                               | Default if unresolved                                         |
| ----- | -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------- |
| OD-C1 | Core                       | Accept fixed mode argv plus an optional absolute config path for the `ai` component spawn?                                                                                                             | No argv change; strict stays off                              |
| OD-C2 | Core                       | Does Core derive the catalog path from its user config root, or does the helper resolve it with an extended env allowlist?                                                                             | Core passes the absolute path (C6)                            |
| OD-C3 | Core                       | Is a descriptor v2 (argv/multi-executable) explicitly out of scope for this contract?                                                                                                                  | Yes, out of scope                                             |
| OD-C4 | Core                       | If argv is rejected, is a second component (`ai-helper`) ever acceptable?                                                                                                                              | No; use M3 instead                                            |
| OD-C5 | Core                       | Confirm Windows ships resident stdio only and strict publication stays unsupported there.                                                                                                              | Yes                                                           |
| OD-A1 | bitty-ai                   | Approve one `bitty-ai` artifact with a required closed `--mode` and removal of `bitty-ai-helper`.                                                                                                      | Keep `bitty-ai-helper`; no integration                        |
| OD-A2 | bitty-ai                   | Approve the metadata-only catalog schema and exclude credentials from v1.                                                                                                                              | No loader                                                     |
| OD-A3 | bitty-ai                   | Choose the loader crate and parser: existing in-repo JSON parser (no new dependency) vs TOML (new dependency plus supply-chain review).                                                                | Existing parser, new small module/crate                       |
| OD-A4 | bitty-ai                   | Confirm load-once/restart-only reload and fail-fast on an invalid catalog (absent = empty).                                                                                                            | Load once; fail fast on invalid                               |
| OD-A5 | bitty-ai                   | May a real zero-provider registry publish an empty projection, or must publication refuse until one provider exists?                                                                                   | Refuse; keep strict off                                       |
| OD-A6 | bitty-ai                   | Is `control-only` compile-time gated (non-default feature) so a release build cannot enter the placeholder mode?                                                                                       | Yes                                                           |
| OD-A7 | bitty-ai                   | Do entries whose adapter or credentials are unresolved appear in `provider_list`? The accepted projection has no availability field.                                                                   | Include configured entries; availability is out of projection |
| OD-J1 | Core + bitty-ai + security | Credential resolution ownership under DIR-030: helper-side file/keyring/command tiers vs Core-brokered handles, given env passthrough is release blocking and OQ-054/OQ-055 are accepted in principle. | No credentials in v1; metadata only                           |
| OD-J2 | Core + bitty-ai            | Confirm the publication control session and `ai.provider_list` live in the same process/registry (M2/M3), or accept a split authority.                                                                 | Same process                                                  |
| OD-J3 | Core + bitty-ai            | Feed revision semantics: view-owned, starts at `1`, non-zero even when empty.                                                                                                                          | Starts at `1`                                                 |
| OD-J4 | Core + bitty-ai            | Which acceptance artifact unblocks AI-0329 and #548 (this document plus owner sign-off, or a canonical docs decision)?                                                                                 | This document plus sign-off                                   |
| OD-J5 | Core + security            | Windows catalog path/ACL validation and publication support matrix.                                                                                                                                    | No publication on Windows                                     |
| OD-J6 | bitty-ai + security        | Refuse or warn on secret-shaped keys in the catalog?                                                                                                                                                   | Refuse                                                        |

## Acceptance criteria

This draft passes document-level review only when all of the following are
true:

- The status block keeps the candidate qualifiers and claims no accepted
  contract, design approval, or implementation evidence.
- C6 and M2 appear only as the recommended proposal; no sentence states that
  an owner accepted either.
- Strict `ai.provider_list` and `Admit` remain disabled, and
  [bitty-ai#521](https://github.com/bitty-terminal/bitty-ai/issues/521) and
  [bitty-ai#548](https://github.com/bitty-terminal/bitty-ai/issues/548)
  remain blocked.
- Every open decision keeps its owner and default-if-unresolved column, and no
  default is presented as an acceptance.
- The option matrices, option analysis, failure modes, and security
  constraints retain their stated content; any change to them is a new
  decision input, not an edit under this task.
- Every changed canonical file is self-contained: it contains no research
  reference, record number, or coverage ledger.
- `just check` passes, and independent architecture, security, and
  docs-curator review records no blocking finding.

## P0 Review Sign-off

No P0 sign-off is claimed by this document. Before any reliance, the Core owner
must answer OD-C1 through OD-C5, the bitty-ai owner must answer OD-A1 through
OD-A7, and the joint owners must answer OD-J1 through OD-J6. The security
reviewer must verify the absence of a credential surface, bounded parsing,
fail-closed behavior, and diagnostics that never carry catalog values, paths,
or file contents. The architecture owner must verify the R1 registry authority
split and DIR-030 compatibility. The docs curator must verify
self-containment, metadata, taxonomy, and links. Repository gate success and
this draft do not constitute those sign-offs.

## References

- [Execution ownership R1](../architecture/execution-ownership-r1.md) (Draft):
  the R1 provider/routing disposition and the accepted provider-routing
  snapshot binding contract.
- [Authenticated provider publication](../architecture/authenticated-provider-publication.md)
  (Draft): supervisor delegation control protocol, metadata feed, and
  revocable helper proof lifetime.
- [Provider plugin boundary](provider-plugin-boundary.md) (Draft): Core versus
  provider-plugin boundary, secret invariant, and frozen v0.1 provider surface.
- [Dependency Strategy](dependency-strategy.md) (Draft): std-only runtime
  kernel and adapter dependency boundaries.
- [Provider transport adapter contract](transport-adapter-contract.md) (Draft):
  adapter input envelope, guarantees, and network delegation boundary.
- [v0.1 Implementation Profile](../product/implementation-profile-v0.1.md)
  (Draft): minimal experimental scope for the first runtime increment.
- [AI Unresolved Questions](../product/ai-unresolved-questions.md) (Draft):
  local AI question register.
- [DIR-030 Native Component Boundary](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/development/native-component-boundary.md)
  (Normative): component, executable, environment, and spawn rules.
- [Authenticated provider publication ingress](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/authenticated-provider-publication-ingress.md)
  (Draft): D1 stdio data RPC plus control stream, CB-2 restart revocation, and
  CB-9 single active control session.
- [Open questions](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/open-questions.md):
  OQ-054, OQ-055, OQ-080, and OQ-081.
- `bitty-ai` `crates/bitty-ai-control/src/bin/bitty-ai-helper.rs`,
  `crates/bitty-ai-control/src/control_registry.rs`,
  `crates/bitty-ai-runtime/src/selection.rs`,
  `crates/bitty-ai-runtime/src/secret.rs`.
- `bitty` `crates/bitty-runtime/src/ai_helper.rs`,
  `crates/bitty-runtime/src/ai_publication_launch.rs`,
  `crates/bitty-runtime/src/component/descriptor.rs`,
  `recording/ai-helper-wiring-options.md` (original A/B/C framing of the
  binary gap; this document refines A into M2 and adds the proof/registry
  consistency argument), and `specifications/ai-helper-resident-singleton.md`.
- [bitty-ai#521](https://github.com/bitty-terminal/bitty-ai/issues/521)
  (AI-0329), [bitty-ai#548](https://github.com/bitty-terminal/bitty-ai/issues/548)
  (packaging), [bitty-ai#519](https://github.com/bitty-terminal/bitty-ai/issues/519)
  (AI-0327), [bitty-ai#520](https://github.com/bitty-terminal/bitty-ai/issues/520)
  (AI-0328), [bitty-ai#522](https://github.com/bitty-terminal/bitty-ai/issues/522)
  (AI-0330).
