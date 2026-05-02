# paarthurnax/ — Agent & Model Governance Index

This file indexes how **every supported agent and model family** in this workspace is wired to the governance scaffold. The single source of truth for behavior is always [`CONVENTIONS.md`](CONVENTIONS.md) → *Prime Directives*. Every per-agent briefing **must** reference it.

**Core rule for all agents/models (add this to any new ones you introduce):**
- Read `CONVENTIONS.md` at session start.
- Obey the Prime Directives (including the new edit guardrails in `GROK.md` for this instance).
- Default to text-only review/help mode unless the user explicitly requests creation, implementation, or edits.
- Lazy-load skills from `.agents/skills/` and `.github/skills/`.
- Honor `.gitignore`, `.clineignore`, etc.

## Agents

| Agent | Governing File | How it loads governance | Notes |
|-------|----------------|-------------------------|-------|
| **GitHub Copilot** (VS Code) | `.github/copilot-instructions.md` | Auto-loaded by Copilot when workspace root contains it (or symlinked). | Mirrors CLAUDE.md. Uses VS Code tools (`vscode_askQuestions`, inline diffs). |
| **Claude Code** | `CLAUDE.md` (workspace root) | Auto-loaded by Claude Code from workspace root. | Points to `CONVENTIONS.md`. Uses its own option-picker and diff affordances. |
| **Grok** (this instance, via Copilot) | `GROK.md` (workspace root) | Explicitly read at start of every session (included in context or priming). | Strengthened "no-edit" guardrails for review/help tasks. Looks at editorContext, conversation-summary, tools list, terminal state. |
| **Cline** | `.cline/README.md` + `.clinerules` (or `.clinerules/`) | `.clinerules` (or folder) is injected into every conversation. | See `.cline/README.md` for recommended rules that reference `CONVENTIONS.md`. Use `.clineignore`. |
| **Continue** | `.continue/README.md` + `.continue/config.yaml` + `rules/*.md` | Rules in `config.yaml` or `rules/` folder; workspace `config.yaml` layers on top. | See `.continue/README.md`. Local Ollama models configured here with rules referencing `CONVENTIONS.md`. |

## Models (Ollama)

| Family | File | Purpose | How it enforces governance |
|--------|------|---------|----------------------------|
| Qwen | `ollama/qwen/qwen.md` | Modelfile template, chat template quirks, sampling defaults | Injected into system prompt or Modelfile `SYSTEM` / `PARAMETER` blocks. |
| Llama | `ollama/llama/llama.md` | Same | Same. |
| Mistral | `ollama/mistral/mistral.md` | Same | Same. |
| DeepSeek | `ollama/deepseek/deepseek.md` | Same | Same. |
| Gemma | `ollama/gemma/gemma.md` | Same | Same. |

See [`ollama/README.md`](ollama/README.md) for why these five families, the tool-calling floor, and how to add a new family (create `ollama/newfamily/newfamily.md` with the same 8-section SKILL-like pattern).

## What to do when changing agents or models

1. **New agent**: Create a dot-folder (e.g. `.cursor/`, `.windsurf/`) with a `README.md` that documents its config files and includes the rule "always load/reference `CONVENTIONS.md` Prime Directives + this project's skills".
2. **New model family**: Add `ollama/<family>/<family>.md` following the existing pattern, then update `ollama/README.md` (but only if you explicitly request the edit).
3. **Verify**: After switching, ask the agent "read CONVENTIONS.md and GROK.md and confirm you are following the Prime Directives and edit guardrails".
4. **MCP**: Keep `.mcp.json` and `.vscode/mcp.json` in sync with real servers (never commit secrets).
5. **Memory**: Use the `memory` tool to record agent-specific preferences in `/memories/repo/` or `/memories/`.

**All agents must treat `CONVENTIONS.md` as immutable canonical rules.** `GROK.md`, `CLAUDE.md`, and `copilot-instructions.md` are thin wrappers that map the directives to each platform's affordances and add platform-specific guardrails (e.g. no unsolicited edits for Grok).

This index ensures consistency when you switch between Copilot/Grok, Claude, Cline, Continue, or local Ollama models.

---

Created to answer "what else should I do" for multi-agent/model governance without modifying any existing files. Place new agent folders at root (`.newagent/README.md`) following the established pattern.