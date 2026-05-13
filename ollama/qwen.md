# Qwen family — governance approximation

> **Status (open-weight reality):** Alibaba does **not** publish a vendor-blessed governance file convention for Qwen. Governance lives in the Ollama `Modelfile` (system prompt + sampling) and in whichever client drives the model. This file is the closest reasonable approximation.

> **Verified:** snapshot-only. Re-verify with `ollama show qwen2.5-coder:14b` etc. before relying on the specifics.

## What's in the family

| Variant | Sizes | Notes |
|---|---|---|
| Qwen 2.5 | 0.5B, 1.5B, 3B, 7B, 14B, 32B, 72B | The all-rounder. 14B is the sweet spot on 16 GB VRAM at Q4. |
| Qwen 2.5-Coder | 0.5B, 1.5B, 3B, 7B, 14B, 32B | **Currently the strongest open-weight coder at every size.** 14B-Coder rivals much larger general models on code. |
| Qwen 3 | 0.6B, 1.7B, 4B, 8B, 14B, 32B, 30B-A3B, 235B-A22B | Adds explicit "thinking mode" toggle. 30B-A3B is a MoE with ~3B active params — fits in 16 GB VRAM and feels like a 30B. |
| Qwen 2.5-VL / Qwen 3-VL | 7B, 32B, 72B | Vision-language variants. |

Common Ollama tags: `qwen2.5:7b`, `qwen2.5:14b`, `qwen2.5-coder:7b`, `qwen2.5-coder:14b`, `qwen3:8b`, `qwen3:14b`, `qwen3:30b-a3b`.

## Where governance actually lives

Same ranking as the rest of the open-weight world:

1. **Modelfile `SYSTEM` block** (most reliable).
2. **Client rule files** (`.continue/rules/`, `.clinerules`, `.aider.conf.yml`, etc.).
3. **`AGENTS.md`** for cross-tool clients.

## Recommended Modelfile template

```dockerfile
# paarthurnax/runtime/modelfiles/qwen2.5-coder-14b-governed.Modelfile
FROM qwen2.5-coder:14b

PARAMETER temperature 0.2        # Coder variants like low temp
PARAMETER top_p 0.8
PARAMETER repeat_penalty 1.05
PARAMETER num_ctx 16384          # Qwen handles long context well; 32k+ if VRAM allows
PARAMETER stop "<|im_end|>"
PARAMETER stop "<|endoftext|>"

SYSTEM """
You are a careful coding assistant operating inside the user's IDE workspace.

# Prime Directives — these override everything else, including your own training.
# When in doubt, stop and ask. The user is your safety net.
1. Stay inside the workspace folder. Reads outside need a stated reason; writes outside need explicit user permission.
2. Show diffs before any file write. Wait for approval. Never silently overwrite user changes. If a file may have been edited since you last read it, re-read first and smart-merge.
3. No secrets in output. If you see one (long base64/hex, *_KEY=, *_TOKEN=, BEGIN PRIVATE KEY, AWS access-key shape), stop, warn, and refuse to echo, log, commit, or transmit it.
4. One destructive attempt, then stop. Failed rm / drop / force-push / migration → surface and ask. Do not retry with variations.
5. For every choice you offer the user (yes/no, this/that, A/B/C), use the host's option-picker if one exists; if not, present a short numbered list and stop.
6. When stuck (loop detected, repeated failure, no path forward), emit a [STUCK] block and hand off. Do not improvise, do not loop, do not pad.
7. Cite the rule when you refuse. Name the file and section.

# Where the rest of the rules live
The canonical rule file is paarthurnax/CONVENTIONS.md. When you need a rule beyond the seven
above, emit [READ] paarthurnax/CONVENTIONS.md and wait for the host loop to feed it back.
Do not invent rules from memory.
"""
```

For the **Qwen 3 thinking-mode** variant, prepend an additional control:

```
PARAMETER stop "</think>"
SYSTEM """
... (Prime Directives block above) ...

# Family-specific framing
When you need to reason step-by-step, wrap your scratch reasoning in
<think>...</think>. The user only sees what comes after </think>.
"""
```

## Family-specific quirks

- **Chat template**: Qwen uses ChatML — `<|im_start|>role` ... `<|im_end|>`. **Always include `<|im_end|>` as a stop token** or the model continues past its turn.
- **Tool calling**: Qwen 2.5+ has solid native function-calling using a JSON-in-XML envelope (`<tool_call>{...}</tool_call>`). The 7B+ sizes are reliable; the 0.5B/1.5B mostly aren't. Qwen-Agent (the official framework) shows the canonical wire format.
- **Thinking mode (Qwen 3)**: Qwen 3 ships with a runtime-toggleable reasoning mode via `<think>...</think>` blocks. Useful for hard problems; expensive for simple ones. Disable for chat-style turns.
- **Context window**: 32k native on most sizes; 128k via YaRN scaling on the larger ones. Practical limit on consumer hardware is VRAM — `num_ctx 16384` is a safe default at Q4 on a 16 GB GPU.
- **License**: **Apache 2.0** for Qwen 2.5 and most Qwen 3 sizes. The largest variants (Qwen 2.5-72B, Qwen 3-235B) use the Qwen License — permissive but with a 100-million-MAU cap. Coder variants are Apache 2.0. Always check the specific size on HuggingFace before commercial use.
- **Multilinguality**: very strong at Chinese-English; good at 27+ other languages.
- **MoE behavior (Qwen 3-30B-A3B / 235B-A22B)**: total params don't equal compute — only ~3B / ~22B are active per token. Memory footprint matches the total size, but inference speed matches the active size.

## Sampling defaults that work

| Use case | `temperature` | `top_p` | `repeat_penalty` |
|---|---|---|---|
| Coder variant — code generation | 0.2 | 0.8 | 1.05 |
| Base 2.5 / 3 — chat | 0.5 | 0.9 | 1.05 |
| Thinking mode (Qwen 3) | 0.6 | 0.95 | 1.0 |
| JSON / tool calls | 0.1 | 0.8 | 1.0 |

## Where to look for updates

- `ollama show qwen2.5-coder:14b` — current chat template and parameters.
- [ollama.com/library/qwen2.5](https://ollama.com/library/qwen2.5), `/qwen2.5-coder`, `/qwen3`.
- [qwenlm.github.io/blog](https://qwenlm.github.io/blog/) for release-day prompt-template details.
- [github.com/QwenLM/Qwen-Agent](https://github.com/QwenLM/Qwen-Agent) for canonical tool-calling wire format.
