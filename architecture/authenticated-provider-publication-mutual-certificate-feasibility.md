---
title: Authenticated provider publication mutual-certificate feasibility
description: API-feasibility and requirement-mapping evidence for the launch-scoped mutual-certificate replacement candidate of the authenticated publication proposal
category: architecture
audience: security-reviewer
document_type: reference
status: draft
website_publish: false
sidebar_order: 24
---

# Authenticated provider publication mutual-certificate feasibility

**Feasibility evidence record, not a decision** — this reference records the
options analysis and API feasibility evidence for the launch-scoped
mutual-certificate alternative recorded by
[Authenticated provider publication dependency feasibility](authenticated-provider-publication-dependency-feasibility.md),
mapped against the requirements of
[Authenticated provider publication](authenticated-provider-publication.md),
which remains the authoritative proposal text. It approves no dependency,
accepts no mechanism, selects no fallback, edits no profile or bound, enables
no strict behavior, and closes no question. Strict provider publication
remains disabled.

## Purpose and scope

The authenticated publication proposal selects a conditional TLS 1.3
external-PSK profile whose released-dependency feasibility blocker remains
unresolved: the dependency survey found no candidate demonstrated to satisfy
the entire profile and recorded, unselected, the possibility of weighing
s2n-tls against a design change such as launch-scoped mutual certificates.
This reference records the feasibility analysis of that replacement candidate:
per-launch self-signed certificates pinned by certificate digest, mutual TLS
through released rustls custom verifiers, and key and certificate generation
via rcgen, with no PKI and no external PSK.

In scope:

- The requirement under test: the proposal's profile and cross-cutting
  requirements, restated, with the one deliberate replacement recorded.
- The candidate mechanism and its recorded sub-options.
- First-hand API feasibility evidence from the released source of the pinned
  rustls, rcgen, and rustls-pki-types versions.
- The requirement mapping, the changes the proposal must make if the candidate
  is later accepted, and the open verification gates.

Out of scope:

- Approving, pinning, adding, forking, or vendoring any dependency.
- Changing the proposal's profile, bounds, records, or acceptance criteria.
- Selecting the mechanism, closing any question, or enabling strict behavior.
- Implementing any part of the mechanism.
- Re-adjudicating the dependency survey or the s2n-tls candidate.

## Requirement under test

This restates the requirement set from the proposal and the dependency survey;
the proposal remains the authoritative definition. A replacement candidate is
acceptable only if it can demonstrably provide:

- A mutually authenticated supervisor-helper control channel with fixed roles
  (`core-supervisor` TLS client, `ai-helper` TLS server), never reused for
  another launch or purpose.
- TLS 1.3 only, with no resumption, session tickets, 0-RTT, or renegotiation.
- ALPN `provider-publication-control/1`; no application record accepted
  before the handshake completes.
- No PKI: no certificate authority, no chain trust, no network-fetched trust
  material.
- No network listener or TCP in Core; bounded custom pipe I/O with
  per-work-unit accounting and an audited per-step upper bound for
  library-internal work.
- MSRV 1.85 and Linux, BSD-family, macOS, and Windows coverage.
- Limited cooperative zeroization, with key logging, plaintext tracing, and
  session export disabled.
- No fork, custom cryptography, or weaker fallback; a reviewed dependency
  audit.

One proposal line is deliberately not a requirement of this candidate: the
external-PSK profile's "No certificate/TOFU fallback". This candidate replaces
that line by design; the replacement is recorded under what the design must
change and requires the new reviewed decision, never a silent fallback.

## Candidate description

The candidate mechanism: per-launch self-signed certificates pinned by
certificate digest, mutual TLS through released rustls custom verifiers, and
key and certificate generation via rcgen; no PKI and no external PSK.

- Each side holds one leaf key pair and one self-signed certificate generated
  for the launch; the verifier on each side accepts exactly the peer's pinned
  digest rather than any chain or name. No certificate authority is involved.
- The custom verifiers on both sides make only the trust (pin) decision and
  delegate all signature verification to released rustls helpers, so no custom
  cryptography is written.
