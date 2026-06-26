---
name: test-driven-development
description: "TDD: enforce RED-GREEN-REFACTOR with bun test. Tests before code."
version: 2.0.0
author: Hermes Agent (adapted from obra/superpowers)
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [testing, tdd, development, quality, red-green-refactor, bun]
    related_skills: [eval-harness, factory-mode, systematic-debugging, subagent-driven-development]
---

# Test-Driven Development (TDD) — Bun / TypeScript

## Overview

Write the test first. Watch it fail. Write minimal code to pass.

**Core principle:** If you didn't watch the test fail, you don't know if it tests the right thing.

**Violating the letter of the rules is violating the spirit of the rules.**

## When to Use

**Always:**
- New features
- Bug fixes
- Refactoring
- Behavior changes

**Exceptions (ask the user first):**
- Throwaway prototypes
- Generated code
- Configuration files

## The Iron Law

```
NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST
```

Write code before the test? Delete it. Start over.

**No exceptions:** don't keep it as "reference", don't "adapt" it while writing tests, don't look at it. Delete means delete.

## Red-Green-Refactor Cycle

### RED — Write Failing Test

```typescript
// bun test auto-discovers *.test.ts files
import { describe, expect, test } from 'bun:test'
import { retryOperation } from '../src/retry'

describe('retryOperation', () => {
  test('retries failed operations up to 3 times then succeeds', () => {
    let attempts = 0
    const operation = () => {
      attempts++
      if (attempts < 3) throw new Error('fail')
      return 'success'
    }

    const result = retryOperation(operation)

    expect(result).toBe('success')
    expect(attempts).toBe(3)
  })
})
```

**Requirements:**
- One behavior per test
- Clear descriptive name (has "and"? Split it)
- Real code, not mocks (unless truly unavoidable)
- Arrange → Act → Assert pattern

### Verify RED — Watch It Fail

**MANDATORY. Never skip.**

```bash
bun test __tests__/retry.test.ts
```

Confirm: test fails (not errors), failure is expected, fails because feature is missing.

**Test passes immediately?** You're testing existing behavior. Fix the test.
**Test errors?** Fix the error, re-run.

### GREEN — Minimal Code

```typescript
// Minimal — just enough to pass
export function retryOperation<T>(fn: () => T): T {
  let lastError: unknown
  for (let i = 0; i < 3; i++) {
    try {
      return fn()
    } catch (e) {
      lastError = e
    }
  }
  throw lastError
}
```

Don't add features, refactor other code, or "improve" beyond the test. Cheating is OK in GREEN.

### Verify GREEN — Watch It Pass

```bash
# Specific test
bun test __tests__/retry.test.ts

# Full suite
bun test
```

### REFACTOR — Clean Up

After green only: remove duplication, improve names, extract helpers. Keep tests green.

### Full verification

```bash
bun run check     # typecheck + lint + test (if eval-harness is active)
```

## The Prove-It Pattern (Bug Fixes)

```typescript
// Bug: "completeTask doesn't set completedAt timestamp"

// Step 1: Write reproduction test (MUST fail)
import { describe, expect, test } from 'bun:test'
import { completeTask, createTask } from '../src/tasks'

describe('completeTask', () => {
  test('sets completedAt when task is completed', async () => {
    const task = await createTask({ title: 'Test' })
    const completed = await completeTask(task.id)
    expect(completed.status).toBe('completed')
    expect(completed.completedAt).toBeInstanceOf(Date) // ← fails (bug!)
  })
})

// Step 2: Fix
export async function completeTask(id: string): Promise<Task> {
  return db.tasks.update(id, {
    status: 'completed',
    completedAt: new Date(), // ← was missing
  })
}
```

## Why Order Matters

- **Test-after = "what does this do?"** — biased by implementation, tests pass immediately
- **Test-first = "what should this do?"** — forces edge case discovery before implementing
- **"Already manually tested"** — ad-hoc ≠ systematic. No record, can't re-run.
- **"Deleting X hours is wasteful"** — sunk cost fallacy. Keeping unverified code is technical debt.

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "Too simple to test" | Simple code breaks. Test takes 30 seconds. |
| "I'll test after" | Tests passing immediately prove nothing. |
| "Already manually tested" | Ad-hoc ≠ systematic. No record, can't re-run. |
| "Keep as reference, write tests first" | You'll adapt it. That's testing after. |
| "TDD will slow me down" | TDD faster than debugging. |

## Red Flags — STOP and Start Over

- Code before test
- Test after implementation
- Test passes immediately on first run
- Can't explain why test failed
- Rationalizing "just this once"
- "Tests after achieve the same purpose"

**All of these mean: Delete code. Start over with TDD.**

## Hermes Agent Integration

### Running Tests

```bash
# RED — verify failure
bun test __tests__/feature.test.ts

# GREEN — verify pass
bun test __tests__/feature.test.ts

# Full suite
bun test
```

### With eval-harness

After TDD cycle completes, eval-harness fires per-action gates:
- `bun run format` — Biome auto-format changed files
- `bun run check` — typecheck + lint + test (full verification)

TDD + eval-harness = RED → GREEN → REFACTOR → format → check → commit.

### With delegate_task

```python
delegate_task(
    goal="Implement [feature] using strict TDD",
    context="""
    Follow test-driven-development skill:
    1. Write failing test FIRST (bun test)
    2. Run to verify failure
    3. Write minimal code to pass
    4. Run to verify pass
    5. Refactor if needed
    6. bun run check before done

    Project: bun test runner, Biome format, bun run check = typecheck + lint + test
    """,
    toolsets=['terminal', 'file']
)
```

### With systematic-debugging

Bug found? Write failing test reproducing it. TDD cycle. The test proves the fix AND prevents regression.

## Testing Anti-Patterns

- **Testing mock behavior instead of real behavior**
- **Testing implementation details** — test behavior/results, not internal method calls
- **Happy path only** — always test edge cases, errors, boundaries
- **Brittle tests** — tests should verify behavior, not structure

## Final Rule

```
Production code → test exists and failed first
Otherwise → not TDD
```

No exceptions without the user's explicit permission.
