---
name: factory-mode
description: "Full Factory Model setup: AGENTS.md, formatter+linter, tests, CI, pre-commit hooks, check pipeline. Stack-agnostic — uses toolchain-discovery for commands."
version: 2.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [harness, factory-model, setup, ci, quality]
    category: ship
    related_skills: [toolchain-discovery, eval-harness, code-review-and-quality, requesting-code-review]
---

# Factory Mode — Harness Setup

Sets up the full Factory Model harness for any project: AGENTS.md, formatter+linter, test runner, pre-commit hooks, CI pipeline, and unified `check` script. Load this skill, point at a project, and it audits → configures → verifies.

> **Principle:** The Factory Model builds the system that builds software. The agent generates code; the harness verifies it. Stack doesn't matter — the toolchain map resolves the commands.

## When to Use

- User says "set up factory model for <project>", "настрой harness", "добавь CI", "стандартизируй проект"
- New project onboarding — first thing after `git init`
- Existing project audit — user wants to know what harness components are missing
- After major stack changes — re-audit and fill gaps

**Skip for:** projects with existing mature harness (all 5 components present and working).

## Prerequisites

- Project has a package manager config file (`package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`)
- `toolchain-discovery` has been run to produce the command map
- Git repository initialized

## Phase 1 — Audit

**First, run `toolchain-discovery`** to detect the project stack. This produces a canonical command map (`test`, `lint`, `format`, `typecheck`, `build`, `check`). **Never assume bun, npm, biome, or eslint — detect.**

Then read the existing project state:

```bash
# What exists?
ls AGENTS.md 2>/dev/null && echo "AGENTS.md: YES" || echo "AGENTS.md: NO"
ls .github/workflows/ci.yml 2>/dev/null && echo "CI: YES" || echo "CI: NO"
ls .husky/pre-commit 2>/dev/null && echo "husky: YES" || echo "husky: NO"
```

Check config files for existing scripts: `check`, `test`, `format`, `lint`, `typecheck`, `build`.

Report missing components in a table:
```
| Component    | Status |
|--------------|--------|
| AGENTS.md    | ✅/❌   |
| Formatter    | ✅/❌   |
| Tests        | ✅/❌   |
| Pre-commit   | ✅/❌   |
| CI           | ✅/❌   |
| check script | ✅/❌   |
```

**Ask "set up missing components?" before making changes.** Don't assume — the user may want only specific pieces.

## Phase 2 — Configure Missing Components

All commands below use the **toolchain map** from Phase 1. The examples show a Bun+Biome project; substitute `<toolchain.X>` for your stack.

### AGENTS.md

Read the project to understand:
- Stack (from config files)
- Structure (from directory layout)
- Commands (from the toolchain map)

Write AGENTS.md following user conventions:
- **Compact** — conventions + pitfalls only, no verbose reference
- Sections: Stack, Project structure, Conventions (scripts table from toolchain map), Key architecture decisions, Pitfalls, Verification
- Add toolchain map as the commands reference

### Formatter + Linter

Use the toolchain map to select the right tools. **Do not force a specific formatter.** If the project already has config files (`.eslintrc.*`, `.prettierrc.*`, `biome.json`, `ruff.toml`), respect them.

**Example (Bun+Biome — greenfield TS project):**
```bash
cd <project> && bun add -d @biomejs/biome
```

Write the formatter config (`biome.json`, `.prettierrc`, `ruff.toml`, etc. — match the toolchain):
- Match existing project conventions (indent, quotes, semicolons)
- Include source directories, exclude build artifacts
- Enable recommended rules

### Scripts

Add missing scripts to the project config, using the **toolchain map**. Example:

```json
{
  "check": "<toolchain.check>",
  "format": "<toolchain.format>",
  "lint": "<toolchain.lint>",
  "test": "<toolchain.test>",
  "typecheck": "<toolchain.typecheck>",
  "build": "<toolchain.build>"
}
```

