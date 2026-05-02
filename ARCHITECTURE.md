# `paarthurnax/` — architecture

The `paarthurnax/` folder is a **vendor-neutral, agent-agnostic governance scaffold**. It mirrors the conventions every supported agent already expects (`.github/copilot-instructions.md` + `CLAUDE.md` + `CONVENTIONS.md` + skills/prompts) so the same scaffolding can be dropped into any project, paired with any agent, and pointed at any backend.

This file explains how the folder is laid out, what each piece does, and how the pieces compose.

## At a glance

```
paarthurnax/
├── CLAUDE.md                       ← Claude's auto-loaded session briefing
├── CONVENTIONS.md                  ← canonical operating rules (single source of truth)
│
├── .github/                        ← GitHub Copilot's auto-load conventions
│   ├── copilot-instructions.md     ← Copilot's auto-loaded session briefing (mirrors CLAUDE.md)
│   ├── skills/README.md            ← .github/skills/ convention reference
│   └── prompts/README.md           ← .github/prompts/ slash-command convention reference
│
├── .vscode/                        ← VS Code workspace-config conventions
│   └── README.md                   ← settings.json / tasks.json / mcp.json reference
│
├── .cline/                         ← Cline (VS Code AI agent) governance
│   └── README.md                   ← .clinerules / .clineignore + Ollama setup
│
├── .continue/                      ← Continue (VS Code AI agent) governance
│   └── README.md                   ← config.yaml / rules/ + Ollama setup
│
├── .agents/                        ← cross-tool agent capabilities
│   └── skills/                     ← global, model-independent skills
│       ├── README.md
│       ├── git/SKILL.md
│       ├── github-cli/SKILL.md
│       ├── node-npm/SKILL.md
│       ├── python-environments/SKILL.md
│       ├── powershell-essentials/SKILL.md
│       ├── unix-shell-essentials/SKILL.md
│       ├── docker-essentials/SKILL.md
│       └── tool-discovery/SKILL.md
│
└── ollama/                          ← 5 vetted open-weight model families (see ollama/README.md for the cut)
    ├── README.md                     ← tool-calling floor, kept vs not-included families, how to add more
    ├── qwen/qwen.md
    ├── llama/llama.md
    ├── mistral/mistral.md
    ├── deepseek/deepseek.md
    └── gemma/gemma.md
```

## Layered view — what loads when

```
                ┌─────────────────────────────────────────────────────┐
                │                  USER + WORKSPACE                   │
                └──────────────────────┬──────────────────────────────┘
                                       │
              ┌────────────────────────┼────────────────────────┐
              │                        │                        │
              ▼                        ▼                        ▼
   ┌──────────────────┐    ┌────────────────────┐    ┌──────────────────┐
   │  GitHub Copilot  │    │       Claude       │    │  Local model via │
   │                  │    │                    │    │   Ollama / etc.  │
   │  auto-loads:     │    │   auto-loads:      │    │  loaded via:     │
   │  .github/        │    │   CLAUDE.md        │    │  Modelfile       │
   │   copilot-       │    │                    │    │  SYSTEM block    │
   │   instructions   │    │                    │    │                  │
   └────────┬─────────┘    └─────────┬──────────┘    └─────────┬────────┘
            │                        │                          │
            └────────────┬───────────┴──────────────┬───────────┘
                         │                          │
                         ▼                          ▼
              ┌──────────────────────┐    ┌────────────────────────┐
              │    CONVENTIONS.md    │    │  ollama/<family>/      │
              │  (canonical rules —  │    │  (model quirks +       │
              │   linked from every  │    │   recommended Modelfile│
              │   briefing above)    │    │   for that family)     │
              └──────────┬───────────┘    └────────────────────────┘
                         │
                         ▼
              ┌──────────────────────────────────────────────┐
              │          .agents/skills/<topic>/             │
              │   (lazy-loaded on demand — git, docker,      │
              │    python, powershell, tool-discovery, …)    │
              └──────────────────────────────────────────────┘
```

**Three load tiers, in order:**

1. **Always-on briefings** (`copilot-instructions.md`, `CLAUDE.md`, Modelfile `SYSTEM` block) — read before the agent does anything. Kept short; they delegate to `CONVENTIONS.md` for detail.
2. **Canonical rules** (`CONVENTIONS.md`) — the single source of truth that the briefings link into. Edit once, every agent picks it up.
3. **Lazy-loaded skills** (`.agents/skills/<topic>/SKILL.md`) — pulled in only when a task matches the skill's `description` frontmatter. Keeps the always-on context window small.

The model-family files (`ollama/<family>/<name>.md`) sit *outside* this tier system — they're **reference**, not auto-loaded. They tell you how to build the Modelfile for tier 1.

