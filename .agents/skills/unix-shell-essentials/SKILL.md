---
name: unix-shell-essentials
description: Use bash or zsh from the terminal on macOS or Linux — pipelines, ls/grep/find/sed/awk, redirection, common idioms, scripting basics. Load when the user is working on macOS, Linux, WSL, or any POSIX shell environment.
---

# Unix shell essentials (bash / zsh)

## When to load this skill

Any task on macOS, Linux, WSL, or in a CI runner running bash/zsh. Pair with [`powershell-essentials`](../powershell-essentials/SKILL.md) for cross-platform work.

## Quick reference

### Files and directories
| Command | What it does |
|---|---|
| `pwd` | Print current dir. |
| `ls -lah` | Long form, all (incl. hidden), human sizes. |
| `cd -` | Jump to previous dir. |
| `mkdir -p a/b/c` | Make dirs incl. parents. |
| `cp -r src dst` | Copy recursively. |
| `mv src dst` | Move / rename. |
| `rm -rf foo` | ⚠ Recursive force delete. **Cannot undo.** Run `ls foo` first. |
| `touch file` | Create empty file or update mtime. |
| `ln -s target link` | Create symlink. |
| `stat file` | Detailed file info. |
| `du -sh *` | Size of each item, human-readable. |
| `df -h` | Disk free, human-readable. |
| `tree -L 2` | Directory tree (often needs install). |

### Viewing and slicing files
| Command | What it does |
|---|---|
| `cat file` | Print whole file. |
| `less file` | Pager — `q` quits, `/pattern` searches, `g`/`G` to top/bottom. |
| `head -n 20 file` | First 20 lines. |
| `tail -n 20 file` | Last 20 lines. |
| `tail -f file` | Follow as it grows. |
| `wc -l file` | Line count. `-w` words, `-c` bytes. |
| `sort file \| uniq -c \| sort -rn` | Frequency-count classic. |
| `cut -d, -f1,3 file.csv` | Pick columns from CSV. |
| `column -t -s, file.csv` | Pretty-print CSV. |

### Search
```sh
grep "pattern" file                         # search a file
grep -r "pattern" .                          # recursive
grep -rn "pattern" .                         # with line numbers
grep -ri "pattern" --include="*.py" .        # case-insensitive, restrict to Python
grep -v "pattern" file                       # invert: lines NOT matching
grep -E "foo|bar" file                       # extended regex (alternation)
rg "pattern"                                 # ripgrep — much faster, recursive by default, respects .gitignore
fd "pattern"                                 # fd — fast `find` replacement, friendlier syntax
```

### `find` — locate files by attributes
```sh
find . -name "*.log"                         # by name
find . -iname "*.LOG"                        # case-insensitive
find . -type f -size +100M                   # files larger than 100 MB
find . -type f -mtime -1                     # modified in last 24h
find . -type d -name node_modules -prune -o -type f -print   # skip node_modules
find . -name "*.tmp" -delete                 # delete matches (be careful)
find . -name "*.log" -exec gzip {} \;        # run a command per match
find . -name "*.log" -exec gzip {} +         # batch — pass many to one gzip call
```

### Stream editing
```sh
# sed — substitute
sed 's/foo/bar/g' file                       # print with substitutions
sed -i 's/foo/bar/g' file                    # in-place edit (Linux)
sed -i '' 's/foo/bar/g' file                 # in-place edit (macOS — note empty quotes)

# awk — field-aware processing
awk '{print $1, $3}' file                    # print 1st and 3rd whitespace-separated columns
awk -F, '{sum += $2} END {print sum}' f.csv  # sum column 2 of CSV
awk 'NR==FNR{a[$1]; next} $1 in a' a.txt b.txt   # lines in b.txt whose first field is in a.txt

# tr — character translation
echo "Hello" | tr 'a-z' 'A-Z'                # uppercase
tr -d '\r' < dos.txt > unix.txt              # strip Windows CRs
```

### Pipes, redirects, substitution
```sh
cmd1 | cmd2                                  # pipe stdout → stdin
cmd > file                                   # redirect stdout (overwrite)
cmd >> file                                  # redirect stdout (append)
cmd 2> err.log                               # redirect stderr
cmd > out.log 2>&1                           # both to same file
cmd 2>&1 | tee log.txt                       # capture both AND see on screen
cmd < input.txt                              # feed file as stdin
$(cmd)                                       # command substitution — capture output
$(<file)                                     # read file content (faster than $(cat file))
<(cmd)                                       # process substitution — pass output as a "file"
diff <(sort a.txt) <(sort b.txt)             # diff two sorted streams
```

### Processes and signals
```sh
ps aux                                       # all processes
ps aux | grep node                           # find by name (use pgrep instead)
pgrep -fl node                               # process IDs matching "node", with command line
pkill -f "node server.js"                    # kill matches
kill 1234                                    # SIGTERM (polite)
kill -9 1234                                 # ⚠ SIGKILL (cannot be caught)
top                                          # live process view
htop                                         # nicer top (often needs install)
jobs; fg %1; bg %1                           # background jobs
nohup cmd &                                  # run detached, immune to hangup
```

