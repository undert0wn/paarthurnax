---
name: python-environments
description: Use Python with venv, pip, uv, pyenv, conda — virtual environments, dependency files (requirements.txt, pyproject.toml), lockfiles, version management. Load when the user wants to install Python deps, set up a project, or troubleshoot environment issues.
---

# Python environments + packaging

## When to load this skill

Any Python project task — creating venvs, installing deps, picking between `pip` / `uv` / `poetry` / `conda`, choosing a Python version, debugging "wrong Python", working with `requirements.txt` or `pyproject.toml`, or untangling system vs project Python.

## The golden rule

**Never `pip install` into the system Python.** Always use a virtual environment, `uv`, `pipx`, or `conda`. On modern Linux distros (Debian 12+, Ubuntu 24.04+, Fedora 38+) the system Python actively refuses with PEP 668 "externally-managed-environment" — that's the OS protecting itself. Don't bypass it; use a venv.

## Pick a workflow

| Tool | Best for | Notes |
|---|---|---|
| **`uv`** ([astral-sh/uv](https://github.com/astral-sh/uv)) | Most new projects in 2024+ | 10-100x faster than pip; manages venvs, deps, AND Python versions. **Default recommendation.** |
| `venv` + `pip` | Simple scripts, broad compat | Built into Python; works everywhere; no extras to install. |
| `poetry` | Library development with publishing | Strong dep resolver, lockfile, build/publish workflow. |
| `pipx` | Installing CLI tools (black, ruff, httpie) | Each tool gets its own isolated venv. |
| `conda` / `mamba` | Scientific Python with binary deps (numpy, scipy, pytorch, GIS) | Handles non-Python binaries that pip can't. |
| `pyenv` | Managing multiple Python versions on one machine | Often paired with `venv` or `poetry`. (`uv` replaces this for most users.) |

## Quick reference

### `uv` (recommended for new work)
```sh
uv python install 3.12              # download and install Python 3.12
uv venv                             # create .venv/ in current dir
uv venv --python 3.12               # pin Python version
uv pip install requests             # install into the active venv (no activation needed)
uv pip install -r requirements.txt
uv pip compile requirements.in -o requirements.txt   # lockfile-style pin

# Project mode (uses pyproject.toml + uv.lock):
uv init my-project                  # scaffold a new project
uv add requests                     # add a dep + update lockfile
uv add --dev pytest                 # dev dep
uv sync                             # install exactly what's in uv.lock
uv run python script.py             # run with project's deps, no manual activation
uv run pytest                       # run any installed tool the same way
```

### `venv` + `pip` (built-in, universal)
```sh
python -m venv .venv                # create venv
# Activate:
.venv\Scripts\Activate.ps1          # Windows PowerShell
.venv\Scripts\activate.bat          # Windows cmd
source .venv/bin/activate           # macOS / Linux
# Once activated:
pip install -r requirements.txt
pip install requests
pip freeze > requirements.txt       # snapshot installed versions
deactivate                          # exit the venv
```

### `pipx` (CLI tools)
```sh
pipx install ruff                   # install ruff in its own venv, on PATH
pipx list
pipx upgrade-all
pipx run black .                    # one-off without installing
```

## Dependency files

| File | Purpose | Used by |
|---|---|---|
| `requirements.txt` | Flat list of pins. Simple, broad compat. | `pip`, `uv pip` |
| `requirements.in` + `requirements.txt` | `.in` is the source of truth (loose ranges); `.txt` is the compiled lockfile. | `pip-tools`, `uv pip compile` |
| `pyproject.toml` | Modern standard. Project metadata + deps + tool config in one file. | `uv`, `poetry`, `hatch`, `pdm`, `setuptools` |
| `uv.lock` / `poetry.lock` / `pdm.lock` | Hash-pinned lockfile for reproducible installs. | tool-specific |
| `setup.py` / `setup.cfg` | Legacy — only edit if you must maintain an old project. | `setuptools` |
| `environment.yml` | Conda env spec. | `conda`, `mamba` |

### `pyproject.toml` minimum example
```toml
[project]
name = "my-thing"
version = "0.1.0"
requires-python = ">=3.10"
dependencies = [
    "requests>=2.31",
    "pydantic>=2",
]

[project.optional-dependencies]
dev = ["pytest", "ruff", "mypy"]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

## Common workflows

**Fresh checkout with `uv`:**
```sh
uv sync                              # creates .venv if needed, installs from uv.lock
uv run pytest                        # run tests
```

**Fresh checkout with `pip`:**
```sh
python -m venv .venv
source .venv/bin/activate            # or Windows equivalent
pip install -r requirements.txt
pytest
```

**Add a dep (uv project mode):**
```sh
uv add httpx
git add pyproject.toml uv.lock
git commit -m "deps: add httpx"
```

**Pin Python version per project**: add `.python-version` (used by `pyenv`, `uv`, `rye`):
```
3.12
```

## Gotchas

- **Which `python`/`pip`?** Run `which python` (or `where python` on Windows) and `python -c "import sys; print(sys.executable)"` to confirm. Activated venv should show `.venv/bin/python` (or `.venv\Scripts\python.exe`).
- **`pip install --user`** installs to `~/.local/lib/python3.X/site-packages/`. Often a footgun — confuses which Python sees what. Prefer venv or pipx.
- **`python` vs `python3`**: on most macOS / Linux, `python` is either Python 2 or doesn't exist; use `python3`. On Windows from python.org installer, `python` works. `py -3.12` (the [Python launcher for Windows](https://docs.python.org/3/using/windows.html#python-launcher-for-windows)) is unambiguous.
- **PowerShell execution policy** can block venv activation. One-time fix: `Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned`.
- **`requirements.txt` without hashes is not reproducible** — same file can install different transitive versions tomorrow. Use `pip-tools` / `uv pip compile` for hash pinning, or use a real lockfile.
- **Windows wheels for compiled deps** (numpy, lxml, cryptography, pillow): almost always available on PyPI — don't try to build from source unless you have to.
- **`conda` and `pip` mix poorly**: install everything via conda first, then `pip` only for what conda lacks. Reversed order breaks the conda env.
- **`__pycache__` and `.pyc`**: gitignore them. Never commit. `find . -type d -name __pycache__ -exec rm -rf {} +` to nuke.

## Where to learn more

- `python -m pip help`, `python -m pip help install`, `uv help`, `uv pip help install` — built-in help.
- [packaging.python.org](https://packaging.python.org/) — official Python Packaging User Guide. **The single best reference.**
- [docs.astral.sh/uv](https://docs.astral.sh/uv/) — `uv` docs (also covers `pip`-equivalent commands).
- [pip.pypa.io](https://pip.pypa.io/) — pip reference.
- [docs.python.org/3/library/venv.html](https://docs.python.org/3/library/venv.html) — venv reference.
- [pypi.org](https://pypi.org/) — search packages, check release dates, read project URLs.
- [peps.python.org](https://peps.python.org/) — PEP 517 (build system), PEP 621 (pyproject.toml metadata), PEP 668 (externally-managed) explain the *why* behind modern packaging.
