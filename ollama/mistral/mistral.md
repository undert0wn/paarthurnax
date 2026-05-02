# Mistral family — governance approximation

> **Status (open-weight reality):** Mistral AI does **not** publish a vendor-blessed governance file convention. Governance lives in the Ollama `Modelfile` and in whichever client drives the model. This file is the closest reasonable approximation.

> **Verified:** snapshot-only. Re-verify with `ollama show mistral` etc.

## What's in the family

| Variant | Sizes | Notes |
|---|---|---|
| Mistral 7B | 7B | The original. Still a good baseline at Q5 on 16 GB. |
| Mistral-Nemo | 12B | **128k context.** Standout for long-document work on 16 GB VRAM. |
| Mistral-Small | 22B / 24B *(verify)* | Mid-tier general; closer to GPT-3.5-class on many tasks. |
| Mixtral 8x7B | MoE, ~12B active | Older MoE; Q4 ≈ 26 GB. Borderline for 32 GB systems. |
| Codestral | 22B | Coder-specialized. Strong **fill-in-the-middle**. |
| Devstral | 24B *(verify)* | **Agent-tuned** — trained for read/edit/test loops. Pair with Cline / OpenHands / Aider. |

Common Ollama tags: `mistral`, `mistral-nemo`, `mistral-small`, `mixtral:8x7b`, `codestral`, `devstral`.

## Where governance actually lives

1. **Modelfile `SYSTEM` block** (most reliable).
2. **Client rule files**.
3. **`AGENTS.md`** for cross-tool clients.

## Recommended Modelfile templates

### Mistral-Nemo (long context generalist)

```dockerfile
# paarthurnax/runtime/modelfiles/mistral-nemo-governed.Modelfile
FROM mistral-nemo:12b

PARAMETER temperature 0.3        # Nemo prefers low temp — high temp produces drift
PARAMETER top_p 0.9
PARAMETER repeat_penalty 1.05
PARAMETER num_ctx 32768          # 128k nominal; 32k is a safe high-quality window
PARAMETER stop "[/INST]"
PARAMETER stop "</s>"

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
"""
```

### Codestral / Devstral (coding + agent loops)

```dockerfile
# paarthurnax/runtime/modelfiles/codestral-governed.Modelfile
FROM codestral:22b

PARAMETER temperature 0.2
PARAMETER top_p 0.9
PARAMETER repeat_penalty 1.05
PARAMETER num_ctx 16384
PARAMETER stop "[/INST]"

SYSTEM """
You are a coding assistant that produces minimal, focused diffs. The seven Prime
Directives in paarthurnax/CONVENTIONS.md → *Prime Directives* always apply (workspace
boundary, diffs-before-writes, no secrets, one destructive try, option-picker,
[STUCK] hand-off, cite-the-rule). In addition:

1. Read existing files before editing.
2. When fill-in-the-middle is requested, return only the missing span — no
   surrounding code, no explanations.
3. For agent loops (Devstral), use the read/plan/edit/test cycle and report
   each step concisely.
"""
```

## Family-specific quirks

- **Chat template**: most Mistral variants use the `[INST] ... [/INST]` envelope. **Always include `[/INST]` and `</s>` as stop tokens.** Nemo and newer variants also support a v3 / v7 ChatML-style template — `ollama show <model>` to see which is active.
- **Tool calling**: Mistral-Small, Mistral-Nemo, Codestral, Devstral all support a JSON `[TOOL_CALLS]` token convention. Reasonably reliable on 12B+; check the model card for the exact wire format. Mixtral 8x7B has weak tool-calling — wrap your own JSON validator.
- **Codestral FIM**: send `[SUFFIX]` and `[PREFIX]` markers per the model card. Don't try to coerce FIM out of a chat template.
- **Devstral agent quirks**: trained on agent traces — works much better with explicit step-by-step protocols (read → plan → edit → verify) than with raw "do X" prompts. Pair with a real agent loop, not chat.
- **Context window**: Mistral 7B = 32k. Mistral-Nemo = 128k (best of the family for long docs). Codestral = 32k. Mixtral = 32k. `num_ctx` defaults in Ollama are usually conservative — bump to match.
- **License**: **Apache 2.0** for Mistral 7B, Mistral-Nemo, Mixtral, Mistral-Small (some sizes). **Mistral Research License (MRL)** for Codestral and some larger variants — non-commercial only. **Always check the specific Ollama tag's license** before commercial use.

## Sampling defaults that work

| Use case | `temperature` | `top_p` | `repeat_penalty` |
|---|---|---|---|
| Codestral / Devstral — code | 0.2 | 0.9 | 1.05 |
| Mistral-Nemo — long-doc analysis | 0.3 | 0.9 | 1.05 |
| Mistral 7B / Small — chat | 0.5 | 0.9 | 1.1 |
| Mixtral — chat | 0.5 | 0.95 | 1.1 |
| Tool calls / JSON | 0.1 | 0.9 | 1.0 |

## Where to look for updates

- `ollama show mistral-nemo:12b` etc. for the current template + params.
- [ollama.com/library/mistral](https://ollama.com/library/mistral), `/mistral-nemo`, `/codestral`, `/devstral`, `/mixtral`.
- [docs.mistral.ai](https://docs.mistral.ai/) for upstream tool-call schemas.
- [mistral.ai/news](https://mistral.ai/news/) for release notes that include prompt-template changes.
