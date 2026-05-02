# `.continue/` — Continue (VS Code AI agent) governance

[Continue](https://continue.dev/) is the open-source VS Code (and JetBrains) AI extension. It supports chat, autocomplete, edit, and agent modes, and is designed from the ground up to work well with **local models via Ollama**. It's the other main way (alongside [Cline](../.cline/README.md)) to drive a local model as a coding agent from VS Code.

This README documents Continue's **governance file conventions** and the opinionated defaults this scaffold recommends. Pair with [`../.agents/skills/`](../.agents/skills/README.md) for cross-tool capability and [`../CONVENTIONS.md`](../CONVENTIONS.md) for canonical operating rules.

## Where Continue reads governance from

In load order:

| Path | Scope | What it's for |
|---|---|---|
| **`~/.continue/config.yaml`** (or `config.json` legacy) | User-global | Default models, providers, custom commands, MCP servers. |
| **`~/.continue/assistants/*.yaml`** | User-global | Multiple named "assistants" — each with its own model + rules + tools. |
| **`<workspace>/.continue/config.yaml`** | Workspace | Workspace-specific override layered on top of user config. |
| **`<workspace>/.continue/rules/*.md`** | Workspace | Markdown rule files. Modern, file-per-rule form. |
| **Inline `rules:` in `config.yaml`** | Per-config | Rules can also live directly in YAML — fine for one or two short ones. |

> Continue migrated from `config.json` to `config.yaml` in 2024+. Both still load; `config.yaml` is the recommended modern form.

## File shape — `config.yaml`

```yaml
name: my-project
version: 1.0.0
schema: v1

models:
  - name: Local Coder
    provider: ollama
    model: qwen2.5-coder:14b
    apiBase: http://localhost:11434
    roles:
      - chat
      - edit
      - apply
    defaultCompletionOptions:
      temperature: 0.2
      contextLength: 16384

  - name: Local Autocomplete
    provider: ollama
    model: qwen2.5-coder:1.5b-base
    apiBase: http://localhost:11434
    roles:
      - autocomplete

context:
  - provider: code
  - provider: docs
  - provider: diff
  - provider: terminal
  - provider: problems
  - provider: folder
  - provider: codebase

rules:
  - "Read existing files before editing."
  - "Show diffs before any write."
  - "Default scope is this workspace. Ask before touching anything outside."
  - file: ./.continue/rules/security.md
  - file: ./.continue/rules/style.md
```

## File shape — `.continue/rules/*.md`

Each rule file is plain Markdown with optional YAML frontmatter that scopes when the rule loads:

```markdown
---
name: Python style
globs: "**/*.py"
description: Loaded only when the user is editing or asking about Python files.
---

- Use ruff for formatting and linting.
- Prefer pathlib over os.path.
- Type-hint all public functions.
- Use `pytest`, not `unittest`.
```

`globs` makes a rule **file-pattern-conditional** — Continue only injects it when the active file matches. This is the cleanest way to ship per-language rules without bloating every prompt.

## Recommended starter — `.continue/config.yaml`

```yaml
name: my-project-assistant
version: 1.0.0
schema: v1

# ----- Models -----
# Pair a strong chat/edit model with a tiny fast autocomplete model.
models:
  - name: Coder (chat / edit / agent)
    provider: ollama
    model: qwen2.5-coder:14b           # see paarthurnax/ollama/qwen/qwen.md for alternatives
    apiBase: http://localhost:11434
    roles: [chat, edit, apply]
    defaultCompletionOptions:
      temperature: 0.2
      contextLength: 16384

  - name: Autocomplete
    provider: ollama
    model: qwen2.5-coder:1.5b-base     # base, not -instruct, for FIM
    apiBase: http://localhost:11434
    roles: [autocomplete]

  - name: Embeddings
    provider: ollama
    model: nomic-embed-text
    apiBase: http://localhost:11434
    roles: [embed]

# ----- Context providers (what shows up in @-mentions and auto-context) -----
context:
  - provider: code
  - provider: docs
  - provider: diff
  - provider: terminal
  - provider: problems
  - provider: folder
  - provider: codebase
  - provider: file

# ----- Rules (always-on) -----
rules:
  - "This is a real codebase. Treat every change as production-bound."
  - "Default scope is this workspace folder. Reads outside need a stated reason; writes outside need explicit permission."
  - "Read files before modifying them. Show a diff before every write."
  - "Prefer minimal, focused edits. Don't refactor code that wasn't asked about."
  - "Use the package manager indicated by the lockfile — never mix npm/pnpm/yarn."
  - "Never include .env, secrets, or anything resembling a token in your output."
  - file: ./.continue/rules/style.md
  - file: ./.continue/rules/security.md
```

And split the longer per-language rules into `.continue/rules/`:

```
.continue/
├── config.yaml
└── rules/
    ├── security.md          ← always-on
    ├── style.md             ← always-on
    ├── python.md            ← globs: "**/*.py"
    ├── typescript.md        ← globs: "**/*.{ts,tsx}"
    └── tests.md             ← globs: "**/{tests,__tests__}/**"
```

## Local-model setup (Ollama path)

Continue is the **most opinionated about local models** of any major VS Code AI extension — it's basically the reference setup. Recommended pairings on consumer hardware (16-32 GB RAM, 8-16 GB VRAM):

| Role | Model | Why |
|---|---|---|
| Chat / edit / agent | [`qwen2.5-coder:14b`](../ollama/qwen/qwen.md) | Best open-weight tool-caller in this size class. |
| Chat / edit (alt) | [`deepseek-coder-v2:16b`](../ollama/deepseek/deepseek.md) | MoE — feels like 14B, runs like ~2.4B active. |
| Autocomplete (FIM) | `qwen2.5-coder:1.5b-base` | Tiny, fast, FIM-tuned. Use the **base** model, not -instruct. |
| Autocomplete (alt) | `starcoder2:3b` | Very fast; broad language coverage. *(StarCoder is excluded from `ollama/` because it isn't suited to agent loops; it remains a fine FIM-only autocomplete model. See [`../ollama/README.md`](../ollama/README.md).)* |
| Embeddings | `nomic-embed-text` | The standard local embedding model. |
| Reasoning (heavy lifts only) | [`deepseek-r1:14b`](../ollama/deepseek/deepseek.md) | Slow but strong; switch to it when stuck. |

The `roles` field in each model entry is what lets Continue route the right job to the right model. **Always give autocomplete its own tiny model** — sharing the chat model for autocomplete makes both feel sluggish.

## Modes Continue ships

- **Chat** — sidebar conversation. Read-only by default; can suggest edits.
- **Edit** (`Ctrl+I`) — inline edit on selected text. Diff-confirmed.
- **Autocomplete** — ghost text as you type. FIM-driven.
- **Apply** — applies a code block from chat into the active file.
- **Agent** — multi-step autonomous mode (Continue 1.0+). Behaves more like Cline; honors the same rules + a stricter approval flow.

## How this composes with the rest of `paarthurnax/`

```
.continue/config.yaml         ← Continue-specific setup (this folder's recommendations)
       │
       ├── points at → Ollama models built from ../ollama/<family>/<name>.md
       │
       └── rules/             ← Continue-specific rule files
              │
              └── delegate to → ../CONVENTIONS.md (canonical rules)
                                ../.agents/skills/ (cross-tool capabilities)
```

Keep workspace-specific Continue rules **short** — they delegate to `CONVENTIONS.md` for any rule that should also apply to Copilot, Claude, or Cline.

## Gotchas

- **`config.yaml` is loaded on save** — Continue hot-reloads. If a model goes missing after edit, check the bottom-right *Continue* status indicator for parse errors.
- **Glob-scoped rules don't fire in agent mode** the same way they do in chat — Continue's agent loop sometimes pulls all rules. Test before relying on glob exclusion for security-sensitive content.
- **`apiBase` defaults to `http://localhost:11434/v1`** for OpenAI-compatible providers; for the native Ollama provider, use `http://localhost:11434` (no `/v1`). Mixing these silently fails to connect.
- **Autocomplete using a chat-tuned model**: prepends a chat template to FIM context, ruining completions. Always use a `-base` (or explicitly FIM-trained) model for the autocomplete role.
- **Indexing on a large repo**: Continue builds an embedding index of the workspace. First-run can take a while and uses your local embedding model heavily. `Continue: Reindex Workspace` if it gets stale.
- **Two configs collision**: workspace `.continue/config.yaml` is **layered on top of** `~/.continue/config.yaml`, not a replacement. A model defined in user config still appears unless explicitly removed at workspace scope.

## Where to learn more

- [docs.continue.dev](https://docs.continue.dev/) — official docs.
- [github.com/continuedev/continue](https://github.com/continuedev/continue) — source + issue tracker.
- [docs.continue.dev/customize/deep-dives/configuration](https://docs.continue.dev/customize/deep-dives/configuration) — config.yaml reference.
- [docs.continue.dev/customize/deep-dives/rules](https://docs.continue.dev/customize/deep-dives/rules) — rules system reference.
- [docs.continue.dev/customize/model-providers/ollama](https://docs.continue.dev/customize/model-providers/ollama) — Ollama provider setup.
- [hub.continue.dev](https://hub.continue.dev/) — assistant + rule library you can copy from.
