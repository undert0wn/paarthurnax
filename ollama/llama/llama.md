# Llama family — governance approximation

> **Status (open-weight reality):** Meta does **not** publish a vendor-blessed governance file convention for Llama models. There is no `LLAMA.md` standard. Governance lives in (a) the **Ollama `Modelfile`** that defines the model's `SYSTEM` block, sampling defaults, and chat template, and (b) the **client** that drives the model (Continue, Cline, Aider, Cursor, etc.). This file is the closest reasonable approximation.

> **Verified:** snapshot-only. Re-verify with `ollama show llama3.1` etc. before relying on the specifics below.

## What's in the family

| Variant | Sizes | Notes |
|---|---|---|
| Llama 3.1 | 8B, 70B, 405B | The 8B is the practical workhorse on a 16 GB rig at Q4. 70B needs 48 GB+ VRAM or a quantized 32 GB-unified-memory Mac. |
| Llama 3.2 | 1B, 3B, 11B-Vision, 90B-Vision | 3B is great as a draft / autocomplete model. 11B-Vision adds image input. |
| Llama 3.3 | 70B-instruct | Same architecture as 3.1-70B with stronger instruction tuning. |
| Llama 4 *(if present in your Ollama lib)* | varies by tag | New MoE family; check the model card before assuming size = compute. |

Common Ollama tags: `llama3.1:8b`, `llama3.1:70b`, `llama3.2:3b`, `llama3.2:11b-vision`, `llama3.3:70b`.

## Where governance actually lives

Ranked by reliability:

1. **Modelfile `SYSTEM` block** — baked into every chat against the custom-tagged model. Most reliable.
2. **Client rule files** — `.continue/rules/*.md`, `.clinerules`, `.aider.conf.yml` `read:` directive, `.cursor/rules/*.mdc`, `.windsurfrules`, etc.
3. **`AGENTS.md`** at the workspace root if your client supports the cross-tool standard.
4. **Per-turn system prompt** override from your client UI — fragile, easy to forget.

## Recommended Modelfile template

```dockerfile
# paarthurnax/runtime/modelfiles/llama3.1-8b-governed.Modelfile
FROM llama3.1:8b

# Llama 3 chat template — Ollama ships the correct one by default.
# Override only if you know what you're doing.

PARAMETER temperature 0.4        # 0.2-0.4 for code, 0.6-0.8 for prose
PARAMETER top_p 0.9
PARAMETER repeat_penalty 1.05    # mild — Llama 3 doesn't loop much
PARAMETER num_ctx 8192           # 128k available; raise only if VRAM allows
PARAMETER stop "<|eot_id|>"
PARAMETER stop "<|end_of_text|>"

SYSTEM """
You are a careful coding and ops assistant operating inside the user's IDE workspace.

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

# Family-specific framing
Don't name internal tools to the user — say "I'll run a command in a terminal",
not the literal tool name.
"""
```

Build with `ollama create llama3.1-8b-governed -f paarthurnax/runtime/modelfiles/llama3.1-8b-governed.Modelfile`.

## Family-specific quirks

- **Chat template**: Llama 3 uses `<|begin_of_text|>`, `<|start_header_id|>role<|end_header_id|>`, and `<|eot_id|>` markers. **Always include `<|eot_id|>` as a stop token** or the model rambles past its turn.
- **Tool calling**: Llama 3.1 introduced **JSON-style tool calling** with explicit `<|python_tag|>` markers. The 70B is reliable; the 8B is okay but needs few-shot examples in the system prompt for unfamiliar tools.
- **Context window**: nominal 128k. Practical limit on consumer hardware is whatever VRAM holds — `num_ctx 8192` is safe; `16384`–`32768` works on a 16 GB GPU at Q4.
- **License**: **Llama Community License** — permissive for most uses but has a 700-million-MAU cap and an acceptable-use policy. Read [llama.com/llama3_1/license](https://www.llama.com/llama3_1/license/) before commercial deployment.
- **Multilinguality**: Llama 3.1+ officially supports English, German, French, Italian, Portuguese, Hindi, Spanish, Thai. Other languages work but quality drops noticeably.

## Sampling defaults that work

| Use case | `temperature` | `top_p` | `repeat_penalty` |
|---|---|---|---|
| Code generation | 0.2 | 0.9 | 1.05 |
| Code explanation / chat | 0.4 | 0.9 | 1.05 |
| Creative prose | 0.7 | 0.95 | 1.0 |
| JSON / structured output | 0.1 | 1.0 | 1.0 |

## Where to look for updates

- `ollama show llama3.1:8b` — current chat template, license, parameters as Ollama sees them.
- [ollama.com/library/llama3.1](https://ollama.com/library/llama3.1) and sister pages.
- [github.com/meta-llama](https://github.com/meta-llama) for upstream prompt-template updates.
- The model card on HuggingFace (`meta-llama/Meta-Llama-3.1-8B-Instruct` etc.) for the canonical chat-template Jinja.
