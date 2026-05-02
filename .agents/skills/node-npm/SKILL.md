---
name: node-npm
description: Use Node.js with npm, pnpm, or yarn — package.json, scripts, dependencies, lockfiles, npx, version management. Load when the user wants to install dependencies, run scripts, or work in a Node project.
---

# Node.js + package managers

## When to load this skill

Any task in a Node.js project — installing or upgrading deps, running scripts, debugging `package.json`, comparing `npm` / `pnpm` / `yarn`, using `npx`, picking a Node version, or troubleshooting "it works locally but not in CI".

## Pick a package manager once, stick with it

The lockfile in the repo tells you which to use. **Never mix.**

| Lockfile | Manager | Install command |
|---|---|---|
| `package-lock.json` | `npm` | `npm ci` (clean install from lockfile) |
| `pnpm-lock.yaml` | `pnpm` | `pnpm install --frozen-lockfile` |
| `yarn.lock` (Berry: `.yarn/`) | `yarn` | `yarn install --immutable` |
| `bun.lockb` | `bun` | `bun install --frozen-lockfile` |

`packageManager` field in `package.json` (e.g. `"packageManager": "pnpm@9.4.0"`) is the modern way to declare this; **enable [Corepack](https://nodejs.org/api/corepack.html)** (`corepack enable`) and Node will use the right manager + version automatically.

## Quick reference (all managers)

| Action | npm | pnpm | yarn (classic) |
|---|---|---|---|
| Install all from lockfile | `npm ci` | `pnpm install --frozen-lockfile` | `yarn install --frozen-lockfile` |
| Add a dep | `npm i <pkg>` | `pnpm add <pkg>` | `yarn add <pkg>` |
| Add a dev dep | `npm i -D <pkg>` | `pnpm add -D <pkg>` | `yarn add -D <pkg>` |
| Add at exact version | `npm i <pkg>@1.2.3 --save-exact` | `pnpm add <pkg>@1.2.3 --save-exact` | `yarn add <pkg>@1.2.3 --exact` |
| Remove | `npm uninstall <pkg>` | `pnpm remove <pkg>` | `yarn remove <pkg>` |
| Run a script | `npm run <name>` | `pnpm <name>` (alias) | `yarn <name>` |
| Run a one-off binary | `npx <pkg>` | `pnpm dlx <pkg>` | `yarn dlx <pkg>` (Berry) |
| Show outdated | `npm outdated` | `pnpm outdated` | `yarn outdated` |
| Upgrade interactively | `npm-check-updates` (sep. tool) | `pnpm update -i --latest` | `yarn upgrade-interactive --latest` |
| Audit | `npm audit` | `pnpm audit` | `yarn npm audit` (Berry) |

## `package.json` essentials

```jsonc
{
  "name": "my-thing",
  "version": "1.2.0",
  "type": "module",                    // "module" = ESM, "commonjs" = CJS (default if omitted)
  "engines": { "node": ">=20" },       // CI / installers can enforce this
  "packageManager": "pnpm@9.4.0",      // Corepack uses this
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "test": "vitest run",
    "lint": "eslint .",
    "format": "prettier --write ."
  },
  "dependencies":   { "react": "^18.3.0" },
  "devDependencies": { "vitest": "^2.0.0" }
}
```

**Version range syntax** (semver):
- `1.2.3` — exactly 1.2.3.
- `^1.2.3` — anything `>=1.2.3 <2.0.0` (compatible upgrades). **Default for `npm install`.**
- `~1.2.3` — anything `>=1.2.3 <1.3.0` (patch only).
- `>=1.2.3 <2`, `1.2.x`, `*`, `latest` — also valid.

## Common workflows

**Fresh checkout:**
```sh
corepack enable                      # one-time per machine
pnpm install --frozen-lockfile       # or `npm ci`, depending on lockfile
pnpm test                            # confirm everything works
```

**Add a dep and commit the lockfile:**
```sh
pnpm add lodash
git add package.json pnpm-lock.yaml
git commit -m "deps: add lodash"
```

**Pin a transitive dep version (security override):**
- pnpm: `package.json` → `"pnpm": { "overrides": { "vulnerable-pkg": "^2.0.0" } }`
- npm:  `package.json` → `"overrides": { "vulnerable-pkg": "^2.0.0" }`
- yarn (Berry): `package.json` → `"resolutions": { "vulnerable-pkg": "^2.0.0" }`

**Run a one-off tool without installing globally:**
```sh
npx create-vite@latest my-app -- --template react-ts
pnpm dlx prettier --write .
```

**Switch Node versions** (recommended: [`fnm`](https://github.com/Schniz/fnm), [`nvm`](https://github.com/nvm-sh/nvm), or [Volta](https://volta.sh/)):
```sh
fnm install 20
fnm use 20
node -v
```
A `.nvmrc` or `.node-version` file in the repo root pins the version per project.

## Gotchas

- **`npm install` without `ci`** modifies your lockfile; `npm ci` doesn't. Use `ci` whenever you don't intend to add/remove deps.
- **`node_modules` blowups**: when in doubt, `rm -rf node_modules` (PowerShell: `Remove-Item -Recurse -Force node_modules`) and reinstall.
- **`peerDependencies` warnings**: pnpm is strict and refuses; npm warns and continues. Read the warning — usually means you need to add the peer dep yourself.
- **CommonJS vs ESM**: `"type": "module"` makes `.js` ESM; otherwise it's CJS. `.mjs` is always ESM, `.cjs` is always CJS. Mixing causes `ERR_REQUIRE_ESM` errors.
- **Workspaces**: monorepos use `workspaces` (`package.json`) for npm/yarn or `pnpm-workspace.yaml` for pnpm. Commands like `pnpm -r test` run across all packages.
- **`postinstall` scripts** run automatically — review what you `pnpm install`. `--ignore-scripts` to skip if untrusted.
- **`engines` is advisory by default** for npm; pnpm honors it strictly. Add `"engine-strict": true` in `.npmrc` to force npm.

## Where to learn more

- `npm help` / `pnpm help` / `yarn help` — top-level command list.
- `npm <cmd> --help` / `pnpm <cmd> --help` / `yarn <cmd> --help` — detailed help.
- [docs.npmjs.com](https://docs.npmjs.com/) — npm CLI + package.json reference.
- [pnpm.io/cli/install](https://pnpm.io/cli/install) — pnpm reference.
- [yarnpkg.com/cli](https://yarnpkg.com/cli) — yarn (Berry) reference.
- [nodejs.org/docs/latest/api/](https://nodejs.org/docs/latest/api/) — Node.js standard library.
- [npmjs.com](https://www.npmjs.com/) — search packages, read READMEs, check weekly downloads (a rough quality signal).
- [bundlephobia.com](https://bundlephobia.com/) — see how big a dep is before adding it.
