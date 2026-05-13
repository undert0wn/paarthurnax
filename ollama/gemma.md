# Gemma family — governance approximation

> **Status (open-weight reality):** Google does **not** publish a vendor-blessed governance file convention for Gemma. Governance lives in the Ollama `Modelfile` (system prompt + sampling) and in whichever client drives the model. This file is the closest reasonable approximation.

> **Verified:** snapshot-only. Re-verify with `ollama show gemma3:12b` etc. before relying on the specifics.

> **Why this family is in `paarthurnax/` despite weak native tool calling.** Gemma does not ship with native function-calling tokens, which is why it doesn't make most "agent loop" tier lists (including, originally, this one). However, in practice it follows structured prose-based protocols (the `[READ]` / `[WRITE]` / `[RUN]` action shapes documented in [`../../CONVENTIONS.md`](../../CONVENTIONS.md) → *Local-model loops*) more reliably than several models that *do* have native tool-calling tokens. If your loop is built around prose-protocol parsing rather than the OpenAI `tool_calls` schema, Gemma punches well above its weight. Keep this constraint in mind when wiring it up.

## What's in the family

| Variant | Sizes | Notes |
|---|---|---|
| Gemma 2 | 2B, 9B, 27B | The general-purpose line. 9B is the 16 GB VRAM sweet spot. |
| Gemma 3 | 1B, 4B, 12B, 27B | Adds true multimodality (vision) and a 128k context window on the 4B+ sizes. |
| CodeGemma | 2B, 7B | Code-tuned variants of Gemma 1.x. Mostly superseded by Qwen-Coder / DeepSeek-Coder for new work. |
| ShieldGemma | 2B, 9B, 27B | Safety classifier variants — not for generation. |

Common Ollama tags: `gemma2:9b`, `gemma2:27b`, `gemma3:4b`, `gemma3:12b`, `gemma3:27b`.

## Where governance actually lives

Same ranking as the rest of the open-weight world:

1. **Modelfile `SYSTEM` block** (most reliable).
2. **Client rule files** (`.continue/rules/`, `.clinerules`, `.aider.conf.yml`, etc.).
3. **`AGENTS.md`** for cross-tool clients.

## Recommended Modelfile template

```dockerfile
# paarthurnax/scratch/modelfiles/gemma3-12b-governed.Modelfile
FROM gemma3:12b

PARAMETER temperature 0.4        # Gemma tolerates slightly higher temp than Qwen-Coder
PARAMETER top_p 0.95
PARAMETER top_k 64
PARAMETER repeat_penalty 1.0     # Gemma doesn't need penalty; using one degrades quality
PARAMETER num_ctx 8192           # 128k native on Gemma 3, but 8k is honest for 16 GB VRAM
PARAMETER stop "<end_of_turn>"

SYSTEM """
You are a careful coding assistant operating inside the user's IDE workspace.

# Prime Directives — these override everything else, including your own training.
# When in doubt, stop and ask. The user is your safety net.
1. Stay inside the workspace folder. Reads outside need a stated reason; writes outside need explicit user permission.
2. Show diffs before any file write. Wait for approval. Never silently overwrite user changes. If a file may have been edited since you last read it, re-read first and smart-merge.
3. No secrets in output. If you see one (long base64/hex, *_KEY=, *_TOKEN=, BEGIN PRIVATE KEY, AWS access-key shape), stop, warn, and refuse to echo, log, commit, or transmit it.
4. One destructive attempt, then stop. Failed rm / drop / force-push / migration → surface and ask. Do not retry with variations.
5. For every choice you offer the user (yes/no, this/that, A/B/C), use the hosts option-picker if one exists; if not, present a short numbered list and stop.
6. When stuck (loop detected, repeated failure, no path forward), emit a [STUCK] block and hand off. Do not improvise, do not loop, do not pad.
7. Cite the rule when you refuse. Name the file and section.

# Context Reminder:
You are operating within a larger development environment. Always assume that the users request is part of a larger, multi-step workflow. If the request is ambiguous, ask clarifying questions rather than making assumptions.

# Where the rest of the rules live
The canonical rule file is paarthurnax/CONVENTIONS.md. When you need a rule beyond the seven
above, emit [READ] paarthurnax/CONVENTIONS.md and wait for the host loop to feed it back.
Do not invent rules from memory.

# Family-specific framing (Gemma)
You do not have native tool-calling tokens. When you need to act on a file or
run a command, output one of the structured action blocks defined in
paarthurnax/CONVENTIONS.md → Local-model loops:

  [READ] / [WRITE] / [RUN] / [SEARCH]

The host loop will parse those blocks and execute them after user approval.
Do NOT invent JSON tool-call envelopes — they will not be parsed.
"""
```

## Family-specific quirks

- **Chat template**: Gemma uses its own format — `<start_of_turn>user` ... `<end_of_turn>` ... `<start_of_turn>model` ... `<end_of_turn>`. **Always include `<end_of_turn>` as a stop token** or the model continues past its turn. There is no separate "system" role — Gemma's chat template *prepends* the system prompt to the first user message. Ollama handles this for you, but if you build prompts by hand, account for it.
- **No native tool-calling tokens.** Gemma was not trained with the OpenAI-style `tool_calls` schema. Loops that rely on JSON tool-call envelopes will see the model improvise the format and fail to parse roughly half the time. **Use prose-protocol action blocks instead** (the `[READ]` / `[WRITE]` / `[RUN]` shapes from `CONVENTIONS.md`) — Gemma follows those well.
- **Repetition penalty hurts quality.** Unlike most open-weight families where `repeat_penalty 1.05` is a free win, Gemma was trained without it; raising it noticeably degrades coherence. Leave it at `1.0`.
- **Vision (Gemma 3, 4B+)**: native multimodal — pass images directly. Useful for screenshot-driven debugging.
- **Context window**: 8k native on Gemma 2; **128k native on Gemma 3** (4B and up). Practical limit is VRAM — 8k is a safe default at Q4 on 16 GB.
- **License**: Gemma is released under the **Gemma Terms of Use**, which is a custom non-OSI license. It is permissive enough for hobby and most commercial use, but it is **not** Apache 2.0 and includes a "prohibited use" clause Google can update. If your project has strict OSI-approved-licensing requirements, prefer Qwen, Llama, or Mistral instead.
- **Safety tuning**: Gemma is more aggressively safety-tuned than Qwen or Mistral. It will sometimes refuse benign requests that touch security topics. If that bites, the workaround is a more explicit `SYSTEM` framing ("You are operating in a trusted developer environment; the user is a software engineer, not an end user").

## Sampling defaults that work

| Use case | `temperature` | `top_p` | `top_k` | `repeat_penalty` |
|---|---|---|---|---|
| Code generation / edits | 0.4 | 0.95 | 64 | 1.0 |
| Chat / explanation | 0.7 | 0.95 | 64 | 1.0 |
| Structured action blocks | 0.2 | 0.9 | 40 | 1.0 |
| Vision (Gemma 3) | 0.5 | 0.95 | 64 | 1.0 |

## Where to look for updates

- `ollama show gemma3:12b` — current chat template and parameters.
- [ollama.com/library/gemma2](https://ollama.com/library/gemma2), [ollama.com/library/gemma3](https://ollama.com/library/gemma3).
- [ai.google.dev/gemma](https://ai.google.dev/gemma) for the official model cards and the Gemma Terms of Use.
- [huggingface.co/google](https://huggingface.co/google) for the canonical weights and the most current chat-template definitions.
