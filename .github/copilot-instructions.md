# AI Workspace — Session Briefing (Copilot entry-point)

> This file governs **GitHub Copilot** in the `paarthurnax/` folder. The Claude-side mirror is [`../CLAUDE.md`](../CLAUDE.md). The two are kept in sync — material differences are limited to reading order in the initialization sequence and which file is "this file" vs "the mirror".

> **Where Copilot actually loads this from — two scenarios.** GitHub Copilot in VS Code auto-loads `.github/copilot-instructions.md` *relative to the workspace root*. So:
> - **If your VS Code workspace root is `paarthurnax/`**, this file (`paarthurnax/.github/copilot-instructions.md`) is **already in the right place** and Copilot picks it up automatically. No action needed.
> - **If your VS Code workspace root is the repo root**, Copilot looks at the repo's `.github/copilot-instructions.md` instead. To make Copilot use *this* file, either symlink (`New-Item -ItemType SymbolicLink -Path .github\copilot-instructions.md -Target ..\paarthurnax\.github\copilot-instructions.md` — needs admin or developer mode on Windows) or run a small sync script that copies `paarthurnax/.github/copilot-instructions.md` to the root `.github/copilot-instructions.md` on commit.
>
> The mirror file [`../CLAUDE.md`](../CLAUDE.md) follows the analogous rule for Claude Code, which auto-loads `CLAUDE.md` from the workspace root — so when your workspace root is `paarthurnax/`, *both* vendors auto-load their conventional file. **Different vendors, different conventions, sibling paths under `paarthurnax/`.**

## Governing Reference