- Fixed roles stay as proposed: the supervisor is the TLS client and the
  helper is the TLS server.

Recorded sub-options, not selected here:

- Material ownership: the supervisor generates both sides' material and
  delivers the helper's key material plus the pin(s) the helper needs through
  the existing one-shot bootstrap (which currently flows supervisor to helper
  only); or each side generates its own material and pins are exchanged,
  which would change the bootstrap direction.
- Pin shape: SHA-256 over the exact certificate DER bytes, which the verifier
  can hash without parsing; or SPKI extraction, which requires DER parsing.
- Algorithm and provider: the key algorithm (for example P-256 or Ed25519)
  and the rustls crypto provider (aws-lc-rs or ring) remain open.

## API feasibility evidence

Evidence basis: the released source of rustls 0.23.46, rcgen 0.13.2, and
rustls-pki-types 1.15.1, as resolved into the local Cargo registry source
trees, plus the workspace lockfiles and manifests cited inline. Claim
strength: first-hand API facts at pinned releases, not a conformance run.

### rustls 0.23.46

The artifact is the same release the bitty-ai workspace lockfile already
resolves (checksum
`48e13bd8c0e9365c43cfa5c9e8f9ad49d3c8444926c9aac819e0e4dc503c8fdf`,
`bitty-ai/Cargo.lock:1276-1279`), so no new TLS crate family is introduced to
that graph. Paths below are relative to the crate root.

- Custom trust decision exists for both roles under the audited `danger`
  modules:
  - `rustls::client::danger` re-exports `ServerCertVerifier`,
    `ServerCertVerified`, and `HandshakeSignatureValid` (`src/lib.rs:608-611`).
    The trait is defined at `src/verify.rs:69` with `verify_server_cert`
    (`:83`), `verify_tls12_signature` (`:107`), `verify_tls13_signature`
    (`:128`), `supported_verify_schemes` (`:139`), `requires_raw_public_keys`
    (default `false`, `:143`), and `root_hint_subjects` (default `None`,
    `:153`). `verify_server_cert` receives the end-entity certificate as
    unparsed `CertificateDer` bytes plus intermediates; the trait doc states
    none of the certificates have been parsed yet (`src/verify.rs:74-76`), so
    the exact DER bytes are directly available for a SHA-256 pin comparison.
  - `rustls::server::danger` re-exports `ClientCertVerifier` and
    `ClientCertVerified` (`src/lib.rs:658-660`). The trait is defined at
    `src/verify.rs:160` with `offer_client_auth` (default `true`, `:163`),
    `client_auth_mandatory` (default follows the offer, `:170`),
    `root_hint_subjects` (`:203`), `verify_client_cert` (`:219`),
    `verify_tls12_signature` (`:241`), `verify_tls13_signature` (`:257`),
    `supported_verify_schemes` (`:268`), and `requires_raw_public_keys`
    (default `false`, `:272`).
- The role builders accept the custom verifiers and the launch material:
  client `ClientConfig::builder().dangerous()` (`src/client/builder.rs:85`)
  `.with_custom_certificate_verifier(Arc<dyn ServerCertVerifier>)` (`:107`)
  then `.with_client_auth_cert(cert_chain, key_der)` (`:146`); server
  `ConfigBuilder::with_client_cert_verifier(Arc<dyn ClientCertVerifier>)`
  (`src/server/builder.rs:16`) then `with_single_cert(cert_chain, key_der)`
  (`:65`, which also fails if the private key's public key does not match the
  end-entity certificate).
- Signature verification delegates to
  `rustls::crypto::{verify_tls12_signature, verify_tls13_signature}`
  (re-exported at `src/crypto/mod.rs:14`, defined at
  `src/webpki/verify.rs:155,192`), so the custom verifier performs only the
  pin comparison.