> ⚠️ **User preference:** DO NOT modify config files without permission. ASK before changing.

### Tests

```bash
# Create test directory if missing
mkdir -p __tests__  # or tests/, spec/ — match project conventions
```

Add **one smoke test** for a pure function — something simple that exercises the test runner. Pick a file with zero side effects (no I/O, no network). The exact test syntax depends on the language; use the project's test framework.

Do NOT write tests for business logic without understanding it. One smoke test proves the harness works.

### Pre-commit hooks

Install and configure pre-commit tooling appropriate for the project's package manager:

```bash
# Example (Bun project)
cd <project> && bun add -d husky lint-staged
bunx husky init
```

Write `.husky/pre-commit`:
```bash
bunx lint-staged
<toolchain.check>
```

Configure `lint-staged` to format changed files on commit. Adapt the file globs to the project's languages.

### CI (GitHub Actions)

Create `.github/workflows/ci.yml` using the toolchain map. Example for a Bun project:

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
          bun-version: "<detect from bun.lock>"
      - run: bun install --frozen-lockfile
      - run: <toolchain.check>
      - run: <toolchain.build>
```

For other stacks, adapt the setup action (e.g., `actions/setup-python`, `actions-rs/toolchain`) and use the toolchain map commands. If project uses Docker, adapt for Docker-based CI.

## Phase 3 — Verify

After all components are configured:

```bash
cd <project> && <toolchain.format> && <toolchain.check> && <toolchain.build>
```

- **Format fails:** warn but continue (first run often fixes line-ending issues)
- **Check fails:** show errors, flag which are pre-existing vs new
- **Build fails:** critical — stop and diagnose
- **Tests fail:** if new test fails, fix it; if pre-existing, note and continue

Report final state:
```
✅ Factory Model active for <project>

Pipeline: spec → AI generates → <toolchain.check> → pre-commit hook → push → CI gate → merge

| Component    | Status |
|--------------|--------|
| AGENTS.md    | ✅     |
| Formatter    | ✅     |
| Tests        | ✅ (N tests) |
| Pre-commit   | ✅     |
| CI           | ✅     |
| check script | ✅     |
```

## Phase 4 — Update AGENTS.md

After setup, verify AGENTS.md reflects the current state. Add if missing:
- Verification section with `<toolchain.check>` and `<toolchain.build>`
- Reference to eval-harness for ongoing quality
- Known pre-existing lint issues (if found during Phase 3)

## Pitfalls

- **Don't modify configs without asking** — user enforces this rule. Audit first, propose changes, get confirmation.
- **Don't force a specific formatter** — use the toolchain map. If the project already has eslint/prettier/ruff configs, respect them.
- **Don't overwrite** — if a config file already exists, read it first and only add what's missing.
- **Monorepo awareness** — if subdirectories have their own configs, configure per-package, not at root.
- **CI setup** — detect the correct setup action and toolchain commands for the project's stack. Don't blindly use `oven-sh/setup-bun`.
- **GitHub repo creation** — the user's `gh_token` may lack `createRepository` scope. If `gh repo create` fails, tell the user to create the repo manually.
- **Don't rename busy directory** — if `mv` fails with "Device or resource busy", the shell has cwd inside the target. Try `cd .. && mv`.

## Verification

After setting up the harness for a project:

- [ ] AGENTS.md exists and covers stack, commands (from toolchain map), conventions, and pitfalls
- [ ] Formatter config exists and `<toolchain.format>` works
- [ ] Config file has `check`, `test`, `format`, `lint`, `typecheck`, `build` scripts
- [ ] `<toolchain.check>` passes (typecheck + lint + test)
- [ ] `<toolchain.build>` succeeds
- [ ] Pre-commit hook is installed and functional
- [ ] CI workflow (`.github/workflows/ci.yml`) exists and matches the project stack
- [ ] At least one smoke test runs successfully
