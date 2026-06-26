---
name: toolchain-discovery
description: "Detects project toolchain at factory-mode stage and resolves canonical commands for test, lint, format, typecheck, build. Use when setting up a harness for a new project, or when any skill needs to know the project's test/lint/build commands. Makes all other skills stack-agnostic."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [toolchain, discovery, stack-detection, infrastructure, factory-model]
    category: ship
    related_skills: [factory-mode, eval-harness]
---

# Toolchain Discovery

## Overview

Detect the project's toolchain at setup time and produce a **canonical command map** that all other harness skills consume. Instead of every skill hardcoding `bun test` or guessing `pytest` vs `jest`, they reference the toolchain map. One source of truth for how this project runs tests, lints, formats, typechecks, and builds.

**Principle:** The agent generates code; the harness verifies it. But verification commands depend on the stack. Toolchain discovery resolves that dependency once, at setup, so no skill needs to guess.

## When to Use

- **factory-mode Phase 1** — detect the stack before configuring the harness
- **eval-harness** — substitute correct gate commands instead of assuming `bun`
- **Any skill** that needs to run project-specific commands (test, lint, build)

**When NOT to use:** The project has no config files at all (bare scripts). In that case, ask the user.

## Detection Process

### Step 1: Scan for config files

```bash
ls package.json 2>/dev/null && echo "NODE:YES" || echo "NODE:NO"
ls pyproject.toml 2>/dev/null && echo "PYTHON:YES" || echo "PYTHON:NO"
ls Cargo.toml 2>/dev/null && echo "RUST:YES" || echo "RUST:NO"
ls go.mod 2>/dev/null && echo "GO:YES" || echo "GO:NO"
ls Makefile 2>/dev/null && echo "MAKE:YES" || echo "MAKE:NO"
```

### Step 2: Resolve package manager (Node/TS projects)

```bash
# Check packageManager field first (most reliable)
grep -o '"packageManager": *"[^"]*"' package.json 2>/dev/null

# Fallback: check lockfile
ls bun.lock 2>/dev/null && echo "BUN" && exit
ls package-lock.json 2>/dev/null && echo "NPM" && exit
ls yarn.lock 2>/dev/null && echo "YARN" && exit
ls pnpm-lock.yaml 2>/dev/null && echo "PNPM" && exit

# Default
echo "NPM"
```

### Step 3: Build the command map

Use the table below. Read the project's existing `package.json` scripts first — if `test`/`lint`/`format` scripts already exist, use those. Only generate defaults for missing ones.

## Command Map (Canonical Defaults)

### Node.js / TypeScript

| Package Manager | Install | Test | Lint | Format | Typecheck | Build |
|----------------|---------|------|------|--------|-----------|-------|
| **bun** | `bun install` | `bun test` | `biome check <src>` | `biome format --write <src>` | `tsc --noEmit` | `bun run build` |
| **npm** | `npm ci` | `npm test` | `npx eslint <src>` | `npx prettier --write <src>` | `npx tsc --noEmit` | `npm run build` |
| **yarn** | `yarn install --frozen-lockfile` | `yarn test` | `yarn eslint <src>` | `yarn prettier --write <src>` | `yarn tsc --noEmit` | `yarn build` |
| **pnpm** | `pnpm install --frozen-lockfile` | `pnpm test` | `pnpm eslint <src>` | `pnpm prettier --write <src>` | `pnpm tsc --noEmit` | `pnpm build` |

### Python

| Tool | Test | Lint | Format | Typecheck | Build |
|------|------|------|--------|-----------|-------|
| **pytest** | `pytest` | `ruff check .` | `ruff format .` | `mypy .` | `python -m build` |
| **unittest** | `python -m pytest` | `ruff check .` | `ruff format .` | — | — |

### Rust

| Command | Test | Lint | Format | Typecheck | Build |
|---------|------|------|--------|-----------|-------|
| **cargo** | `cargo test` | `cargo clippy -- -D warnings` | `cargo fmt -- --check` | `cargo check` | `cargo build --release` |

### Go

| Command | Test | Lint | Format | Build |
|---------|------|------|--------|-------|
| **go** | `go test ./...` | `go vet ./...` | `gofmt -w .` | `go build ./...` |

## Output Format

After detection, produce a map like this:

```yaml
# Toolchain map for <project-name>
package_manager: bun
test: bun test
lint: biome check src/ server/ __tests__/
format: biome format --write src/ server/ __tests__/
typecheck: tsc --noEmit
build: bun run build
check: bun run typecheck && biome check src/ server/ __tests__/ && bun test
```

**Rules:**
- If `package.json` already has `lint`/`test`/`format` scripts, use those verbatim — don't override
- If `biome.json` exists, use biome for lint+format regardless of package manager
- If `.eslintrc.*` exists, use eslint for lint regardless of package manager
- The `check` entry combines typecheck + lint + test (the standard harness verification command)

## Integration

### factory-mode

Phase 1 audit runs toolchain-discovery first. Phase 2 uses the command map to write correct scripts into `package.json` and verify commands into AGENTS.md.

### eval-harness

Code gates reference the toolchain map instead of hardcoded commands:

```
Before (Bun-only): ① npx @biomejs/biome format --write <files>
                    ② bun run check
                    ③ bun run build
                    ④ bun test

After (agnostic):  ① <toolchain.format>
                    ② <toolchain.check>
                    ③ <toolchain.build>
                    ④ <toolchain.test>
```

### test-driven-development

Instead of `bun test __tests__/feature.test.ts`, the skill says "run the project's test command on the new test file." The actual command comes from `<toolchain.test>`.

## Hermes Integration

```python
# In factory-mode, after detection:
toolchain = {
    "test": "bun test",
    "lint": "biome check src/",
    "format": "biome format --write src/",
    "typecheck": "tsc --noEmit",
    "build": "bun run build",
    "check": "bun run typecheck && biome check src/ && bun test"
}

# Write to AGENTS.md so other skills can reference it
write_file("AGENTS.md", f"""...
## Commands
- Test: `{toolchain['test']}`
- Lint: `{toolchain['lint']}`
- Format: `{toolchain['format']}`
- Typecheck: `{toolchain['typecheck']}`
- Build: `{toolchain['build']}`
- Check: `{toolchain['check']}`
...""")
```

## Verification

After running toolchain discovery:

- [ ] Config files were scanned (`package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`)
- [ ] Package manager was resolved (not assumed)
- [ ] Existing scripts in config files were read and preserved
- [ ] Command map was produced with all 5 entries (test, lint, format, typecheck, build)
- [ ] The `check` entry combines typecheck + lint + test correctly
- [ ] The map was written into AGENTS.md for downstream skills to reference

## Pitfalls

- **Don't assume bun** — always check lockfiles and `packageManager` field first
- **Don't override existing scripts** — if the project already has `"test": "jest"`, use that, not `bun test`
- **Don't force Biome** — if `.eslintrc.*` or `.prettierrc.*` exists, respect the existing tooling
- **Don't force ruff on Python** — check if `flake8`/`pylint` configs exist first
- **Monorepo awareness** — check each subdirectory's config; different packages may use different tools
- **The map is input, not output** — toolchain-discovery PRODUCES the map for other skills to consume. It doesn't modify config files (that's factory-mode's job)
