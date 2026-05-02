# Paarthurnax

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE)
[![Status: alpha](https://img.shields.io/badge/status-alpha-orange.svg)](CHANGELOG.md)

> *Codename: **Paarthurnax**. A powerful ancient being (not unlike AI) who chooses to help mortals, teach them, and guide them. Wise, patient, and never gatekeeping.*

A drop-in governance scaffold that (hopefully) gives local open-source LLMs the behavioral structure they need to be useful coding partners inside an IDE.

> **Legal Disclaimer:** This project is an independent, unofficial project inspired by The Elder Scrolls V: Skyrim. It is not affiliated with, endorsed by, or connected to Bethesda Softworks, Zenimax Media, Microsoft, or any official Skyrim/Dragon assets. This project uses the character name "Paarthurnax" purely for thematic inspiration and does not include any official Skyrim imagery, logos, dragon fonts, or direct quotes.

*In plain English: make your local LLM useful in VS Code.*

> **About this project.** `Paarthurnax` is a solo hobby project, one person learning in the open. The author runs Ollama on a desktop and is using this scaffold to figure out how to make small local models behave well inside an IDE. It might grow into something bigger; for now it's a sketchbook with a license. **Expect churn**: folder layout, file names, and conventions will shift as it gets exercised. If you're depending on any of it, pin a commit. Issues and forks are welcome but there's no SLA on responses, and no contribution policy beyond "be decent." The Apache-2.0 license is here for legal clarity, NOT to imply a team.

---

## The problem

Local LLMs (DeepSeek, Qwen, Llama, Mistral, Gemma) are excellent in a CLI chat. You ask a question, get an answer. But the moment you point one at a real codebase inside VS Code (or any IDE) and ask it to *build something*, it falls apart:

- It doesn't know which files are safe to read or write.
- It invents file paths, hallucinates dependencies, edits the wrong file.
- It forgets your project's conventions between turns.
- It can't tell scratch work from production code.
- It re-asks the same yes/no question fifteen times.

Cloud agents (GitHub Copilot, Claude Code, Cursor) solved this with tens of thousands of words of internal system prompts, tooling, and behavioral rules baked into the product. Local models don't get that for free. **Paarthurnax is that missing layer, written in the open and tuned for small local models.**

---

## What Paarthurnax actually is

A folder you copy into your project. Inside it:

- **One source of truth** for how any AI agent (cloud or local) should behave in your workspace — [`CONVENTIONS.md`](CONVENTIONS.md).
- **Per-vendor briefings** that point every supported agent at the same rules: GitHub Copilot, Claude Code, Cline, Continue, and any local model running through Ollama.
- **Per-model-family folders** under [`ollama/`](ollama/) ([`qwen/`](ollama/qwen/qwen.md), [`llama/`](ollama/llama/llama.md), [`mistral/`](ollama/mistral/mistral.md), [`deepseek/`](ollama/deepseek/deepseek.md), [`gemma/`](ollama/gemma/gemma.md)) with the chat-template quirks, recommended `Modelfile`, and sampling defaults each family needs to behave well. See [`ollama/README.md`](ollama/README.md) for why the cut is just these five and how to add more.
- **Lazy-load skills** for cross-cutting tools (git, docker, python, powershell, …) so the always-on context stays small enough for a 14B model to think clearly.

Paarthurnax is **not a VS Code extension** (and not tied to VS Code at all). It's the *brain*. An extension or custom Ollama loop is the body — and you can build either, or use the existing in-IDE agents (Cline, Continue, Copilot, Claude Code) that already know how to load this kind of scaffold.

---

## Who this is for

- Hobbyists and beginners who want to build real software with a local model and don't want to pay for cloud APIs.
- Privacy-conscious devs who can't or won't send code to a third party.
- Anyone running Ollama who's been frustrated that Qwen / DeepSeek / Llama feel powerful in chat but useless when pointed at a project.

---

## Quick start

1. **[Install Ollama](https://ollama.com/)** and pull a few strong coding models:

   ```bash
   ollama pull qwen2.5-coder:14b
   # or
   ollama pull llama3.1:8b
   # or
   ollama pull deepseek-coder-v2:16b
   ```

2. **Set up your IDE**. I started with **VS Code** as the primary environment. Install some extensions and tools:

   **Recommended VS Code Extensions**

   | Extension              | Purpose                              |
   |------------------------|--------------------------------------|
   | GitHub Copilot         | Primary AI coding assistant          |
   | Cline                  | Powerful local-first agent           |
   | Continue               | Open-source AI coding assistant      |
   | GitLens                | Enhanced Git experience & history    |
   | Docker                 | Container & MCP server support       |
   | Markdown All in One    | Better documentation editing         |

   **Recommended Local Tools**

   - **[Git](https://git-scm.com/)** — version control and submodules
   - **[Node.js](https://nodejs.org/)** — required for many MCP servers (via npx)
   - **[Docker](https://www.docker.com/)** — containers, volumes & MCP servers (Windows/macOS/Ubuntu)

3. **Add Paarthurnax** to your project from the official repository at https://github.com/undert0wn/paarthurnax:

   - **New project (recommended)**: Clone the repo (replace `your-project` with your own folder name)
     ```bash
     git clone https://github.com/undert0wn/paarthurnax.git your-project
     cd your-project
     ```

   - **Existing project**: Add as a git submodule
     ```bash
     git submodule add https://github.com/undert0wn/paarthurnax.git paarthurnax
     git submodule update --init --recursive
     ```

   - Or download the latest zip from the GitHub repository and extract the `paarthurnax` folder.

4. **Point your agent at it:**
   - **VS Code + Cline** → see [`.cline/README.md`](.cline/README.md).
   - **VS Code + Continue** → see [`.continue/README.md`](.continue/README.md).
   - **GitHub Copilot** → already auto-loads [`.github/copilot-instructions.md`](.github/copilot-instructions.md).
   - **Claude Code** → already auto-loads [`CLAUDE.md`](CLAUDE.md).
   - **Custom Ollama loop** → inject [`CLAUDE.md`](CLAUDE.md) as the system prompt and implement the action protocol from [`CONVENTIONS.md`](CONVENTIONS.md) → *Local-model loops*; see [`ARCHITECTURE.md`](ARCHITECTURE.md) → *Local-model deployment*.

5. Ask your model to build something. It now knows where it is, what it can touch, and how to behave.

> **Note**: The `paarthurnax/` folder (or submodule) should be committed to your repository so the governance rules travel with your project.

> When an external system has to be named in any Paarthurnax file, the scaffold uses the placeholder `<REMOTE-AI>` (and `<REMOTE-CLI>` for a direct CLI to the same system).

---

## Start here

If you're new to this folder, read in this order:

1. **[`ARCHITECTURE.md`](ARCHITECTURE.md)** — the at-a-glance map: what every folder is for, what loads when, the layered diagram showing how Copilot / Claude / local models all see the same rules, and the *Local-model deployment* section covering Flavor A (model behind a client) vs Flavor B (custom Ollama loop).
2. **[`CONVENTIONS.md`](CONVENTIONS.md)** — the single source of truth for agent behavior. Every other file in `paarthurnax/` either points to it or implements one of its rules.

That's the orientation pass. Everything below is reference.

---

## Root files

| File | Purpose | Loaded by |
|---|---|---|
| [`README.md`](README.md) | This file. Entry point and index. | Humans. |
| [`ARCHITECTURE.md`](ARCHITECTURE.md) | The folder map: at-a-glance tree, layered loading diagram, per-folder tables, design principles, *Adding to this scaffolding* decision table. | Humans + agents (when asked "where does X go?"). |
| [`CONVENTIONS.md`](CONVENTIONS.md) | Canonical operating rules — Tool Selection rubric, Live State Reconciliation, File-System Boundaries, IDE-Embedded AI Agents rules, Sync Checklist. Single source of truth. | Referenced by every per-vendor briefing. |
| [`CLAUDE.md`](CLAUDE.md) | Claude's auto-loaded session briefing. Mirrors `.github/copilot-instructions.md`. Both are thin — they enforce the rules; `CONVENTIONS.md` defines them. | Claude Code (auto-loads from workspace root). |

---

## Folders at a glance

> **Documentation-only in the published scaffold.** The IDE/extension dot-folders below ([`.vscode/`](.vscode/), [`.cline/`](.cline/), [`.continue/`](.continue/)) ship with a single `README.md` each — no working `settings.json`, `.clinerules`, `config.yaml`, or other live config. They document the **shape** an adopter should fill in for their own repo (which files to create, what each key means, what's safe to commit). Treat them as templates to copy or reference, not turn-key configs. The same applies to [`.mcp.json`](.mcp.json) and [`.vscode/mcp.json`](.vscode/mcp.json), which ship with dummy Skyrim placeholder server entries to demonstrate the schema.

| Folder | Purpose |
|---|---|
| [`.github/`](.github/) | GitHub Copilot's auto-load convention. `copilot-instructions.md` is Copilot's session briefing; `skills/` and `prompts/` document the lazy-load skill and slash-command conventions. |
| [`.vscode/`](.vscode/) | *Documentation-only.* VS Code workspace-config conventions: `settings.json` / `tasks.json` / `extensions.json` / `mcp.json` reference, what's safe to commit, agent integration. |
| [`.cline/`](.cline/) | *Documentation-only.* Cline (in-editor AI agent) governance: `.clinerules` / `.clineignore` shape, opinionated starter rules, local-model setup. |
| [`.continue/`](.continue/) | *Documentation-only.* Continue (in-editor AI agent) governance: `config.yaml` + `rules/` shape, opinionated starter, three-model local setup (chat + autocomplete + embed). |
| [`.agents/`](.agents/) | Cross-tool, model-independent agent capabilities. `.agents/skills/<name>/SKILL.md` is the lazy-load convention used by every agent that respects it. Currently ships eight generic skills (git, github-cli, node-npm, python-environments, powershell-essentials, unix-shell-essentials, docker-essentials, tool-discovery). |
| [`ollama/`](ollama/) | Open-weight model-family docs, one subfolder per family (`qwen/`, `llama/`, `mistral/`, `deepseek/`, `gemma/`) plus an [`ollama/README.md`](ollama/README.md) explaining the tool-calling floor, the five kept families, the seven intentionally-not-included families, and how to add more. Each `<family>/<family>.md` covers chat-template quirks, recommended `Modelfile`, and sampling defaults. Index: [`ARCHITECTURE.md`](ARCHITECTURE.md) → *Model-family folders*. |

---

## The two governing files (per-vendor entry points)

The same content, mirrored into the file each vendor auto-loads:

- **GitHub Copilot** → [`.github/copilot-instructions.md`](.github/copilot-instructions.md)
- **Claude Code** → [`CLAUDE.md`](CLAUDE.md)

Both are intentionally thin. Their job is to (a) enforce a small set of *Always-On Behaviors* every turn and (b) point the agent at [`CONVENTIONS.md`](CONVENTIONS.md) for everything else. Keeping them in sync is mandatory — see *Sync Checklist* in [`CONVENTIONS.md`](CONVENTIONS.md).

When new vendors are added (Cursor's `.cursor/rules/`, the cross-tool `AGENTS.md` standard, Gemini CLI's `GEMINI.md`, …), they get the same treatment: a thin briefing that points at `CONVENTIONS.md`. The rules don't fork.

---

## Design principles

1. **One source of truth.** `CONVENTIONS.md` defines every rule. Every vendor briefing references it instead of restating it.
2. **Vendor-neutral root, vendor-specific edges.** The root files are vendor-agnostic. Vendor specifics live in the dot-folder for that vendor (`.github/`, `.cline/`, `.continue/`) and model-family specifics live under [`ollama/`](ollama/).
3. **Lazy-load by default.** Always-on context stays small. Skills, prompts, and family files load on demand. Critical for local models where context budget is tight.
4. **Clickable choices over prose prompts.** Every yes/no or A/B/C is rendered as labelled buttons in the chat UI (`vscode_askQuestions` or equivalent). A mouse click beats typing "yes" every time.
5. **Workspace-scoped by default.** Reads outside the workspace need a stated reason; writes outside need explicit permission. Subagents inherit. Defined in [`CONVENTIONS.md`](CONVENTIONS.md) → *File-System Boundaries*.
6. **Per-instance scratch is gitignored, never committed.** Anything tied to a specific `<REMOTE-AI>` instance (IDs, URLs, exported resources) lives under `paarthurnax/scratch/<INSTANCE-ID>/` and stays local. The committed `paarthurnax/` tree stays generic.
7. **Importable, not bespoke.** Any platform-flavored content brought into `paarthurnax/` should be genericized first — keep the structural lessons, drop the identifiers.

---

## Adding to this scaffolding

The full decision table lives in [`ARCHITECTURE.md`](ARCHITECTURE.md) → *Adding to this scaffolding*. Quick version:

| You're adding… | Where it goes |
|---|---|
| A new operating rule that should apply across agents | [`CONVENTIONS.md`](CONVENTIONS.md). |
| A rule specific to in-editor AI agents | [`CONVENTIONS.md`](CONVENTIONS.md) → *IDE-Embedded AI Agents* section. |
| A new cross-tool skill | `.agents/skills/<name>/SKILL.md` + update [`.agents/skills/README.md`](.agents/skills/README.md). |
| A new model family | `ollama/<family>/<family>.md` + update the *Model-family folders* table in [`ARCHITECTURE.md`](ARCHITECTURE.md). |
| A new IDE-AI extension (Cursor, Roo Code, Cody, Windsurf, …) | New `.<extension>/README.md` mirroring [`.cline/README.md`](.cline/README.md) / [`.continue/README.md`](.continue/README.md), plus a row in the IDE-Embedded AI Agents table in [`CONVENTIONS.md`](CONVENTIONS.md). |

---

## Future work

Things the author knows are missing or rough, in rough priority order. Not promises — just visibility into what would be valuable next.

- **A working example.** `examples/hello-paarthurnax/` — a minimal pre-wired project (one source file + a real `.continue/config.yaml` or equivalent) that a first-time user can open in VS Code and see Paarthurnax "do something" without reading three READMEs first. Will be added once the author has actually run Paarthurnax against a real Ollama setup; building it now would just bake in guesses.
- **A smoke-test prompt in the README** — a single ask-the-model question whose answer confirms the briefing loaded correctly. Cheap, will land alongside the example folder.
- **A 30-second screencast or GIF.** Lowest priority — high maintenance burden as IDE/extension UIs change.
- **Validation of the per-vendor briefings against actual local models.** The current contents are written from theory + experience with cloud agents; need a real Qwen / DeepSeek / Llama session to confirm tool-calling reliability and prompt-following.

---

## License & provenance

Licensed under the **[Apache License, Version 2.0](LICENSE)**. See [`NOTICE`](NOTICE) for attribution.

In plain English: you can use, modify, fork, redistribute, and **sell** Paarthurnax (or anything you build on top of it) freely, including commercially. You just need to keep the copyright notice and `LICENSE` file in your copies, mark any files you modify, and preserve the `NOTICE` file's attribution. The software comes with **no warranty** and the authors are **not liable** for any damages arising from its use. The Apache-2.0 patent grant means contributors can't sue you over patents covering code they contributed; the patent-retaliation clause means anyone who sues the project over patents loses their license to use it.

Paarthurnax is a vendor-neutral restatement of governance patterns. Every committed file is generic — no platform-specific identifiers, URLs, or vocabulary — so the folder is safe to copy into any project, public or private.

---

## Contributing

Issues, PRs, and ideas are welcome — see [`CONTRIBUTING.md`](CONTRIBUTING.md). For security issues, please use private reporting per [`SECURITY.md`](SECURITY.md). Release notes live in [`CHANGELOG.md`](CHANGELOG.md).
