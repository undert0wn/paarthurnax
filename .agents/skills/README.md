# Global agent skills (`paa/.agents/skills/`)

This folder ships **broad, cross-tool starter skills** that any agent (Copilot, Claude, a local Ollama model behind a custom loop, etc.) can lazy-load when it needs to interact with a common dev tool. Each skill is a **getting-started reference**, not a full database — it covers the 10–20 commands most users actually need, the gotchas that bite on day one, and **where to look for the rest**.

## What "skill" means here

Same shape as the `paa/.github/skills/` convention defined in [`../.github/skills/README.md`](../.github/skills/README.md):

- One folder per skill: `<skill-name>/SKILL.md` (+ optional `references/`, `scripts/`).
- YAML frontmatter with `name` and `description`. The `description` is the **load-trigger**: clients (Copilot Chat, Claude Code, the lazy-load layer of a local-model loop) read only the description by default and pull the full body when a task matches.
- Plain Markdown body, organized for skim-reading.

## Skills shipped here

| Skill | When to load |
|---|---|
| [`git/SKILL.md`](git/SKILL.md) | Any version-control task — commits, branches, rebase, conflict resolution. |
| [`github-cli/SKILL.md`](github-cli/SKILL.md) | Working with GitHub from the terminal — PRs, issues, releases, gists. |
| [`node-npm/SKILL.md`](node-npm/SKILL.md) | Node.js, `npm` / `pnpm` / `yarn`, package.json, scripts. |
| [`python-environments/SKILL.md`](python-environments/SKILL.md) | Python venvs, `pip`, `uv`, `pyenv`, dependency files. |
| [`powershell-essentials/SKILL.md`](powershell-essentials/SKILL.md) | Windows PowerShell — pipelines, `Get-*` discovery, common cmdlets. |
| [`unix-shell-essentials/SKILL.md`](unix-shell-essentials/SKILL.md) | bash / zsh — pipelines, find/grep/awk, common idioms. |
| [`docker-essentials/SKILL.md`](docker-essentials/SKILL.md) | `docker run` / `build` / `compose` basics, image hygiene. |
| [`tool-discovery/SKILL.md`](tool-discovery/SKILL.md) | How to ask any CLI for help — `--help`, `man`, `tldr`, `info`, `Get-Help`, web docs. |

## Design principles

These principles apply when adding a new skill or editing an existing one:

1. **Getting-started, not exhaustive.** Cover the 10–20 commands a working developer uses weekly. The long tail belongs in the official docs — link to them.
2. **Show the canonical "how do I find more" path.** Every skill ends with a *Where to learn more* section pointing at the tool's own help system + official docs + a community resource (Stack Overflow tag, awesome-list, etc.).
3. **OS-aware where it matters.** Where Windows/macOS/Linux diverge (path separators, shell quoting, file-system semantics), call it out explicitly. Don't assume bash.
4. **Safety first.** Destructive commands (`git push --force`, `rm -rf`, `docker system prune`) are flagged with a clear ⚠ and a safer alternative. The agent's [`../CONVENTIONS.md`](../CONVENTIONS.md) → *Operational Safety* rules apply on top.
5. **Tool-agnostic phrasing.** Don't assume a specific agent or IDE — these skills ride alongside whatever client is loading them.
6. **Update by observation.** If a new command becomes part of the everyday workflow, add it. If an old one stops being relevant, remove it. Skills are living docs.

## Adding a new skill

1. Pick a focused topic (one tool family or one workflow). If the topic is broader than ~150 lines, split it.
2. Create `paa/.agents/skills/<skill-name>/SKILL.md` with YAML frontmatter:
   ```markdown
   ---
   name: <skill-name>
   description: <one sentence — this is the load-trigger>
   ---
   ```
3. Body sections (suggested order): *When to load this skill* → *Quick reference* → *Common workflows* → *Gotchas* → *Where to learn more*.
4. Keep the skill generic — no platform-specific names, identifiers, or URLs. Committed skills must work in any project.
5. Add a row to the table above.

## Relationship to the family folders

The `paa/ollama/<family>/` folders (indexed in [`../ARCHITECTURE.md`](../ARCHITECTURE.md) → *Model-family folders*) define **what a particular model can do and how to talk to it**. The skills in this folder define **what an agent does in the user's workspace** independent of which model is driving. Both layers compose: a Modelfile from `paa/ollama/qwen/qwen.md` plus the skills from this folder gives a Qwen-driven agent both its personality and its toolbelt.
