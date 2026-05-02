---
name: git
description: Use git from the terminal — clone, branch, commit, push, pull, merge, rebase, stash, diff, log, conflict resolution. Load whenever the user asks about version control or you need to inspect or modify the repo's git state.
---

# git essentials

## When to load this skill

Any task that touches `.git/` — committing, branching, fetching, comparing histories, resolving merge conflicts, inspecting blame/log, working with remotes, or recovering from "I think I broke git".

## Quick reference

### Inspect state (read-only, always safe)
| Command | What it does |
|---|---|
| `git status` | What's staged, unstaged, untracked. **Run this first, every time.** |
| `git status -sb` | Compact form with branch tracking info. |
| `git log --oneline -20` | Last 20 commits, one line each. |
| `git log --oneline --graph --all -30` | Visual graph of all branches. |
| `git diff` | Unstaged changes vs working tree. |
| `git diff --staged` | Staged changes vs HEAD. |
| `git diff <branch1>..<branch2>` | Difference between two refs. |
| `git show <sha>` | Full diff of one commit. |
| `git blame <file>` | Who last touched each line. |
| `git branch -vv` | All local branches + tracking remotes + last commit. |
| `git remote -v` | Configured remotes. |

### Common workflows

**Make a branch and commit:**
```sh
git switch -c feature/my-thing      # modern; replaces `git checkout -b`
# ...edit files...
git add -p                          # interactive add — review hunks one by one
git commit -m "feat: add my thing"
git push -u origin feature/my-thing # -u sets upstream tracking
```

**Update from remote:**
```sh
git fetch origin                    # download refs without changing working tree
git pull --ff-only                  # fast-forward only — refuses if a merge would be needed
git pull --rebase                   # rebase local commits on top of fetched ones
```

**Clean rewind / undo:**
```sh
git restore <file>                  # discard unstaged changes to a file
git restore --staged <file>         # unstage but keep changes
git reset HEAD~1 --soft             # undo last commit, keep changes staged
git reset HEAD~1 --mixed            # undo last commit, unstage changes (default)
git reset HEAD~1 --hard             # ⚠ undo last commit AND discard changes
git revert <sha>                    # create a NEW commit that undoes <sha> (safe, public-history-friendly)
```

**Stash:**
```sh
git stash push -m "wip notes"       # save current changes
git stash list                      # see saved stashes
git stash pop                       # apply most recent and drop it
git stash apply stash@{1}           # apply a specific stash without dropping
```

**Rebase (rewrite local history before sharing):**
```sh
git fetch origin
git rebase origin/main              # replay your branch on top of latest main
git rebase -i HEAD~5                # interactive — squash, reorder, reword last 5 commits
```
⚠ Don't rebase commits that have been pushed and pulled by others.

**Resolve a merge conflict:**
```sh
git status                          # find conflicted files
# edit each file, look for <<<<<<< / ======= / >>>>>>> markers
git add <file>                      # mark resolved
git commit                          # finish the merge (or `git rebase --continue`)
git merge --abort                   # bail out entirely
```

### Destructive commands — handle with care

| ⚠ Command | What it does | Safer alternative |
|---|---|---|
| `git push --force` | Overwrites remote history. Can destroy others' work. | `git push --force-with-lease` — fails if someone else pushed since you fetched. |
| `git reset --hard <sha>` | Throws away uncommitted changes AND moves HEAD. | `git stash` first; `git reset --keep` to refuse if uncommitted changes exist. |
| `git clean -fd` | Deletes untracked files and dirs. **Cannot undo.** | `git clean -nd` first to preview. |
| `git branch -D <name>` | Force-deletes a branch even if unmerged. | `git branch -d <name>` (lowercase d) refuses if unmerged. |
| `git checkout <sha> -- <file>` | Overwrites local file with version from `<sha>`. | Read with `git show <sha>:<file>` first. |

## Recovery — "I think I broke git"

`git reflog` is the single most underused recovery tool. It shows **every move HEAD has made** in the last 90 days, including resets and destroyed commits.

```sh
git reflog                          # see what HEAD has done recently
git reset --hard HEAD@{2}           # rewind HEAD to where it was 2 moves ago
```

If `git reflog` shows the commit you want, you can almost always recover.

## Gotchas

- **Windows line endings**: `core.autocrlf=true` (Git for Windows default) rewrites line endings on checkout/commit. Often invisible until a CI lint job fails. Set per-repo via `.gitattributes`.
- **Case-insensitive filesystems**: macOS and Windows filesystems treat `Foo.md` and `foo.md` as the same file; Linux doesn't. Renames that only change case need `git mv -f`.
- **Submodules**: `git clone --recurse-submodules`; `git submodule update --init --recursive` after pulling. Forgetting this is the #1 cause of "missing files" complaints.
- **`HEAD` vs `origin/HEAD`**: `HEAD` is your local current ref; `origin/HEAD` is what *origin's* default branch was when you last fetched. They drift.
- **`git pull` without `--rebase` or `--ff-only`** creates merge commits silently. Configure once:
  ```sh
  git config --global pull.ff only
  ```

## Where to learn more

- `git help <command>` — opens the full man page (e.g. `git help rebase`).
- `git <command> --help` — same thing; works on Windows/Linux/macOS.
- `git <command> -h` — short usage summary, no pager.
- [git-scm.com/docs](https://git-scm.com/docs) — official reference; same content as `git help`.
- [git-scm.com/book](https://git-scm.com/book/en/v2) — *Pro Git*, free book. Chapters 2 (basics), 3 (branching), and 7 (tools) cover 90% of daily use.
- [oh-my-git.com](https://ohmygit.org/) — interactive game for branching/merging/rebasing.
- Stack Overflow tag [`git`](https://stackoverflow.com/questions/tagged/git) — almost any error message has a top-voted answer there.
