---
name: tool-discovery
description: Figure out how to use any unfamiliar CLI — --help, man, info, tldr, Get-Help, version flags, completions, finding the right docs online. Load when the user asks about a tool you don't know, or when you need to discover what flags or subcommands a tool supports.
---

# Tool discovery — "what can this CLI do?"

## When to load this skill

Whenever you encounter a CLI you don't already know inside-out, or whenever the user asks "how do I X with `<tool>`?" and you'd rather check than guess. Also: when an existing skill is silent on a specific flag and you need the authoritative answer.

## The discovery ladder — try in this order

### 1. The tool's own help (always first; works offline; always current)

| You're on… | Try, in order |
|---|---|
| **Any OS, modern CLI** | `<tool> --help` — usually the most useful single command. |
| | `<tool> -h` — short form; some tools differ between `-h` and `--help`. |
| | `<tool> help` — subcommand-style (git, gh, docker, kubectl, cargo, go). |
| | `<tool> help <subcommand>` — drill into a specific subcommand. |
| | `<tool> <subcommand> --help` — same thing, different syntax. |
| **Linux / macOS** | `man <tool>` — full manual page. `q` quits, `/pattern` searches, `n`/`N` next/prev match. |
| | `info <tool>` — GNU info; longer, hyperlinked. |
| | `apropos <keyword>` — search man-page descriptions. |
| **Windows PowerShell** | `Get-Help <cmdlet>` — built-in. Add `-Full`, `-Examples`, `-Online`, `-ShowWindow`. |
| | `Get-Command <name>` — confirms the cmdlet exists; `Get-Command *foo*` for wildcard search. |
| | `<obj> \| Get-Member` — discover what properties/methods a piped object has. |

### 2. Practical-examples shortcut

[`tldr`](https://tldr.sh/) is a community-maintained collection of *examples-only* pages for ~6,000 tools. Often more useful than `man` when you just need the canonical command:

```sh
tldr git                                     # examples for git
tldr docker run                              # examples for a subcommand
tldr -u                                      # update local cache
```

Install: `npm i -g tldr` / `brew install tldr` / `pip install tldr` / `cargo install tealdeer`.

### 3. Version + capabilities

Knowing the version unlocks the right docs page:

```sh
<tool> --version
<tool> -V
<tool> version          # subcommand form (go, kubectl, terraform)
```

Some tools list installed plugins / extensions:
```sh
git --list-cmds=main,others,alias
gh extension list
kubectl plugin list
docker info
```

### 4. Shell completions — they reveal subcommands

Tab-completion is documentation. Most modern CLIs ship completion scripts:

```sh
<tool> completion bash > /etc/bash_completion.d/<tool>     # bash
<tool> completion zsh  > "${fpath[1]}/_<tool>"             # zsh
<tool> completion fish | source                             # fish
<tool> completion powershell | Out-String | Invoke-Expression   # PowerShell
```

Once installed, hitting `<TAB><TAB>` after the tool name lists every subcommand and flag — instant discovery.

### 5. Source / official docs

| Resource | Use when |
|---|---|
| `<tool> --help` says "see <URL>" | Visit it. Tools usually point at their canonical doc. |
| GitHub README + `docs/` folder | Most CLIs link to source from `--version` or `--help`. |
| Vendor docs (e.g. docs.docker.com, learn.microsoft.com, cloud.google.com/sdk) | Authoritative reference + tutorials. |
| Tool's GitHub repo — `AGENTS.md` / `CONTRIBUTING.md` / `CHANGELOG.md` | Tells you how it's *supposed* to be used, often with examples agents miss. |
| Release notes | Find the release that introduced or changed a flag. |

### 6. Community knowledge (when official docs fall short)

- **Stack Overflow** tag for the tool — `[git]`, `[docker]`, `[kubernetes]`, etc. Top-voted answers age well.
- **GitHub issues** in the tool's repo — search for the error message verbatim. The maintainers' responses are gold.
- **awesome-* lists** on GitHub — curated link directories (e.g. `awesome-docker`, `awesome-python`).
- [explainshell.com](https://explainshell.com/) — paste any shell command, get an annotated breakdown of every flag.

## Tactics for fast discovery

**Search the help output instead of reading top-to-bottom:**
```sh
<tool> --help | grep -i "<keyword>"          # bash / zsh
<tool> --help | Select-String "<keyword>"    # PowerShell
```

**Find a tool you can't remember the name of:**
```sh
apropos "<keyword>"                          # man-page descriptions (Linux/macOS)
which <tool>                                 # is it on PATH? (POSIX)
where.exe <tool>                             # Windows
Get-Command -Module *                        # all PowerShell cmdlets
```

**Inspect what a binary actually is:**
```sh
file $(which <tool>)                         # binary type, language, arch
ldd $(which <tool>)                          # dynamic libs (Linux)
otool -L $(which <tool>)                     # dynamic libs (macOS)
```

**Trace what a tool does at runtime** (advanced — when docs lie):
```sh
strace -f -e openat <tool>                   # Linux: which files does it read?
dtrace / dtruss <tool>                       # macOS equivalent
Procmon                                      # Windows GUI (Sysinternals)
```

## When the help is bad or missing

Some older tools have terse or wrong help text. In that order:
1. `man <tool>` — often more thorough than `--help`.
2. The tool's GitHub repo `README.md`.
3. The tool's source: `<tool>-source/cmd/` or wherever flags are parsed. CLI tools written in Go, Rust, Cobra-based ones almost always have a `cmd/` tree readable like documentation.
4. Search the issue tracker for `"flag X"` or the exact symptom.

## Checklist for "I just learned a new tool"

When you've used a tool in earnest for the first time, capture:
- The 3–5 commands you actually used.
- The one gotcha that surprised you.
- The URL of the official docs page.

If it's something you'll use again, that's a candidate for a new file in [`../`](../README.md). Skills are cheap; lookup time isn't.

## Where to learn more

- `man man` — yes, `man` has a man page. `man 7 man-pages` describes the conventions.
- [tldr.sh](https://tldr.sh/) — the practical-examples project.
- [explainshell.com](https://explainshell.com/) — visual command parser.
- [cheat.sh](https://cheat.sh/) — `curl cheat.sh/<tool>` for quick examples in any terminal.
- [devhints.io](https://devhints.io/) — concise cheatsheets for ~400 topics.
- [pkg.go.dev](https://pkg.go.dev/), [docs.rs](https://docs.rs/), [npmjs.com](https://www.npmjs.com/), [pypi.org](https://pypi.org/) — package registries with READMEs and source links for the underlying libraries CLIs are built from.
