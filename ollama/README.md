# Ollama model-family reference

> **What this folder is.** One subfolder per open-weight model family that paa has actually-vetted notes for. Each `<family>/<family>.md` documents that family's chat-template quirks, recommended Ollama `Modelfile`, sampling defaults, and known tool-calling reliability. **It is not a how-to-run-Ollama doc** — it assumes you already have Ollama installed (see [ollama.com](https://ollama.com)).

> **Status.** Snapshot only. Open-weight model behavior shifts every release. Re-verify with `ollama show <model>` before relying on the specifics.

---

## How Ollama models reach an MCP server

Ollama itself doesn't speak MCP — it's a model runner. MCP is a client-side protocol; the **agent loop around the model** is what speaks it. Three working patterns today, easiest → hardest:

1. **Put the Ollama model behind an MCP-aware client.** Cline, Continue, Cursor, and Claude Code (via OpenAI-compatible proxies like LiteLLM) all support pointing at a local Ollama endpoint and they handle the MCP side themselves. **This is the path that "just works"** and matches *Flavor A* in [`../ARCHITECTURE.md`](../ARCHITECTURE.md) → *Local-model deployment*. If you have one of those clients installed, this is the recommended setup.
2. **Use a third-party MCP-host loop that wraps Ollama.** Projects like [`mcphost`](https://github.com/mark3labs/mcphost) (mark3labs), [`mcp-cli`](https://github.com/chrishayuk/mcp-cli), [`oterm`](https://github.com/ggozad/oterm), and [`ollmcp`](https://github.com/jonigl/mcp-client-for-ollama) are CLI/TUI loops built specifically to bridge a local Ollama model to MCP servers. They translate MCP `tools/list` into the model's tool-calling format, dispatch tool calls, feed results back. Quality varies wildly with model size.
3. **Roll your own loop** (*Flavor B* in `ARCHITECTURE.md`). Ollama's API supports OpenAI-style `tools` since ~v0.3. The loop fetches `tools/list` from each MCP server, hands those schemas to the model on each turn, parses the model's `tool_calls`, dispatches them to the right server, returns the results. This is what [`../CONVENTIONS.md`](../CONVENTIONS.md) → *Local-model loops* is implicitly setting up the contract for.

**Tool-calling reliability has a floor.** Sub-7B open-weight models spend most of their time recovering from malformed tool calls. The realistic "good enough" floor is **Llama 3.1 8B / Qwen 2.5 7B / Mistral Nemo**, with **14B+** the comfortable target for serious agent work.

---

## Families included

These are the five families that ship with vetted notes. Selection criterion: **proven to work end-to-end against the paa contract in actual local-model loops** (which is a slightly different bar than "native tool-calling tokens" — see Gemma's row).

| Family | Folder | Sweet spot | Why it's here |
|---|---|---|---|
| Qwen | [`qwen/`](qwen/qwen.md) | `qwen2.5-coder:14b` | Currently the strongest open-weight tool caller. Native function-calling tokens. The default recommendation. |
| DeepSeek | [`deepseek/`](deepseek/deepseek.md) | `deepseek-coder-v2:16b` / `deepseek-v3` | Strong, especially for code. V3 is a heavyweight MoE. |
| Gemma | [`gemma/`](gemma/gemma.md) | `gemma3:12b` / `gemma2:9b` | **No native tool-calling tokens** — but follows the prose-protocol action blocks (`[READ]`/`[WRITE]`/`[RUN]`) from `CONVENTIONS.md` more reliably than several models that *do* have native tokens. Empirically validated. See the family file for the constraint and the matching loop shape. |

**Recommended starting points:**
- **16 GB VRAM, general coding** → `qwen2.5-coder:14b` (Q4_K_M).
- **8 GB VRAM, willing to trade speed for quality** → `qwen2.5-coder:7b`, `llama3.1:8b`, or `gemma3:4b`.
- **Big rig (24 GB+)** → `qwen2.5-coder:32b`, `mistral-small:22b`, or `gemma3:27b`.

---

## Families intentionally not included

These were considered and removed. None are *bad* models — they're just not strong fits for the **MCP/agent-loop use case** paa targets, *and* (unlike Gemma) the author hasn't personally validated them against the paa contract. Listed here so future-you (or a contributor) doesn't waste time wondering why they're missing.

| Family | Why it was cut |
|---|---|
| Phi (3 / 4) | Small models, weak at structured tool calling. Good at reasoning benchmarks, less good at sustained multi-turn tool use. |
| SmolLM | Tiny by design (sub-2B). Below the agent-loop floor by definition. Useful for embedded / on-device use. |
| StarCoder (2) | Pure code-completion model. No instruction tuning, no chat, no tool calling. Use it via Continue's autocomplete role, not as an agent. |
| Yi | Older, less actively maintained, weaker tool-calling story. |
| GLM (4) | Has tool calling on paper, less proven in real-world agent loops. Worth revisiting in a future release. |
| Granite (IBM) | Mixed reports on tool-calling reliability. Strong on enterprise-flavored benchmarks. |
| Command-R / Command-R+ (Cohere) | **Purpose-built for RAG and tool use** — would otherwise be a top pick — but the smallest variant is 35B. Too heavy for the "hobby on a desktop" target. Add it back if you have the VRAM. |

### Adding a removed family back (or a brand-new one)

1. Create `ollama/<family>/<family>.md`.
2. Copy the structure from [`qwen/qwen.md`](qwen/qwen.md) — sections: *What's in the family*, *Where governance actually lives*, *Recommended Modelfile template*, *Sampling defaults*, *Known quirks*, *Tool-calling reliability* (be honest — if it's weak, say so).
3. Add a row to the *Families included* table above (or *Families intentionally not included* if it didn't make the floor and you want the rationale on record).
4. Update the *Model-family folders* section in [`../ARCHITECTURE.md`](../ARCHITECTURE.md).
5. If the family is genuinely below the agent-loop floor but useful in some other role (autocomplete, embedding, summarization), document the role explicitly so it doesn't get accidentally pointed at MCP and disappoint.

The bar for inclusion is **"I have personally driven this through an MCP loop and it didn't fall apart."** Anything else belongs in the not-included table with a one-line why.
