---
name: factory-mode
description: "Full Factory Model setup for TS/Bun projects: AGENTS.md, Biome, tests, CI, pre-commit hooks, check pipeline."
version: 1.1.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [harness, factory-model, setup, ci, quality]
    category: ship
    related_skills: [eval-harness, code-review-and-quality, requesting-code-review]
---

# Factory Mode — Harness Setup for TS/Bun Projects

Sets up the full Factory Model harness for a TypeScript/Bun project: AGENTS.md, Biome formatting+linking, test runner, pre-commit hooks, CI pipeline, and unified `check` script. Load this skill, point at a project, and it audits → configures → verifies.

> **Principle:** The Factory Model builds the system that builds software. The agent generates code; the harness verifies it.

## When to Use

- User says "set up factory model for <project>", "настрой harness", "добавь CI", "стандартизируй проект"
- New project onboarding — first thing after `git init`
- Existing project audit — user wants to know what harness components are missing
- After major stack changes — re-audit and fill gaps

**Skip for:** non-TS projects, projects with existing mature harness (all 5 components present and working).

## Prerequisites

- Project has `package.json` with `bun` as package manager
- TypeScript-based (`.ts`/`.tsx` files)
- Git repository initialized

## Phase 1 — Audit

Read the existing project state. Never assume — always check.

```bash
# What exists?
ls AGENTS.md 2>/dev/null && echo "AGENTS.md: YES" || echo "AGENTS.md: NO"
ls biome.json 2>/dev/null && echo "biome.json: YES" || echo "biome.json: NO"
ls .github/workflows/ci.yml 2>/dev/null && echo "CI: YES" || echo "CI: NO"
ls .husky/pre-commit 2>/dev/null && echo "husky: YES" || echo "husky: NO"
ls __tests__/ 2>/dev/null && echo "tests/: YES" || echo "tests/: NO"
```

Check `package.json` scripts for: `check`, `test`, `format`, `lint`, `typecheck`, `build`.

Report missing components in a table:
```
| Component | Status |
|-----------|--------|
| AGENTS.md | ✅/❌   |
| Biome     | ✅/❌   |
| Tests     | ✅/❌   |
| Pre-commit| ✅/❌   |
| CI        | ✅/❌   |
| check     | ✅/❌   |
```

**Ask "set up missing components?" before making changes.** Don't assume — the user may want only specific pieces.

## Phase 2 — Configure Missing Components

### AGENTS.md

Read the project to understand:
- Stack (from `package.json` dependencies)
- Structure (from `ls` / `search_files`)
- Build/test/lint commands (from `package.json` scripts)

Write AGENTS.md following user conventions:
- **Compact** — conventions + pitfalls only, no verbose reference
- Sections: Stack, Project structure, Conventions (scripts table), Key architecture decisions (if applicable), Pitfalls, Verification
- Add eval-harness reference: mention `bun run check` = typecheck + lint + test

### Biome

> **If the project already uses ESLint + Prettier (or `ruff` for Python), skip Biome.** Only install Biome for greenfield TS/Bun projects. For existing projects, adapt the `check`/`format`/`lint` scripts to the project's existing toolchain.

```bash
# Install (TS/Bun projects only)
cd <project> && bun add -d @biomejs/biome
```

Write `biome.json`:
- `indentStyle: space`, `indentWidth: 2` (or match project conventions)
- `quoteStyle: single`, `semicolons: asNeeded`, `trailingCommas: all`
- `includes: ["src/**", "server/**", "__tests__/**"]` (adapt to project structure)
- `ignore: ["dist/**", "node_modules/**"]`
- `linter.rules.recommended: true`
- Respect user preference: user REJECTED Tailwind — don't add Tailwind-specific rules unless project uses it

### Scripts (package.json)

Add if missing:
```json
{
  "check": "bun run typecheck && biome check <sources> && bun run test",
  "format": "biome format --write <sources>",
  "lint": "biome check <sources>",
  "test": "bun test",
  "typecheck": "tsc --noEmit"
}
```

> ⚠️ User preference: DO NOT modify .env, biome.json, tsconfig.json, package.json scripts without permission. ASK before modifying configs.

### Tests

```bash
# Create __tests__/ directory if missing
mkdir -p __tests__
```

Add **one smoke test** for a pure function (filters, utils, helpers) — something simple that exercises the test runner. Pick a file with zero side effects (no I/O, no network).

