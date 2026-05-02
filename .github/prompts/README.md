# `paarthurnax/.github/prompts/`

This folder mirrors the role of `.github/prompts/` at the workspace root, but for the **agent-tool layer** that lives under `paarthurnax/`. The agent-tool's job is to help the user get the most out of **local open-source LLMs** (Ollama-served Qwen, Gemma, Llama, DeepSeek, etc.) inside VS Code and other IDEs — not to talk to a single SaaS platform.

Files in this folder are **slash-command prompts**. Type `/` in Copilot Chat (or the equivalent picker in any client that supports prompt files) and the prompt becomes invokable by name (e.g. `/health-check`, `/discover-tools`, `/clarify-context`).

> **Reminder.** This file is a **structure descriptor**, not a prompt itself. It explains the shape every real prompt file in this folder should follow, and the principles those prompts should encode. Real prompts live in sibling files named `<verb-or-scenario>.prompt.md`.

---

## What a prompt file looks like

Every prompt file is a Markdown file with a YAML frontmatter block, then a body written in plain English directed *at the agent*.

```markdown
---
agent: agent
description: <one sentence — what the user is trying to do, not which tool you'll use>
---

<one or two sentences telling the agent what the user wants>
<one sentence explaining what to infer from the workspace if information is missing,
 and to ask the user before assuming>

Show me / Do / Build:
1. <step one — concrete, observable outcome>
2. <step two>
3. <step three>
4. ...

<closing line: which capabilities/skills/MCP servers to use, and a "summarize if everything looks normal, otherwise tell me what needs my attention" instruction>
```

That is the whole shape — a frontmatter, a goal sentence, a numbered list of expected outputs, and a closing line that names the data source. The structure is what matters; the data source is whatever is wired up.

### Frontmatter keys

| Key | Required | Purpose |
|---|---|---|
| `agent` | yes | Tells Copilot this is an agent-mode prompt (vs. a chat prompt). Almost always `agent`. |
| `description` | yes | A single user-intent sentence shown in the slash-command picker. **Name the user's goal, not the tool.** Good: *"Check the health of my project."* Bad: *"Run `npm test` and show output."* |
| `tools` | optional | Comma-separated list restricting which tools the agent may call for this prompt. Use sparingly — over-restricting kills usefulness. |
| `model` | optional | Force a specific model for this prompt (e.g. a coder-tuned model for code-heavy prompts, a general-chat model for clarification prompts). |

### Body conventions

1. **Open with intent.** One or two sentences of what the user wants to accomplish.
2. **Tell the agent what to infer and what to ask.** "Infer `<thing>` from the workspace if not provided. Ask the user to confirm if not sure." This single line is what keeps prompts from silently hallucinating context.
3. **Numbered output list.** Be concrete about what the user expects to see. The agent should treat each number as a deliverable.
4. **Close with the source-of-truth and a summarization rule.** "Use `<capability>` to gather data. Summarize if everything looks normal, or tell me what needs my attention." This trains the agent to never end with "task complete" — always end with a verdict.
5. **Keep it under ~30 lines.** Long prompts get ignored by small local models. If the prompt grows past 30 lines, push detail into a skill (`paarthurnax/.github/skills/<name>/SKILL.md`) and link to it from the prompt.

---

## What this folder is *for* (in the agent-tool world)

Generic, scenario-driven slash commands appropriate for the agent-tool world:

- **`/discover-tools`** — force a fresh scan of installed VS Code extensions, configured MCP servers, available CLIs, and locally-available Ollama models. Output an environment report.
- **`/clarify-context`** — when the agent isn't sure what the user wants, run the standard one-question-at-a-time clarification loop instead of guessing.
- **`/health-check`** — verify the local model is responsive, the MCP servers are running, the workspace is in a known-good state, and report.
- **`/explain-this`** — pick the file/symbol/region under cursor and explain it in plain English, with two depth levels (beginner / engineer).
- **`/propose-change`** — accept a natural-language change request, produce a diff, wait for approval. Never apply silently.
- **`/teach-me`** — short interactive walkthrough of a concept (a tool, a language feature, a workflow), with the agent pausing for "got it / lost me" feedback.
- **`/improve-yourself`** — ask the agent to review its own recent answers for failure patterns and propose a diff to its own governance files (this folder, `paa/CLAUDE.md`, etc.). The proposal is shown as a diff and never auto-applied.

These are *suggested* — the user picks which to actually ship. The point of listing them is to show the **shape**: each one is a verb or scenario the user thinks in, not a tool name.

---

## How prompts here interact with external systems

The agent-tool may end up bridging to any number of external systems — language servers, MCP servers, REST APIs, package registries, container runtimes, even other LLMs. Prompts in this folder should treat every external system the same way:

