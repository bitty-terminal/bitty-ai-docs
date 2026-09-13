# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to
[Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- Repository bootstrap (CTX-0187 Phase 1): docs-quality toolchain (justfile,
  `.github/scripts/check-docs.mjs`, docs-quality workflow), CarryCtx baseline,
  and an empty documentation skeleton.
- AI-core documentation corpus migrated from bitty-docs
  (`docs/projects/bitty/specifications/` at `c664214`) with history preserved
  (CTX-0001): AI Architecture, IPC and Agent RFC, and the Browser and Agent
  Panel Integration Pre-Study. Cross-repository links were rewritten to
  absolute URLs into `bitty-docs`, `bitty-terminal-docs`, and
  `bitty-plugins-docs`.

### Changed

- Docs-quality metadata validation now covers the root content trees in
  addition to `docs/`, and SVG validation scans every tracked or new SVG.