## What lives where, and why

### Top-level governance

| File | Role | Loaded by |
|---|---|---|
| [`CONVENTIONS.md`](CONVENTIONS.md) | Canonical operating rules (prime directives, file-system boundaries, smart-merge contract, local-model loop contract, etc.). The single source of truth. | Linked from every briefing. |
| [`CLAUDE.md`](CLAUDE.md) | Short briefing pointing Claude at `CONVENTIONS.md`. | Claude (auto). |
| [`.github/copilot-instructions.md`](.github/copilot-instructions.md) | Mirror of `CLAUDE.md` for Copilot. Kept in lock-step. | Copilot (auto). |

### Convention reference folders (`.github/`, `.vscode/`)

These folders **document the conventions** of those folders rather than acting as live config. They exist so any agent (or new contributor) can read *"here's what `.github/skills/` is for, here's the file shape"* without guessing.

| File | Documents |
|---|---|
| [`.github/copilot-instructions.md`](.github/copilot-instructions.md) | Copilot's auto-load rule file. |
| [`.github/skills/README.md`](.github/skills/README.md) | The `.github/skills/<name>/SKILL.md` lazy-load convention. |
| [`.github/prompts/README.md`](.github/prompts/README.md) | The `.github/prompts/<slash-command>.prompt.md` convention. |
| [`.vscode/README.md`](.vscode/README.md) | What `settings.json` / `tasks.json` / `extensions.json` / `mcp.json` are for, what's safe to commit. |
| [`.cline/README.md`](.cline/README.md) | Cline extension: `.clinerules` / `.clineignore` conventions, opinionated defaults, Ollama provider setup. |
| [`.continue/README.md`](.continue/README.md) | Continue extension: `config.yaml` + `rules/` conventions, opinionated defaults, Ollama provider setup. |

The Cline and Continue folders also feed the cross-cutting **IDE-Embedded AI Agents** section in [`CONVENTIONS.md`](CONVENTIONS.md), which holds the rules that apply to every in-editor agent regardless of vendor.

In a real project these folders would contain actual configs alongside the README. In `paarthurnax/` they're scaffolding to copy into a new repo.

### Cross-tool skills (`.agents/skills/`)

These are **agent capabilities, not model traits**. They describe *how an agent uses a tool in your workspace* independent of which model is driving. Eight starter skills cover the most common cross-platform dev surfaces:

```
.agents/skills/
├── git/                       ← version control
├── github-cli/                ← GitHub from the terminal
├── node-npm/                  ← Node + package managers
├── python-environments/       ← venv / uv / pip / pyenv
├── powershell-essentials/     ← Windows PowerShell pipelines + cmdlets
├── unix-shell-essentials/     ← bash / zsh + classic Unix tools
├── docker-essentials/         ← containers + compose
└── tool-discovery/            ← --help / man / tldr / Get-Help
```

Each skill is a **getting-started reference** (10–20 commands, gotchas, *where to look for more*) — not an exhaustive database. Each ships YAML frontmatter with `name` + `description`; clients read only the description until a task triggers the load.

### Model-family folders (`ollama/<family>/`)