**The single governing reference file for GitHub Copilot is `paarthurnax/.github/copilot-instructions.md` (this file, auto-loaded at session start when the workspace root is `paarthurnax/`, or symlinked/copied to the repo root's `.github/` otherwise). For Claude it is [`paarthurnax/CLAUDE.md`](../CLAUDE.md). Both are kept in sync.**

This file enforces a small set of *Always-On Behaviors* every turn and points at the canonical rules in [`../CONVENTIONS.md`](../CONVENTIONS.md). The behaviors below are tuned for working with VS Code (the surface Copilot lives in) and for cleanly using whatever skills, tools, extensions, and MCP servers the workspace exposes.

## Initialization sequence

Run this in order at the start of every session:

1. **Read the canonical files.** This file + [`../CLAUDE.md`](../CLAUDE.md) (mirror) + [`../CONVENTIONS.md`](../CONVENTIONS.md) (canonical rules). Skim [`../ARCHITECTURE.md`](../ARCHITECTURE.md), [`prompts/README.md`](prompts/README.md), [`skills/README.md`](skills/README.md), [`../.agents/skills/README.md`](../.agents/skills/README.md) so you know the shapes the agent is expected to produce.
2. **Scan the environment.** One-line summary covering: workspace folder, git branch + dirty/clean state, active terminal, OS, and any MCP servers configured (see *Environment awareness* below).
3. **Note the configured MCP servers.** Read [`../.mcp.json`](../.mcp.json) (workspace root: `.mcp.json` and/or `.vscode/mcp.json`). Treat the entries as *available*, not necessarily *running* — the user starts servers from the MCP UI. Do not auto-call MCP tools until you've confirmed the relevant server is running.
4. **Index the skills folders** ([`skills/`](skills/) and [`../.agents/skills/`](../.agents/skills/)) by reading each `SKILL.md`'s frontmatter `description` only. Lazy-load the body when a task matches the description.

## Always-On Behaviors

The seven **Prime Directives** that override everything live in [`../CONVENTIONS.md`](../CONVENTIONS.md) → *Prime Directives*. Read those first and re-read them whenever you feel uncertain about whether an action is allowed.

The bullets below do **not** add new top-level rules. They (a) name the VS Code Copilot-specific affordance for each directive that needs one, and (b) carry operational guidance (lockfile choice, MCP discipline, ignore-list handling, etc.) that isn't in the directives.

### How Copilot implements the directives

- **Directive 2 (diffs before writes)** → VS Code shows the diff inline. Watch it land before claiming success.
- **Directive 5 (option-picker)** → call `vscode_askQuestions` with labelled options. Leave `allowFreeformInput` on (default). Cap labelled options at **6 maximum**.
- **Directive 6 (`[STUCK]`)** → emit the block as plain text in chat. Copilot has no structured stuck-state tool — the user reads it and decides.
- **Directive 7 (cite the rule)** → markdown link to the file and section, e.g. *"I won't do that because [`../CONVENTIONS.md`](../CONVENTIONS.md) → *Prime Directives* rule 1 says reads outside the workspace need a reason."*

### Environment awareness

- **Stay grounded in concrete, freshly observed facts.** When the user's request depends on the environment (which extension is installed, which terminal is active, which MCP server is running, which Python interpreter is selected), check before answering. Do not guess. The most common Copilot failure mode is confidently invoking a tool the user doesn't have.
- **Detect what's installed when it matters.** For VS Code extensions, language servers, MCP servers, and shells: prefer asking the editor through its own commands (extension list, MCP server list, terminal profile) over assuming based on file shapes alone. When detection isn't possible from inside chat, ask one short question.
- **Re-scan on workspace change.** Branch switch, container reload, extension install/uninstall — any of these invalidates the previous environment scan. Refresh before the next non-trivial action.

### Skills, prompts, and tools

- **Lazy-load skills, never preload them all.** Index by `description` frontmatter; load the full `SKILL.md` body only when a task triggers it. The two canonical skills folders are [`skills/`](skills/) (Copilot-side) and [`../.agents/skills/`](../.agents/skills/) (cross-tool starter skills).
- **Use the right slash command** when the user invokes one. Real prompts live in [`prompts/`](prompts/); see [`prompts/README.md`](prompts/README.md) for the file shape. Don't reinvent a prompt's logic in plain prose if a `.prompt.md` already encodes it.
- **Prefer the project's own tasks.** When `.vscode/tasks.json` defines `test`, `lint`, or `build`, run those rather than reverse-engineering the command. Tasks encode the project's conventions.
- **Match the lockfile.** `package-lock.json` → `npm`, `pnpm-lock.yaml` → `pnpm`, `yarn.lock` → `yarn`, `uv.lock` → `uv`, `poetry.lock` → `poetry`. Never mix.

### MCP awareness

- **Treat `.mcp.json` as the menu, not the kitchen.** The file declares servers the workspace knows about; whether each one is actually running is a separate question. The shipped [`../.mcp.json`](../.mcp.json) contains *dummy Skyrim-flavored placeholder entries* showing the expected shape — replace them with real servers before pointing the agent at them.
- **One MCP server is plenty.** Most workspaces only need one remote bridge. If multiple servers are configured, pick the one most relevant to the current task and route all MCP calls through it for the rest of the turn — don't fan out across servers without a reason.
- **MCP servers are third parties.** Each server connects to its own external system. Treat its tool calls like any other untrusted code path: scoped to the declared capability, never granted blanket auto-approve, never given access to secrets.
- **Don't commit private MCP entries.** [`../.mcp.json`](../.mcp.json) is repo-tracked. Real production servers (with URLs, tokens, etc.) belong in a local-only override or in environment-driven config, not in the committed file.

### File-system boundaries

- **Honor the project's ignore lists.** `.gitignore`, `.clineignore`, Continue's `excludeFiles`, Cursor's `.cursorignore`, etc. — never read files matched by them, never include them in prompts, never pass them to sub-tools.
- **Live-state reconciliation before any modification.** Before writing to any resource the user might have edited (a notebook, a config, a workflow definition fetched from a remote system), re-read the current live state first and smart-merge. Stop and ask only on conflicting overwrites. Keep a timestamped snapshot for revert. Full contract in *Resource Update Contract* below.

### Safety and reversibility

- **Never bypass safety prompts** (`--force`, `--no-verify`, `-y` on uninstalls) without naming the flag and explaining why.

### Communication

- **Speak when you start, when you wait, when you finish.** One sentence on entry ("I'll do X by doing Y."). Announce long-running tool calls before they start. Short verdict line at the end. Never end silently.
- **Use the todo list** for multi-step work so the user can see progress.
- **Don't say tool names to the user.** Say "I'll run a command in a terminal", not "I'll use `run_in_terminal`".

## Skills

Domain knowledge skills live in two canonical folders:

- [`skills/`](skills/) — Copilot-side skills (this sibling).
- [`../.agents/skills/`](../.agents/skills/) — cross-tool starter skills any agent can lazy-load (git, github-cli, node-npm, python-environments, powershell-essentials, unix-shell-essentials, docker-essentials, tool-discovery).

They load automatically when relevant — no manual loading required. See [`skills/README.md`](skills/README.md) for the file shape, frontmatter conventions, and the eight-section pattern for skills that wrap external systems. For a local Ollama-driven loop the skills must be loaded by the loop itself — see [`../CONVENTIONS.md`](../CONVENTIONS.md) → *Local-model loops* and [`../ARCHITECTURE.md`](../ARCHITECTURE.md) → *Local-model deployment* (Flavor B).

## Prompts

Type `/` in Copilot Chat to access slash commands. Real prompts live in [`prompts/`](prompts/) — see [`prompts/README.md`](prompts/README.md) for the file shape and naming conventions. Name slash commands after **what the user is trying to do**, not after the tool that does it.

## MCP servers

The workspace ships an example [`../.mcp.json`](../.mcp.json) at the root. It demonstrates the expected shape for MCP-aware clients (Claude Code, Cursor, Cline) — the dummy entries are Skyrim-flavored placeholders showing stdio servers (npx-launched, node-launched with `cwd`) and HTTP/SSE servers. VS Code Copilot Chat reads the same shape from `.vscode/mcp.json` (top-level key `servers` instead of `mcpServers`).

Replace the dummy entries with real servers before connecting the agent to anything that matters. The transports MCP supports are stdio (most common), HTTP+SSE (legacy), and Streamable HTTP (preferred for new servers); see [modelcontextprotocol.io](https://modelcontextprotocol.io) for the full spec.

When MCP servers are configured and running, prefer them for: structured-JSON-direct-to-the-AI tasks, tool-calling against the remote system from chat, ad-hoc messaging integrations exposed by the server. Prefer plain shell or the project's own CLI when: you need terminal-visible output, declarative `apply`/`diff`/`history`, or anything the user wants to *see* happen.

## Resource Update Contract

This workspace follows a smart-merge contract for any resource that has both a local representation and a live remote version (a notebook synced to a remote app, a workflow definition, a settings document, etc.). Full details in [`../CONVENTIONS.md`](../CONVENTIONS.md) → *Resource Update Contract*.

- **One file per resource** at a stable path with a top-level marker indicating its remote origin. Per-type subfolders are auto-created on first use.
- **Refresh before edit.** Re-read the live state from the remote system into the local file before applying any modification.
- **Smart-merge unrelated user UI edits** into the local representation. Stop and ask (stop / let AI overwrite / do something else) only on conflicting overwrites.
- **Snapshot before reconciling.** Keep a timestamped before-user-edit snapshot for revert.
- **Prefer JSON payloads, ID-based operations, explicit metadata** for any embedded query/code, and re-export + verify after apply.

---

## Communication discipline (Copilot-specific reinforcement)

The Copilot chat surface gives the agent a uniquely interactive UI (option pickers, inline diffs, todo lists, terminal panels). *Prime Directive 5* (option-picker for every choice) and the *Communication* bullets in *Always-On Behaviors* above are the contract; this section just calls out the affordances:

- **Option pickers** for choices.
- **Inline diffs** for writes.
- **Todo list** for multi-step work.
- **Terminal panels** for shell commands you want the user to actually see (vs. silent background calls).

The Copilot UI rewards visible work. Use it.

---

## Design notes (target sizes & patterns)

Treat as targets, not laws.

- **Briefing length.** Aim for ~140 lines for the auto-loaded entry-point file. Push detail into `CONVENTIONS.md` and skills.
- **Skill prefix patterns.** Group skills by prefix (`<domain>-data-*` per data shape, `<domain>-app-*` per user-facing artifact type, plus standalone knowledge skills and unprefixed verb-named workflow skills).
- **Prompt set size.** Aim for a small set (~5–7) of slash commands, all verb-or-scenario named (no command named after the tool that does the work).
- **Lazy-load every skill body.** Index by `description` only; load the body on demand. Critical for keeping context budget small enough that local models can also use the same scaffold.
- **Memory tiers** (Copilot-Chat construct, but the layout is portable):
  - `/memories/` (cross-conversation, always loaded) — keep tiny, bullets only.
  - `/memories/session/` (per-conversation) — listed not loaded.
  - `/memories/repo/` (workspace-scoped, generic only) — never put per-project facts here.

---

<!-- changelog -->
- See [`../CHANGELOG.md`](../CHANGELOG.md) for the project changelog. This file mirrors `paarthurnax/CLAUDE.md` structurally; differences are reading order in the initialization sequence and Copilot-specific UI affordances called out in *Communication discipline*.
