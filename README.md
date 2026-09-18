# bitty-ai-docs

`bitty-ai-docs` is the canonical documentation repository for `bitty-ai`, the
independent AI-core sub-platform of Bitty. It owns the English-language
architecture, specification, provider, context, and reference documentation for
the AI core.

**Current state: AI-core corpus migrated (CTX-0001).** The AI-core documents
were imported from `bitty-docs` (`docs/projects/bitty/specifications/` at
`c664214`) with history preserved. Canonical content lives in root topic trees;
this repository's process documents stay under `docs/`. Every cross-repository
link uses an absolute URL.

## Scope

This repository owns:

- AI-core architecture and runtime lifecycle.
- Provider integration: model and tool providers, capability boundaries, and
  configuration contracts.
- Context management: assembly, provenance, and budget semantics.
- AI-specific trust boundaries and failure semantics, coordinated with the
  canonical security corpus in `bitty-docs`.
- Reference material derived from verified implementation evidence.

This repository does not own:

- Shared cross-project governance — decisions, the security corpus, findings,
  reviews, handoff, roadmap, releases, and project state — which stays in
  [bitty-docs](https://github.com/bitty-terminal/bitty-docs).
- Terminal-platform documentation, which lives in
  [bitty-terminal-docs](https://github.com/bitty-terminal/bitty-terminal-docs).
- Plugin-ecosystem documentation, which lives in
  [bitty-plugins-docs](https://github.com/bitty-terminal/bitty-plugins-docs).

Cross-project contracts and registers are linked, never copied.

Canonical AI-core content is organized into root topic trees —
`architecture/`, `context/`, `providers/`, `agent/`, `persistence/`,
`interfaces/`, `integration/`, `product/`, and `specifications/` — each with a
route-only index. The
[documentation map](https://github.com/bitty-terminal/bitty-ai-docs/blob/main/docs/README.md)
records every tree and entry point plus the one remaining planned tree
(`reference/`), which lands only when reviewed implementation evidence exists.

## Composition

The repository is intended to be mounted at `bitty-ai/docs` as a Git submodule
once the `bitty-ai` implementation repository exists, so documentation
version-matches the implementation it describes. The standalone repository is
fully self-contained and passes its own gates. AI-core content lives in root
topic trees; this repository's process documents stay under `docs/`.

## Structure

| Path                             | Purpose                                                      |
| -------------------------------- | ------------------------------------------------------------ |
| `docs/README.md`                 | Documentation map and authority rules for this repository.   |
| `docs/development/`              | Contributor workflow and the normative documentation policy. |
| `docs/sources/`                  | Research coverage ledger and provenance registers.           |
| `<topic>/`                       | Canonical AI-core documents in root topic trees.             |
| `TODO.md`                        | Work register for this repository.                           |
| `AGENTS.md`                      | Agent scope, CarryCtx workflow, and local gate rules.        |
| `.github/scripts/check-docs.mjs` | Links, metadata, language, budgets, and hygiene checks.      |
| `justfile`                       | Pinned docs-quality commands; `just check` is the gate.      |

## Authority and status

- The
  [documentation workflow](https://github.com/bitty-terminal/bitty-ai-docs/blob/main/docs/development/documentation-workflow.md)
  is normative for authoring, metadata, status, and review.
- Every canonical document (root topic trees and `docs/`) carries the flat
  frontmatter schema and declares its own status; design intention must never
  read as implemented behavior.
- When statements conflict, the canonical bitty-docs security corpus takes
  precedence. Implementation claims require evidence from the owning code
  repository.

## Local checks

```sh
just fmt           # format supported files
just check         # full local gate pipeline (same logical gates as CI)
```

`just check` verifies Prettier formatting, markdownlint, repository-local links,
frontmatter metadata, English-only content, file budgets, hygiene, SVG
well-formedness, and GitHub Actions syntax. JavaScript tooling runs through Bun
only; `npm`, `npx`, and `yarn` are not used here.

## Contributing

Read [CONTRIBUTING.md](https://github.com/bitty-terminal/bitty-ai-docs/blob/main/CONTRIBUTING.md)
and [AGENTS.md](https://github.com/bitty-terminal/bitty-ai-docs/blob/main/AGENTS.md)
before editing. The normal lifecycle is Issue, scoped CarryCtx task,
branch/worktree, commit, pull request, independent review plus CI, merge, then
task closure and a final checkpoint.

## Related repositories

| Repository                                                                   | Role                                           |
| ---------------------------------------------------------------------------- | ---------------------------------------------- |
| [bitty](https://github.com/bitty-terminal/bitty)                             | Terminal platform implementation.              |
| [bitty-docs](https://github.com/bitty-terminal/bitty-docs)                   | Shared cross-project governance and registers. |
| [bitty-terminal-docs](https://github.com/bitty-terminal/bitty-terminal-docs) | Terminal-platform documentation.               |
| [bitty-plugins-docs](https://github.com/bitty-terminal/bitty-plugins-docs)   | Plugin-ecosystem documentation.                |

## License

MIT — see [LICENSE](https://github.com/bitty-terminal/bitty-ai-docs/blob/main/LICENSE).
