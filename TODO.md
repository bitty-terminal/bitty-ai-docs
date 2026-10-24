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
  is the single authoritative gate: after the `3091977` verification, six rows
  are design-review approved (Architecture PX-1580, Documentation PX-1579,
  bitty-ai PX-1582, Plugin PX-1583, Wheel PX-1585, Product PX-1586) and three
  remain `Pending — NEEDS-FIX` (Security PX-1581 and Accessibility PX-1587 for
  the AT approval trust boundary, Core PX-1584 for re-review plus accepted
  `bitty#2546`). Core's final sign-off is conditional on `bitty#2546`. The 19
  open points carry an explicit disposition or deferral in the document;
  product decisions (`D1`-`D7`; `D7` discharged) and packaging
  (`P-1`-`P-4`) decisions and the non-blocking P2/P3 review follow-ups are listed
  in the review follow-ups. Promotion into the owning registers and issues runs
  through #553 and the named owners, not from this work register.