These folders document **what each open-weight model family can do and how to talk to it** — the closest thing each family has to a vendor-blessed governance convention (which mostly doesn't exist for open weights). Open-weight models inherit their behavior from whichever client drives them, so each folder contains a single Markdown file describing:

1. The honest status — there is no auto-loaded governance file for this family.
2. Where governance actually lives — the Ollama `Modelfile` (`SYSTEM` block + sampling defaults) is the only model-side hook; everything else is client-side (`AGENTS.md`, `.continue/rules/`, `.clinerules`, `.aider.conf.yml`, `.cursor/rules/`, etc.).
3. A recommended `Modelfile` template with a sane `SYSTEM` block, sampling params, and `num_ctx` for that family.
4. Family-specific quirks: chat template, special tokens, tool-calling shape, license caveats, recommended sizes for a "modern but not enterprise" rig (~16–32 GB RAM, 8–16 GB VRAM or Apple Silicon 16–32 GB unified).

All 12 follow the same 7-section template:

1. Status callout (snapshot-only verification)
2. Family overview table
3. Where governance actually lives (Modelfile → client rule files → AGENTS.md)
4. Recommended Modelfile template
5. Family-specific quirks (chat template, tool calling, context window, license)
6. Sampling defaults table
7. Where to look for updates

| Folder | Family | Best use |
|---|---|---|
| [`ollama/qwen/`](ollama/qwen/qwen.md) | Alibaba Qwen 2.5 / Qwen 3 / Qwen Coder | Strongest open-weight all-rounder; best coder at every size; best native tool calling. |
| [`ollama/llama/`](ollama/llama/llama.md) | Meta Llama 3.1 / 3.2 / 3.3 | General workhorse; large ecosystem; reference for "good enough" tool calling. |
| [`ollama/mistral/`](ollama/mistral/mistral.md) | Mistral / Nemo / Mixtral / Codestral / Devstral | Long-context Nemo, agent-tuned Devstral. |
| [`ollama/deepseek/`](ollama/deepseek/deepseek.md) | DeepSeek V2 / V3 / R1 / Coder | Strong reasoning (R1 distills); strong coder. |
| [`ollama/gemma/`](ollama/gemma/gemma.md) | Google Gemma 2 / Gemma 3 | No native tool-calling tokens, but follows prose-protocol action blocks reliably. Multimodal on Gemma 3 (4B+). Gemma TOS license. |

> Seven other open-weight families (Phi, SmolLM, StarCoder, Yi, GLM, Granite, Command-R) were considered and intentionally excluded. See [`ollama/README.md`](ollama/README.md) → *Families intentionally not included* for the rationale on each and the procedure for adding them back.

**How to use these folders**

- **Pick the family you actually run.** Read its file end-to-end before writing your own `Modelfile`. Family quirks (chat template, stop tokens, tool-call format) matter more than people expect — getting them wrong silently degrades quality.
- **Keep your customized `Modelfile` outside `paarthurnax/`** (e.g. under `paarthurnax/runtime/modelfiles/`, gitignored) if it contains anything instance-specific. The templates in these folders are generic and committable.
- **For the actual always-on rules**, point your Modelfile's `SYSTEM` block at the **Prime Directives** block at the top of [`CONVENTIONS.md`](CONVENTIONS.md). Paste the seven directives verbatim — they are deliberately short enough to fit. Do not paste the rest of `CONVENTIONS.md`; tell the model to emit `[READ] paarthurnax/CONVENTIONS.md` when it needs more, and have your loop honor that. See any of the family files under [`ollama/`](ollama/) for a worked example of the full SYSTEM block.
- **Cross-client governance** (one file works everywhere) is best served by writing an `AGENTS.md` at the workspace root per [agents.md](https://agents.md/) and pointing each client's config at it (Aider, Gemini CLI, Continue, Cursor, Cline, Roo, Zed, Warp, goose, Windsurf, Factory, Amp, Jules, GitHub Copilot's coding agent, VS Code agent mode all read it).

**Updating these files**

Open-weight model conventions evolve fast — recommended params and even prompt templates change between point releases. Treat each family's file as a snapshot and re-verify with `ollama show <model>`, the model card on [ollama.com/library](https://ollama.com/library), and the family's announcement post. When you re-verify, update the family file in place and add a `> **Verified:** YYYY-MM-DD` line at the top.

```
ollama/qwen/   ollama/llama/   ollama/mistral/   ollama/deepseek/   ollama/gemma/
```

## Composition — how the layers stack

A working agent setup pulls from **multiple layers simultaneously**:

```
   Modelfile from ollama/qwen/qwen.md      ← personality + sampling
            +
   SYSTEM block pointing to CONVENTIONS.md  ← canonical rules
            +
   Client briefing (CLAUDE.md or            ← always-on context
   copilot-instructions.md)
            +
   .agents/skills/git/SKILL.md (when        ← lazy-loaded capability
   the user asks about git)
            +
   .vscode/ + .github/ scaffolding in       ← workspace setup
   the actual project repo
            =
   A governed, capable, model-independent agent.
```

## Local-model deployment — two flavors

The scaffold supports two ways to drive a local open-weight model. The governance layer is identical; the loading mechanism differs.

**Flavor A — local model behind an existing client (Copilot / Claude Code / Cline / Continue / Cursor / Aider).**
The client auto-loads the governance files. You point Ollama at the client (most expose an OpenAI-compatible endpoint) and add a thin pointer file for whichever convention the client uses (`AGENTS.md`, `.cline/rules/`, `.continue/rules/`, etc.) carrying the same rules. Skills, prompts, MCP wiring — all reused. Ollama becomes "just another model" behind the editor; the governance layer doesn't care which model is on the other end.

**Flavor B — Ollama-driven custom agent loop you write yourself.**
No client is doing auto-loading for you. You replicate the loading order in code:

1. On startup, read [`CLAUDE.md`](CLAUDE.md) (or your equivalent entry-point) and inject as system prompt.
2. Index `paarthurnax/.agents/skills/*/SKILL.md` frontmatter; on each user turn, score the request against each `description` and load full SKILL bodies that match.
3. Index `paarthurnax/.github/prompts/*.prompt.md` and expose them as `/`-commands.
4. Implement the seven *Prime Directives* from [`CONVENTIONS.md`](CONVENTIONS.md) as deterministic middleware around the model call — small local models will not reliably follow long prose rules; *enforce* them in code (path checks before any write, mandatory diff display before any mutating call, etc.). The *Failure-mode guardrails* in [`CONVENTIONS.md`](CONVENTIONS.md) → *Local-model loops* are the loop-side counters/timers that pair with each directive.
5. Read `paarthurnax/runtime/mcp.json` and start the listed servers; expose their tools to the model via your loop.

Items 1–3 transfer structurally from any client setup. Item 4 is the *real* leverage. See [`CONVENTIONS.md`](CONVENTIONS.md) → *Local-model loops (custom Ollama agents)* for the action protocol, environment-scan shape, and small-local-model pitfalls that motivate the in-code enforcement.

## Design principles

1. **One source of truth** — rules live in `CONVENTIONS.md`. Briefings are short pointers, not duplicates.
2. **Vendor-neutral** — no platform-specific identifiers in committed files. Live data, instance IDs, and exported resources stay in `runtime/` or `scratch/` (gitignored).
3. **Tier separation** — always-on context stays small; detail is reachable via links and lazy-loaded skills.
4. **Mirror the patterns agents already expect** — `.github/`, `.vscode/`, `CLAUDE.md`, `AGENTS.md`-shaped files. Don't invent new conventions when an existing one works.
5. **Skills are starter references, not databases** — cover the daily 80%, link to authoritative docs for the rest.
6. **Models are reference, not loaded** — the family folders document Modelfile-building knowledge; they don't run inside the agent context.

## Quick reference — what to read first when authoring

| You're adding… | Read first | Then read |
|---|---|---|
| A new agent vendor | [`CLAUDE.md`](CLAUDE.md) and [`.github/copilot-instructions.md`](.github/copilot-instructions.md) (compare side by side) | [`CONVENTIONS.md`](CONVENTIONS.md) |
| A new knowledge skill | Any existing `SKILL.md` in [`.agents/skills/`](.agents/skills/) (a small one) | A larger `SKILL.md` for layout cues (sections, frontmatter, references folder) |
| A new workflow skill | Any procedural skill in `.agents/skills/` | Another workflow skill in the same folder |
| A new slash command | The smallest `.github/prompts/*.prompt.md` for shape | A richer one for advanced patterns |
| A new local tool / MCP server | [`.mcp.json`](.mcp.json) for the wiring shape | [`CONVENTIONS.md`](CONVENTIONS.md) |
| A new `<REMOTE-AI>` endpoint | [`.mcp.json`](.mcp.json) for the wiring shape | [`CONVENTIONS.md`](CONVENTIONS.md) → instance-onboarding section |
| An always-on rule | The *Prime Directives* section at the top of [`CONVENTIONS.md`](CONVENTIONS.md) | The vendor-specific *How <vendor> implements the directives* subsection in [`CLAUDE.md`](CLAUDE.md) and [`.github/copilot-instructions.md`](.github/copilot-instructions.md) |

## Adding to this scaffolding

| You're adding… | Where it goes |
|---|---|
| A new operating rule that should apply across agents | [`CONVENTIONS.md`](CONVENTIONS.md) — then make sure briefings still summarize it accurately. |
| A new rule that applies specifically to in-editor AI agents | [`CONVENTIONS.md`](CONVENTIONS.md) → *IDE-Embedded AI Agents* section. |
| A new cross-tool skill (e.g. `kubectl`, `terraform`) | `.agents/skills/<name>/SKILL.md` with `name` + `description` frontmatter. Update [`.agents/skills/README.md`](.agents/skills/README.md). |
| A new model family | `ollama/<family>/<family>.md` following the 7-section template. Update the *Model-family folders* section above. |
| A new IDE-AI extension (Cursor, Roo Code, Cody, Windsurf, …) | New `.<extension>/README.md` mirroring [`.cline/README.md`](.cline/README.md) / [`.continue/README.md`](.continue/README.md). Add a row to the IDE-Embedded AI Agents table in [`CONVENTIONS.md`](CONVENTIONS.md). |
| A vendor-specific convention reference | New folder mirroring the vendor's expected path (e.g. `.cursor/README.md`). |

## Where to learn more

- [`CONVENTIONS.md`](CONVENTIONS.md) — the rules themselves, including *Local-model loops* (action protocol, environment scan, small-local-model pitfalls).
- [`.agents/skills/README.md`](.agents/skills/README.md) — skills index + adding-a-skill checklist.
