---
name: agent-observability
description: "Track agent actions, decisions, and outcomes: structured traces, health checks, failure patterns."
version: 1.0.1
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [observability, monitoring, tracing, metrics, production]
    category: ship
    related_skills: [eval-harness, rho-retrospective-harness-optimization]
---

# Agent Observability

Track what the agent did, what worked, what failed, and why. Without observability, you're vibe-operating your harness.

> **Principle:** If you can't measure it, you can't improve it. RHO needs data.

## When to Use

- **During development:** track after every significant action
- **After task completion:** structured outcome report
- **Weekly health check:** aggregate metrics from past sessions
- **Before/after harness changes:** compare metrics to validate improvement

## The Observation Loop

```
Agent acts → Log action + outcome → Aggregate metrics → Surface anomalies → Improve harness (RHO)
```

## What to Track

### Level 1: Task Outcomes (always)

After every task completion, emit a structured outcome:

```markdown
## Task Outcome — [YYYY-MM-DD HH:MM]

**Goal:** [one-line description]
**Tools used:** terminal(2), write_file(1), read_file(3)
**Files changed:** src/auth.ts, src/auth.test.ts
**Verification:** bun run check → PASS
**Time:** 12 tool calls, ~3 min wall time
**Status:** ✅ SUCCESS / ⚠️ PARTIAL / ❌ FAILED
**Failure reason (if any):** [root cause]
**User feedback:** [correction received?]
```

### Level 2: Action Traces (when debugging)

For complex multi-step operations, trace each tool call:

```
[1] read_file("src/auth.ts")          → 120 lines read, 0.3s
[2] terminal("bun test auth")         → 3/3 pass, 2.1s
[3] patch(auth.ts, old, new)          → 1 replacement, 0.1s
[4] terminal("bun test auth")         → 4/4 pass, 2.0s
[5] write_file("CHANGELOG.md")         → 450 bytes written

✅ Complete: auth middleware with session refresh. 5 tool calls, 4.5s total.
```

### Level 3: Aggregate Metrics (weekly)

```markdown
## Weekly Harness Health — Week 26, 2026

**Sessions:** 14
**Tasks completed:** 23 (85% success, 15% partial)
**Avg tool calls/task:** 8.2
**Avg wall time/task:** 4.3 min

**Top corrections:**
1. "use bunx not npx" — 3 occurrences → RHO proposed AGENTS.md patch
2. "don't modify tsconfig" — 2 occurrences → already in memory

**Verification gaps:**
- 4 instances of agent skipping `bun run check` after code changes
→ RHO proposed eval-harness trigger update

**Token usage:**
- Context compaction saved ~40% tokens vs baseline (estimate)
```

## Health Check Command

Run periodically to assess harness effectiveness:

```bash
# Check recent session outcomes
session_search(query="SUCCESS FAILED PARTIAL", limit=20)

# Count verification gaps
session_search(query="bun run check")

# Find repeated corrections
session_search(query="no wrong don't instead", limit=10)
```

### Health Signals

| Signal | Good | Warning | Critical |
|--------|------|---------|----------|
| Task success rate | > 90% | 70-90% | < 70% |
| Repeated corrections | 0/week | 1-2/week | 3+/week |
| Verification gaps | 0 | 1-2/week | 3+/week |
| Context compaction | Active | Inconsistent | Never used |
| RHO proposals applied | 1-2/week | 0/week (stale) | N/A |

## Integration with eval-harness

eval-harness can be extended to log verification outcomes:

```
eval-harness code-gate:
  format → PASS
  lint   → FAIL (3 warnings, 1 error)
  test   → PASS

→ Logged as: "Code gate: 2/3 passed, lint needs fix"
→ Fed into RHO for pattern detection
```

## Integration with RHO

Observability feeds RHO with data. Without observability, RHO has nothing to analyze:

```
Observability logs → RHO detects patterns → RHO proposes fixes → RHO applies → Observability confirms
```

## Pitfalls

- **Don't over-log** — tracking every `read_file` adds noise. Focus on writes and decisions.
- **Structured over verbose** — one-line outcome beats 20-line trace for most tasks
- **Privacy** — don't log user content or secrets. Log structure, not data.
- **Session search is your DB** — Hermes sessions already store everything. Use `session_search` instead of building a separate log store.
- **Metrics without action are waste** — if you're not using metrics to improve (RHO), don't track them

## Verification

After applying agent-observability:

- [ ] Task outcome was logged (Goal, Files changed, Verification result, Status, Time)
- [ ] For complex tasks: action traces recorded (tool calls with timing)
- [ ] Weekly health check: success rate, repeated corrections, verification gaps analyzed
- [ ] Observability data was fed into RHO for pattern detection
- [ ] Privacy: no user content or secrets in logs — structure only