- TLS 1.3 only is expressible exactly:
  `builder_with_protocol_versions(&[&rustls::version::TLS13])` on both client
  and server configs (`src/client/client_conn.rs:332`,
  `src/server/server_conn.rs:472`; the version statics are at
  `src/versions.rs:25-30`). Defaults otherwise enable TLS 1.2 as well, so the
  version restriction is required, not automatic.
- Resumption and related knobs are suppressible exactly: client
  `resumption: Resumption` (`src/client/client_conn.rs:191`) with
  `Resumption::disabled()` (`:513`); `enable_early_data` default `false`
  (`:226-227`); server `send_tls13_tickets` (`src/server/server_conn.rs:399`,
  default 2, set to 0 for no tickets) and `max_early_data_size` default 0,
  which disables early data (`:366`, `:724`). TLS 1.3 has no renegotiation.
- Bounded custom pipe I/O is the only I/O mode: rustls performs no I/O
  itself; the caller drives `read_tls` (`src/conn.rs:784`), `write_tls`
  (`:812`), `process_new_packets` (`:461`), and `wants_read` / `wants_write`
  (`src/common_state.rs:745,103`), so the wrapper keeps the proposal's byte,
  record, step, and work accounting. TLS fragments application frames rather
  than requiring one frame per record.
- ALPN is a plain field on both configs (`alpn_protocols`,
  `src/client/client_conn.rs:167`, `src/server/server_conn.rs:338`).
- Key logging is absent unless installed: the `key_log` sink
  (`src/client/client_conn.rs:217`, `src/server/server_conn.rs:349`) is never
  populated by the standard builders.
- Zeroization: rustls uses zeroize across key-schedule and record layers
  (for example `src/crypto/tls13.rs`, `src/crypto/cipher.rs`,
  `src/msgs/base.rs`, `src/crypto/mod.rs`); the crypto provider's key objects
  are not promised. Recorded as limited cooperative cleanup pending the
  adoption-time zeroization review.
- MSRV `1.71` (`Cargo.toml:14`), at or below the project's 1.85. License
  `Apache-2.0 OR ISC OR MIT` (`:34`).
- Crypto provider: default features include `aws_lc_rs` and
  `prefer-post-quantum`; `ring` is an alternative feature. The provider's
  `kx_groups` (`src/crypto/mod.rs:201`) is where group preference is
  expressed; provider and group policy remain an adoption gate.

Workspace direction: `bitty-ai` consumes the `bitty-network` family at rev
`2aa466387db8e7b2092ce31a09c51040e38ec3e0`
(`bitty-ai-slice/Cargo.toml:44,49,61`); at that revision `bitty-network-tls`
pins `rustls = "=0.23.46"` with `["std", "aws-lc-rs"]` and
`rustls-pki-types = "=1.15.1"`. The `bitty` Core workspace has no rustls
anywhere today (no match in its `Cargo.lock` or crate manifests), so the Core
side of the control channel is an adoption decision: a reviewed rustls edge or
a reviewed transport crate.

### rcgen 0.13.2

- `KeyPair::generate()` generates a key pair for `PKCS_ECDSA_P256_SHA256` by
  default; `generate_for(alg)` selects the algorithm; generation uses
  `SystemRandom` (`src/key_pair.rs:68-110`, `:77`, `:88`). Backends: default
  features are `crypto`, `pem`, `ring` (`Cargo.toml:165-169`); `aws_lc_rs`
  and `fips` are alternative features.
- `CertificateParams` (`src/certificate.rs:67`) with
  `self_signed(self, key_pair)` (`:177`) produces a self-signed `Certificate`;
  `Certificate::der()` returns `&CertificateDer<'static>`
  (`src/certificate.rs:47`) — the same rustls-pki-types DER type rustls
  consumes.
- Bridge: rcgen depends on rustls-pki-types (`pki-types` alias,
  `Cargo.toml:102-104`); `KeyPair::serialize_der()` returns PKCS#8 DER bytes
  (`src/key_pair.rs:476`), and rustls-pki-types 1.15.1 provides
  `TryFrom<Vec<u8>> for PrivateKeyDer` (`src/lib.rs:285`) and
  `From<Vec<u8>> for PrivatePkcs8KeyDer` (`:484`), so generation output feeds
  `with_single_cert` / `with_client_auth_cert` directly.
