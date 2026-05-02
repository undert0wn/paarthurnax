# `paa/.github/skills/`

This folder mirrors the role of `.agents/skills/` (and `.github/skills/`) at the workspace root, but for the **agent-tool layer** under `paa/`. Skills here are **lazy-loaded reference packets** the agent reads only when it needs them — they keep the always-on context tiny so local open-source models can think clearly.

> **Reminder.** This file is a **structure descriptor**, not a real skill. It explains the shape every skill file in this folder should follow, what skills are *for*, and how the agent uses them. Real skills live in sibling folders named `<skill-name>/SKILL.md`.

---

## What a skill is (and what it isn't)

A **skill** is a small, focused, single-purpose reference document the agent loads **only when a relevant task is at hand**. It is not a system prompt, not a tutorial, not a piece of documentation aimed at humans — though humans can certainly read it.

| A skill **is** | A skill **is not** |
|---|---|
| A focused how-to for one capability or domain (e.g. "how to read a file safely on Windows") | A general overview of a topic |
| Discoverable via its `description` line in the frontmatter (the picker reads only the description) | A wall of text the agent has to load up-front |
| Procedural ("here are the steps") or referential ("here are the rules / commands / patterns") | Conversational |
| Read in full only when the agent decides it needs it | Always-on context |
| Allowed to link out to deeper references in `references/<topic>.md` siblings | A monolith |

There are exactly two flavors:

1. **Knowledge skill** — a reference packet. "Here are the facts and patterns for X." Loaded when the agent needs to *understand* something.
2. **Workflow skill** — a procedural recipe. "When the user asks for X, do these steps in this order." Loaded when the agent needs to *do* something.

Don't try to mix the two in one file. Split them.

---

## What a skill file looks like

Every skill lives in its own folder: `paa/.github/skills/<skill-name>/SKILL.md`. The folder structure exists so a skill can carry its own references without polluting the skill index.

```
paa/.github/skills/<skill-name>/
  SKILL.md                  # required — the entry point
  references/
    <topic>.md              # optional — deeper detail the SKILL.md links to
    <example-output>.json   # optional — concrete fixtures
  scripts/
    <helper>.ps1            # optional — ready-to-suggest helpers
```

The `SKILL.md` itself follows this shape:

```markdown
---
name: <skill-name>
description: <one or two sentences — when to load this skill, in the user's words>
---

# <Skill name in title case>

<one-paragraph summary: what this skill covers and what it does NOT cover>

## When to use this skill

- <bullet — a user phrasing that should trigger loading>
- <bullet — another user phrasing>
- <bullet — a situation, not a phrasing>

## Getting started

<the smallest read-only first step the agent should take>

## <Section per dimension or step>

<actionable detail, with code blocks, tables, and concrete commands>

## Pitfalls

- <a known wrong path and how to spot it>
- <another known wrong path>

## When NOT to use this skill

- <bullet — a case that looks like it fits but doesn't, with the right alternative>
```

### Frontmatter keys

| Key | Required | Purpose |
|---|---|---|
| `name` | yes | The skill's invokable name. Lowercase, hyphens. Must match the folder name. |
| `description` | yes | The one or two sentences a model uses to **decide whether to load this skill at all**. This is the most important field in the whole file. Write it as a description of the *user's situation*, not the skill's contents. Good: *"Use when the user asks to debug a failing build, including phrases like 'the build broke', 'CI is red', 'why won't this compile'."* Bad: *"Reference for build tooling."* |
| `model` | optional | If the skill needs a specific model class (e.g. a coder-tuned model for refactoring), name it. |
| `tools` | optional | Restrict which tools the agent may use while this skill is active. Use sparingly. |

### Body conventions

1. **The `description` is the trigger.** Local models scan all skill descriptions and load only the ones that match the current task. Make the description rich with the *user phrasings* and *situations* that should trigger it. This is more important than the body.
2. **Lead with "When to use this skill" and "When NOT to use this skill".** The agent uses these two sections to confirm or rule out the skill *after* loading it but before acting on it.
3. **Short body, deep references.** SKILL.md should be reading-budget-friendly (~100–200 lines max). Push deep detail into `references/<topic>.md` siblings and link to them. The agent only opens references when actively needed.
4. **Concrete > abstract.** Show the actual commands, the actual code shapes, the actual error messages. Local models hallucinate generic advice; concrete examples ground them.
5. **Pitfalls section is mandatory.** Every skill includes at least one known-wrong-path so the agent has a way to recognize it's off-track.

---

## What this folder is *for* (in the agent-tool world)

The agent-tool helps the user leverage local open-source LLMs to their fullest. Skills here should cover the capabilities the agent itself needs to wield well, plus the protocols for talking to whatever external systems show up. Suggested initial set:

