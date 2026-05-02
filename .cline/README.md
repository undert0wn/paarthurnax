# `.cline/` — Cline (VS Code AI agent) governance

[Cline](https://cline.bot/) (formerly *Claude Dev*) is a VS Code extension that turns any LLM — cloud or local — into an autonomous coding agent inside the editor. It's one of the two main ways (alongside [Continue](../.continue/README.md)) to drive a local Ollama model from VS Code with file-edit + terminal + browser tool access.

This README documents Cline's **governance file conventions** and the opinionated defaults this scaffold recommends for safe use. Pair with [`../.agents/skills/`](../.agents/skills/README.md) for cross-tool capability and [`../CONVENTIONS.md`](../CONVENTIONS.md) for canonical operating rules.

## Where Cline reads governance from

In load order (later layers override earlier ones for conflicting instructions):

| Path | Scope | What it's for |
|---|---|---|
| **VS Code settings** (`Cline > *` keys) | User or workspace | API provider, model, custom instructions, auto-approve toggles. |
| **Custom Instructions** (settings UI textbox) | User-global | A persistent system prompt prepended to every Cline conversation. |
| **`.clinerules`** at workspace root | Workspace | Markdown file. Loaded into every Cline conversation in this workspace. **The main lever.** |
| **`.clinerules/`** folder at workspace root | Workspace | Newer split-file form: every `.md` file inside gets concatenated. Use when rules grow past one file. |
| **`.clineignore`** at workspace root | Workspace | gitignore-style exclude list — files Cline can't read, edit, or include in context. |

> If both `.clinerules` (file) and `.clinerules/` (folder) exist, Cline's behavior is version-dependent. **Pick one form per workspace.** The folder form is the modern recommendation for anything more than a one-pager.

## File shape — `.clinerules`

Plain Markdown. No frontmatter required. Cline pastes the contents into the system context.

```markdown
# Project rules for Cline

## Always
- Read existing files before editing.
- Show diffs before any write.
- Default scope: this workspace folder. Ask before touching anything outside.
- Honor every rule in paarthurnax/CONVENTIONS.md (link to it when in doubt).

## Never
- Run destructive shell commands (rm -rf, git push --force, DROP TABLE) without explicit confirmation.
- Commit or print secrets, tokens, or .env contents.
- Edit files matched by .clineignore.

## Tool use
- Prefer running tasks defined in .vscode/tasks.json over hand-rolling commands.
- Use the project's package manager (see lockfile) — never mix npm/pnpm/yarn.
```

## File shape — `.clinerules/` (split form)

Each `.md` file is concatenated alphabetically. Use prefixes to control order:

```
.clinerules/
├── 00-always-and-never.md
├── 10-tooling.md
├── 20-style.md
└── 30-domain-knowledge.md
```

## File shape — `.clineignore`

Same syntax as `.gitignore`. Cline will refuse to read or pass these files into model context — the headline use case is keeping secrets and noisy generated files out of the prompt.

```gitignore
# Secrets
.env
.env.*
**/secrets/
**/*.pem
**/*.key

# Noise
node_modules/
.venv/
dist/
build/
**/*.log

# Big binaries that blow context
*.zip
*.tar.gz
*.sqlite
```

## Recommended `.clinerules` starter (opinionated defaults)

Drop this into a new project verbatim and edit from there. Everything below is **deliberately conservative** — loosen only when you understand the consequence.

```markdown
# Cline rules

## Default operating mode
- This is a real codebase. Treat every change as if it will reach production.
- Default scope is this workspace folder. Reads outside need a stated reason.
  Writes outside need explicit user permission.
- Read files before modifying them. Show a diff before every write.
- Prefer minimal, focused edits. Don't refactor or "improve" code that wasn't asked about.
- When unsure, ask using a labelled choice — never guess.

## Approval discipline
- Auto-approve is OFF for: shell commands that write outside the workspace,
  any git operation that pushes, any package install, any credential operation.
- Auto-approve is OK for: read-only file ops, read-only shell commands
  (ls, cat, grep, git status, git log), running existing tasks from .vscode/tasks.json.

## Secrets and PII
- Never include .env, *.pem, *.key, or anything matched by .clineignore in your context.
- If you encounter what looks like a secret in code (long hex strings, keys, tokens),
  stop and report it — don't echo it back, don't include it in commits.

## Tooling
- Use the package manager indicated by the lockfile (package-lock.json → npm,
  pnpm-lock.yaml → pnpm, yarn.lock → yarn, uv.lock → uv).
- Run tests via the task in .vscode/tasks.json when one exists.
- For git ops, follow the safety rules in paarthurnax/.agents/skills/git/SKILL.md.

## Communication
- Be brief. Code blocks > prose. Diffs > descriptions of diffs.
- Mark suggested edits as suggestions until the user confirms.
```

## Local-model setup (Ollama path)

Cline supports Ollama natively as a provider:

1. **Settings → Cline → API Provider → Ollama**.
2. **Base URL**: `http://localhost:11434` (the Ollama default).
3. **Model**: pick from `ollama list`. For agent loops on consumer hardware, [`../ollama/qwen/qwen.md`](../ollama/qwen/qwen.md) (Qwen2.5-Coder 14B) and [`../ollama/deepseek/deepseek.md`](../ollama/deepseek/deepseek.md) (DeepSeek-Coder V2 16B Lite) are the workhorses. [`../ollama/llama/llama.md`](../ollama/llama/llama.md) (3.1 8B) and [`../ollama/mistral/mistral.md`](../ollama/mistral/mistral.md) (Nemo) also work well. See [`../ollama/README.md`](../ollama/README.md) for the full tier rationale.
4. Build a governed Modelfile from the family folder's template, then `ollama create cline-coder -f /path/to/Modelfile`. Point Cline at `cline-coder`.

⚠ **Tool-calling reliability matters more than benchmark scores for Cline.** Cline depends on the model emitting well-formed XML/JSON tool calls; a model that scores higher on MMLU but drifts on tool-call format will be a worse Cline driver. Qwen2.5-Coder, DeepSeek V3 / Coder V2, and Granite 3+ are reliable. R1-distills and Gemma 2 are not (they reason well but tool-call poorly).

## Auto-approve — the single most important Cline knob

Cline's *Auto-approve* settings decide which tool calls run without asking. This is **the** safety setting — read it before you click *Approve All*.

| Auto-approve toggle | Recommended | Why |
|---|---|---|
| Read files | ✅ On | Read-only; can't break anything. |
| Edit files | ⚠ Off (or workspace-only) | One bad edit = one bad commit. Worth the click. |
| Execute commands | ⚠ Off | Even read-only commands can leak data on a misconfigured terminal. |
| Use browser | ⚠ Off | Network egress; risk of credential exposure. |
| Use MCP servers | Per-server decision | Trust the same way you'd trust any third-party server. |

The cost of one wrong click is much higher than the cost of clicking *Approve* a few extra times.

## How this composes with the rest of `paarthurnax/`

```
.clinerules            ← workspace-specific Cline rules (this folder's recommendations)
       │
       ├── links to → ../CONVENTIONS.md          (canonical operating rules)
       ├── links to → ../.agents/skills/         (cross-tool capabilities)
       └── points at → cline-coder Ollama model  (built from ../ollama/<family>/<name>.md)
```

The Cline-specific rules in `.clinerules` should be **short** — they delegate to `CONVENTIONS.md` for any rule that should apply across every agent (Copilot, Claude, Continue, Cline). Keep `.clinerules` focused on Cline-specific guidance: approval discipline, ignore-list pointers, tool-use preferences.

## Gotchas

- **`.clinerules` is loaded every turn** — it eats context. Keep it under ~200 lines, link to bigger docs instead of inlining them.
- **Settings UI "Custom Instructions" is user-global** — fine for personal style, wrong for project-specific rules. Project rules belong in `.clinerules` (committed).
- **`.clineignore` is enforced at read-time, not parse-time** — once context is in the prompt, the model has it. Adding a path to `.clineignore` after a session started doesn't retroactively scrub.
- **MCP servers configured for Cline are independent from Copilot's MCP servers**. They have separate settings and approval state.
- **Ollama context window**: Cline can blow past `num_ctx` and silently truncate. Pick a Modelfile with `PARAMETER num_ctx 16384` or higher for real coding work. See the family folders for recommended values.

## Where to learn more

- [docs.cline.bot](https://docs.cline.bot/) — official docs.
- [github.com/cline/cline](https://github.com/cline/cline) — source + issue tracker.
- [docs.cline.bot/features/cline-rules](https://docs.cline.bot/features/cline-rules) — rules system reference.
- [docs.cline.bot/getting-started/model-selection-guide](https://docs.cline.bot/getting-started/model-selection-guide) — provider setup incl. Ollama.
- The Cline subreddit and Discord are unusually high-signal for tuning advice.