- Hygiene: license `MIT OR Apache-2.0` (`Cargo.toml:30`), repository
  `github.com/rustls/rcgen` (`:31`), edition 2021 (`:13`); no `rust-version`
  declared (an MSRV gate, below). Required dependencies are `pki-types`,
  `time`, and `yasna`; optional `pem`, `ring`, `aws_lc_rs`, `x509-parser`,
  `zeroize` (`Cargo.toml:93-128`).
- Zeroization: the optional `zeroize` feature implements `Zeroize for
KeyPair`, zeroizing rcgen's own serialized DER copy (`src/lib.rs:656-659`);
  the backend key object is not covered — the same limited cooperative
  framing the proposal already states.

### Cross-check: per-step work accounting

The proposal's per-step upper bound on library-internal work was the
requirement no opaque C candidate demonstrated. rustls is a pure Rust library
whose internal per-record and handshake work is auditable and instrumentable,
so this requirement is structurally more tractable here; the audited bound
and instrumented tests remain adoption work and are not established by this
record.

## Requirement mapping

| Requirement                                              | Status                                                        | Evidence                                                                                                        |
| -------------------------------------------------------- | ------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| Mutual authentication, fixed roles, one launch, no reuse | Candidate-shaped; material ownership is a recorded sub-option | Both-direction custom verifiers; per-launch material generation                                                 |
| TLS 1.3 only                                             | Satisfied at API level                                        | `builder_with_protocol_versions(&[&TLS13])` both roles                                                          |
| No resumption, tickets, 0-RTT, renegotiation             | Satisfied with exact knobs                                    | `Resumption::disabled()`; `send_tls13_tickets = 0`; early-data defaults 0 / false; TLS 1.3 has no renegotiation |
| ALPN `provider-publication-control/1`                    | Satisfied                                                     | `alpn_protocols` on both configs                                                                                |
| No PKI                                                   | Satisfied by construction                                     | self-signed leaf plus digest pin; no roots, chains, or fetched trust material                                   |
| No certificate/TOFU fallback (PSK profile)               | Must change, deliberately                                     | this candidate replaces the line; requires the new reviewed decision, never a silent fallback                   |
| No network listener or TCP in Core                       | Satisfied structurally                                        | rustls performs no I/O; caller-owned `Read`/`Write` only                                                        |
| Bounded custom pipe I/O with accounting                  | Satisfied structurally                                        | `read_tls` / `write_tls` / `process_new_packets` over caller buffers; accounting stays wrapper-owned            |
| Audited per-step bound                                   | Improved, not established                                     | pure-Rust internals auditable; adoption task must bound and test                                                |
| MSRV 1.85                                                | rustls yes (1.71); rcgen undeclared                           | adoption gate for rcgen                                                                                         |
| Linux, BSD-family, macOS, Windows                        | Not locally established                                       | vendor-level claims only; platform evidence gate                                                                |
| Limited cooperative zeroization                          | Partial                                                       | rustls zeroize usage; rcgen copy-only `zeroize` feature; provider key objects unclaimed; review gate            |
| No fork, custom crypto, or weaker fallback               | Satisfied by construction                                     | released crates plus delegated signature helpers                                                                |
| Reviewed dependency audit                                | Open                                                          | rcgen is a new dependency; rustls already resolves in bitty-ai                                                  |

## What the design must change (if this candidate is later accepted)

Recorded for the acceptance task; not performed by this record and not a
selection:

1. Rewrite the exchange-profile bullet: replace the external-PSK clause with
   the mutual-certificate clause (TLS 1.3 only; one self-signed leaf per side;
   digest pinning; no PKI; no resumption, tickets, or 0-RTT).
2. Invert "No certificate/TOFU fallback" explicitly as a deliberate reviewed
   profile change; the pin is launch-scoped, not web-PKI trust on first use.
