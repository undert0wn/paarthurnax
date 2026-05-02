---
name: powershell-essentials
description: Use Windows PowerShell (5.1) and PowerShell 7+ from the terminal — pipelines, Get-* discovery, common cmdlets, file/process/string operations, scripting basics. Load when the user is working on Windows or in a cross-platform PowerShell environment.
---

# PowerShell essentials

## When to load this skill

Any task on Windows where the shell is PowerShell, or any cross-platform task where the user has chosen `pwsh` (PowerShell 7+). Pair with [`unix-shell-essentials`](../unix-shell-essentials/SKILL.md) for cross-platform context.

## Two PowerShells

| Edition | Binary | Where it ships | Notes |
|---|---|---|---|
| Windows PowerShell 5.1 | `powershell.exe` | Built into Windows 10/11. | Last 5.x version; no new features, only security fixes. .NET Framework only. |
| PowerShell 7+ | `pwsh.exe` (or `pwsh` on macOS/Linux) | Separate install ([github.com/PowerShell/PowerShell](https://github.com/PowerShell/PowerShell)). | Cross-platform, .NET 8+, all new features. **Default recommendation for new work.** |

`$PSVersionTable.PSVersion` tells you which one is running.

## The single most important idea: pipelines pass *objects*

`bash` pipelines pass **bytes**. PowerShell pipelines pass **typed .NET objects**. This means you don't parse text — you access properties:

```powershell
# Get the 5 largest files in the current dir, recursively, by size
Get-ChildItem -Recurse -File | Sort-Object Length -Descending | Select-Object -First 5 Name, Length

# Find all .log files modified in the last 24h
Get-ChildItem -Recurse -Filter *.log | Where-Object { $_.LastWriteTime -gt (Get-Date).AddHours(-24) }

# Convert a JSON file to objects, filter, and back
Get-Content data.json | ConvertFrom-Json | Where-Object status -eq "active" | ConvertTo-Json
```

`$_` (or `$PSItem`) is the current pipeline object inside `Where-Object` / `ForEach-Object`.

## Quick reference

### Discovery (the *Get-* family)
| Command | What it does |
|---|---|
| `Get-Command <name>` | Find a cmdlet, function, or executable. `gcm` is the alias. |
| `Get-Command *user*` | Wildcard search — find anything matching. |
| `Get-Help <cmdlet>` | Help for a cmdlet. Add `-Full`, `-Examples`, `-Online`. |
| `Get-Help <cmdlet> -ShowWindow` | Help in a separate searchable GUI window. |
| `Update-Help` | Download latest help (one-time per machine). |
| `Get-Member` | Show properties + methods of whatever's piped in. **The discovery superpower.** |
| `<obj> \| Get-Member` | What can I do with this thing? |

### Files and directories
```powershell
Get-ChildItem                         # ls equivalent. Aliases: gci, ls, dir
Get-ChildItem -Recurse -File          # files only, recursive
Get-ChildItem -Filter *.log           # filter (faster than -Include for simple patterns)
Get-Item file.txt                     # single item info
Get-Content file.txt                  # cat equivalent. Alias: cat, gc
Get-Content file.log -Tail 50         # last 50 lines (like tail)
Get-Content file.log -Wait -Tail 0    # tail -f
Set-Content file.txt "new content"    # overwrite
Add-Content file.txt "append line"    # append
New-Item -ItemType Directory -Path foo  # mkdir
Remove-Item -Recurse -Force foo       # ⚠ rm -rf equivalent. Alias: rm, del
Copy-Item src dst -Recurse            # cp -r
Move-Item src dst                     # mv
Rename-Item old.txt new.txt
Test-Path foo.txt                     # returns $true / $false
Resolve-Path .\foo                    # absolute path
```

### Searching
```powershell
Select-String -Path *.log -Pattern "error"           # grep equivalent
Select-String -Path *.log -Pattern "error" -Context 2,3   # 2 lines before, 3 after
Get-ChildItem -Recurse -Filter *.cs | Select-String "TODO"  # recursive grep
```

### Filtering and shaping
```powershell
... | Where-Object { $_.Status -eq "Running" }       # filter. Alias: ?, where
... | Where-Object Status -eq "Running"              # short form (one property)
... | Sort-Object Name                                # sort. Alias: sort
... | Sort-Object Length -Descending
... | Select-Object Name, Length                      # pick columns. Alias: select
... | Select-Object -First 10                         # head
... | Select-Object -Last 5                           # tail (after collection)
... | ForEach-Object { $_.Name.ToUpper() }            # map. Aliases: %, foreach
... | Group-Object Status                             # group by property
... | Measure-Object Length -Sum -Average             # stats
... | Format-Table Name, Length -AutoSize             # pretty print
... | Format-List *                                   # all properties as list
```

### Processes and services
```powershell
Get-Process                                # ps equivalent. Alias: ps
Get-Process node                           # filter by name
Stop-Process -Name node                    # kill
Get-Service                                # services list
Start-Service <name>; Stop-Service <name>; Restart-Service <name>
```

### Networking
```powershell
Invoke-WebRequest https://example.com -OutFile out.html      # curl equivalent. Alias: iwr, wget
Invoke-RestMethod https://api.example.com/users              # auto-parses JSON. Alias: irm
Test-NetConnection example.com -Port 443                     # ping + port check
Resolve-DnsName example.com
```

### Variables, environment, and paths
```powershell
$x = 42                                # variable
$env:PATH                              # read env var
$env:MY_VAR = "hello"                  # set for current session
[Environment]::SetEnvironmentVariable("MY_VAR", "hello", "User")  # persist for user

# Common path conveniences:
$HOME           # user home
$PSScriptRoot   # directory of the running script
$PWD            # current dir
```

### Exporting / importing data
```powershell
... | Export-Csv out.csv -NoTypeInformation
Import-Csv data.csv | ...
... | ConvertTo-Json -Depth 10
Get-Content data.json | ConvertFrom-Json
... | Export-Clixml out.xml          # round-trip-safe object serialization
```

## Common workflows

**Find all files larger than 100 MB in a tree:**
```powershell
Get-ChildItem -Recurse -File | Where-Object Length -gt 100MB | Sort-Object Length -Descending
```

**Tail-follow a log:**
```powershell
Get-Content app.log -Wait -Tail 0
```

**Bulk rename:**
```powershell
Get-ChildItem *.txt | Rename-Item -NewName { $_.Name -replace '\.txt$', '.md' }
```

**HTTP API call:**
```powershell
$resp = Invoke-RestMethod -Uri https://api.github.com/repos/cli/cli -Method Get
$resp.stargazers_count
```

**Run a quick script with strict mode:**
```powershell
Set-StrictMode -Version Latest
$ErrorActionPreference = 'Stop'   # turn warnings/non-terminating errors into hard stops
```

## Gotchas

- **Aliases lie**: `ls`, `cat`, `wget`, `curl` in PowerShell are aliases to *PowerShell cmdlets*, not the Unix tools. `wget` is `Invoke-WebRequest` — different flag syntax. In scripts, **use full cmdlet names**.
- **Quoting**: single quotes are literal; double quotes interpolate (`"$var"`). Backtick `` ` `` is the escape character (not `\`). Newline-continuation uses backtick at end-of-line — fragile; prefer splatting or a here-string.
- **Execution policy**: by default, scripts won't run. One-time per user: `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`. CI uses `-ExecutionPolicy Bypass`.
- **Errors**: PowerShell has *terminating* and *non-terminating* errors. `try/catch` catches only terminating. Set `$ErrorActionPreference = 'Stop'` (or `-ErrorAction Stop` per call) to convert.
- **`&&` and `\|\|`**: only work in PowerShell 7+. In 5.1, chain with `;` and check `$?` or `$LASTEXITCODE`.
- **Path separators**: PowerShell accepts `/` and `\` on Windows. Use `Join-Path` for portability: `Join-Path $HOME 'docs' 'file.txt'`.
- **`Get-Content` is slow on huge files**: use `[System.IO.File]::ReadAllLines()` or `-ReadCount` for performance.
- **External commands return strings**, not objects. To structure them, parse: `git log --format=json` then `ConvertFrom-Json`, or use `ConvertFrom-StringData` / regex.

## Where to learn more

- `Get-Help <cmdlet> -Online` — opens the official docs for that cmdlet in your browser. Best per-cmdlet reference.
- `Get-Help about_*` — conceptual topics: `about_Pipelines`, `about_Variables`, `about_Operators`, `about_If`, `about_Foreach`, `about_Try_Catch_Finally`, `about_Splatting`, `about_Execution_Policies`.
- `Get-Help about_*` (no arg) — list all conceptual topics.
- [learn.microsoft.com/powershell](https://learn.microsoft.com/powershell/) — official docs hub. Bookmark.
- [github.com/PowerShell/PowerShell](https://github.com/PowerShell/PowerShell) — source + release notes for `pwsh`.
- [PSGallery](https://www.powershellgallery.com/) — module repository (`Install-Module <name>`).
- Stack Overflow tag [`powershell`](https://stackoverflow.com/questions/tagged/powershell).