- **`local-model-tuning/SKILL.md`** — how to read and propose changes to Ollama Modelfile parameters (temperature, num_ctx, top_p, repeat_penalty, …) per [`../../CONVENTIONS.md`](../../CONVENTIONS.md) → *Local-model loops* → *Communication style* (`[PARAMETER SUGGESTION]` block). Includes a parameter-effect table, sample diffs, and the "always show as a suggestion" rule.
- **`vscode-extension-discovery/SKILL.md`** — how to enumerate installed extensions, MCP servers, available shells, and language servers, then format an Environment Scan per [`../../CONVENTIONS.md`](../../CONVENTIONS.md) → *Local-model loops* → *Environment scan*.
- **`external-system-protocol/SKILL.md`** — the universal "discover → announce → call → show → on-failure-stop → never-cross-systems-silently" protocol from [`../prompts/README.md`](../prompts/README.md). Every prompt that touches an external system loads this skill first.
- **`safe-file-edits/SKILL.md`** — diff-first writes, backup-then-modify for risky edits, never-touch lists (`.git/`, `~/.ssh/`, `node_modules/`, the workspace's own `.vscode/settings.json` without explicit permission).
- **`terminal-shell-awareness/SKILL.md`** — PowerShell vs Git Bash vs WSL command differences, `;` vs `&&`, path separators, environment variable syntax. Loaded whenever the agent is about to run a shell command.
- **`clarification-loop/SKILL.md`** — the "ask one question at a time" pattern. Common batteries: user preferences (preferred shell, output verbosity), available tools (which extension is installed, which test runner), project goals (quick fix vs long-term, target platforms), technical constraints (supported tool versions, lint rules, CI green-rule).
- **`memory-tier-discipline/SKILL.md`** — what goes in `/memories/`, `/memories/session/`, `/memories/repo/`, and `paa/memory.md`. Loaded whenever the agent is about to write to any of them.
- **`self-improvement-proposal/SKILL.md`** — how to notice a recurring failure pattern, write it up, propose a diff to a governance file, wait for approval, and log the change. The mechanism behind the `/improve-yourself` prompt.

These are *suggested*. Pick which to actually ship.

---

## How skills interact with external systems

Skills are the right place to encode the **rules for one specific external system**. A skill named `<system>/SKILL.md` should answer:

1. **Identity.** What is this system, in one sentence? What category does it belong to?
2. **Reachability.** How is it reached (CLI, MCP, HTTP, library, browser-only)? What's the smallest read-only call that proves it's working?
3. **Vocabulary.** What are the system's own names for its core concepts? Quote them, don't paraphrase — paraphrasing causes the agent to invent terms.
4. **Read patterns.** What are the most useful read-only calls? Show the exact shape of the request and a fenced example response.
5. **Write patterns.** What writes does it support? Each write gets a row in a table with: the call, what it changes, whether it's reversible, and the safeguard (confirmation, dry-run flag, etc.).
6. **Error vocabulary.** What error shapes does it return, and what does each one mean in plain English?
7. **Pitfalls.** What does the agent get wrong about this system most often? (Write this section *after* you've actually used the skill in anger; leave it as `## Pitfalls\n\n- (none observed yet — add as encountered)` to start.)
8. **Docs link.** Where to read more when the agent needs detail not in this skill.

A skill that follows this shape is enough for the agent to be useful with the external system without ever loading external system docs into the prompt by default.

---

## How to ask "what does this external system do?"

When the agent meets a new external system and there is no skill for it yet, the right move is **not** to guess. The right move is to:

1. Tell the user: "I see system X but I don't have a skill for it yet. Mind if I ask you a few short questions and write one?"
2. Use the clarification ladder from [`../prompts/README.md`](../prompts/README.md) → *How to ask "what does this external system do?"*.
3. Draft a new skill at `paa/.github/skills/<system>/SKILL.md` using the *eight-section shape* above.
4. **Show the draft as a diff.** Wait for approval. Don't commit a skill the user hasn't seen.
5. After approval, the skill is now part of the lazy-load index and any future prompt that touches this system will load it.

This is how the skill library grows — one observed system at a time, captured by the agent, reviewed by the user, then permanent.

---

## Self-improvement loop for skills

Same loop as for prompts (see [`../prompts/README.md`](../prompts/README.md)), with two skill-specific additions:

1. **Notice a failure or friction point** while a skill is loaded.
2. **Tell the user** what was missing or wrong.
3. **Propose a diff** to the skill's `SKILL.md` or to a `references/<topic>.md` sibling.
4. **Wait for approval.**
5. **Append a changelog entry** at the bottom of the skill file.
6. **If the skill grew past ~200 lines**, propose a split: pull the deepest detail into `references/<topic>.md` and replace it in `SKILL.md` with a one-line link. Skills stay small or they stop being lazy-loadable.
7. **If two skills overlap heavily**, propose merging them — or factor the shared part into a third skill that both link to. Duplication in skills means the agent loads twice the context for the same job.

---

## Communication discipline (skill flavor)

Skills are loaded silently — the user doesn't see the load happen. That makes communication *more* important, not less:

- **Announce when a skill changes the plan.** "I just loaded the `<skill>` skill — based on its rules, I'm going to do X instead of Y because Z."
- **Quote the skill, don't paraphrase**, when stating a rule. "The `safe-file-edits` skill says: *never modify files under `.git/` without explicit permission*. So I'll stop here."
- **Cite the skill on disagreement.** If the user pushes against something a skill says, the agent surfaces the skill's exact wording and the user decides whether to override for this turn or update the skill.
- **Propose a skill update when the skill is wrong.** Don't just override silently. Use the self-improvement loop above.

---

## Naming conventions

- Folder name = `<skill-name>` (kebab-case). Must match the `name` frontmatter key in `SKILL.md`.
- Group related skills with shared prefixes: `local-model-*`, `vscode-*`, `external-system-*`, `safe-*`. The prefix is human convention, not enforced by tooling.
- Knowledge skills tend to take noun-ish names (`vscode-extension-discovery`); workflow skills take verb-ish names (`safe-file-edits`, `clarification-loop`).
- Don't name a skill after a specific brand or product unless it *is* a skill for that specific product (`<system>/SKILL.md`). Generic capabilities get generic names.

---

<!-- changelog -->
- 2026-05-01 — Initial structure descriptor created. No real skills yet — copy this file's SKILL.md template to start the first one.
