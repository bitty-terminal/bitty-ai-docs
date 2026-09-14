# TODO

Repository work register for `bitty-ai-docs`. This file stays under 300 lines
(enforced by `just agents`); completed items move to git history rather than
accumulating here.

## Current

- [x] Bootstrap repository scaffold (CTX-0187 Phase 1): docs-quality toolchain,
      docs-quality workflow, CarryCtx baseline, documentation skeleton, labels, and
      repository metadata.
- [x] AI-core content migration (CTX-0001): import bitty-docs
      `docs/projects/bitty/specifications/` at `c664214` with history preserved;
      rewrite cross-repository links to absolute URLs; preserve each document's
      status and `website_publish` flag.
- [ ] Later phase: wire this repository into `bitty-ai` as the `docs/`
      submodule once the implementation repository exists.

## Blocked / open

- Further AI-core topic trees (architecture, context, providers, reference)
  land as reviewed content is produced; empty placeholder pages are avoided.
- Vertical-slice pressure-test gaps G-1 through G-6
  ([AI Vertical Slice Pressure Test](specifications/ai-vertical-slice-pressure-test.md))
  are proposals owned by `bitty`, `bitty-terminal-docs`, and `bitty-docs`; they
  are tracked there and not resolved in this repository.
