---
title: Authenticated provider publication dependency feasibility
description: Released-dependency survey evidence for the conditional external-PSK TLS profile of the authenticated publication proposal
category: architecture
audience: security-reviewer
document_type: reference
status: draft
website_publish: false
sidebar_order: 22
---

# Authenticated provider publication dependency feasibility

**Feasibility evidence record, not a decision** — this reference records the
released-dependency survey for the conditional external-PSK TLS profile required
by [Authenticated provider publication](authenticated-provider-publication.md).
It approves no dependency, accepts no mechanism, selects no fallback, enables no
strict behavior, and closes no question. Strict provider publication remains
disabled.

## Purpose and scope

The authenticated publication proposal selects a conditional TLS 1.3
external-PSK profile and states a released-dependency feasibility blocker: a
separate dependency/security task must demonstrate a released implementation
that meets an exact requirement, or strict publication stays disabled. This
reference records the outcome of that survey for the candidate set below and the
verdict the survey supports.

In scope:

- The requirement under test and the candidates surveyed.
- The released API/prototype evidence and upstream tracker state for each
  candidate.
- The verdict on whether a released Rust dependency meets the requirement.
- The secondary structural observation about audited per-step work bounds.

Out of scope:

- Approving, pinning, forking, or vendoring any dependency.
- Changing the reviewed profile, its bounds, its records, or its acceptance
  criteria in the proposal.
- Re-opening, re-accepting, or closing any question, risk, or review gate.
- Selecting a different authentication mechanism or protocol version.

## Requirement under test

The survey tests whether a released dependency provides all of the following.
This restates the proposal's profile; the proposal remains the authoritative
definition.

- TLS 1.3 external PSK in **both** client and server roles, with fixed roles.
- A raw length-delimited PSK identity for a 32-byte random launch nonce, with no
  NUL-termination.
- ALPN `provider-publication-control/1`.
- No resumption, session tickets, 0-RTT, or renegotiation.
- No network listener or TCP in Core.
- Bounded custom pipe I/O with per-work-unit accounting and an audited per-step
  upper bound for library-internal work.
- MSRV 1.85.
- Linux, BSD-family, macOS, and Windows coverage.
- Limited cooperative zeroization support.
- No fork, custom crypto, or weaker fallback.

A candidate meets the requirement only if it satisfies every point. A partial
result, an unreviewed binding, or a future or unreleased API is not sufficient.

## Survey basis and method

- The candidate set is anchored on [RFC 9257](https://www.rfc-editor.org/rfc/rfc9257)
  section 6.1, the published form of draft-ietf-tls-external-psk-guidance. That
  section is the authoritative implementation survey for external PSK: it lists
  OpenSSL/BoringSSL, mbedTLS, gnuTLS, and wolfSSL, and it does **not** list
  rustls. The Rust-reachable candidates were added from the TLS stacks the
  proposal's profile could use (rustls, its wrappers, and released FFI
  bindings).
- For each candidate the survey inspected the released public API surface
  (crate and `docs.rs` documentation) for client and server external-PSK
  configuration, and the relevant upstream issue and pull-request state.
- Limits of the evidence. Absence in an inspected released API is evidence about
  that released artifact, not proof about every future release. The survey did
  not run a runtime conformance test, and it did not demonstrate a
  Rust-reachable API for any candidate. Upstream tracker state is quoted as
  observed at survey time.

## Candidate survey results