### Networking
```sh
curl -fsSL https://example.com               # fetch (fail-on-error, silent, follow-redirects, location)
curl -X POST -H 'Content-Type: application/json' -d '{"a":1}' https://api/...
wget https://example.com/file.zip            # download
ssh user@host                                # remote shell
scp file user@host:/path                     # remote copy
rsync -avz src/ user@host:/path/             # incremental sync
ping example.com
dig example.com                              # DNS lookup
nc -zv example.com 443                       # netcat — port check
```

### Permissions
```sh
chmod +x script.sh                           # make executable
chmod 644 file                               # rw- r-- r--
chmod 755 dir                                # rwx r-x r-x
chown user:group file
sudo cmd                                     # run as root (with prompt for password)
```

## Common workflows

**Count log lines per status code:**
```sh
awk '{print $9}' access.log | sort | uniq -c | sort -rn | head
```

**Find and delete empty dirs:**
```sh
find . -type d -empty -delete
```

**Search code with context, exclude vendor dirs:**
```sh
rg "TODO" --type py --glob '!**/node_modules/**' --glob '!**/.venv/**' -C 2
```

**Tail two logs at once with labels:**
```sh
tail -F app.log error.log
```

**Tar + gzip a directory:**
```sh
tar -czf archive.tar.gz some-dir/
tar -tzf archive.tar.gz                      # list contents
tar -xzf archive.tar.gz                      # extract
```

## Scripting basics

```sh
#!/usr/bin/env bash
set -euo pipefail        # exit on error, undefined var, pipe failure
IFS=$'\n\t'              # safer word-splitting

# Variables (no spaces around =)
name="world"
echo "Hello, $name"

# Conditionals
if [[ -f "$file" ]]; then
    echo "exists"
elif [[ -d "$file" ]]; then
    echo "is a dir"
else
    echo "missing"
fi

# Loops
for f in *.log; do
    echo "Processing $f"
done

while IFS= read -r line; do
    echo "Line: $line"
done < input.txt

# Functions
greet() {
    local who="$1"
    echo "Hi, $who"
}
greet "Alice"
```

## Gotchas

- **`set -e` doesn't catch everything**: pipelines need `set -o pipefail`; subshells and command substitutions have their own rules. The `set -euo pipefail` triple is the standard "strict mode" opener.
- **Quote your variables**: `"$file"` not `$file`. Unquoted vars word-split on whitespace and glob-expand. Bites everyone eventually.
- **`[ ]` vs `[[ ]]`**: in bash, prefer `[[ ]]` — it doesn't word-split, supports `&&` / `||`, and has regex matching (`=~`). `[ ]` is POSIX-portable but trickier.
- **macOS `sed` ≠ GNU `sed`**: macOS ships BSD sed (`-i ''` requires empty arg). On macOS, `brew install gnu-sed` and use `gsed` for scripts you want portable.
- **macOS `bash` is 3.2** (from 2007, GPL2 reasons). Modern features (associative arrays, `mapfile`) need `brew install bash` and `#!/usr/bin/env bash` shebang. Or write for `zsh` (default macOS shell).
- **`rm -rf "$var"` with empty `$var` becomes `rm -rf` of cwd**. Always check: `[[ -n "$var" ]] || exit 1` before destructive ops.
- **`cd` in a subshell doesn't affect the parent**: `(cd foo && do_stuff)` returns to original dir afterward — useful idiom.
- **Glob doesn't match dotfiles by default**: enable with `shopt -s dotglob` (bash) or `setopt globdots` (zsh).
- **Exit codes**: `0` = success, non-zero = failure. `$?` holds the last exit code. Most tools return `1` for general errors, `2` for usage errors.

## Modern replacements worth installing

| Classic | Modern alternative | Why |
|---|---|---|
| `grep` | [`ripgrep`](https://github.com/BurntSushi/ripgrep) (`rg`) | Much faster, recursive by default, respects `.gitignore`. |
| `find` | [`fd`](https://github.com/sharkdp/fd) | Friendlier syntax, much faster, smart defaults. |
| `cat` | [`bat`](https://github.com/sharkdp/bat) | Syntax-highlighted, paginated, git-aware. |
| `ls` | [`eza`](https://github.com/eza-community/eza) | Colorful, git-aware, tree mode built in. |
| `top` | [`htop`](https://htop.dev/), [`btop`](https://github.com/aristocratos/btop) | Interactive, prettier. |
| `du` | [`dust`](https://github.com/bootandy/dust) | Visual disk-usage tree. |
| `man` | [`tldr`](https://tldr.sh/) | Practical examples instead of full reference. |

## Where to learn more

- `man <command>` — full reference page. `q` quits, `/pattern` searches.
- `<command> --help` or `-h` — short usage summary.
- `info <command>` — GNU's hyperlinked manuals (often more detail than `man`).
- `tldr <command>` — practical examples, community-maintained.
- `apropos <keyword>` — find man pages mentioning a keyword.
- [explainshell.com](https://explainshell.com/) — paste any command, get an annotated breakdown.
- [tldp.org](https://tldp.org/) — Linux Documentation Project; the *Bash Guide for Beginners* and *Advanced Bash-Scripting Guide* are classics.
- [mywiki.wooledge.org/BashFAQ](https://mywiki.wooledge.org/BashFAQ) — the BashFAQ; answers most "why doesn't this work" questions.
- [shellcheck.net](https://www.shellcheck.net/) — paste a script, get linter warnings. Run locally with `shellcheck script.sh`.