Template:
```typescript
import { describe, expect, test } from 'bun:test'
import { somePureFunction } from '../path/to/module'

describe('Module name', () => {
    test('handles basic case', () => {
        expect(somePureFunction(input)).toBe(expected)
    })
})
```

Do NOT write tests for business logic without understanding it. One smoke test proves the harness works.

### Pre-commit hooks

```bash
cd <project> && bun add -d husky lint-staged
bunx husky init
```

Write `.husky/pre-commit`:
```bash
bunx lint-staged
bun run check
```

Add to `package.json`:
```json
"lint-staged": {
  "*.{ts,tsx,js,jsx,json,md}": ["bun run format"]
}
```

### CI (GitHub Actions)

Create `.github/workflows/ci.yml`:
```yaml
name: CI
on:
  push:
    branches: [main, dev]
  pull_request:
    branches: [main, dev]
jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: oven-sh/setup-bun@v2
        with:
          bun-version: "<detect from project>"
      - run: bun install --frozen-lockfile
      - run: bun run check
      - run: bun run build
```

If project uses Docker, adapt for Docker-based CI.

## Phase 3 — Verify

After all components are configured:

```bash
cd <project> && bun run format && bun run check && bun run build
```

- **Format falls:** warn but continue (first run often fixes CRLF issues)
- **Check falls:** show errors, flag which are pre-existing vs new
- **Build falls:** critical — stop and diagnose
- **Tests fall:** if new test fails, fix it; if pre-existing, note and continue

Report final state:
```
✅ Factory Model active for <project>

Pipeline: spec → AI generates → bun run check (typecheck + lint + test) → pre-commit hook → push → CI gate → merge

| Component | Status |
|-----------|--------|
| AGENTS.md | ✅     |
| Biome     | ✅     |
| Tests     | ✅ (N tests) |
| Pre-commit| ✅     |
| CI        | ✅     |
| check     | ✅     |
```

## Phase 4 — Update AGENTS.md

After setup, verify AGENTS.md reflects the current state. Add if missing:
- Verification section with `bun run check` and `bun run build`
- Reference to eval-harness for ongoing quality
- Known pre-existing lint issues (if found during Phase 3)

## Pitfalls

- **Don't modify configs without asking** — user enforces this rule. Audit first, propose changes, get confirmation.
- **Respect tsconfig.json** — never change `include`/`exclude` without asking. If server is excluded from typecheck, note it, don't fix it.
- **Biome 2.5 schema** — `files.ignore` was removed in 2.x, use `vcs.useIgnoreFile: true` instead. `linter.rules` was flattened — check current schema.
- **Don't overwrite** — if a config file already exists (biome.json, .husky/pre-commit, ci.yml), read it first and only add what's missing.
- **Node vs Bun** — use `@biomejs/biome` (npm package), not `@biomejs/cli`. Run via `bun x biome` or `npx @biomejs/biome`.
- **Monorepo awareness** — if `web/` subdirectory has its own package.json, configs go there, not root.
- **Test location** — Bun discovers `*.test.ts` anywhere. `__tests__/` is conventional but not required.
- **CI bun version** — detect from `package.json` `packageManager` field or `bun.lock`.
- **GitHub repo creation** — the user's `gh_token` is a fine-grained PAT that blocks `.github/workflows/` writes and may lack `createRepository`. If `gh repo create` fails with `Resource not accessible by personal access token`, tell the user to create the repo manually on GitHub, provide the remote URL, and wait for confirmation before pushing. Don't loop-retry `gh repo create`.
- **Don't rename busy directory** — if `mv` fails with "Device or resource busy", the shell has cwd inside the target. Try `cd .. && mv` or tell the user to do it after the session ends.

## Verification

After setting up the harness for a project:

- [ ] AGENTS.md exists and covers stack, commands, conventions, and pitfalls
- [ ] `biome.json` is configured and `bun run format` works
- [ ] `package.json` has `check`, `test`, `format`, `lint`, `typecheck`, `build` scripts
- [ ] `bun run check` passes (typecheck + lint + test)
- [ ] `bun run build` succeeds
- [ ] Pre-commit hook is installed and functional (`bunx husky` or equivalent)
- [ ] CI workflow (`.github/workflows/ci.yml`) exists and matches the project stack
- [ ] At least one smoke test runs successfully
