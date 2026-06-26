---
name: rho-retrospective-harness-optimization
description: "Self-improving harness: analyze past failures, detect patterns, auto-patch skills and AGENTS.md to prevent recurrence."
version: 1.0.0
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [harness, optimization, self-improvement, evals, feedback-loop]
    category: verify
    related_skills: [eval-harness, factory-mode, session_search]
---

# RHO — Retrospective Harness Optimization

**Retrospective Harness Optimization (RHO):** use the agent's own past trajectories — successes and failures — to improve the harness without manual intervention. Inspired by the technique that moved SWE-Bench Pro from 59% to 78%.

> "Don't just fix the bug. Fix the harness that allowed the bug."

## When to Use

- After a task fails or produces suboptimal output
- After the user corrects the agent ("no, do it this way instead")
- Periodic health check (weekly): analyze last N sessions for patterns
- After any manual skill patch — RHO validates the fix worked

## The RHO Loop

```
1. DETECT → session_search for failures, corrections, user steering
2. CLASSIFY → categorize the root cause
3. PROPOSE → suggest harness improvement (skill patch, AGENTS.md update, memory entry)
4. VALIDATE → apply and verify on next similar task
5. OBSERVE → did recurrence stop? if yes → keep, if no → iterate
```

## Phase 1 — Detect

Find failure signals in recent sessions:

```bash
# Search for user corrections (strongest signal)
session_search(query="no wrong incorrect fix don't instead", limit=10)

# Search for test failures
session_search(query="test failed FAIL error regression", limit=10)

# Search for manual overrides
session_search(query="I'll do it myself let me manual", limit=5)
```

**Failure signals (descending priority):**
1. User says "no", "wrong", "don't", "instead" — explicit correction
2. User repeats a request — first attempt didn't work
3. User manually does something the agent tried to do — agent's approach rejected
4. Tests/lint fail after agent changes — quality gate caught error
5. Agent asks clarifying question that should be in memory — missing context

## Phase 2 — Classify

Categorize each failure by root cause:

| Category | Example | Fix in |
|----------|---------|--------|
| **Missing context** | Agent didn't know project uses `bun` not `npm` | AGENTS.md or memory |
| **Wrong tool** | Used `web_search` when `web_extract` was better | Skill instructions |
| **Skipped verification** | Didn't run `bun run check` after code change | eval-harness trigger |
| **Over-engineering** | Added unnecessary abstractions | AGENTS.md pitfalls |
| **Repeated correction** | Same mistake across multiple sessions | Memory entry |
| **Skill gap** | Task needed a skill that doesn't exist | New skill |

## Phase 3 — Propose

Generate a specific harness improvement:

```markdown
## RHO Finding #N

**Session:** [session ID]
**Trigger:** User said "no, use bunx not npx"
**Classification:** Missing context
**Root cause:** AGENTS.md for project X doesn't specify bun as package manager

**Proposed fix — patch AGENTS.md:**
Add to pitfalls: "Never use npx — project uses bun. Use bunx for one-off packages."

**Expected impact:** Prevents similar correction in all future sessions for this project.
```

## Phase 4 — Validate

Apply the fix and monitor:

1. **Apply:** patch the skill/AGENTS.md/memory as proposed
2. **Verify:** run a similar task to confirm the fix works
3. **Monitor:** check next 3 sessions for recurrence
4. **Escalate if needed:** if fix doesn't work, try a different category

## Phase 5 — Observe

After 3+ sessions with the fix active:
- Did the failure pattern stop? → **Keep** — fix validated
- Did it recur? → **Iterate** — try a different approach
- Did a new pattern emerge? → **New RHO cycle**

## Integration with eval-harness

RHO feeds into eval-harness. When RHO detects a repeated verification gap:

```
RHO detects: "Agent forgot to run bun run check 3 times this week"
       │
       ▼
RHO proposes: "Add bun run check to eval-harness triggers for *.ts files"
       │
       ▼
RHO patches eval-harness → next session, check fires automatically
```

## Self-Validation Pattern

RHO validates itself. Track:
- **RHO proposals made:** N
- **Proposals applied:** M
- **Recurrence prevented:** K
- **Efficacy rate:** K / M

If efficacy < 50%, RHO is over-patching — increase threshold for proposing changes.

## Pitfalls

- **Don't over-patch** — one failure isn't a pattern. Look for 2+ occurrences.
- **Don't remove valid flexibility** — "never use X" rules narrow agent capability
- **Context window decay** — old sessions may not reflect current harness state
- **False positives** — user saying "no" might be a preference, not a failure
- **Skill bloat** — don't create a new skill for every minor fix; patch existing ones