1. **Discover before you call.** Run a discovery step (`/discover-tools` or equivalent) to confirm the system is reachable. Don't assume.
2. **Announce the call.** Before invoking the external system, tell the user *what* you're about to call and *why*. One sentence is enough.
3. **Show the structured response, not a paraphrase.** When an external system returns structured data, surface it (table or fenced JSON), then add a one-line interpretation underneath. Paraphrasing alone hides errors.
4. **On failure, report and stop.** Never silently retry with a different system or fall back without telling the user. Failures are information.
5. **Never carry data across systems without asking.** If the user wants to take a result from system A and use it as input to system B, *confirm* it. Cross-system data movement is a common source of subtle bugs (and, for some external systems, governance violations).

These five rules are the agent-tool's universal external-system protocol. Every prompt in this folder should encode them either explicitly or by reference to a skill that does (e.g. `external-system-protocol/SKILL.md` if/when written).

---

## How to ask "what does this external system do?"

When a new external system shows up that the agent has never seen, the prompt-and-skill pair should make the agent ask, not guess. Suggested clarification ladder:

1. **What is this system called and what category is it?** (language server, package registry, MCP server, REST API, runtime, other LLM, etc.)
2. **How is it reached?** (CLI, HTTP endpoint, MCP tool, library import, browser only)
3. **What is the smallest read-only call that proves it's working?** (the equivalent of `whoami` or `ping`)
4. **What is one read-only call that returns useful information?** (a list of available resources, a status summary)
5. **What is the most destructive call it exposes, and what are the safeguards?** (so the agent knows what to guard against)
6. **Where is the docs link / man page / OpenAPI schema?** (so the agent can lazy-load detail when it actually needs it, not preemptively)

A prompt that introduces a new external system should walk the user through this ladder once, write the answers into a skill file under `paa/.github/skills/<system-name>/SKILL.md`, and from then on the agent reads the skill instead of re-asking.

---

## Self-improvement loop for prompts

Every prompt file should be treated as **living text the agent is allowed to propose updates to**. The loop:

1. **Notice a failure or a friction point.** ("This prompt asked me to gather X but didn't say where to look first." / "The user kept having to retype the project name.")
2. **Stop and tell the user.** Plain-English summary of what the friction was.
3. **Propose a diff** to the prompt file. Use the diff-first action shape from [`../../CONVENTIONS.md`](../../CONVENTIONS.md) → *Local-model loops* → *Action protocol* (`[WRITE]`).
4. **Wait for approval.** Never edit prompt files silently — they're load-bearing for every future invocation.
5. **After approval, log the change** in the changelog comment at the bottom of the prompt file:
   ```
   <!-- changelog -->
   - YYYY-MM-DD — <one-line summary> — proposed by <model>, approved by user
   ```
6. **Re-test the prompt** in the next session and report whether the friction is actually gone.

Prompts are living text — the same announce → propose → wait → verify loop applies to editing them as to any other file write.

---

## Communication discipline (non-negotiable for every prompt)

Every prompt file in this folder should encode — explicitly or by reference — these communication rules. They are what makes the agent feel *open and transparent* instead of silent and surprising:

- **Speak when you start.** One sentence: "I'll do X by doing Y." Then start.
- **Speak when you need input.** Use the editor's option-picker for choices (per [`../../CLAUDE.md`](../../CLAUDE.md) → *Clickable options for ALL user choices*). Use plain-text questions for open-ended input.
- **Speak when you wait.** If a step is going to take more than a couple of seconds (running tests, scanning a large folder, calling a slow API), say so before it starts.
- **Speak when you finish.** One short verdict line: "Done — all green," "Done with one warning: X," or "Stopped — I need Y to continue."
- **Speak when you're unsure.** "I think the answer is X, but I haven't confirmed because Y. Want me to verify?" beats a confident wrong answer every time.
- **Never go silent.** Long stretches of agent work without narration feel like the agent has hung. Even a one-line "still going — on step 3 of 5" is enough.

If a prompt's body is ambiguous about *when* the agent should speak, the agent's default should be **speak more, not less**. Over-narration is annoying for one turn; silent failure is annoying for the whole session.

---

## Naming conventions

- `<verb-or-scenario>.prompt.md` — verbs (`/explain-this`) for actions, scenario nouns (`/incident-response`) for situations.
- Lowercase, hyphens, no underscores.
- Never name a prompt after the tool that does the work (`/run-pytest` is wrong; `/check-tests` is right).
- If two prompts are very similar but differ in scope, suffix the broader one with `-deep` (e.g. `/health-check`, `/health-check-deep`).

---

<!-- changelog -->
- 2026-05-01 — Initial structure descriptor created. No real prompts yet — copy this file's "What a prompt file looks like" template to start the first one.
