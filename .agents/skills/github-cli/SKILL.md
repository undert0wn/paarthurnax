---
name: github-cli
description: Use the GitHub CLI (gh) from the terminal — pull requests, issues, releases, repo management, gists, workflow runs, auth. Load when the user wants to work with GitHub without leaving the terminal.
---

# GitHub CLI (`gh`) essentials

## When to load this skill

Any GitHub task the user wants done from the terminal: opening / reviewing / merging PRs, creating or commenting on issues, viewing CI runs, cutting releases, working with gists, or scripting GitHub interactions. Pair with the [`git`](../git/SKILL.md) skill for the local-VCS half.

## Setup (one-time)

```sh
gh auth login                 # interactive — pick GitHub.com or Enterprise, HTTPS or SSH
gh auth status                # confirm you're signed in
gh repo set-default           # set the default repo for the current clone
```

If you'll script `gh` in CI, use `gh auth login --with-token < tokenfile` or set `GH_TOKEN` / `GITHUB_TOKEN`.

## Quick reference

### Pull requests
| Command | What it does |
|---|---|
| `gh pr create` | Interactive PR creation from the current branch. |
| `gh pr create --fill` | Auto-fill title/body from commit messages. |
| `gh pr create --draft` | Open as draft. |
| `gh pr list` | Open PRs in current repo. |
| `gh pr list --author "@me"` | Just yours. |
| `gh pr view <num>` | PR details in terminal. |
| `gh pr view <num> --web` | Open in browser. |
| `gh pr checkout <num>` | Check out the PR's branch locally. |
| `gh pr diff <num>` | See the diff without checking out. |
| `gh pr review <num> --approve` | Approve. |
| `gh pr review <num> --comment -b "msg"` | Comment without approval. |
| `gh pr review <num> --request-changes -b "..."` | Request changes. |
| `gh pr merge <num> --squash --delete-branch` | Squash-merge and clean up. |
| `gh pr ready <num>` | Mark a draft as ready. |
| `gh pr status` | Your PRs across repos. |

### Issues
| Command | What it does |
|---|---|
| `gh issue create` | Interactive issue creation. |
| `gh issue create --title "..." --body "..." --label bug` | Scripted form. |
| `gh issue list` | Open issues. |
| `gh issue list --label "good first issue"` | Filter by label. |
| `gh issue view <num>` | Issue details. |
| `gh issue comment <num> -b "msg"` | Add a comment. |
| `gh issue close <num>` / `gh issue reopen <num>` | Close / reopen. |

### Repos
```sh
gh repo create <name> --public --source=. --push   # turn current dir into a new GitHub repo
gh repo clone <owner>/<repo>                        # clone (uses your auth)
gh repo view --web                                  # open current repo in browser
gh repo fork <owner>/<repo> --clone                 # fork and clone
```

### Releases
```sh
gh release create v1.2.0 --generate-notes           # auto-generate notes from PRs since last tag
gh release create v1.2.0 ./dist/* --generate-notes  # also upload artifacts
gh release list
gh release view v1.2.0
gh release download v1.2.0 --pattern "*.zip"
```

### Workflow runs (GitHub Actions)
```sh
gh run list                          # recent workflow runs
gh run list --workflow=ci.yml        # filter by workflow file
gh run view <run-id>                 # details
gh run view <run-id> --log           # full logs
gh run view <run-id> --log-failed    # only failed step logs
gh run watch <run-id>                # live-tail until done
gh run rerun <run-id>                # re-run failed jobs
```

### Gists
```sh
gh gist create file.txt --public --desc "demo"
gh gist list
gh gist view <id>
gh gist clone <id>
```

## Common workflows

**Open a PR for the branch you just pushed:**
```sh
git push -u origin HEAD
gh pr create --fill --web    # --fill uses commit messages, --web opens browser to confirm
```

**Review a teammate's PR locally:**
```sh
gh pr checkout 123           # creates local branch tracking the PR
# ...test it...
gh pr review 123 --approve   # or --request-changes -b "..."
```

**Address Copilot or human comments on a PR:**
```sh
gh pr view 123 --comments    # see all review threads
# ...edit, commit, push...
gh pr comment 123 -b "Addressed in <new-sha>"
```

**Cut a release after merging:**
```sh
git tag v1.2.0 && git push origin v1.2.0
gh release create v1.2.0 --generate-notes
```

## JSON output for scripting

Almost every `gh` command supports `--json` + `--jq`:
```sh
gh pr list --json number,title,author --jq '.[] | "\(.number) \(.author.login) \(.title)"'
gh run list --json conclusion,name,createdAt --jq '.[] | select(.conclusion=="failure")'
```

`gh api` is the universal escape hatch:
```sh
gh api repos/:owner/:repo/issues --paginate --jq '.[].title'
gh api -X POST repos/:owner/:repo/issues -f title="bug" -f body="..."
```

## Gotchas

- **Two-factor auth**: HTTPS push needs a personal access token, not your password. `gh auth login` handles this for you; don't paste your password into git prompts.
- **`gh` auth vs `git` auth are separate** — `gh auth login` configures `gh`; you may also need `gh auth setup-git` to make `gh` provide credentials to plain `git push`.
- **Default repo confusion**: in a clone with multiple remotes (origin + upstream), `gh` uses the first one it finds. `gh repo set-default` to be explicit.
- **Rate limits**: `gh api` shares your user rate limit. Use `--paginate` carefully on large repos.
- **`--web` flag is your friend**: when terminal output is too dense, append `--web` to most read commands to open the browser version.

## Where to learn more

- `gh help` — top-level command list.
- `gh <command> --help` — detailed help for any subcommand.
- `gh <command> --help | less` — for the long ones.
- [cli.github.com/manual](https://cli.github.com/manual/) — full online manual.
- [docs.github.com/en/rest](https://docs.github.com/en/rest) — REST API reference (for `gh api` calls).
- [github.com/cli/cli](https://github.com/cli/cli) — source + issue tracker; check here when behavior is unexpected.
