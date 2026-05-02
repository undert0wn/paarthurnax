# Changelog

All notable changes to **Paarthurnax** are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.1.0] — Initial public release

First public release of the `Paarthurnax` governance scaffold.

> **Expectation-setting.** This is a solo hobby project at the "post it publicly so others can poke at it" stage. The folder layout, file names, placeholder vocabulary, and conventions **will shift** as the scaffold gets exercised against more real local-model setups. If you're depending on this in your own workflow, **pin a specific commit** rather than tracking `main`. There is no deprecation policy yet — breaking changes between 0.x releases should be assumed.

### Added

- **Root governance files**
  - `CONVENTIONS.md` — single source of truth for agent behavior (Tool Selection, Live State Reconciliation, File-System Boundaries, IDE-Embedded AI Agents, *Local-model loops* with the `[READ]` / `[WRITE]` / `[RUN]` action protocol and *Small-local-model pitfalls*, Sync Checklist).
  - `ARCHITECTURE.md` — folder map, layered loading diagram, *Local-model deployment* (Flavor A: model behind a client; Flavor B: custom Ollama loop), *Model-family folders* index for the twelve open-weight families, *Adding to this scaffolding* decision table, *Quick reference — what to read first when authoring*.
  - `CLAUDE.md` and `.github/copilot-instructions.md` — thin per-vendor briefings that point at `CONVENTIONS.md`.
- **Per-vendor folders**
  - `.github/` (Copilot), `.vscode/`, `.cline/`, `.continue/` — each with a `README.md` describing the convention and what's safe to commit vs. keep local.
- **Cross-tool capabilities**
  - `.agents/skills/` — eight lazy-load skills: `git`, `github-cli`, `node-npm`, `python-environments`, `powershell-essentials`, `unix-shell-essentials`, `docker-essentials`, `tool-discovery`.
- **Per-model-family folders** — five open-weight families with chat-template quirks, recommended `Modelfile`, sampling defaults under [`ollama/`](ollama/): `qwen/`, `llama/`, `mistral/`, `deepseek/`, `gemma/`. Selection criterion is empirical — each family has been driven through an actual local-model loop against the Paarthurnax contract (Gemma is included despite no native tool-calling tokens because it follows the prose-protocol action blocks reliably). Seven other considered families (Phi, SmolLM, StarCoder, Yi, GLM, Granite, Command-R) are documented in [`ollama/README.md`](ollama/README.md) with rationale for exclusion and a procedure for adding them back.
- **Project hygiene**
  - `LICENSE` — Apache License, Version 2.0.
  - `.gitignore` — covers `scratch/`, per-vendor secret-bearing configs, standard ecosystem ignores, OS noise.
  - `CHANGELOG.md`.

<!--
  Footer compare/release links intentionally omitted until the repo is published.
  Once it is, restore them in the standard Keep a Changelog shape, e.g.:
    [Unreleased]: https://github.com/<user>/<repo>/compare/v0.1.0...HEAD
    [0.1.0]: https://github.com/<user>/<repo>/releases/tag/v0.1.0
-->
