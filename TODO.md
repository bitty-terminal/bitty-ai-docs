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
- [x] Wire this repository into `bitty-ai` as the `docs/` submodule
      (AI-0004, AI-0007): gitlink tracks canonical main.

## Blocked / open

- The `reference/` AI-core topic tree lands only when reviewed implementation
  evidence exists; empty placeholder pages are avoided. Its owning open
  questions, prerequisite evidence, and landing order are recorded in the
  [Planned trees](docs/README.md#planned-trees) table of the documentation map.
- Vertical-slice pressure-test gaps G-1 through G-6
  ([AI Vertical Slice Pressure Test](product/ai-vertical-slice-pressure-test.md))
  are proposals owned by `bitty`, `bitty-terminal-docs`, and `bitty-docs`; they
  are tracked there and not resolved in this repository.
- AI-0347 ([bitty-ai#553](https://github.com/bitty-terminal/bitty-ai/issues/553))
  tracks the P0 dispositions and open-point promotion for the
  [Headed Agent contracts](agent/headed-agent-contracts-candidate.md) candidate
  design. The document stays `draft`. Its nine-row
  [P0 Review Sign-off](agent/headed-agent-contracts-candidate.md#p0-review-sign-off)
  is the single authoritative gate: every row remains pending, no row is
  approved, and all nine independent reviews recorded `NEEDS-FIX` (Wheel
  PX-1550, Documentation PX-1551, Architecture PX-1552, Security PX-1553, Core
  PX-1564/AI-0350, bitty-ai PX-1565/AI-0351, Plugin PX-1566/AI-0352, Product
  PX-1563/AI-0353, Accessibility PX-1562/AI-0354) require re-review. The 17 open
  points carry an explicit disposition or deferral in the document, and open
  product (`D1`-`D6`) and packaging (`P-1`-`P-4`) decisions are listed in the
  review follow-ups. Promotion into the owning registers and issues runs through
  #553 and the named owners, not from this work register.
