---
name: eval-harness
description: "Automatic verification gates for agent actions — lint, format, test, file checks. Part of the Factory Model harness."
version: 1.2.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [harness, verification, evals, factory-model, quality]
    category: verify
    related_skills: [factory-mode, requesting-code-review, context-compaction, agent-observability]
---

# Eval Harness — Automatic Verification Gates

Automatic verification gates after every agent action. Part of the Factory Model — AI generates, harness verifies.

> **WHEN TO APPLY:** after EVERY code change (write_file, patch, terminal with code) — do NOT wait for user prompt. Eval harness must fire as a reflex.

## When to Use

- After ANY code change — fires as a reflex, no user trigger needed
- After `write_file`, `patch`, or `terminal` with code edits
- Before committing changes (commit gates)
- After writing files to verify they exist and are valid

**When NOT to use:** Documentation-only changes, pure config without code changes.

## Gates by Operation Type

### 1. Code Gates (TypeScript/React projects)

After any change to `.ts/.tsx/.js/.jsx` files in a project:

```
① Format: npx @biomejs/biome format --write <changed-files> 2>&1
② Lint:  bun run check 2>&1               # if "check" script exists in package.json
③ Build: bun run build 2>&1               # ONLY for commit gate
④ Tests: bun test 2>&1                    # if "test" script exists
```

> **Non-Bun projects:** substitute — `npm test`/`pytest`/`cargo test` for ④, `npm run build`/`cargo build` for ③, `ruff check`/`eslint`/`clippy` for ②. The gate structure is stack-agnostic; only the commands change.

**Rules:**
- ① and ② — ALWAYS after code changes
- ③ — only before git commit
- ④ — if files with logic were touched (not just styles/config)
- If any check fails — stop, explain the error, ask confirmation before fixing
- If `bun run check` doesn't exist in the project — skip, not an error

### 2. File Gates

After `write_file`:
- **File exists** — `ls -la <path>` or `search_files`
- **Non-empty** — size > 0 bytes
- **Valid format** — for `.md`: frontmatter present, wikilinks correct
- For `.json`: `python -m json.tool <path> > /dev/null`
- For `.yaml/.yml`: `python -c "import yaml; yaml.safe_load(open('<path>'))"`

### 3. Git Gates

After any git action:
- **Branch didn't switch unexpectedly:** `git branch --show-current` — verify against expected
- **No pending changes** (if commit not planned): `git diff --stat`
- **If commit needed** — after commit: `git log -1 --oneline` — verify commit landed

### 4. Wiki Gates

After writing to any wiki source (project `docs/`, global `$WIKI_PATH`, or LLM-wiki):
- **File created/updated** — verify via `search_files` at the path agent wrote to
- **Non-empty** — size > 0 bytes

**LLM-wiki conventions (auto-detect):** if target dir contains `index.md` → verify index was updated. If `log.md` exists → verify log entry added. If files use YAML frontmatter → verify `title`, `created`, `type`, `tags`, `sources` fields present.

**Wikilinks (optional):** if `[[...]]` wikilinks present in the page → verify each target exists (resolve relative to the wiki root, not agent's cwd).

> **Key rule:** check the path the agent ACTUALLY wrote to — don't assume a global wiki path. Detect conventions from the target directory, not from environment variables.

### 5. Commit Gates (before `git commit` / opening PR)

In addition to code gates:
```
① Full format: npx @biomejs/biome format --write . 2>&1
② Full lint:   bun run check 2>&1
③ Build:       bun run build 2>&1       # if present
④ Tests:       bun test 2>&1            # if present
⑤ Summary:     git diff --stat
```

## Severity Levels

**CRITICAL (stop immediately):**
- Code doesn't compile/run
- Tests fail
- Git in unexpected state (wrong branch, detached HEAD)

**IMPORTANT (stop, report, wait for decision):**
- Linter finds errors (not warnings)
- Formatter reports unformatted files
- File not created / empty after write

**INFO (report, don't stop):**
- Linter warnings
- File over 200 lines (candidate for split)
- No AGENTS.md in project

## LM-as-a-Judge (for non-deterministic tasks)

For tasks where you can't write a deterministic test (summaries, translations, analysis):
- **Self-review:** after generating summary/analysis — re-read and ask 3 questions:
  1. Are all key points covered?
  2. Are there any factual errors?
  3. Is the tone/style appropriate?
- **If response > 500 words** — offer TL;DR at the top

## Pitfalls

- **Don't run ③ (build) and ④ (tests) if project isn't configured** — skip, not an error
- **Don't auto-fix failing tests without explicit permission** — first explain what failed
- **Biome format ≠ Biome check** — format auto-fixes, check only verifies. Use format for fixing, check for verification.
- **Biome 2.x config schema changed:** `files.ignore` and `linter.rules` — unknown keys in 2.5+. Use `vcs.useIgnoreFile: true` for ignore, `"linter": {"enabled": true}` for default recommended rules. Check current $schema URL.
- **Wiki checks are non-blocking** — if index/log wasn't updated, just update them, don't stop everything. Missing wikilinks → flag but don't block.
- **Don't run biome format on others'/unchanged files** — only on files you touched
- **On Windows: `npx @biomejs/biome format --write .`** (full package name) — if biome not installed globally
- **`bun run check` is a non-standard script** — package.json may have `lint`, `typecheck` instead. Check scripts before running.

## Verification

After applying eval-harness to a task:

- [ ] Format gate ran on changed files and passed (or issues were auto-fixed)
- [ ] Code gate (`bun run check` or equivalent) passed — typecheck + lint + test
- [ ] File gate: all written files exist, are non-empty, and have valid format
- [ ] Git gate: branch is as expected, no unintended pending changes
- [ ] For non-deterministic tasks: LM-as-a-Judge self-review completed (3 questions)
