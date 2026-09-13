# TODO

Repository work register for `bitty-ai-docs`. This file stays under 300 lines
(enforced by `just agents`); completed items move to git history rather than
accumulating here.

## Current

- [x] Bootstrap repository scaffold (CTX-0187 Phase 1): docs-quality toolchain,
      docs-quality workflow, CarryCtx baseline, documentation skeleton, labels, and
      repository metadata.
- [ ] Phase 3 (separately tracked): migrate the AI-core documents from
      bitty-docs (`ai-architecture.md` and other AI runtime, provider, and context
      documents) with history preserved, rewriting links and preserving each
      document's status and `website_publish` flag.
- [ ] Later phase: wire this repository into `bitty-ai` as the `docs/`
      submodule once the implementation repository exists.

## Blocked / open

- Content migration is not part of Phase 1. Until migration lands, the tree
  intentionally contains only the documentation map and development workflow.
