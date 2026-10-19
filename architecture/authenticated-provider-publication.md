---
title: Authenticated provider publication
description: Proposed supervisor delegation control protocol metadata feed and revocable helper proof lifetime
category: architecture
audience: security-reviewer
document_type: specification
status: draft
website_publish: false
sidebar_order: 21
---

# Authenticated provider publication

## Document status

**DRAFT — independently reviewed conditional proposal, not an accepted mechanism.**
Architecture, security, and documentation reviewers approved delivery of this
bounded conditional draft on 2026-10-17; the remaining acceptance gates below
are not discharged by those reviews. This specification proposes the residual mechanism for
the accepted [R1 provider/routing contract](execution-ownership-r1.md#providerrouting-snapshot-binding-contract-accepted-2026-10-17).
It does not change that contract's acceptance, authorize implementation, enable
strict dispatch, or close an issue, question, risk, or review gate.

The proposal selects a launch-bound dedicated control stream, explicit
supervisor delegation bootstrapped through a restricted inherited channel, a
launch-scoped mutual-certificate TLS 1.3 control profile, a helper-to-Core
metadata feed, and helper-owned revocable state lifetimes. Per the owner
direction recorded with CTX-0164
([AI-docs #290](https://github.com/bitty-terminal/bitty-ai-docs/issues/290)),
the [Mechanism ruling (2026-10-19)](#mechanism-ruling-2026-10-19) below replaces
the conditional external-PSK profile with the launch-scoped mutual-certificate
profile, so that profile's unresolved released-dependency blocker is resolved by
replacement, not by a PSK dependency. Platform adapters and cryptographic
dependencies still require separately scoped approval and evidence; the selected
profile is not an approved runnable mechanism. The launch-material ownership and
bootstrap-direction gate is completed in draft below; its remaining carried
gates keep authenticated publication unavailable and strict dispatch
`Unbound` until discharged.

### Pinned starting point

These are pinned source/review facts, including the AI-0327 merge confirmed
during review. The older dispatch snapshot is identified separately; none is
live authenticated integration evidence:

| Component             | Pinned evidence                                                                                 | Limit of claim                                                                                                                                                                                                                                                                                                  |
| --------------------- | ----------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Core                  | `aceb327e16ff0f2fafe9c3915c9b26a313355514`, including #2127 / `27ab3f7b` and #2184 / `08aa903e` | Digest-verified resident helper supervision, bounded stdio, restart fences, generic snapshot versions, and local metadata acknowledgement; no authenticated provider publication or remote acknowledgement.                                                                                                     |
| Core-consumed IPC     | `aa7259d78e50d46e5362c341f2dfdd98b4db4ed6`                                                      | 38 generic methods, no `ai.*`; UID verification attests the user, not the Core component. Windows SID comparison is test-only evidence.                                                                                                                                                                         |
| Endpoint-consumed IPC | `f20191183f6ae9464889c5d0a6cc8606670314ee`                                                      | 13 `ai.*` registrations from CTX-0006 / `bd13d9c` / IPC #24. Registration does not authenticate publication.                                                                                                                                                                                                    |
| AI endpoint           | `6757b1e21ed27941b910ce0452834806fafc9c4b`                                                      | Canonical parsing/projection and digest-based binding; parsed publication bytes are discarded. All strict dispatch remains `Unbound`.                                                                                                                                                                           |
| AI-0327               | Merged PR #523, `a2bb62a50582b060e3c1e1173e4af2ab0bf35813`                                      | Retained exact publication bytes and offline exact-byte binding, with 132 endpoint tests. Replaces the pending `7f9f589` dispatch snapshot; does not prove authenticated origin, cross-reset lifetime isolation, or live Core integration. Strict wiring remains blocked by AI-0330 and this mechanism's gates. |
| AI-0330               | Issue #522                                                                                      | Existing offline reconstruction can make an old proof compare current after identical publication/acknowledgement/bind inputs. Equality of public tuples does not prove same-state provenance.                                                                                                                  |

## Purpose and scope

Resolve the authority, transport, source mapping, and lifetime choices requested
by Core #2199 and AI-docs #287 / CTX-0162 for later review and implementation.
The only later strict target is `ai.provider_list` in `provider-routing`.
The helper remains registry, selection, provider I/O, and credential authority.
Core owns its accepted read-only publication view and publication lifecycle.

This design introduces no durable usage/session state, daemon, TCP listener,
provider credential API, new `ai.*` method, or provider behavior in Core.
Default standalone/tolerant behavior and every other domain retain their
existing contracts. MCP `list_version` and generic AI `snapshot_version` do not
become publication authority.

## Normative sources this specification must not weaken

- [R1 provider/routing contract](execution-ownership-r1.md#providerrouting-snapshot-binding-contract-accepted-2026-10-17): projection, canonical bytes, ownership, exact equality, and fail-closed strict admission.
- [IPC and Agent RFC](../specifications/ipc-agent-rfc.md): bounded framing,
  per-action scopes, current-user-local external IPC, read-only defaults, and
  child-scope/secret minimization. This internal control protocol is a separate
  proposed boundary; it does not replace external IPC authentication.
- Canonical [security overview](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/overview.md),
  [threat model](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/threat-model.md),
  [P0 acceptance criteria](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/p0-acceptance-criteria.md),
  [risk register](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/risk-register.md),
  and [evidence matrix](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/evidence-matrix.md).
- [Native component boundary / DIR-030](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/development/native-component-boundary.md):
  verified component resolution, explicit Core grants, cleared allowlisted
  environment, process ownership, and no ambient network/AI capability in Core.

These sources prevail on conflict. Accepting this proposal would still require
the owning security corpus's change/review process and separate implementation
tasks; this draft does not alter foreign repositories or risk states.

## Terminology

| Term                | Meaning                                                                                                                                                    |
| ------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Launch authority    | One random secret delegated by the trusted Core supervisor to exactly one verified helper launch for provider-publication control only.                    |
| Launch manifest     | Bounded, non-secret binding of Core instance, component descriptor, protocol, channel, and roles; delivered alongside the secret through bootstrap.        |
| Control session     | One mutually authenticated TLS connection plus the matching application establishment exchange.                                                            |
| Lifetime            | One helper-created, non-reconstructible internal capability and its live revocation cell; distinct from public publication tuples and Core authentication. |
| Receipt             | Random opaque lookup handle for a helper-resident proof. It is not a serialized proof or general credential.                                               |
| Feed revision       | Helper-owned registry mutation revision, covering aliases as well as projected model fields.                                                               |
| Publication version | Core-owned monotonic publication revision; never inferred from a feed revision, MCP list version, or generic snapshot version.                             |

## Proposed authority decision

### Select A with explicit bootstrap and standard authenticated transport

Select candidate A: keep existing stdio for bounded data RPC and add a dedicated
bidirectional inherited control stream plus a one-shot bootstrap channel.
The trusted Core supervisor, after policy grants this exact component launch,
generates fresh launch key material and the expected launch manifest (see
[Launch material and bootstrap direction (2026-10-19)](#launch-material-and-bootstrap-direction-2026-10-19)).
It installs them only in its private launch object and the selected helper
through the bootstrap channel. No plugin, project file,
IPC caller, shell, or Lua request may choose, replace, export, or obtain it.

The manifest binds, in binary field order:

1. Bootstrap format version `u16` (`1`); control protocol version `u16` (`1`);
   fixed role bytes `1`, then `2`, representing
   `core-supervisor` (TLS client) and `ai-helper` (TLS server).
2. Core instance nonce and fresh launch nonce, each `RANDOM_ID_BYTES`.
3. Component name, exact descriptor version, executable name (each a bounded
   `u32`-length-prefixed UTF-8 string), and descriptor's
   verified executable SHA-256; variable strings use the bounds below.
4. Selected component protocol version `u16`, fresh control-channel nonce, and fresh
   data-channel nonce, each channel nonce `RANDOM_ID_BYTES`.
5. Sole delegation purpose `provider-routing-publication`, and control profile
   `provider-publication-control/1`, each encoded as a bounded string as above.

Bootstrap appends exactly the launch key-material block after that manifest;
`Establish` and `Established` omit all key material. The executable digest is
raw `bytes32`; no textual digest, filesystem path, username, hostname, or
credential field is present.

These values come from trusted supervisor configuration and verified component
metadata, never caller parameters. Bootstrap is one exact bounded record then
EOF; trailing records, missing bytes, unknown versions, wrong descriptor, or
non-EOF close revoke the launch. The helper consumes it before accepting control
or data admission, closes the bootstrap handle, marks surviving control handles
non-inheritable, and prevents secret-bearing diagnostic formatting. Core closes
its bootstrap endpoint after delivery. No material enters argv, environment,
logs, stderr tails, traces, application-generated crash exports, filesystem, ordinary RPC, or caller
parameters. Non-secret handle locators may use reviewed platform launch
metadata; the handle number itself is not an authentication secret.

Use an approved TLS implementation, not a handwritten handshake/MAC, over the
reliable inherited control byte stream. **Superseded (2026-10-19):** the
originally proposed external-PSK profile below, together with its
dependency-approval paragraph, is retained as historical record and replaced by
the launch-scoped mutual-certificate profile selected in the
[Mechanism ruling (2026-10-19)](#mechanism-ruling-2026-10-19). Proposed profile:

- TLS 1.3 only; external pairwise PSK with SHA-256 and `psk_dhe_ke`, X25519,
  and `TLS_AES_128_GCM_SHA256`. No certificate/TOFU fallback, PSK-only mode,
  resumption, session tickets, 0-RTT, renegotiation, or alternate group/profile.
- PSK is the OS-random launch secret. Its sole opaque public identity is the
  launch nonce; compare bytes exactly. It is shared by exactly one client and
  one server with fixed roles, never reused for another launch or purpose.
- ALPN must select `provider-publication-control/1`. TLS Finished authenticates
  the handshake transcript in PSK mode; application establishment below confirms
  both roles and the exact bootstrap manifest before accepting publications.
  Follow RFC 8446 and RFC 9257 pairwise provisioning guidance. No application
  record is accepted until both Finished checks complete.
- Standard TLS record AEAD, sequence numbers, traffic-key limits, and failure
  handling provide transport integrity/replay protection; application sequence
  and lifecycle checks below add state-order protection. There is no extra
  invented MAC construction. Key updates are subject to the wrapper's explicit
  record/work/update budgets below as well as TLS key-use limits; exhaustion
  revokes without extending application authority.
- Disable key logging, plaintext tracing, and session export. On cooperative
  cleanup, zeroize owned accessible bootstrap/PSK storage once no longer needed
  and owned accessible TLS key storage when destroyed, using reviewed library
  support. This makes no guarantee about inaccessible library/compiler copies,
  registers, stack remnants, kernel buffers, core dumps, or snapshots, or about
  destructor execution after abrupt crash/kill. Authoritative revocation does
  not depend on successful erasure; see lifetime cleanup below.

Crypto and OS RNG dependencies are **not approved by this draft**. A separate
dependency/security task must demonstrate this precise profile, bounded memory,
pipe transport, MSRV/platform support, no TCP listener or network functionality
in Core, dependency audit, and reviewed secret disposal. If no acceptable
implementation meets it, strict publication stays disabled; do not replace it
with custom crypto or weaker authentication. Standard TLS does not automatically
bound the application's I/O wrapper, scheduler, buffering, or CPU work.

### Released-dependency feasibility blocker

**Historical record (superseded 2026-10-19).** This subsection records the
released-dependency survey and the blocker it left open for the external-PSK
profile. Per the [Mechanism ruling (2026-10-19)](#mechanism-ruling-2026-10-19)
below, that profile is no longer the selected profile: the launch-scoped
mutual-certificate profile replaces it, and the blocker is resolved by
replacement rather than by demonstrating external-PSK support. The survey
evidence below remains historical; it approves no dependency and enables no
strict behavior.

Commander-supplied documentation evidence inspects **rustls 0.23.46**:
its public `ClientConfig` and `ServerConfig` do not expose configuration of the
proposed external pairwise PSK. Session resumption configuration is a different
mechanism and does not satisfy this requirement. The Context7 rustls query
returned no external-PSK documentation; that search absence alone is not proof
of API absence. The release API evidence is reinforced by upstream external-PSK
PR #2424 being closed pending future work, and client draft PR #2524 being closed
unmerged on 2025-09-15. This is evidence about that released pin and those PRs,
not a claim about every newer rustls release or every TLS implementation.

The external-PSK profile was therefore **conditional and BLOCKED** as recorded,
not a claim that existing rustls dependencies implement it. The separate
dependency/security feasibility task did not identify and verify a released
implementation supporting **both client and server external PSK**, exact
fixed-role/profile behavior, bounded custom pipe I/O and processing/scheduling
below, the project MSRV, non-network feature selection, all enabled platforms,
and the stated limited cooperative cleanup. No library substitution, pin/fork,
handwritten crypto, or insecure fallback is selected or authorized. A different
concrete standard mechanism required a new reviewed decision; the
[Mechanism ruling (2026-10-19)](#mechanism-ruling-2026-10-19) below is that
decision, and it resolves the blocker by replacement. Strict publication
remains disabled until the selected profile's carried gates are discharged.

No OpenSSL dependency is approved or selected by mentioning external-PSK
feasibility. Any later candidate must preserve the raw length-delimited nonce
identity: an API accepting only a NUL-terminated C string cannot represent all
32-byte random identities. A candidate would need an explicitly reviewed
length-delimited adapter, or a separately reviewed profile/identity encoding
change; do not silently hex-encode, truncate at NUL, or substitute a resumption
callback. Both client and server paths must be demonstrated at the proposed
released pin before unconditional mechanism acceptance.

**Survey outcome (CTX-0163).** The released-dependency survey is recorded
separately in
[Authenticated provider publication dependency feasibility](authenticated-provider-publication-dependency-feasibility.md).
No surveyed candidate was demonstrated to satisfy the entire profile: s2n-tls
implements TLS 1.3 external PSK with a Rust-crate surface, but its MSRV, platform
matrix, zeroization, external-mode reachability from the Rust API, and this
specification's per-step work-accounting requirement remain unestablished, and
every other candidate is unmet or not demonstrated. The external-PSK mechanism
was therefore not demonstrated to be implementable with a released dependency,
and a new reviewed decision was required; the
[Mechanism ruling (2026-10-19)](#mechanism-ruling-2026-10-19) below records that
decision. That survey approves no dependency and enables no strict behavior; it
is retained as historical record.

**Replacement candidate analysis (CTX-0164).** The launch-scoped
mutual-certificate alternative recorded by the dependency survey is analysed
separately in [Authenticated provider publication mutual-certificate feasibility](authenticated-provider-publication-mutual-certificate-feasibility.md).
That record maps the candidate against this specification's requirements as
API-feasibility evidence only: it selects no mechanism, edits no profile or
bound, and enables no strict behavior.

### Mechanism ruling (2026-10-19)

Acting under the owner direction recorded with CTX-0164
([AI-docs #290](https://github.com/bitty-terminal/bitty-ai-docs/issues/290)) and
the API-feasibility evidence in
[Authenticated provider publication mutual-certificate feasibility](authenticated-provider-publication-mutual-certificate-feasibility.md),
this ruling replaces the external-PSK profile selection with the launch-scoped
mutual-certificate profile. It records a selection decision; it grants no
implementation permission, enables no strict behavior, and claims no verified or
implemented behavior. The superseded external-PSK profile text and feasibility
discussion above are retained as historical record.

- **Superseded: TLS 1.3 external-PSK profile.** The conditional external-PSK
  profile and its released-dependency blocker above are no longer the selected
  profile. The blocker is resolved by replacement, not by a PSK dependency; no
  library substitution, pin, fork, or fallback is selected.
- **Selected: launch-scoped mutual-certificate profile.** A per-launch
  self-signed certificate pair generated at launch, one leaf per side; mutual
  authentication through released rustls custom certificate verifiers on both
  sides (`ServerCertVerifier` for the `core-supervisor` TLS client, and
  `ClientCertVerifier` for the `ai-helper` TLS server); each verifier makes only
  the certificate-digest pin decision and delegates signature checks to the
  rustls `verify_tls12_signature` and `verify_tls13_signature` crypto helpers;
  certificate-digest pinning, with the pin shape — SHA-256 over certificate DER
  versus SPKI extraction — explicitly left as an open implementation choice;
  TLS 1.3 only; ALPN `provider-publication-control/1`; no PKI, no external PSK,
  no resumption, session tickets, or 0-RTT; and no listener or network
  functionality in Core.
- **Rationale.** The external-PSK profile had no demonstrated released
  implementation for either role (CTX-0163), while the mutual-certificate
  candidate is API-viable in both halves on released dependencies without
  custom cryptography (CTX-0164).
- **Carried over unchanged.** The launch-bound dedicated control stream, the
  restricted one-shot inherited bootstrap channel, the exact control records
  with their named size/count/deadline bounds, fail-closed behavior before
  acknowledgement, proof, handler, provider, credential, or tool contact, and
  the limited cooperative-zeroization framing (owned accessible storage only, no
  guarantee for inaccessible copies, crash remnants, or destructor execution)
  stay as specified above.
- **Remaining verification gates (carried from the CTX-0164 record).** The
  candidate is API-viable on released dependencies, but these gates remain open
  and each needs separately scoped evidence before the mechanism is runnable:
  the certificate-digest pin shape (SHA-256 certificate DER versus SPKI
  extraction); key-generation cost and resource bounds measured under an audited
  bound; launch key-material zeroization across rustls internals, the crypto
  provider, and rcgen's copy-only coverage; the rcgen 0.13.2
  supply-chain/maturity review (undeclared MSRV, platform matrix, `time` and
  `yasna` chain); the rustls crypto-provider decision (aws-lc-rs versus ring,
  `kx_groups` and post-quantum policy, feature selection); launch material
  ownership and bootstrap direction — how the certificate/key material is
  provisioned and pinned through the trusted one-shot bootstrap, the gate that
  keeps the mutual-authentication claim non-circular; handshake byte/record
  bound re-validation with real certificate sizes; platform evidence for Linux,
  the BSD family, macOS, and Windows; and a per-step work-accounting
  instrumentation plan for rustls internals.
- **Dependency scope.** The profile uses rustls 0.23.46, already resolved in the
  bitty-ai workspace through the bitty-network family, so it adds no new TLS
  dependency family there; rcgen 0.13.2 would be a new dependency. The Core side
  has no rustls today, so its reviewed dependency or transport edge remains a
  separately scoped adoption decision.
- **Scope of this ruling.** Implementation remains separately scoped; the
  helper proof-lifetime binding (AI-0330) and Core delivery remain gates, and
  strict dispatch stays disabled. The profile-dependent mechanics text written
  for the superseded external-PSK profile (for example the raw-PSK bootstrap
  append, PSK identity wording, the PSK-specific `AUTHORITY_SECRET_BYTES` bound,
  and the no-certificate handshake rationale) stays historical draft text;
  replacing it with launch key-material, pin, and certificate-bound handling
  belongs to the separately scoped adoption work. This ruling closes no issue,
  question, or risk, and mints no identifier.

### What the authority actually proves

The root of trust is an uncompromised running Core supervisor, its policy and
component descriptor verification, the kernel's restricted spawn/inheritance
operation, OS RNG, and trusted helper startup code. Under those assumptions,
successful pairwise TLS plus exact manifest confirmation proves possession of
the **specific supervisor delegation** bound to that component and channel.
Private control-pipe possession alone does not provide it: an attacker with
only a duplicated control handle cannot produce TLS Finished or valid records.
UID/SID/ACL equality is defense in depth, never the delegation itself.

This is capability authentication rooted in a trusted launch, not universal
attestation that a process named Core is trustworthy. A helper started by an
arbitrary process with an attacker-chosen bootstrap has no trusted Core
provenance and cannot enter the real supervisor's launch slot or authorize its
requests. The real Core accepts only its own descriptor-verified child/channel
and its private expected manifest, never a discovered replacement endpoint.
There is no implemented product binary-signing root of trust to assert here.

A same-user process with ptrace/debug/process-memory access, ability to duplicate
bootstrap handles, inject code, replace trusted startup code, or compromise the
kernel/root can steal the delegation. The pinned launch certificate authenticates
its bearer and the session nonce binds it to the launch; neither identifies an
unauthorized bearer after key theft. Prevention depends on platform
process-access and handle protections, not an impossible cryptographic promise.
Detected leakage, debugger attachment under deployment policy, grant loss, or
suspected compromise requires launch revocation, child teardown, and fresh
bootstrap; old launch material must never authenticate a replacement session. An
undetected theft within the trusted-memory boundary is a stated residual risk.

### Launch material and bootstrap direction (2026-10-19)

This section completes, in draft, the carried gate "launch material ownership
and bootstrap direction" for the selected mutual-certificate profile: how
certificate and key material is provisioned, owned, delivered, and pinned
through the trusted one-shot bootstrap. It records proposed mechanics only. It
adopts no Core dependency, grants no implementation permission, enables no
strict behavior, and closes no review gate. The dated ruling text above remains
the record of what was open when the mechanism was selected.

**Ownership and generation.** The supervisor owns launch key material. At every
helper launch it generates fresh material for that launch only: one ECDSA P-256
key pair and one self-signed leaf certificate per role — a `core-supervisor`
client leaf and an `ai-helper` server leaf — bound to the manifest's fresh
launch nonce. Both leaves carry a fresh random serial, no subject alternative names,
`basicConstraints` CA false, `keyUsage` digitalSignature, and the
role-appropriate `extendedKeyUsage` (`clientAuth` or `serverAuth`); the validity
window is set with bounded margins for hygiene, not as an authorization input.
No key, certificate, or nonce is reused across launches, helper restarts, or
reconnects; a restart is a fresh launch with fresh material. The helper
generates no key material and contacts no CA, PKI, OCSP, or directory service.

**Delivery.** Immediately after spawn the supervisor writes exactly one launch
manifest to the dedicated one-shot bootstrap handle and closes its write end.
The manifest is the already specified bounded record — bootstrap and protocol
versions, roles, Core and launch nonces, component identity and executable
digest, channel nonces, delegation purpose, and control profile — followed by
the launch key-material block below in place of the superseded raw-PSK append.
The block carries the helper leaf certificate and its PKCS#8 private key, the
Core leaf certificate, and the profile and key-type tags. The Core leaf is
delivered so the helper can pin it; the helper leaf key is delivered so the
helper can serve TLS. The supervisor retains the helper leaf digest as its pin
and keeps its own private key in process memory. No key material, private key,
or manifest payload enters argv, the environment, the filesystem, logs, the
control stream, or any control record.

The key-material block uses the same integer and variable-string conventions as
the manifest (unsigned big-endian, `u32` byte lengths) and ends at the manifest
EOF:

| Field             | Type  | Bound                   | Content                               |
| ----------------- | ----- | ----------------------- | ------------------------------------- |
| `profile`         | u8    | `1`                     | Launch-scoped mutual certificates     |
| `key_type`        | u8    | `1`                     | ECDSA P-256                           |
| `core_cert_len`   | u32   | `LAUNCH_CERT_MAX_BYTES` | Length of the Core leaf certificate   |
| `core_cert`       | bytes | DER                     | Self-signed `core-supervisor` leaf    |
| `helper_cert_len` | u32   | `LAUNCH_CERT_MAX_BYTES` | Length of the helper leaf certificate |
| `helper_cert`     | bytes | DER                     | Self-signed `ai-helper` leaf          |
| `helper_key_len`  | u32   | `LAUNCH_KEY_MAX_BYTES`  | Length of the helper private key      |
| `helper_key`      | bytes | PKCS#8 DER              | Helper ECDSA P-256 private key        |

The helper checks the total against `CONTROL_BOOTSTRAP_MAX_BYTES` before
allocation, every length with checked arithmetic, the exact field order and
EOF, and rejects unknown profile or key type. It derives the peer pin by hashing
the received Core leaf; digests are not transmitted in the manifest. A
malformed, truncated, oversize, or trailing-byte manifest fails the launch.

**Pin shape.** The pin is SHA-256 over the leaf certificate DER, not over the
SPKI. The verifier callback receives the end-entity certificate DER directly, so
the pin decision needs no X.509 parsing, SPKI extraction, or re-encoding; the
leaf is minted per launch and never reused, so pinning the complete certificate
is at least as strong as pinning its public key; and exactly one self-signed
leaf is accepted, with no intermediates, chains, or alternate encodings.
Rejected alternatives: SPKI pinning (adds a parser and re-encoding ambiguity for
no gain on a single-use leaf), exact-DER retention (equivalent security but
larger retained state than a 32-byte digest), and public-key-value pinning
without launch material (replayable across launches).

**One-shot bootstrap semantics.** The bootstrap handle is a dedicated inherited
channel: never standard input or output, never a named path or socket path,
never the control stream, and never multiplexed with data framing. It is
unidirectional supervisor-to-helper, written once, read once to the exact
manifest end, and closed by both sides; handle placement follows the launch
convention and carries no secret. The helper starts TLS, adopts a projection, or
accepts any record only after it has consumed the manifest and built its TLS
server configuration. Any read error, establishment-deadline timeout (the named
five-second bound), duplicate delivery, or extra byte fails closed; the
supervisor revokes that launch. The control stream is a separate inherited
duplex channel whose first bytes are TLS, never manifest bytes.

Unix uses an anonymous pipe with pre-exec descriptor placement; Windows uses an
anonymous pipe with an inherited handle. Per-platform handle inheritance,
close-on-exec behavior, and absence of accidental grandchildren inheritance
remain platform evidence gates.

**Zeroization.** Both sides zeroize their serialized manifest buffer and any raw
private-key copy once the TLS configuration has consumed it. The limits already
recorded apply: only owned accessible storage is covered; copies inside rustls
internals, the crypto provider, rcgen, or inaccessible allocator remnants may
persist; crash remnants and destructor non-execution are not covered. A process
that fails after reading the manifest discards the launch rather than continuing
with partial material.

**Failure and revocation.** A missing handle, manifest timeout, malformed or
oversize manifest, pin mismatch in either role, certificate or signature failure,
or TLS establishment failure terminates the launch: the supervisor closes the
control stream, tears down the recorded child, discards that launch's material,
and retries only with fresh material and a fresh nonce. Superseded or old launch
material never authenticates a replacement session.

**Non-circularity.** The pins are provisioned out of band from the channel they
protect: the supervisor's trusted startup code generates both leaves and hands
the helper its material through the kernel-mediated inherited bootstrap. The
helper's pin of the Core leaf is therefore rooted in supervisor generation, not
in anything observed on the TLS channel; the handshake then proves possession of
the pinned keys and binds the session to the launch nonce and descriptor digest.
A same-user attacker able to read the bootstrap handle or process memory has
already crossed the stated trust boundary; this design does not claim to repair
a compromised bootstrap.

**Bound revalidation.** With ECDSA P-256 self-signed leaves (certificate DER
roughly 400–500 bytes, PKCS#8 key roughly 140 bytes), the proposed manifest stays
well under `CONTROL_BOOTSTRAP_MAX_BYTES` (4 KiB), and the handshake stays well
under `CONTROL_HANDSHAKE_MAX_BYTES` (64 KiB) and
`TLS_PREAUTH_WIRE_MAX_BYTES` (128 KiB) per direction. These are design-time
estimates; measurement with the selected provider and real certificate sizes is
a verification gate, not a current claim.

**Remaining gates after this section.** The rcgen 0.13.2 supply-chain and
maturity review; the Core rustls crypto-provider choice (aws-lc-rs versus ring,
`kx_groups` and post-quantum policy, feature selection) and the reviewed Core
dependency-adoption decision; per-platform bootstrap-handle evidence; the
key-generation cost and resource bound measured under an audited bound; the
handshake byte/record bound revalidation above; the launch key-material
zeroization evidence across rustls internals, the crypto provider, and rcgen's
copy-only coverage; and the per-step work-accounting instrumentation plan for
rustls internals. Each remains separately scoped; none is discharged here.

## Control protocol and resource policy

### Transport discrimination and exact encoding

Control is on a distinct inherited descriptor/handle, never multiplexed into
legacy stdio. It begins with TLS, not a bitty IPC v1 frame. ALPN selects the
control protocol only on that stream. An upgraded component descriptor must
explicitly advertise this separate control profile; absent mutual support,
strict provider publication is unavailable. Never sniff an RPC payload to
auto-enable control or `ai.*`; bytes arriving on the wrong stream revoke the
affected launch before acknowledgement/proof/handler contact.

Inside authenticated TLS application data use `u32` big-endian length followed
by exactly one binary control record. Integers are unsigned big-endian; fixed
IDs/digests are raw bytes. A variable byte string has a `u32` byte length followed
by that many bytes. No padding, extensions, duplicate fields, compression,
unknown tags, or trailing bytes are accepted. UTF-8 appears only in bounded
manifest strings and canonical payload bytes. Domain enum `1` means exactly
`provider-routing`; no other domain value is valid in this version.

Every record has this fixed header, in order: magic ASCII `BPPC`, `version:u16`
(`1`), `kind:u16`, `sender:u8` (`1` Core, `2` helper), `sequence:u64`, and
`session_id:bytes32` (the launch nonce). The receiver checks stream identity,
established TLS role, session, version, legal kind/direction, exact expected
per-direction sequence (starting at `1`), and state-machine order before body
acceptance. No caller field can override these checks. Application sequence
overflow, replay, duplicate, skipped, or reordered records revoke the session.
TLS already rejects modified/replayed ciphertext; sequence fields do not replace
that authentication.

### Named bounds

All values below are proposed policy constants defined once by the eventual
wire contract, with shared tests; they are not current implementation claims.
Checked arithmetic is mandatory before allocation, conversion, or increment.

| Constant                          | Proposed value                                | Rationale                                                                                                                                                                                                                                    |
| --------------------------------- | --------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `CONTROL_FRAME_MAX_BYTES`         | 256 KiB, excluding the outer four-byte length | Reuse accepted IPC frame ceiling for each complete decrypted record.                                                                                                                                                                         |
| `CONTROL_INFLIGHT_MAX_BYTES`      | 512 KiB per direction                         | Accepted two-frame buffering ceiling; account for decoder, queued records, and decrypted partial records together.                                                                                                                           |
| `CONTROL_TLS_BUFFER_MAX_BYTES`    | 64 KiB per direction                          | Bound ciphertext/handshake buffering separately; TLS fragments application frames rather than assuming one frame fits one TLS record.                                                                                                        |
| `CONTROL_HANDSHAKE_MAX_BYTES`     | 64 KiB total per peer                         | No certificates or unbounded certificate chain; refuse excess before allocation.                                                                                                                                                             |
| `CONTROL_SMALL_RECORD_MAX_BYTES`  | 4 KiB                                         | Establishment, invalidation, bind, permits, and errors contain only fixed/bounded fields.                                                                                                                                                    |
| `CONTROL_BOOTSTRAP_MAX_BYTES`     | 4 KiB                                         | Exactly one launch manifest carrying launch certificates and the helper key; no provider metadata.                                                                                                                                           |
| `CONTROL_PAYLOAD_MAX_BYTES`       | `CONTROL_FRAME_MAX_BYTES - 512`               | Reserve a checked fixed budget for the header and largest publication/feed envelope; stricter than R1's payload ceiling.                                                                                                                     |
| `CONTROL_QUEUE_MAX_RECORDS`       | 2 per direction                               | Two-frame byte budget also applies; no drop-oldest acknowledgement or invalidation.                                                                                                                                                          |
| `CONTROL_PENDING_PUBLICATIONS`    | 1                                             | One current tuple; replacements revoke before replacement acceptance.                                                                                                                                                                        |
| `CONTROL_PENDING_STRICT_REQUESTS` | 1                                             | Initial provider-list gate needs one exact data-frame permit, no unbounded admission table.                                                                                                                                                  |
| `CONTROL_ACTIVE_RECEIPTS`         | 1                                             | A newer proof revokes the prior receipt.                                                                                                                                                                                                     |
| `CONTROL_ENDPOINT_HANDLES_MAX`    | 64 per supervised helper                      | Bound retained endpoint references and revoked cells; align with the accepted IPC pending-request ceiling.                                                                                                                                   |
| `CONTROL_MODELS_MAX`              | 64 total model entries across all providers   | Reuse the accepted registry/projection model-entry bound. Each group must contain at least one model, so the group count cannot exceed this same total.                                                                                      |
| `CONTROL_CAPABILITIES_MAX`        | 8 per model                                   | Reuse accepted closed label set.                                                                                                                                                                                                             |
| `CONTROL_STRING_MAX_BYTES`        | 128                                           | Reuse R1 identity/model/label bound; also cap each manifest string.                                                                                                                                                                          |
| `CONTROL_JSON_DEPTH_MAX`          | 8                                             | Sufficient for the fixed provider/model/capability schema, stricter than IPC depth 32.                                                                                                                                                       |
| `LAUNCH_CERT_MAX_BYTES`           | 1024                                          | One DER-encoded self-signed ECDSA P-256 leaf certificate per role.                                                                                                                                                                           |
| `LAUNCH_KEY_MAX_BYTES`            | 256                                           | One PKCS#8 DER ECDSA P-256 private key delivered to the helper.                                                                                                                                                                              |
| `RANDOM_ID_BYTES`                 | 32                                            | Fresh launch/channel/lifetime/receipt identifiers with cryptographic collision margin.                                                                                                                                                       |
| `LIFETIME_SECRET_BYTES`           | 32                                            | Helper-private lifetime capability value; never serialized or reconstructed from public data.                                                                                                                                                |
| `CONTROL_ESTABLISH_TIMEOUT`       | 5 seconds from spawn                          | Proposed bounded local bootstrap/TLS/establishment policy; matches the numerical candidate default in the IPC RFC, not an accepted default deadline.                                                                                         |
| `CONTROL_TRANSACTION_TIMEOUT`     | 5 seconds                                     | Whole feed/publication/ack/bind/permit transaction, not renewed per partial byte.                                                                                                                                                            |
| `CONTROL_SESSION_MAX_AGE`         | 60 seconds                                    | Expire provider-publication authority and close its dedicated control stream only. Strict stays disabled until an independently safe fresh helper launch; routine expiry never kills tolerant data RPC or automatically respawns the helper. |
| `CONTROL_RECORD_RATE`             | 100 records/second, burst 200 in one second   | Reuse accepted IPC sustained/burst policy; excess revokes rather than retaining an unlimited backlog.                                                                                                                                        |

TLS budgets below are additional **wrapper obligations**, not inferred library
properties. Limits apply separately to received and sent traffic unless a row
explicitly says aggregate work. A TLS lifetime here is the entire launch/control
session, not the replaceable publication lifetime. Mutation/reset/key update
never replenishes launch counters. Rates are token buckets, initially at the
specified burst capacity, replenished only by monotonic elapsed time; no
activity extends any deadline.

| Constant                      | Proposed value                                      | Accounting and rationale                                                                                                                                                                                                        |
| ----------------------------- | --------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `TLS_PREAUTH_WIRE_MAX_BYTES`  | 128 KiB per handshake per direction                 | Count all raw bytes, including TLS headers, ciphertext, padding, ignored/rejected/partial records and compatibility CCS, before handing them to TLS. Allows the 64 KiB logical handshake ceiling plus bounded framing overhead. |
| `TLS_PREAUTH_RECORDS_MAX`     | 64 per handshake per direction                      | Charge each five-byte TLS header before processing its body; no minimum record size exempts empty/junk/control records.                                                                                                         |
| `TLS_PREAUTH_STEPS_MAX`       | 256 per handshake, aggregate local work             | Charge every bounded TLS processing invocation, including no-output/no-progress calls; fragmented input cannot trigger unlimited retries.                                                                                       |
| `TLS_PREAUTH_WORK_MAX`        | 4096 work units per handshake, aggregate local work | Bound expensive operations separately from bytes and reusable buffers.                                                                                                                                                          |
| `TLS_LAUNCH_WIRE_MAX_BYTES`   | 64 MiB per direction including pre-auth             | Absolute launch budget for all wire traffic, independent of application records and the 60-second age limit.                                                                                                                    |
| `TLS_LAUNCH_RECORDS_MAX`      | 32768 per direction including pre-auth              | Bound lifetime record dispatch, including ignored/rejected/non-application records.                                                                                                                                             |
| `TLS_LAUNCH_STEPS_MAX`        | 65536 aggregate local invocations                   | Bound repeated processing, polling, fragments and no-progress work over the whole launch.                                                                                                                                       |
| `TLS_LAUNCH_WORK_MAX`         | 131072 aggregate local work units                   | Absolute computation budget, never reset by traffic or publication replacement.                                                                                                                                                 |
| `TLS_AUTH_WIRE_RATE`          | 4 MiB/second, burst 8 MiB, per direction            | Post-auth all-wire budget; allow bounded full-frame transactions without permitting unlimited empty/control traffic.                                                                                                            |
| `TLS_AUTH_RECORD_RATE`        | 1024/second, burst 2048, per direction              | Applies to every TLS record, not only decrypted application records.                                                                                                                                                            |
| `TLS_AUTH_WORK_RATE`          | 4096 units/second, burst 8192, aggregate local work | CPU-work admission budget also covers control messages with no application output.                                                                                                                                              |
| `TLS_KEY_UPDATE_RATE`         | 1/second, burst 2, per direction                    | Charge requested and response KeyUpdate messages alike; no reply amplification outside budgets.                                                                                                                                 |
| `TLS_KEY_UPDATES_MAX`         | 4 per direction over entire launch                  | Additional hard limit; if TLS needs another update, disable publication control rather than exceed the budget or force a shared-helper restart.                                                                                 |
| `TLS_TURN_WIRE_MAX_BYTES`     | 32 KiB combined read/write bytes                    | One scheduler turn cannot drain a continuously readable stream.                                                                                                                                                                 |
| `TLS_TURN_RECORDS_MAX`        | 4 combined received/sent records                    | Bound record dispatch before yielding to write/timer/data work.                                                                                                                                                                 |
| `TLS_TURN_IO_CALLS_MAX`       | 8 nonblocking read/write calls                      | Charge success, partial, interrupted, and would-block calls; no busy retry loop.                                                                                                                                                |
| `TLS_TURN_STEPS_MAX`          | 4 bounded TLS invocations                           | Do not let a library drain callback process unlimited records.                                                                                                                                                                  |
| `TLS_TURN_WORK_MAX`           | 256 aggregate units                                 | Yield when the next operation would exceed the remaining turn budget.                                                                                                                                                           |
| `TLS_TURN_TIME_MAX`           | 1 millisecond of monotonic elapsed time             | Check before/after each I/O/library operation; yield on expiry.                                                                                                                                                                 |
| `TLS_SYNC_STEP_TIME_MAX`      | 5 milliseconds                                      | Admission/measurement bound for one synchronous library step under supported deployment load; a non-preemptible call may cross the turn slice but must not cross this bound.                                                    |
| `TLS_TIMER_SERVICE_MAX_DELAY` | 10 milliseconds                                     | Native stress-test bound for checking/revoking expired authority despite continuous readability or TLS-only traffic; requires independently scheduled deadline service.                                                         |

### TLS work accounting and fair deadline service

A work unit charges one header parse, one TLS processing invocation, one
AEAD record operation, or one symmetric/hash/key-schedule operation; a DHE key
agreement/key generation or other public-key operation charges 128 units.
Reserve the maximum work for a bounded step **before** invoking it and reconcile
actual operations afterward, never refunding ignored/malformed/no-output work.
Library-internal work must be covered by an audited per-step upper bound and
instrumented tests, not omitted because it is invisible to the wrapper. Exactly
the configured TLS profile is allowed; unrecognized operations or inability to
bound a library call fail the dependency feasibility gate. These units and byte
caps bound admitted operations; they do not assert equal CPU cost for every
primitive or replace wall-clock checks.

A step whose maximum reserved work cannot fit a fresh `TLS_TURN_WORK_MAX`
budget is unsupported and revokes/fails feasibility, rather than yielding
forever. A rate bucket with insufficient allowance for new inbound work closes
the launch rather than retaining an unlimited retry backlog. Exhausted lifetime
allowance is terminal. A partially used turn may yield to a fresh turn only
when the next bounded step fits all remaining launch/rate limits. Once TLS
Finished completes, post-auth budgets apply even before `Established`; all
launch counters include both phases and are monotonic.

Read the fixed TLS header incrementally under wire/I/O budgets; count a record
as soon as its header completes, and validate its claimed length against TLS's
legal record maximum and the local buffer allowance before allocating or
processing a body. Fragment bytes, incomplete headers, zero-length records,
compatibility CCS, alerts, padding, malformed records, and ignored/rejected
records all consume wire/work budgets even if no application record emerges.
After authentication also charge every KeyUpdate before processing it, including
automatic responses before emission. Alerts/error writes use remaining send
budgets only; when exhausted, close without a diagnostic write. No extra record
or expensive operation is admitted after a launch/rate/work limit is reached.

The adapter must expose bounded custom I/O: offer at most the remaining
turn/buffer allowance and process at most one complete TLS record per bounded
step, with auditable limits on handshake messages and cryptographic work inside
that record. Do not call a convenience method that drains the transport or
unbounded internal messages. A fragmented input must not be repeatedly re-fed
or reprocessed as new work for free. If the released library cannot provide
these properties, it is not feasible for this profile.

Each scheduler turn checks establishment/transaction/absolute launch deadlines
and revocation first, services bounded pending writes as well as reads, then
yields at the first turn limit. When both directions are ready, alternate which
direction is serviced first and reserve an I/O opportunity for the other;
continuous reads may not starve writes. A pending write does not block deadline
service. Recheck deadlines after every bounded operation and immediately before
Ack, proof, reservation, and handler admission. No loop runs until would-block
without also obeying every turn limit.

A separate scheduled deadline service uses the monotonic expiry/revocation gate
without waiting on the TLS worker or its I/O lock. TLS processing cannot hold the
authority gate across a library call. The deployment must demonstrate the
`TLS_SYNC_STEP_TIME_MAX` and `TLS_TIMER_SERVICE_MAX_DELAY` bounds under supported
load; if it cannot, strict stays disabled. This is not a hard real-time promise
under arbitrary OS suspension: even after suspension, every admission checks
absolute expiry before effects. Exhausted launch/pre-auth/rate/work budgets,
deadline expiry, or a synchronous-step overrun revoke provider-publication
authority and close its dedicated control channel; the separate security-fault
policy may also require compromised-launch teardown. Cooperative cleanup is
separate. No Ack, proof, or
handler admission may be issued by processing a flood after that condition.

The sum of a record's checked header/body lengths must fit the frame ceiling,
even when its nested payload fits `CONTROL_PAYLOAD_MAX_BYTES`. Publication and
feed bodies carry **raw canonical UTF-8 bytes**, not a JSON-escaped nested
string or base64. SHA-256 still covers only the accepted canonical JSON payload.
A 256 KiB canonical payload cannot fit this envelope and is refused; it is not
silently chunked or truncated. A future JSON envelope would require an exact
post-escaping full-frame check and a separate version decision. Provider strings
may expand during canonical JSON escaping, so bound the resulting bytes too.

### Records and direction

The common header is implicit below. Fields are mandatory, in listed order.
`tuple` means `domain:u8`, `publication_id:bytes40`, `snapshot_version:u64`,
`digest:bytes32`. `publication_id` is launch nonce followed by publication
version; it is an opaque 80-character lowercase hexadecimal identity when
mapped to the existing bounded string vocabulary. This identity never repeats
within a Core run, and across fresh launches relies on OS-random nonce
uniqueness. Version starts at `1`, strictly increases on every republish, and
never wraps. The control session owns its version high-water mark outside the
replaceable publication-state object.

| Kind            | Direction        | Body and acceptance effect                                                                                                                                                                                                                                                                         |
| --------------- | ---------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `1 Establish`   | Core → helper    | Exact launch manifest echo (profile, key type, session nonce, descriptor digest, Core and helper certificate digests), never key material. Helper compares every field against private bootstrap expectations; no publication acceptance yet.                                                      |
| `2 Established` | Helper → Core    | Same manifest, fresh public `lifetime_id:bytes32`. Core confirms roles/channel/descriptor; establishes current lifetime only.                                                                                                                                                                      |
| `3 Projection`  | Helper → Core    | `lifetime_id`, `feed_revision:u64`, `digest:bytes32`, `payload:bytes`. Helper sends a bounded atomic registry projection; Core validates/canonicalizes before adopting its read-only source view.                                                                                                  |
| `4 Publish`     | Core → helper    | `lifetime_id`, `feed_revision`, `tuple`, `payload:bytes`. Validate authenticated origin, current source/lifetime/version, exact schema/canonical bytes and digest before storing one pending publication.                                                                                          |
| `5 Ack`         | Helper → Core    | `lifetime_id`, `feed_revision`, exact `tuple`, fresh `ack_id:bytes32`. Confirms validation/retention of that publication, not yet strict proof.                                                                                                                                                    |
| `6 Bind`        | Core → helper    | `lifetime_id`, `feed_revision`, exact `tuple`, `ack_id`. Consume only current exact pending acknowledgement once; compare retained bytes with current authoritative registry bytes.                                                                                                                |
| `7 Bound`       | Helper → Core    | `lifetime_id`, `feed_revision`, exact `tuple`, `receipt:bytes32`. Receipt maps to current helper proof after equality; no proof internals leave the helper.                                                                                                                                        |
| `8 Invalidate`  | Either direction | Predecessor `lifetime_id` and its pre-rotation `feed_revision`, `reason:u8`. Reasons: source mutation, registry mutation, reset, supersession, grant loss, shutdown. Immediately invalidate locally before enqueueing; helper revokes/rotates or disables under the exact predecessor rules below. |
| `9 Reserved`    | Neither          | Reserved tag; receipt is a protocol error. Refresh is unnecessary: every valid invalidation response announces the new lifetime and projection without another request.                                                                                                                            |
| `10 Lifetime`   | Helper → Core    | Fresh `lifetime_id`, `feed_revision`. Follows local reset/clone/reconstruction or registry invalidation; Core discards all old tuples and waits for `Projection`.                                                                                                                                  |
| `11 Admit`      | Core → helper    | Current `lifetime_id`, `feed_revision`, exact `tuple`, `receipt`, `request_id:u64`, `data_frame_digest:bytes32`. The request ID equals the existing numeric JSON u64 data RPC ID; reserve one exact forthcoming strict provider-list frame.                                                        |
| `12 Admitted`   | Helper → Core    | Exact `request_id`, `data_frame_digest`, `lifetime_id`. Confirms reservation only; handler admission still rechecks live proof on frame consumption.                                                                                                                                               |
| `13 Error`      | Either direction | `code:u8`: invalid control, unavailable, resource limit, or timeout. Fixed local diagnostic text only; no echoed payload, IDs, paths, model strings, secrets, or parser details. Revoke and close.                                                                                                 |

Fields such as `lifetime_id` and `receipt` are fixed byte arrays as defined by
the bound table. Publication tuples, registry revisions, and receipt lookups
have no unauthenticated setter. Static reason/error enums have closed values;
unknown values revoke. On unauthenticated failures, close without application
error; an authenticated error is best effort, never a blocking dependency of
local revocation. Never include launch key material or a private key in a control record.

Reason tags are `1` source mutation, `2` registry mutation, `3` reset, `4`
supersession, `5` grant loss, and `6` shutdown. Error tags are `1` invalid
control, `2` unavailable, `3` resource limit, and `4` timeout. Fixed roles,
purpose, and profile are exact byte matches, not case-folded identifiers.
Feed revision starts at `1`; every mutation increments it with checked
arithmetic. A publication-free initial lifetime still has a feed revision.

### Message order and strict handoff

1. Supervisor verifies descriptor/digest, grants the narrow launch, generates
   fresh launch key material, creates bootstrap/control/data channels, starts
   the helper, delivers the launch manifest over the one-shot bootstrap handle,
   and closes it. TLS and `Establish → Established` finish within the
   establishment deadline.
2. Helper snapshots the actual registry and sends `Projection`. Core adopts a
   validated read-only source view, derives its exact canonical bytes, allocates
   a fresh publication ID/version, and records one private pending tuple.
3. `Publish → Ack → Bind → Bound` completes within the transaction deadline.
   Core accepts only the exact pending ack/receipt on its current launch.
   Helper consumes the exact ack and proves retained-byte equality under the
   registry read/mutation gate. An Ack alone does not authorize a handler.
4. For later strict provider-list use, Core creates its existing scoped data RPC
   frame and authenticates `Admit` containing SHA-256 of those **exact data-frame
   payload bytes**, separately from the canonical publication digest. After
   `Admitted`, it sends exactly that frame over the bound data channel. Helper
   matches frame bytes/digest, method, exact request ID, and current lifetime,
   consumes the reservation once, and rechecks proof while holding the registry
   read gate through the provider-list projection. No cross-process proof
   serialization or new `ai.*` request parameter is necessary.
5. Only `ai.provider_list` on an explicitly configured strict supervisor channel
   uses this gate. An unpermitted frame, another method presented as the strict
   target, changed bytes, reused request ID, revoked receipt, or crossed channel refuses before
   handler/provider/credential/tool contact. A receipt is useless from normal
   IPC, Lua, another launch, or without authenticated `Admit`. Ordinary scopes
   and consent checks remain necessary; control delegation grants none of them.

The sole outstanding strict reservation expires on the transaction deadline,
frame consumption, or any invalidation. Normal request-ID correlation must
still hold; a reused completed ID within a session is refused by retaining only
an increasing supervisor request ordinal encoded in the bounded ID. Its checked
overflow disables publication authority without wrapping. This is correlation,
not Core authentication.

Out-of-order or duplicate Ack/Bind/Bound, Projection outside the initial or
explicit lifetime-rotation sequences below, future/stale versions, and cross-session
records revoke. A locally authorized supersession invalidates Core's current
view and sends `Invalidate`, then consumes `Lifetime → Projection` and creates a
fresh publication without requesting an unknown lifetime. No simultaneous publication pipeline exists. Receiver deadlines
continue while blocked on a partial frame. Queue overflow, byte-budget excess,
write timeout, or inability to deliver an invalidation immediately revokes and
closes publication control; strict use remains disabled. Never drop an
invalidation or ack and continue strict serving; no retry of old records on a
replacement stream. Data RPC follows the separate fault policy below.

A live receipt may authorize successive distinct provider-list requests within
its lifetime; each `Admit` reservation and request ID is single-use. Control
encodes the existing Core `request_id:u64` as eight big-endian bytes; the data
frame contains the same unsigned integer in decimal JSON. Preserve Core's
existing numeric response correlation. Zero is representable; after the first
strict ID, each strict ID must exceed the prior strict ID on that data channel.
The supervisor's existing request allocator must avoid collisions with tolerant
pending IDs too. A string ID or lossy floating-point comparison never matches.

Every `Invalidate` identifies the **predecessor** lifetime and its feed revision
before rotation. Helper is the sole lifetime creator. For a valid Core-origin
invalidation of its current L1/R1, helper revokes L1 and immediately sends
`Lifetime(L2,R1) → Projection(L2,R1)`; no registry mutation or Refresh is needed.
Core-origin grant loss/shutdown instead leaves authority disabled and closes
control. Helper-origin reset/clone/reconstruction/supersession without registry
mutation sends `Invalidate(L1,R1) → Lifetime(L2,R1) → Projection(L2,R1)`.
Helper-origin registry mutation sends `Invalidate(L1,R1)` before changing the
registry, then `Lifetime(L2,R2) → Projection(L2,R2)`. Each Projection is legal
only immediately after its matching Lifetime (or Established initially), once.
All such helper announcements are allowed unsolicited; they do not mint Core
publication authority, and Core must adopt the view and issue a fresh tuple.

Retain one immediately retired predecessor L1/R1 for crossed invalidations.
If Core's `Invalidate(L1,R1)` arrives after helper already rotated L1 to L2,
consume it once as correlation only, without rotating L2 or duplicating its
announcement; Core consumes the in-order announcement already sent. A second
duplicate or an unrelated older predecessor is refused. If multiple helper
rotations outrun that single predecessor window, close publication control and
leave strict disabled; never guess/revive a lifetime or restart unrelated work.
No rotation changes session sequence, TLS budgets, or publication-version
high-water marks.

### Numeric request and control vector

The pinned Core `build_request(request_id: u64, method, params,
snapshot_version)` emits the following exact UTF-8 payload for numeric request
ID `1`, method `ai.provider_list`, empty caller params, and generic snapshot
version `0` (no spaces or newline):

```text
{"v":1,"id":1,"method":"ai.provider_list","params":{"snapshot_version":0}}
```

Its data frame is big-endian prefix `0000004a` followed by those 74 bytes.
SHA-256 of the payload alone is
`b41526c9f648f9a12537daa96f98be50498c0f251d280f4210661beadf443105`.
The existing generic snapshot injection remains; it is **not** the control
publication version, which is independently `1` in this fixture. No provider
tuple, receipt, scope, or authentication field is added to method params.

For a deterministic codec fixture only, let launch/session nonce be 32 bytes of
`11`, lifetime ID 32 bytes of `22`, receipt 32 bytes of `33`, feed revision and
publication version `1`, empty-registry canonical bytes
`{"domain":"provider-routing","providers":[]}`, and Core control sequence `4`
(after Establish, Publish, Bind). The canonical projection digest is
`c2c6d4a1a7b593a4888956ebb74b129d30e4f37fc980cb43279cbb2e26783dbb`.
The complete decrypted `Admit` frame below is hex, whitespace removed before
decoding; each line identifies the next field by the record order above:

```text
000000f2
42505043 0001 000b 01 0000000000000004
1111111111111111111111111111111111111111111111111111111111111111
2222222222222222222222222222222222222222222222222222222222222222
0000000000000001
01
1111111111111111111111111111111111111111111111111111111111111111
0000000000000001
0000000000000001
c2c6d4a1a7b593a4888956ebb74b129d30e4f37fc980cb43279cbb2e26783dbb
3333333333333333333333333333333333333333333333333333333333333333
0000000000000001
b41526c9f648f9a12537daa96f98be50498c0f251d280f4210661beadf443105
```

The prefix is 242 bytes excluding itself; the header is 49 bytes and body 193.
These public fixture values are not runtime nonces or authentication evidence;
TLS ciphertext is deliberately not a fixed vector. `Admitted` echoes numeric
u64 `1`, the exact data digest, and lifetime ID. The later scoped endpoint
adapter must preserve numeric `id:1` in its response envelope and resolve only
the same pending Core data-channel request; empty-registry success result is
`{"models":[]}`. This response-correlation requirement is future integration,
not a claim that current Core source already processes live strict responses.
The corresponding proposed success-envelope fixture is prefix `0000002f`
followed by exactly these 47 bytes (no spaces or newline):

```text
{"v":1,"id":1,"ok":true,"result":{"models":[]}}
```

This fixture scopes numeric response-envelope integration to the existing
Core request path; it does not change provider-list params/result semantics.
It does not import generic IPC's string-ID convention into this Core data path.
Tests exercise the pinned actual request builder and adapter correlation:
string `"1"`, a duplicated numeric ID, changed generic snapshot version, altered
whitespace/payload bytes with an old digest, wrong-channel response, or numeric
response ID `2` cannot consume request `1`'s reservation/pending result. A future
consumer unable to preserve numeric IDs must stay disabled pending an explicit
compatibility decision rather than silently stringify IDs.

## Metadata source and mutation synchronization

Select the helper-to-Core read-only projection feed rather than independent
shared registration metadata. `Projection` is constructed from the same
populated `ProviderRegistry` read view as `ai.provider_list`: group by
`provider_id`, then model `name`, sorted closed capability labels,
`context_window_tokens`, `input_cost_weight`, and `output_cost_weight` under
R1's exact sorting, scalar, integer, schema, and SHA-256 rules. Do not include
provider display names, free-form routing facts, aliases, selection results,
provider clients, request data, credentials, or implementation state.

Core parses the bounded feed into a typed read-only metadata view and derives
the canonical projection itself using the shared pure canonicalizer. Its
policy accepts this view for publication and assigns Core-owned ID/version;
mere reflection by a caller is never this acceptance. Existing generic
`ServiceMetadata` and its local acknowledgement are not this source. Core
neither populates the registry nor calls provider implementations or credential
resolvers to validate the feed.

The helper's subsequent retained-byte equality check proves that the
Core-accepted view still equals the registry used for this operation. Because
the helper supplies the feed, it is **not independent evidence** that a malicious
helper supplied truthful registration data. The trusted helper and its
registration policy remain inside that semantic trust boundary. This mechanism
prevents stale/misbound publication and unauthorized Core-origin injection; it
does not audit a compromised registry by echoing its own feed.

Every provider/model/capability/context/cost mutation and every alias-table
mutation performs the following under one helper mutation gate:

1. Revoke the shared lifetime and all pending ack, proof, receipt, and request
   reservations **before** changing registry state or allowing another strict
   read. Increment the checked feed revision even if canonical projected bytes
   happen to remain equal, as with an alias-only mutation.
2. Apply the atomic mutation, create a fresh lifetime, and emit
   `Invalidate(old lifetime) → Lifetime(new lifetime) → Projection(new revision)`.
   If output cannot be queued within budgets, revoke/close rather than omit it.
3. Core immediately invalidates its accepted view, pending publication, ack,
   and receipt on the invalidation/lifetime change; it adopts only the newest
   authenticated feed and publishes a new ID with a greater version.
4. Helper rejects late `Publish`/`Bind`/`Admit` from older revisions/lifetimes.
   During the asynchronous gap strict requests refuse. A mutator serialized
   after a strict read's admission may run only after that bounded read finishes;
   the response belongs to the pre-mutation revision. A mutator serialized first
   prevents handler entry. No check-then-unlocked-read window is permitted.

Core source-view mutation or policy withdrawal first invalidates its local
state and sends `Invalidate`; helper revokes on receipt. Buffered data is never
allowed to bypass the helper's current gate. Local Core revocation cannot be
instantaneously observed across processes: an already admitted bounded read may
complete before the authenticated invalidation arrives. Control close, grant
checks at admission, bounded transactions, and absolute publication-authority expiry bound that
window; this design promises no retroactive rollback of a completed read.
Delayed invalidation is covered explicitly by race tests.

Alias-dependent strict behavior remains excluded. An alias mutation still
forces a new publication/lifetime, even though it does not add alias data to
the initial provider-list projection.

## Publication-state lifetime and shared revocation

### Helper-owned authority object

Create a helper-private authority owner associated with the established
control session, separate from a replaceable `PublicationState`. It owns a
serialized revocation gate, the current lifetime capability, one current proof
slot, session/version high-water marks, feed revision, and deadlines. The
lifetime capability includes a fresh OS-random `LIFETIME_SECRET_BYTES` value
that is never serialized or reconstructed from a public tuple. A separate
random public `lifetime_id` names it on the authenticated control session.
Proofs reference the shared revocation cell and the private lifetime capability;
they do not merely compare the public ID or publication fields.

The only creator is the helper's authenticated-session authority owner. A
public `PublicationState::new`, deserializer, caller boolean, pointer identity,
or process-global counter cannot mint Core authority. The private lifetime
capability authenticates **local provenance and revocability only**. Authorized
Core origin comes separately from trusted bootstrap/TLS/manifest checks.
Both conditions are required before a lifetime can hold an acknowledged proof.

### Reset, clone, reconstruction, and old endpoint objects

Any reset, semantic clone, or reconstruction is an explicit operation on that
owner: under the shared gate, irreversibly mark the old revocation cell revoked,
clear pending/ack/proof/receipt/reservation slots, and create fresh internal
capability/public lifetime ID. The replacement starts `Unbound` and announces
`Invalidate → Lifetime`; new publication/ack/bind must follow. Merely retaining
an old endpoint/proof object does not retain authority: every admission checks
its original shared cell under the same gate, which now refuses forever.
Revocation makes the shared cell irreversibly unusable before cooperative
cleanup zeroizes its owned accessible private material using reviewed support;
old proofs hold no independent secret copies. Logical invalidation does not
wait for erasure or depend on its success. A local reset,
semantic clone, or reconstruction emits `Invalidate → Lifetime → Projection`
on its existing authenticated session without a Core refresh request.
Revocation must reach **all** old cloned endpoint objects, not only the new
state's current-check method.

Copying secret-bearing state or implementing an ordinary value `Clone` is
forbidden. A requested state clone revokes its source before returning a fresh
unbound state. Cloning a read-only endpoint handle may share the owner/cell, but
cannot copy a live proof into a new independent owner. If ownership cannot be
preserved, the clone/reset operation fails closed. Drop of a live authority owner
revokes its cells and closes the control session; a replacement owner cannot
attach itself to an existing session using copied public IDs.

This requirement holds even within the same process, data channel, control
connection, and session, with identical bytes, publication identity/version,
registry revision, and local epoch. The fresh private capability and permanently
revoked old cell distinguish lifetimes. The AI-0330 regression must reconstruct
an offline state and replay identical publish/ack/bind inputs as the old equality
seam allowed, then test the old proof against **both old and new endpoint
objects**. It must refuse even if a fixture repeats every public tuple. Such an
offline fixture does not establish authenticated Core origin or create a
production replay exception. Production separately rejects the replay at
session sequence/lifetime/version high-water checks before ack.

### Rollback, replay, storage, and expiry

Resetting publication state never rolls back session sequence numbers, Core
publication high-water marks, or the helper mutation revision. Attempted
rollback, checked `u64` overflow, RNG failure, repeated random lifetime/receipt
equal to the current or immediately retired value, or attempted restoration of
secret-bearing state revokes/closes. Do not retry with a timestamp, pointer,
counter, fixed seed, or guessed random value. Fresh nonces are probabilistic
cryptographic uniqueness; this does not claim mathematical uniqueness over
infinite launches. VM/process-memory snapshot rollback and compromised RNG are
outside the trusted-memory assumption and require a new trusted launch.

Live storage is bounded: one owner, one current and one immediately retired
lifetime identifier for collision checks, one publication/ack, one receipt/proof,
one strict reservation, and fixed high-water marks per supervised helper. No
ever-growing replay/receipt history or durable state is needed: session ordering
and publication versions reject older messages, and old revocation cells can
only remain alive through already existing bounded endpoint handles. Endpoint
handle creation must reserve from `CONTROL_ENDPOINT_HANDLES_MAX`; exhaustion
refuses the new handle without copying authority. Accounting follows the shared
owner, including old revoked handles, and a clone must reserve before it can
return. Creating a new unbounded handle table is forbidden. A revoked cell cannot
become live again and exposes no payload, secret, receipt table, or registry
clone through admission. On cooperative cleanup release its retained payloads
and zeroize its owned accessible secret storage; inaccessible remnants are
outside that cleanup guarantee.

Authorization loss, expiry, timeout, framing/authentication fault, control/data
EOF, shutdown, crash, process/endpoint replacement, executable digest/version
change, or generation/channel replacement invalidates the authoritative slot.
Ordinary supersession and registry mutation rotate the lifetime. Absolute
session expiry revokes provider-publication authority and closes only its
dedicated control stream, even if metadata remains unchanged. All clocks are monotonic; wall-clock changes cannot
extend a lease. Nothing persists across restart.

### Routine publication expiry versus shared helper lifecycle

Choose the bounded initial policy of **no live control rebootstrap**. At the
60-second authority deadline or a control transaction/budget expiry, Core and
helper permanently disable provider-publication authority for that resident
launch, revoke all old cells/receipts/pending strict reservations, and close
control. Do not reissue launch key material, reset TLS counters, reuse a token/session/channel
ID, or attach a fresh TLS session to the existing helper. This avoids an
unreviewed secondary bootstrap/rekey protocol. Ordinary reset/mutation before
expiry may rotate publication lifetime using the still-live authority only;
it never extends the absolute deadline.

The data stream and tolerant requests keep their existing scopes, deadlines,
response correlation, and supervisor lifecycle. A tolerant operation submitted
at helper age 59 seconds continues beyond authority age 60 subject to its own
deadline; no routine authority timer kills it, resets helper registry/session
state, or triggers respawn. Strict requests not yet admitted fail closed; an
already admitted bounded provider-list read may finish under its existing
deadline with its pre-expiry linearization. No post-expiry Ack/proof/new handler
entry is allowed. A long-lived helper can therefore remain tolerant-only after
expiry indefinitely under its ordinary lifecycle.

Publication authority returns only on an **independently safe fresh launch**
already permitted by the existing supervisor lifecycle (for example ordinary
idle stop with nothing in flight, followed by demand, or an explicitly authorized
restart). Publication expiry does not request that restart. The fresh launch
repeats verified bootstrap with fresh launch key material, launch/control/data
IDs, TLS session, private lifetime and tuples; all old objects stay revoked.
This design makes no seamless strict-refresh/availability promise after
authority expiry.

Security faults are a separate policy: detected bootstrap leakage, failed
authentication/manifest, malicious framing/injection, or suspected process
compromise may require supervisor-owned teardown of the compromised launch.
The owning security task must specify that fault disposition and notify affected
requests under existing error/deadline rules. It must not classify routine
authority age, ordinary budget exhaustion/timeout, or source refresh as process
compromise just to terminate unrelated tolerant work. No external PID or
unowned process is affected.

Revocation and disposal are separate obligations. A surviving supervisor/helper
invalidates its authoritative slot when it observes peer termination, channel
closure, or expiry, regardless of whether the peer ran cleanup. A process killed
abruptly cannot run destructors or guarantee key erasure; the surviving side
must fence old slots/receipts and require a new trusted launch, not infer that
death erased memory. Cooperative cleanup covers only owned accessible buffers,
revocation-cell storage, and library key storage for which the approved library
provides reviewed zeroization support. It does not guarantee erasure of compiler
copies, registers, stack remnants, kernel/pipe buffers, core dumps, snapshots,
or inaccessible internal allocations. Crash export/key logging remain disabled
and deployment dump/debug restrictions remain necessary. Tests can verify
supported owned-buffer cleanup and irreversible admission refusal; they cannot
prove universal key erasure after a crash.

Projection/canonicalization costs are `O(B + M log M)` time and `O(B + M)`
space for bounded canonical bytes `B` and model count `M`; exact equality is
`O(B)` time. Revocation/receipt admission is `O(1)` time and space excluding the
bounded exact-byte comparison and existing shared-handle references.

## Platform boundary

These are proposed adapter requirements, not claims of tested platform
authentication. Anonymous stdio has no Unix socket peer-credential attestation.
The selected control design relies on restricted spawn plus pairwise bootstrap,
not PID/UID/SID inference. External IPC still obeys its existing peer checks.

| Platform   | Required inherited-channel handling                                                                                                                                                                                                                                                                                    | Evidence/enablement boundary                                                                                                                                                                                                          |
| ---------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Linux      | Create non-inheritable descriptors atomically; explicit per-child spawn actions allow only intended data/control/bootstrap/stderr ends. Close all opposite/unused ends. After bootstrap, control/data are close-on-exec and excluded from every helper descendant. Audit concurrent spawns and descriptor duplication. | Current supervisor/source evidence is Linux-local only; it does not prove the new bootstrap. `/proc`, ptrace, or process-memory access can defeat secrecy within the stated compromise boundary.                                      |
| BSD family | Reviewed platform-specific spawn actions/close-on-exec allowlist, including concurrent descriptor creation and fork behavior; no reliance on a PID supplied as zero by existing peer code.                                                                                                                             | Verify each supported BSD independently; do not infer support from Linux or generic POSIX behavior. Until native leakage/revocation tests pass, strict mode is disabled there.                                                        |
| macOS      | Use reviewed spawn file actions and close-on-exec default facilities where available; explicitly inherit only selected descriptors, then close/restore flags. Treat task/debugger access as compromise of trusted memory.                                                                                              | macOS-specific native tests are required; neither POSIX citations nor Unix UID evidence prove them. Strict stays disabled pending evidence.                                                                                           |
| Windows    | Use reviewed `STARTUPINFOEX` explicit handle list, inheritable child duplicates only for the target spawn, non-inheritable parent ends, and narrow handle access. Immediately close bootstrap/unused handles; clear inheritance on surviving child handles and exclude all of them from descendants.                   | SID/ACL checks alone are insufficient; existing SID unit models are not live evidence. Native tests must cover concurrent `CreateProcess`, handle duplication/process-access rights, and descendant leakage before strict enablement. |

Cleared child environment preserves only the owning component policy's explicit
allowlist (including its accepted Windows exception); it never carries bootstrap
secrets. No helper child may share control/bootstrap authority. Close-on-exec
does not prevent a raw fork from copying memory/descriptors: helper descendants
must use the reviewed spawn adapter, and uncontrolled fork/secret-bearing
process cloning while authority is live is forbidden. This is a trusted-helper
constraint, not an OS sandbox claim.

There is no discoverable control endpoint to replace. A substituted stdio/control
stream lacks the per-launch secret and exact manifest, so fails authentication;
replacement of the actual verified process revokes the supervisor slot and
requires a fresh launch. Executable verification-to-execution races need a
reviewed stable-file/launch binding or equivalent platform enforcement before
strict enablement. This proposal does not claim the current digest check alone
closes that race.

Remote debugging, ptrace/task-port/process-VM access, crash dumps, privileged
handle duplication, root/administrator compromise, and whole-process snapshot
restore are explicitly outside cryptographic containment. Deployment must
restrict such access or keep strict disabled. No remote forwarding, SSH export,
debug protocol exposure, named-pipe discovery, or new network listener is added.

## Security review

This is the local threat-model extension submitted for review; it does not
change the canonical register or mark any mitigation verified.

| Threat / risk trace                                                          | Required control and test                                                                                                                                       | Residual trust assumption                                                                                                                                                                 |
| ---------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Same-user publication injection / T-09, R-011, P0-AC-021..022                | Restricted per-launch bootstrap plus pairwise TLS/manifest and authenticated ordering; a same-user peer with only a data/control handle is denied before Ack.   | Kernel, supervisor/helper code, and process-memory access protections are trusted.                                                                                                        |
| Bootstrap/key-material leakage / R-012, R-014, P0-AC-023,026                 | No argv/env/log/export exposure; child inheritance allowlist, closure, key-log prohibition; seeded-key-material tests; revoke/terminate on detected leak.       | A stolen launch private key can impersonate its delegated bearer until revocation/expiry. Tests must demonstrate containment after detection, not claim stolen material remains harmless. |
| Reflection, cross-launch or wrong-role/channel / R-011                       | Unique pairwise key, fixed TLS roles, exact manifest/channel confirmation, session/sequence checks, data-frame permit binding.                                  | Cryptographic library correctly implements TLS and rejects bad transcripts.                                                                                                               |
| Old clone/proof replay / R-011                                               | Fresh private helper lifetime, shared irreversible revocation cell, no value cloning/serialization, checks on old and new endpoint objects.                     | Arbitrary memory rollback/tampering is excluded; public tuples may repeat in offline regression fixtures.                                                                                 |
| Feed drift/alias mutation/confused deputy / R-011, R-013, P0-AC-024          | Atomic mutation invalidation, registry read gate, revision/lifetime match, full retained-byte equality; Core feed acceptance kept distinct from semantic truth. | Authoritative helper/registration policy is trusted; an echo is not independent integrity evidence.                                                                                       |
| Exhaustion, malformed/unknown protocol / R-011, overview invariant 7         | Pre-allocation size/depth/count limits, byte and record budgets, rate/deadline/overflow gates, bounded fuzz corpus; no handler side effects.                    | Bounded local denial of service is possible; default startup/tolerant paths remain usable.                                                                                                |
| Supply-chain/platform adapter error / R-015, R-018, R-019, P0-AC-028,033,034 | Separate dependency review/audit, narrow unsafe adapters, descriptor/digest stable launch, native platform matrices.                                            | No code-signing or complete same-user isolation is asserted. Unsupported adapters disable strict.                                                                                         |

Refusal precedes publication Ack/proof issuance on authority failures and
precedes handler/provider implementation/provider I/O/credential/tool contact
on strict admission failures. Publication/schema validation is pure metadata
work. Logs record only static error category and safe counters; typed sensitive
redaction still applies elsewhere. Control authority is not a provider API key,
credential reference, user login token, or runtime-administrator grant and must
never enter the provider credential resolver. Safe mode and standalone startup
do not depend on this control session.

## Verification plan

No row below is reported passed by this draft. Each implementation task records
exact revisions, platform/runner, commands, and independent review evidence.

| Test group                    | Required cases and observable pass condition                                                                                                                                                                                                                                                                                                                            |
| ----------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Exact positive flow           | Trusted launch, TLS/manifest, actual registry feed, Core acceptance, Publish/Ack/Bind/Bound/Admit, exact data frame, provider-list projection. All tuples/bytes agree; only the scoped provider-list handler runs.                                                                                                                                                      |
| Launch material/bootstrap     | Fresh material per launch; one-shot handle read/close; manifest truncation, oversize, trailing bytes, unknown profile/key type; Core and helper pin mismatch; extra bytes after the manifest; key material absent from records, logs, argv, and environment; failed launch never reuses material. No TLS, projection, or record acceptance before manifest consumption. |
| Authority negatives           | Same-user caller, control handle without launch key material, fake bootstrap on unrelated helper, wrong key, role reflection, wrong descriptor, ALPN/version/channel/session, replaced process/stream. No Ack or proof; no handler, provider, credential, or tool contact.                                                                                              |
| Deliberate secret leak        | Give adversarial fixture the bootstrap material and show bearer impersonation is possible inside the stated assumption failure; detect/revoke original launch and prove both captured key and old receipts fail on replacement. Seed launch key material/receipts in diagnostic/error inputs and assert no exposure.                                                    |
| Exact-byte prerequisite       | Reviewed/merged AI-0327 consumed revision; equal digest alone does not substitute for retained-byte equality; field, order, Unicode/escape, integer, schema, domain, and cost/context differences refuse.                                                                                                                                                               |
| Lifetime regression / AI-0330 | Same live process/control/data/session, reset/clone/reconstruct, identical offline publish/ack/bind bytes and public tuple/revision/epoch. Old proof refuses against old cloned endpoint and reconstructed endpoint; fresh internal lifetime is required. Production replay separately refuses before Ack.                                                              |
| Revocation matrix             | Each reset/clone/reconstruction, source/registry/alias mutation, supersession, grant loss, timeout/absolute expiry, closure, crash, shutdown, digest/version change, restart, process/endpoint/generation/channel replacement revokes pending/ack/proof/receipt/reservation.                                                                                            |
| Order/replay/races            | Missing/stale/future/duplicate/replayed/out-of-order Publish/Ack/Bind/Bound/Admit; feed changes between every step; mutate before/after admission; delayed/lost invalidation; old endpoint references. No stale handler entry; bounded already-admitted read has documented linearization.                                                                              |
| Resource/fuzz                 | Every named bound at limit and limit-plus-one; maximum escaped canonical bytes plus full envelope; incomplete headers, huge lengths, unknown kind/version/reason, trailing bytes, invalid TLS, rate burst, queue saturation, blocked writes, RNG failure, sequence/version/revision overflow. No oversized allocation, hang, fallback, or side effect.                  |
| Data/control separation       | TLS/control bytes on data stream, RPC on control stream, wrong frame digest/request ID/method, receipt in caller params, cross-channel injection. No sniffed enablement or schema widening.                                                                                                                                                                             |
| Domain/tolerance              | Standalone/default path without bootstrap, safe mode without providers, other `ai.*` methods, MCP `list_version`, generic snapshot version, context/session/tool/extension domains retain their own behavior; none satisfy provider-routing authority.                                                                                                                  |
| Platform matrix               | Linux, each supported BSD, macOS, Windows native runners: unrelated concurrent spawn, descendant/exec/raw-fork restrictions, fd/handle duplication/leak, endpoint/process replacement, executable race, environment dump, debugger boundary, shutdown/restart. Unsupported platform reports strict unavailable.                                                         |

### TLS flood, fairness, and cleanup acceptance tests

- Core-only source invalidation/supersession: start L1/R1 bound, send
  `Invalidate(L1,R1)`, receive `Lifetime(L2,R1) → Projection(L2,R1)`, adopt and
  publish a fresh greater Core version, then Ack/Bind/Bound. No mutation or
  guessed L2 input is necessary. Helper-only reset/clone/reconstruction emits
  `Invalidate(L1,R1) → Lifetime(L2,R1) → Projection(L2,R1)` unsolicited and the
  same rebind succeeds. Registry mutation instead advances to R2. Cross a Core
  L1 invalidation with the helper L1→L2 announcement; accept the one correlated
  predecessor once without duplicate rotation/feed. Duplicate/unrelated or
  exhausted predecessor-window cases disable publication control, never revive
  old cells or restart tolerant work.
- Exercise the exact numeric request/control vector above through the actual
  Core builder and prospective data adapter, including numeric response
  correlation, duplicate IDs, changed bytes/digests, wrong-channel responses,
  and string-ID mismatch. Generic snapshot injection remains independently
  present and cannot satisfy the provider tuple.
- At helper age 59 seconds start a tolerant provider operation whose own
  deadline extends beyond age 60. Expire publication authority at age 60: close
  control, revoke every pending strict slot and old endpoint proof, but let that
  tolerant request complete under its existing deadline without process/data
  restart. Keep the helper long-lived, verify tolerant domains still operate
  and strict stays disabled; allow an independently safe idle/demand restart,
  then verify all launch key material, channel, lifetime, and receipt identities are fresh and
  old proofs fail. No live control rebootstrap or automatic expiry respawn is
  tested or implied. Test security-compromise teardown separately.
- Before Finished, inject compatibility CCS, zero-length/empty records, junk,
  partial headers, and one-byte handshake fragments without ever completing an
  application control record. Test every pre-auth wire/record/step/work bound
  at its limit and limit-plus-one, within the existing handshake-byte ceiling.
  Count ignored/rejected input before expensive processing; close/revoke on
  exhaustion with zero Ack/proof/handler/provider/credential/tool entries.
- After TLS establishment, flood empty/padded records, alerts, and KeyUpdates
  (including requested automatic replies), with no application records. Verify
  per-direction all-wire/record/update buckets, aggregate work buckets, absolute
  launch counters, and send amplification bounds; no application rate counter
  is needed to trigger closure. Repeat across publication reset/clone/mutation
  to prove TLS launch counters never reset. Refuse Ack/proof/handler entry
  after exhaustion even when the last wire fragment also contains a valid
  application message.
- Keep the control stream continuously readable while a write is pending and
  advance an instrumented monotonic clock past establishment, transaction, and
  absolute publication-authority expiry. Verify turn byte/record/I/O/step/work/time limits,
  alternating read/write opportunities, and independent deadline service within
  `TLS_TIMER_SERVICE_MAX_DELAY`; no TLS lock may block revocation. Repeat with
  TLS-only input and no application progress. An expired launch issues no Ack,
  proof, reservation, or handler entry. Native stress tests complement the clock
  model and must establish the supported-load synchronous-step/timer bounds.
- Force no-progress library calls, fragmented messages, internal multiple-message
  records, synchronous-step overruns, and automatic KeyUpdate responses. Check
  pre-reserved work accounting and yielding/closure rather than an unbounded
  drain loop. Demonstrate this with the actual proposed released TLS library
  and both roles; library conformance alone does not verify wrapper budgets.
- In cooperative teardown test zeroization only of owned accessible storage
  with supported library hooks. Separately force abrupt peer crash/kill and
  channel loss: the survivor revokes the authoritative slot and old endpoint
  clones/receipts refuse without relying on a destructor or memory erasure.
  Do not claim these tests inspect compiler copies, kernel buffers, dumps, or
  snapshots.

Property/fuzz coverage checks arbitrary record sequences against the small
state machine, irreversible revocation, bounded storage, and no-contact
invariants. Instrument handler/provider-I/O/credential/tool entry counters and
assert zero for every refusal. Cryptographic tests use library vectors and
tampered Finished/ciphertext cases; never a mock that merely sets an
`authenticated` boolean. Real native launch tests supplement pure state tests.

## Alternatives considered

| Alternative                                                             | Disposition and reason                                                                                                                                                                                                                                                                                                                                                  |
| ----------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| A: ordinary inherited stdio/private pipe alone                          | Insufficient. Possession of a duplicated handle is not explicit supervisor-delegated component authority; add restricted bootstrap and standard authentication.                                                                                                                                                                                                         |
| B: OS-attested socket/named-pipe endpoint                               | Rejected for this scope. UID/SID/ACL and process ID do not identify authorized Core delegation. It would still need equivalent bootstrap/authentication plus discoverability, replacement/TOCTOU, per-OS peer APIs, and broader lifecycle handling. DIR-030 favors resident coprocesses; no endpoint benefit justifies that extra surface here.                         |
| B plus binary identity/signing attestation                              | Could support a separately reviewed future trust root, but no such product signing/attestation delivery exists. Do not assert it now.                                                                                                                                                                                                                                   |
| Caller metadata, generic version, pointer, or global epoch as authority | Rejected: repeatable correlators and local identity do not authenticate Core or revoke every old endpoint lifetime.                                                                                                                                                                                                                                                     |
| Handwritten challenge/HMAC protocol                                     | Rejected: standard TLS supplies transcript, role, key schedule, record integrity, and replay properties without inventing cryptography. Dependency approval remains a hard gate.                                                                                                                                                                                        |
| Mutual per-launch certificates                                          | Selected by the [Mechanism ruling (2026-10-19)](#mechanism-ruling-2026-10-19) as the launch-scoped mutual-certificate profile: per-launch self-signed leaves pinned by digest, no PKI. It adds certificate generation/validation without removing trusted bootstrap; the superseded pairwise-PSK profile had no demonstrated released client-and-server implementation. |
| Independent shared registration source                                  | Rejected initially: introduces duplicate registry population semantics and drift. The selected feed explicitly trusts the authoritative helper, with exact equality protecting freshness rather than independent truth.                                                                                                                                                 |
| Serialize a helper proof or persist replay history                      | Unnecessary. An opaque receipt refers to one live authoritative slot; session ordering, high-water marks, and irreversible revocation give bounded volatile state.                                                                                                                                                                                                      |
| Expand legacy JSON RPC schemas or multiplex control into stdio v1       | Rejected here. A dedicated stream/descriptor discriminator keeps the existing 13 method schemas and generic IPC framing intact.                                                                                                                                                                                                                                         |

## Affected contracts

### Placement and dependency direction

This proposal needs both sides to encode/validate the same control records and
derive R1 canonical bytes. Recommend a narrow optional pure contract module in
the existing `bitty-ipc-api` package: bounded provider-routing DTOs, binary
control codec, named policy constants, canonicalizer, and test vectors only.
It performs no authentication, OS work, provider selection/I/O, registry
population, secret resolution, or proof minting. Core and helper adapters depend
on it; it never depends on `bitty-ai`, Core runtime, providers, or platform
adapters. Do not move broad AI/runtime types or unrelated crates.

The helper maps its authoritative registry to those DTOs; Core holds only the
read-only view. TLS/bootstrap adapters remain separately owned composition
boundaries. Acceptance of the module placement requires the IPC owner's scoped
task, dependency/security review, and explicit consumer pin upgrades from
Core `aa7259d` and endpoint `f201911` to a reviewed merged contract revision.
Those pins are incompatible evidence baselines, not a claim that either already
supports this control profile. Golden vectors and cross-pin compatibility tests
must pass before either upgraded consumer enables strict. No silent pin upgrade
or compatibility promise is made by this draft.

### Implementation sequence after acceptance

| Order | Separately authorized scope                                                                                                                                                                                                   | Required completion evidence before next gate                                                                                                                                                                                                                            |
| ----- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1     | Architecture/security/docs acceptance of this decision; owning security-corpus synchronization; crypto/RNG/platform dependency approval; IPC pure contract module if accepted.                                                | Independent three-lens sign-off, audited dependencies and wire vectors, explicit platform exclusions and scoped pin-upgrade plan. No product implementation is authorized by the draft itself.                                                                           |
| 2     | Core supervisor control/bootstrap adapter including launch key-material generation and one-shot bootstrap delivery, descriptor profile, metadata source view, Core publication/version/invalidation and ack/receipt handling. | Native launch authority tests, stable descriptor/executable binding, current-channel revocation, bounded codec/transport, safe diagnostics; Core owns its task allocation.                                                                                               |
| 3     | Helper authenticated authority owner, shared live revocation/AI-0330, registry feed/mutation gate, retained bytes, ack/bind/receipt and exact data-frame reservation adapter.                                                 | Reviewed/merged AI-0327 exact-byte prerequisite consumed; old/new cloned-object lifetime regression fixed; cross-component and platform/security matrices green. Scopes 2/3 may develop against fixtures but cannot claim integrated authority until both pass together. |
| 4     | AI-0329 strict `ai.provider_list` admission only.                                                                                                                                                                             | Integrated Core delivery plus helper authority/AI-0330, explicit compatible pins, exact-byte evidence, independent security approval, no-contact negative tests and tolerant regressions.                                                                                |

This sequencing grants no scope for completion/context/MCP/tools/extensions,
Lua integration, new provider APIs, or broader AI method strict enforcement.
R1's accepted projection remains authoritative; this focused page owns only the
proposed residual mechanism. AIQ-38's other facets and all unrelated open
questions retain their existing status.

## Open points

The proposed choices are concrete for review: A with restricted delegation and
the launch-scoped mutual-certificate TLS profile selected by the
[Mechanism ruling (2026-10-19)](#mechanism-ruling-2026-10-19), dedicated stream,
exact records/order/bounds, feed source, pure shared contract placement, and
helper lifetime/revocation. The selected profile is not an approved runnable
mechanism: it is API-viable on released dependencies, but its carried
verification gates remain open, and neither unspecified implementation judgment
nor a silent fallback can close them.

Independent architecture, security, and documentation review approved this
bounded conditional draft (see P0 Review Sign-off). The launch-material
ownership and bootstrap-direction gate is completed in draft by
[Launch material and bootstrap direction (2026-10-19)](#launch-material-and-bootstrap-direction-2026-10-19),
including the certificate-digest pin shape. Remaining acceptance blockers are the
IPC owner's module/dependency direction; the remaining carried verification
gates (rcgen 0.13.2 supply chain, Core rustls crypto-provider and dependency
adoption, per-platform bootstrap-handle evidence, key-generation cost,
handshake bound revalidation, per-step work accounting, and launch key-material
zeroization evidence); and security-corpus synchronization by its owner.
Native platform evidence and implementation tasks remain delivery gates, not invented current support. If reviewers require
resistance to arbitrary same-user process-memory/bootstrap theft or memory
snapshot rollback, this proposal cannot meet that requirement without a stronger
externally anchored isolation/attestation design. Keep strict disabled and
escalate that trust-root decision; do not claim the launch key material repairs
its own compromised bootstrap.

No alias projection or future strict domain is accepted here. There is no
identifier mint, closure, cross-repository task assignment, or implementation
permission implicit in proposing these follow-ups.

## Acceptance criteria

- Independent architecture, security, and docs reviewers accept the explicit
  supervisor delegation and threat assumptions, exact protocol/source mapping,
  fresh private lifetime plus shared irreversible revocation, and narrow strict
  target; no blocking finding remains.
- Authority checks precede Ack/proof; strict proof/lifetime/read gates precede
  handler/provider/credential/tool contact. Same-user identity, control-handle
  possession, generic versions, and caller echoes never replace authentication.
- Old proofs fail after reset/clone/reconstruction with identical public tuples
  in the same live session against both old and new endpoint objects. Production
  replay/high-water checks remain independently required.
- Every named size/count/deadline/storage/overflow bound and platform exclusion
  has an implementation verification owner before strict enablement; none is
  represented as tested by the document's own formatting gates.
- Shared DTO/canonicalizer placement and dependency approvals are explicit;
  consumed revisions and cross-pin tests precede integration and AI-0329.
- TLS all-wire/pre-auth/lifetime/record/KeyUpdate/work limits and scheduler
  fairness have concrete real-library and native flood/timeout evidence.
  Zero-application-progress floods still revoke before Ack/proof/handler effects;
  approved standard TLS alone is not wrapper-resource evidence.
- Cleanup claims cover only owned accessible cooperative zeroization with
  reviewed library support; authoritative revocation after observed death,
  closure, or expiry does not depend on universal erasure or destructor execution.
- The released-dependency feasibility blocker stays resolved by replacement, not
  by a PSK dependency: the selected launch-scoped mutual-certificate profile
  needs its separately approved carried evidence (the full carried-gate list in
  the [Mechanism ruling (2026-10-19)](#mechanism-ruling-2026-10-19)) before the
  mechanism is runnable. Until then the mechanism remains conditional and strict
  stays disabled; no implicit library, pin/fork, custom crypto, or weaker
  fallback is authorized.
- Source/reset/mutation transitions recover with explicit predecessor correlation
  and unsolicited lifetime announcements; numeric Core request IDs and exact
  frame/response correlation are preserved. Routine authority expiry closes
  control and disables strict only, preserving tolerant work and its deadlines;
  a fresh authority requires an independently safe new helper launch.
- English self-contained content follows the specification spine and route
  indexes; accepted R1 and normative IPC/security obligations remain intact.

## P0 Review Sign-off

| Lens                              | Status                     | Required focus                                                                                                                                                                     |
| --------------------------------- | -------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Independent architecture owner    | Conditional draft approved | 2026-10-17, ctx-0162-arch-reviewer, PX-0612: protocol, source mapping, lifetime, request correlation, and expiry corrections reviewed. Mechanism acceptance remains blocked.       |
| Independent security reviewer     | Conditional draft approved | 2026-10-17, ctx-0162-security-reviewer, PX-0611: threat boundaries, resource limits, revocation, and cleanup reviewed. Dependency, corpus-owner, and native evidence gates remain. |
| Independent documentation curator | Conditional draft approved | 2026-10-17, ctx-0162-docs-reviewer, PX-0613: status, spine, routes, references, bounds, and evidence reviewed. No implementation or strict enablement approval.                    |

These are approvals to deliver the conditional draft, not unconditional P0
mechanism sign-offs. Local docs gates do not substitute for review. This draft cannot unblock AI-0329 or
authorize a Core/IPC/helper implementation by its existence.

## References

- [Execution ownership R1](execution-ownership-r1.md), accepted provider/routing
  subsection at AI-docs `abb820bdbb663d7ed1ae061a2ed7551208af02cb`.
- [Core #2199](https://github.com/bitty-terminal/bitty/issues/2199), design request;
  [AI-docs #287](https://github.com/bitty-terminal/bitty-ai-docs/issues/287).
- [Core supervisor baseline](https://github.com/bitty-terminal/bitty/tree/aceb327e16ff0f2fafe9c3915c9b26a313355514),
  [#2127](https://github.com/bitty-terminal/bitty/pull/2127), and
  [#2184](https://github.com/bitty-terminal/bitty/pull/2184).
- [AI endpoint baseline](https://github.com/bitty-terminal/bitty-ai/tree/6757b1e21ed27941b910ce0452834806fafc9c4b),
  [AI-0327 / PR #523](https://github.com/bitty-terminal/bitty-ai/pull/523),
  [AI-0330 / #522](https://github.com/bitty-terminal/bitty-ai/issues/522), and
  [AI-0329 / #521](https://github.com/bitty-terminal/bitty-ai/issues/521).
- [Core IPC pin](https://github.com/bitty-terminal/bitty-ipc/tree/aa7259d78e50d46e5362c341f2dfdd98b4db4ed6),
  [endpoint IPC pin](https://github.com/bitty-terminal/bitty-ipc/tree/f20191183f6ae9464889c5d0a6cc8606670314ee),
  and [IPC #24](https://github.com/bitty-terminal/bitty-ipc/pull/24).
- [RFC 8446](https://www.rfc-editor.org/rfc/rfc8446), especially external PSK,
  Finished transcript binding, record protection, limits, and 0-RTT exclusions.
- [RFC 9257](https://www.rfc-editor.org/rfc/rfc9257), pairwise external PSK
  provisioning, entropy, fixed roles, identity-misbinding and compromise limits.
- [Authenticated provider publication mutual-certificate feasibility](authenticated-provider-publication-mutual-certificate-feasibility.md):
  CTX-0164 API-feasibility and requirement-mapping evidence for the
  launch-scoped mutual-certificate profile selected by the mechanism ruling.
- Released rustls 0.23.46 public
  [ClientConfig](https://docs.rs/rustls/0.23.46/rustls/client/struct.ClientConfig.html)
  and [ServerConfig](https://docs.rs/rustls/0.23.46/rustls/server/struct.ServerConfig.html):
  inspected pin does not expose the required external-PSK configuration;
  resumption does not establish it. Upstream
  [external-PSK PR #2424](https://github.com/rustls/rustls/pull/2424) was closed
  pending future work; [client draft PR #2524](https://github.com/rustls/rustls/pull/2524)
  closed unmerged on 2025-09-15. These do not prove absence in every later release.
- [AI-0327 merged exact-byte baseline](https://github.com/bitty-terminal/bitty-ai/commit/a2bb62a50582b060e3c1e1173e4af2ab0bf35813):
  retained bytes and 132 endpoint tests, not authenticated live publication or
  AI-0330 cross-reset isolation evidence.
- [POSIX spawn descriptor semantics](https://pubs.opengroup.org/onlinepubs/9699919799/functions/posix_spawn.html),
  [Windows explicit handle list](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-updateprocthreadattribute),
  and [Windows inheritance](https://learn.microsoft.com/en-us/windows/win32/procthread/inheritance):
  platform primitive references, not proof of Bitty adapter support.