3. Bootstrap content: replace the appended raw launch PSK with launch key
   material and/or pin delivery; note that the current one-way bootstrap fits
   the supervisor-generates-both option, while per-side generation would
   change the bootstrap direction.
4. Bounds: replace `AUTHORITY_SECRET_BYTES` and the PSK-specific figures with
   key-material and pin sizes; re-validate `CONTROL_HANDSHAKE_MAX_BYTES` and
   the TLS pre-authentication wire/record tables against certificate
   handshake messages (single self-signed leaves, no chains).
5. Zeroization bullets: extend with the verified candidate limits (provider
   key objects outside zeroize; rcgen copy-only coverage).
6. Public identity wording: the launch nonce is no longer the sole opaque
   public identity; pin digests join it.
7. Revocation semantics: unchanged at launch lifetime; state explicitly that
   a no-PKI design has no CRL or OCSP path.

## Open verification gates

The adoption/security task must still produce, before any acceptance:

- rcgen MSRV, platform matrix, maintenance evidence, and a full dependency
  audit of its chain (`time`, `yasna`).
- The rustls crypto-provider decision (aws-lc-rs versus ring), including
  `kx_groups` policy under the `prefer-post-quantum` default and the feature
  set (`std`; the `tls12` feature compiling code the version restriction
  disables at runtime).
- The zeroization review across rustls internals, the provider, and rcgen.
- Key-generation cost and resource bounds measured under an audited bound; no
  unmeasured numbers are asserted by this record.
- The pin-shape choice (certificate DER versus SPKI) and its extraction path.
- The material-ownership and bootstrap-direction decision.
- Handshake byte/record bound re-validation with certificate sizes.
- A per-step accounting instrumentation plan for rustls internals.
- The Core-side dependency direction and platform evidence runs for both
  repositories.

## Verdict

The candidate is API-viable in both halves on released dependencies: rustls
0.23.46, already resolved in the bitty-ai workspace, provides both directions
of digest pinning, TLS 1.3 restriction, resumption suppression, bounded custom
I/O, and delegated signature checking from its released source; rcgen 0.13.2
provides self-signed generation and DER serialization inside the same
rustls-pki-types family; and no custom cryptography or fork is required
anywhere. The candidate is not yet demonstrated to satisfy the entire
requirement set: rcgen's MSRV, platform, and supply-chain evidence, the
zeroization review, key-generation bounds, the pin-shape and material-ownership
options, the recorded design changes, and the Core-side dependency direction
remain open.

This record selects no mechanism, approves no dependency, and enables no
strict behavior. Strict provider publication remains disabled, and the
recorded alternatives remain live: weighing the s2n-tls candidate against
this design change, and an explicitly reviewed protocol or profile change.

## Compatibility and versioning

- The API facts are pinned to released artifacts inspected on 2026-10-19:
  rustls 0.23.46, rcgen 0.13.2, rustls-pki-types 1.15.1, and the workspace
  lockfile states cited inline. They are not a compatibility promise and not
  an ongoing claim about every future release.
- A release that changes these APIs, or that ships an external-PSK
  implementation, would need its own reviewed evidence.
- No dependency, version pin, fork, vendored binding, custom crypto, or
  weaker fallback is selected or authorized by this record.

## References

- [Authenticated provider publication](authenticated-provider-publication.md):
  the specification that defines the conditional external-PSK profile, its
  released-dependency feasibility blocker, and its acceptance criteria.
- [Authenticated provider publication dependency feasibility](authenticated-provider-publication-dependency-feasibility.md):
  the survey that records this replacement candidate as an unselected
  possibility.
- rustls 0.23.46 released source: <https://github.com/rustls/rustls> (verified
  locally in the Cargo registry source tree).
- rcgen 0.13.2 released source: <https://github.com/rustls/rcgen>.
- rustls-pki-types 1.15.1: <https://github.com/rustls/pki-types>.
- [RFC 8446](https://www.rfc-editor.org/rfc/rfc8446): TLS 1.3 handshake,
  Finished transcript binding, and record protection.
