# DeepSeek family — governance approximation

> **Status (open-weight reality):** DeepSeek does **not** publish a vendor-blessed governance file convention. Governance lives in the Ollama `Modelfile` and in whichever client drives the model. This file is the closest reasonable approximation.

> **Verified:** snapshot-only. Re-verify with `ollama show deepseek-r1:14b` etc.

## What's in the family

| Variant | Sizes | Notes |
|---|---|---|
| DeepSeek-Coder | 1.3B, 6.7B, 33B | The original coder series. Strong fill-in-the-middle. |
| DeepSeek-Coder V2 | 16B-Lite (MoE, ~2.4B active), 236B-MoE | The Lite MoE is excellent on consumer hardware — feels like a 16B but runs like a 2.4B. |
| DeepSeek V3 | 671B-MoE (full); distills available | The full model is enterprise-only. Distills (3B–70B) bring much of the strength to consumer rigs. |
| DeepSeek-R1 | 671B-MoE (full); distills 1.5B / 7B / 8B / 14B / 32B / 70B | **Reasoning-tuned.** Distills inherit R1's chain-of-thought style at runnable sizes. |

Common Ollama tags: `deepseek-coder`, `deepseek-coder-v2:16b`, `deepseek-v3`, `deepseek-r1:7b`, `deepseek-r1:14b`, `deepseek-r1:32b`, `deepseek-r1:70b`.

## Where governance actually lives

1. **Modelfile `SYSTEM` block** (most reliable).
2. **Client rule files**.
3. **`AGENTS.md`** for cross-tool clients.

## Recommended Modelfile templates

### DeepSeek-R1 distill (reasoning)

```dockerfile
# paarthurnax/runtime/modelfiles/deepseek-r1-14b-governed.Modelfile
FROM deepseek-r1:14b

PARAMETER temperature 0.6        # R1 is tuned for 0.5-0.7; lower causes truncation
PARAMETER top_p 0.95
PARAMETER repeat_penalty 1.0
PARAMETER num_ctx 16384          # 32k+ if VRAM allows
PARAMETER stop "<｜end▁of▁sentence｜>"
PARAMETER stop "</think>"        # optional — see "thinking mode" below

SYSTEM """
You are a careful coding and reasoning assistant operating inside the user's IDE workspace.

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

# Family-specific framing (R1)
When you reason, wrap the scratch reasoning in <think>...</think>. The user reads
only what comes after </think>.
"""
```

### DeepSeek-Coder V2 Lite (autocomplete + chat)

```dockerfile
# paarthurnax/runtime/modelfiles/deepseek-coder-v2-governed.Modelfile
FROM deepseek-coder-v2:16b

PARAMETER temperature 0.2
PARAMETER top_p 0.9
PARAMETER repeat_penalty 1.05
PARAMETER num_ctx 16384

SYSTEM """
You are a coding assistant that produces minimal, focused diffs. The seven Prime
Directives in paarthurnax/CONVENTIONS.md → *Prime Directives* always apply (workspace
boundary, diffs-before-writes, no secrets, one destructive try, option-picker,
[STUCK] hand-off, cite-the-rule). In addition: read existing files before
editing; for fill-in-the-middle, use the model's native FIM tokens and return
only the missing span.
"""
```

## Family-specific quirks

- **Chat template**: DeepSeek uses its own special tokens — `<｜begin▁of▁sentence｜>`, `<｜User｜>`, `<｜Assistant｜>`, `<｜end▁of▁sentence｜>`. **Note the unusual fullwidth pipe characters (`｜`) — copy them exactly or stop tokens won't fire.**
- **R1 thinking mode**: R1 (and its distills) emit `<think>...</think>` blocks before the final answer. **Do not strip them in your client** if you want the model's full reasoning chain — strip them only at display time. Disable for low-latency chat work.
- **R1 sampling sensitivity**: R1 is unusually picky about temperature. **Below 0.5 it truncates; above 0.8 it drifts.** Stick to the recommended 0.5–0.7 band.
- **Tool calling**: DeepSeek V3 and Coder V2 have solid native tool-calling. R1 does not — its training prioritizes reasoning over tool-following. For tool use, prefer V3 or Coder V2.
- **Context window**: most variants support 64k–128k native. Practical `num_ctx` on a 16 GB GPU at Q4 is 16k–32k.
- **License**: **DeepSeek License Agreement** — generally permissive but with a use-restrictions appendix. Coder V2 is often MIT or Apache-style on individual sizes; **always check the specific Ollama tag's model card** before commercial use.
- **MoE behavior** (Coder V2 16B / V3 / R1 full): total params don't equal compute. Memory matches total; speed matches active.

## Sampling defaults that work

| Use case | `temperature` | `top_p` | `repeat_penalty` |
|---|---|---|---|
| R1 / R1-distill — reasoning | 0.6 | 0.95 | 1.0 |
| Coder V2 — code generation | 0.2 | 0.9 | 1.05 |
| V3 — chat / general | 0.4 | 0.9 | 1.05 |
| JSON / tool calls (V3 / Coder V2) | 0.1 | 0.9 | 1.0 |

## Where to look for updates

- `ollama show deepseek-r1:14b` for the current template + params.
- [ollama.com/library/deepseek-r1](https://ollama.com/library/deepseek-r1), `/deepseek-coder-v2`, `/deepseek-v3`.
- [api-docs.deepseek.com](https://api-docs.deepseek.com/) for tool-call schema (works for the open-weight variants too).
- HuggingFace cards under `deepseek-ai/...` for canonical templates.
