# `.vscode/` — workspace configuration for VS Code (and forks)

A `.vscode/` folder at the root of a workspace is how you ship **opinionated, repo-scoped IDE setup** to anyone who clones the repo. VS Code, Cursor, Windsurf, and other VS Code forks all read it. This README documents the files that matter, the ones agents care about, and the conventions that keep them safe to commit.

## Why ship a `.vscode/` folder at all?

Without it, every contributor starts from a blank slate — wrong Python interpreter, wrong test runner, missing recommended extensions, no debug config, no agent rules. With it, a fresh clone is one *Trust Workspace* click away from a working dev loop.

The folder is **always committed** unless individual files explicitly say otherwise (see *Per-file commit policy* below).

## File map

| File | Purpose | Commit? |
|---|---|---|
| `settings.json` | Per-workspace overrides for any VS Code setting. | ✅ Yes (curated) |
| `extensions.json` | Recommended + unwanted extensions for this repo. | ✅ Yes |
| `tasks.json` | Build / test / lint tasks runnable from the Command Palette. | ✅ Yes |
| `launch.json` | Debugger configurations. | ✅ Yes |
| `mcp.json` | Model Context Protocol server definitions for Copilot Chat (1.99+). | ⚠ See *MCP* section — usually committed but **never with secrets**. |
| `*.code-snippets` | Workspace-scoped snippets. | ✅ Yes |
| `keybindings.json` | ⚠ **Not actually a workspace file** — VS Code only honors user-level keybindings. Don't put one here. | n/a |
| `.gitignore` (inside `.vscode/`) | Local opt-outs (e.g. `settings.json` if you want it user-only). | ✅ Yes |

A typical `.gitignore` rule at the **repo root** keeps the noise out:
```gitignore
# Keep .vscode/ but ignore personal scratch
.vscode/*.log
.vscode/chrome-profile/
```

## Per-file commit policy

| Scenario | Decision |
|---|---|
| Setting helps everyone (`editor.formatOnSave`, `python.testing.pytestEnabled`) | Commit. |
| Setting is personal taste (`workbench.colorTheme`, `editor.fontSize`) | Don't commit. Put it in **User settings**, not workspace. |
| Setting contains a token, key, or absolute path with your username | Never commit. If unavoidable, use `${env:VAR_NAME}` or `${workspaceFolder}` placeholders. |
| Extension is needed for the repo's tooling to work | Commit to `extensions.json` `recommendations`. |
| Extension actively breaks the repo's tooling (linter conflict, formatter conflict) | Commit to `extensions.json` `unwantedRecommendations`. |

## `settings.json`

Workspace-level settings override User settings, which override Defaults. Common shape for a polyglot repo:

```jsonc
{
  // Editor — opinionated but harmless
  "editor.formatOnSave": true,
  "editor.rulers": [80, 120],
  "files.trimTrailingWhitespace": true,
  "files.insertFinalNewline": true,
  "files.eol": "\n",

  // Search — skip noise
  "search.exclude": {
    "**/node_modules": true,
    "**/.venv": true,
    "**/dist": true,
    "**/.git": true
  },
  "files.watcherExclude": {
    "**/node_modules/**": true,
    "**/.venv/**": true
  },

  // Python (if relevant)
  "python.defaultInterpreterPath": "${workspaceFolder}/.venv/bin/python",
  "python.testing.pytestEnabled": true,
  "[python]": {
    "editor.defaultFormatter": "charliermarsh.ruff",
    "editor.codeActionsOnSave": { "source.organizeImports": "explicit" }
  },

  // Node/TS (if relevant)
  "typescript.tsdk": "node_modules/typescript/lib",
  "[typescript]": { "editor.defaultFormatter": "esbenp.prettier-vscode" },
  "[typescriptreact]": { "editor.defaultFormatter": "esbenp.prettier-vscode" },

  // Copilot Chat — workspace-scoped agent behavior
  "chat.tools.autoApprove": false,        // require explicit approval per tool
  "github.copilot.chat.codeGeneration.useInstructionFiles": true
}
```