| Candidate                   | Evidence source                                                                                                                                                                                                                                                                                                            | Result                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| --------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| rustls 0.23.x               | Released `ClientConfig` and `ServerConfig`; [issue #174](https://github.com/rustls/rustls/issues/174); [external-PSK PR #2424](https://github.com/rustls/rustls/pull/2424); [client draft PR #2524](https://github.com/rustls/rustls/pull/2524); [0.24 tracking issue #2400](https://github.com/rustls/rustls/issues/2400) | Exposes no external PSK; session resumption is a different mechanism and does not satisfy the requirement. The released `ClientConfig` and `ServerConfig` expose no external-PSK configuration; upstream PSK request issue #174 ("Status of PSK Support?") is still open; PR #2424 ("add TLS 1.3 PSK support") was closed unmerged on 2025-06-13; PR #2524 ("client: initial support for preshared keys") was closed unmerged on 2025-09-15; the open 0.24 tracking issue #2400 does not list external PSK. The survey inspected the released 0.23.x line; 0.23.46 (2026-10-14) is the latest release and the pin the proposal cites, and it is PSK-free like the rest of the line. Does not meet the requirement.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| tokio-rustls and native-tls | Released wrapper crates                                                                                                                                                                                                                                                                                                    | Thin wrappers over rustls and platform TLS; expose no external-PSK configuration. Do not meet the requirement.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| openssl 0.10.81             | [`SslContextBuilder`](https://docs.rs/openssl/0.10.81/openssl/ssl/struct.SslContextBuilder.html); OpenSSL [`SSL_CTX_set_psk_use_session_callback`](https://docs.openssl.org/master/man3/SSL_CTX_set_psk_use_session_callback/) documentation; [OpenSSL issue #26404](https://github.com/openssl/openssl/issues/26404)      | `SslContextBuilder` exposes only `set_psk_client_callback`, the deprecated `set_psk_callback`, and `set_psk_server_callback`. A client using TLS 1.3 PSKs must use `SSL_CTX_set_psk_use_session_callback()` or `SSL_set_psk_use_session_callback()`, which the crate does not expose. The exposed client callback also requires a NUL-terminated C-string identity, incompatible with a raw 32-byte nonce. Issue #26404 (January 2025) records a TLS 1.3 PSK server-callback defect. No TLS 1.3 external-PSK path.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| boring (BoringSSL bindings) | `SslContextBuilder` on `docs.rs`                                                                                                                                                                                                                                                                                           | Exposes only `set_psk_client_callback`, the deprecated `set_psk_callback`, and `set_psk_server_callback`; no TLS 1.3 external-PSK session callback. No TLS 1.3 external-PSK path.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| openssl-sys                 | `docs.rs` API lookup                                                                                                                                                                                                                                                                                                       | The released openssl-sys 0.9.117 API declares `SSL_CTX_set_psk_client_callback` (its `docs.rs` page resolves) but has no declaration for `SSL_CTX_set_psk_use_session_callback` or `SSL_CTX_set_psk_find_session_callback` (both `docs.rs` lookups return 404). The low-level FFI therefore carries the TLS 1.2-style callback but not the TLS 1.3 session callbacks the profile needs. No released path for the required callbacks.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| s2n-tls (AWS)               | [aws/s2n-tls issue #2326](https://github.com/aws/s2n-tls/issues/2326)                                                                                                                                                                                                                                                      | s2n-tls implements TLS 1.3 external PSK in the C library, and issue #2326 ("S2N external PSK support") is closed with state_reason "completed" on 2021-06-23, not an open request. The Rust crate `s2n-tls` 0.3.45 (published 2026-10-02) exposes a `psk` module: `psk::Psk` (documented as representing an out-of-band pre-shared key) with `psk::Builder` and `set_identity`, `set_secret`, `set_hmac`, and `build`; `enums::PskMode` with `External` and `Resumption`; `enums::PskHmac`; and `Connection::append_psk`, `Connection::negotiated_psk_identity`, and `Connection::negotiated_psk_identity_length`. Custom I/O is reachable: `Connection::set_receive_callback` and `Connection::set_send_callback` with `set_receive_context` and `set_send_context`, so a non-socket pipe transport is not excluded by the API. This makes s2n-tls a live candidate, not a dismissal. Not established, and not claimed either way: the crate's MSRV, its supported platform matrix, limited cooperative zeroization, and whether external-mode selection for both client and server roles is reachable from the Rust surface (no `set_psk_mode` method was found on `Connection` or the `Config` builder; only the `PskMode` enum itself was verified). The proposal's per-step work-accounting requirement also remains unmet by any candidate. Not demonstrated to satisfy the entire profile. |
| mbedTLS                     | RFC 9257 section 6.1                                                                                                                                                                                                                                                                                                       | Its external-PSK API is the TLS 1.2-style one and caps PSK identity and key length at 16 bytes (RFC 9257 section 6.1), which cannot carry a 32-byte identity. Does not meet the requirement.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| gnuTLS and wolfSSL          | RFC 9257 section 6.1                                                                                                                                                                                                                                                                                                       | C libraries with external-PSK support; RFC 9257 section 6.1 notes gnuTLS accepts raw byte strings and wolfSSL uses OpenSSL-like callbacks. Their Rust bindings are thin, and the TLS 1.3 external-PSK API surface, MSRV, platform coverage, and maintenance were not demonstrated. Not demonstrated.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| GatoPSKTLS                  | `docs.rs`                                                                                                                                                                                                                                                                                                                  | Niche `no_std` TLS 1.3 PSK crate; not a credible independently reviewed dependency for a security boundary. Not acceptable.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |

## Verdict

No surveyed candidate was demonstrated to satisfy the entire profile. An
earlier reading of this record was wrong on one candidate and is corrected here:
s2n-tls **does** implement TLS 1.3 external PSK and exposes a Rust-crate surface
for it, including raw-byte identity, an `External` PSK mode, and custom
send/receive callbacks, so it is not rejected for lacking support; it is a live
candidate. What remains unestablished for s2n-tls, and for every other
candidate, is the full requirement: for s2n-tls these are the crate's MSRV, its
supported platform matrix, limited cooperative zeroization, whether external mode
selection for both client and server roles is reachable from the Rust API, and
the proposal's per-step work-accounting requirement.

The profile is therefore **not yet demonstrated to be implementable with a
released dependency**, and a new reviewed decision is still required. Recorded
possibilities, not selected here, are:

- A different concrete authentication mechanism.
- An explicitly reviewed protocol or profile change, such as TLS 1.2 PSK, which
  the current proposal forbids as a silent downgrade.
- Weighing s2n-tls, now a live candidate, against a design change such as
  launch-scoped mutual certificates.

This record approves no dependency and enables no strict behavior. Strict
publication remains disabled.

## Secondary structural observation

The proposal additionally requires a per-step upper bound on library-internal
work, covered by an audited bound and instrumented tests rather than omitted
because it is invisible to the wrapper. That requirement is difficult to satisfy
with any opaque C TLS library, whose internal per-record cryptographic and
handshake work is not exposed as a bounded, auditable step. Even a C-based
candidate such as gnuTLS or wolfSSL would face that accounting gate in addition
to the MSRV, platform, and maintenance evidence the survey left undemonstrated.

## Compatibility and versioning

- This reference records one survey at specific released versions and upstream
  tracker states. Its claims are pinned to those inspected artifacts; they are
  not a compatibility promise and not an ongoing claim about every future
  release.
- The verdict is about released artifacts at survey time (2026-10-18). A future
  release that adds the required client and server external-PSK API would not be
  covered here and would need its own reviewed dependency evidence.
- No dependency, version pin, fork, vendored binding, custom crypto, or weaker
  fallback is selected or authorized by this record.

## References

- [Authenticated provider publication](authenticated-provider-publication.md):
  the specification that defines the external-PSK profile, its TLS work
  accounting, its released-dependency feasibility blocker, and its acceptance
  criteria.
- [RFC 8446](https://www.rfc-editor.org/rfc/rfc8446): TLS 1.3 external PSK,
  Finished transcript binding, record protection, and limits.
- [RFC 9257](https://www.rfc-editor.org/rfc/rfc9257): external-PSK provisioning
  guidance; section 6.1 is the authoritative stack-interface survey cited above.
- rustls upstream state: [issue #174](https://github.com/rustls/rustls/issues/174),
  [external-PSK PR #2424](https://github.com/rustls/rustls/pull/2424),
  [client draft PR #2524](https://github.com/rustls/rustls/pull/2524), and
  [0.24 tracking issue #2400](https://github.com/rustls/rustls/issues/2400).
- [openssl crate `SslContextBuilder`](https://docs.rs/openssl/0.10.81/openssl/ssl/struct.SslContextBuilder.html),
  OpenSSL's [`SSL_CTX_set_psk_use_session_callback`](https://docs.openssl.org/master/man3/SSL_CTX_set_psk_use_session_callback/)
  documentation, and [OpenSSL issue #26404](https://github.com/openssl/openssl/issues/26404).
- [boring crate `SslContextBuilder`](https://docs.rs/boring/latest/boring/ssl/struct.SslContextBuilder.html)
  and [openssl-sys](https://docs.rs/openssl-sys) released API surfaces.
- [aws/s2n-tls issue #2326](https://github.com/aws/s2n-tls/issues/2326).