**Path placeholders** that work inside any `.vscode/*.json`:
- `${workspaceFolder}` — repo root
- `${workspaceFolderBasename}` — just the folder name
- `${file}`, `${fileBasename}`, `${fileDirname}` — current editor file
- `${env:NAME}` — environment variable
- `${userHome}` — user home dir
- `${pathSeparator}` — `/` on Unix, `\` on Windows

## `extensions.json`

```jsonc
{
  "recommendations": [
    "github.copilot",
    "github.copilot-chat",
    "ms-python.python",
    "charliermarsh.ruff",
    "dbaeumer.vscode-eslint",
    "esbenp.prettier-vscode"
  ],
  "unwantedRecommendations": [
    "ms-python.black-formatter"     // example: we use ruff format, not black
  ]
}
```

When someone opens the workspace, VS Code shows a one-time *"This workspace has extension recommendations"* toast. They click *Install All* and they're set.

## `tasks.json`

Tasks turn shell commands into Command Palette entries (`Ctrl+Shift+P` → *Tasks: Run Task*) and let agents run them via `run_vscode_task`.

```jsonc
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "test",
      "type": "shell",
      "command": "uv run pytest",
      "group": { "kind": "test", "isDefault": true },
      "problemMatcher": []
    },
    {
      "label": "lint",
      "type": "shell",
      "command": "uv run ruff check .",
      "problemMatcher": "$ruff"
    },
    {
      "label": "build",
      "type": "shell",
      "command": "npm run build",
      "group": { "kind": "build", "isDefault": true }
    }
  ]
}
```

`Ctrl+Shift+B` runs the default build task. `Ctrl+Shift+P` → *Tasks: Run Test Task* runs the default test.

## `launch.json`

Debugger configs. `F5` runs the first/default one.

```jsonc
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Python: Current File",
      "type": "debugpy",
      "request": "launch",
      "program": "${file}",
      "console": "integratedTerminal"
    },
    {
      "name": "Node: Current File",
      "type": "node",
      "request": "launch",
      "program": "${file}",
      "skipFiles": ["<node_internals>/**"]
    },
    {
      "name": "Pytest: Current File",
      "type": "debugpy",
      "request": "launch",
      "module": "pytest",
      "args": ["${file}", "-v"],
      "console": "integratedTerminal"
    }
  ]
}
```

## `mcp.json` — Model Context Protocol servers

If your workspace uses MCP servers (custom tools the agent can call), declare them here. Available in VS Code 1.99+.

```jsonc
{
  "servers": {
    "my-local-server": {
      "type": "stdio",
      "command": "node",
      "args": ["${workspaceFolder}/scripts/mcp-server.js"]
    },
    "remote-tool": {
      "type": "http",
      "url": "https://mcp.example.com/sse",
      "headers": {
        "Authorization": "Bearer ${env:MCP_TOKEN}"      // pulled from env at runtime
      }
    }
  }
}
```

⚠ **Never commit secrets.** Use `${env:VAR_NAME}` placeholders. If you must keep a private server entry, either:
1. Add the file to `.gitignore` (`/.vscode/mcp.json`) and ship a `mcp.example.json` instead, or
2. `git update-index --skip-worktree .vscode/mcp.json` so local edits don't show up in `git status`.

## How agents use this folder

Once `.vscode/` exists and the workspace is trusted, an agent (Copilot Chat, Claude Code via the VS Code extension, Cursor's agent, etc.) can:

- **Read settings** to honor formatter/linter choices.
- **Run tasks** via `run_vscode_task` instead of guessing the right shell command.
- **Launch debug sessions** via `workbench.action.debug.start`.
- **List/install extensions** (with your approval) — see the [`tool-discovery`](../.agents/skills/tool-discovery/SKILL.md) skill for CLI equivalents.
- **Call MCP tools** declared in `mcp.json`.

Without `.vscode/`, the agent falls back to scanning the repo and guessing — slower and more error-prone.

## Workspace trust — the gate

VS Code's **Workspace Trust** model decides whether the contents of `.vscode/` (and arbitrary task commands, debug configs, automatic extension activations) are allowed to run. First-open shows a *Do you trust the authors?* dialog.

- **Trusted**: tasks run, debug works, extensions activate normally, agents can run shell commands (with per-tool approval still gating destructive ones).
- **Restricted (untrusted)**: many extensions disable themselves; tasks won't auto-run; agents can still read but not execute.

Manage at: `File → Manage Workspace Trust`, or Command Palette → *Workspaces: Manage Workspace Trust*. Decisions persist per folder.

## Setup checklist for a new repo

1. Create `.vscode/` at repo root.
2. Add `settings.json` with the formatter/linter choices the project assumes.
3. Add `extensions.json` listing the extensions someone needs to do real work.
4. Add `tasks.json` for the 2–4 commands you run constantly (test, lint, build, dev server).
5. Add `launch.json` if there's a non-obvious way to debug the project.
6. Add `mcp.json` if the project ships MCP servers — and confirm no secrets leak.
7. Open the repo fresh, click *Trust*, accept extension recommendations, run a task with `Ctrl+Shift+P` → *Tasks: Run Task*. If anything fails, fix it before you commit.

## Where to learn more

- [code.visualstudio.com/docs/getstarted/settings](https://code.visualstudio.com/docs/getstarted/settings) — settings precedence + scopes.
- [code.visualstudio.com/docs/editor/tasks](https://code.visualstudio.com/docs/editor/tasks) — full tasks reference.
- [code.visualstudio.com/docs/editor/debugging](https://code.visualstudio.com/docs/editor/debugging) — launch.json reference.
- [code.visualstudio.com/docs/editor/workspace-trust](https://code.visualstudio.com/docs/editor/workspace-trust) — trust model in depth.
- [code.visualstudio.com/docs/copilot/chat/mcp-servers](https://code.visualstudio.com/docs/copilot/chat/mcp-servers) — MCP server setup for Copilot Chat.
- [code.visualstudio.com/docs/editor/variables-reference](https://code.visualstudio.com/docs/editor/variables-reference) — every `${variable}` placeholder.
- Inside VS Code: `Ctrl+Shift+P` → *Preferences: Open Default Settings (JSON)* shows every setting and its default — searchable and authoritative.
