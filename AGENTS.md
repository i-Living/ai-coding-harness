# AI Coding Harness — Agent Instructions

Factory Model harness for AI-assisted development. Twelve skills that automate quality gates, project infrastructure, optimization, and the full interview→spec→verify→improve pipeline.

## Quick Install

```bash
# Clone into Hermes skills directory
git clone https://github.com/i-Living/ai-coding-harness.git ~/.hermes/skills/ai-coding-harness
```

After install: `/reload-skills` or restart session.

## Available Skills

### Define (pre-code)
| Skill | When to use |
|-------|-------------|
| **interview-me** | User says "interview me", "grill me", "are we sure?". Underspecified asks ("build me X" without "for whom"). Pre-spec intent extraction. |
| **spec-driven-development** | New feature/project, ambiguous requirements. SPECIFY → PLAN → TASKS → IMPLEMENT. |

### Infrastructure
| Skill | When to use |
|-------|-------------|
| **factory-mode** | User says "настрой harness", "добавь CI", "стандартизируй проект". Audits → configures → verifies. |
| **eval-harness** | After EVERY code change — auto-format + check. No user trigger needed — fires as reflex. |

### Workflow
| Skill | When to use |
|-------|-------------|
| **planning-and-task-breakdown** | After spec. Break into ordered, verifiable tasks. Vertical slicing, dependency graphs, checkpoints. |
| **incremental-implementation** | Multi-file features. Build in thin slices: implement → test → verify → commit → next. Rule 0: simplicity first. |
| **test-driven-development** | Any logic change, bug fix. RED → GREEN → REFACTOR with `bun test`. |
| **code-review-and-quality** | Before merge. 5-axis review: correctness, readability, architecture, security, performance. |
| **requesting-code-review** | Before commit. Security scan → independent reviewer → auto-fix → verified commit. |

### Optimization
| Skill | When to use |
|-------|-------------|
| **context-compaction** | Before large payloads AND when agent quality declines. Token economics + context quality (hierarchy, trust levels, confusion management). |
| **rho-retrospective-harness-optimization** | After failures or user corrections. Analyze past sessions → detect patterns → auto-patch skills/AGENTS.md. |
| **agent-observability** | During and after tasks. Structured traces, health checks, metrics — feeds data into RHO. |

## The Pipeline

```
interview-me (intent)
      │
      ▼
spec-driven-dev (spec)
      │
      ▼
planning-and-task-breakdown (tasks)
      │
      ▼
incremental-implementation → TDD → code-review → requesting-code-review → commit
      │                         │        │               │
      └─────────────────────────┴────────┴───────────────┘
                                    │
                               eval-harness (per-action gates)
                               context-compaction (tokens + quality)
                                    │
                               agent-observability (traces + metrics)
                                    │
                               RHO (analyze → improve → validate)
```

## Conventions

### Verification command
All skills use `bun run check` as the standard verification command:
```bash
bun run check    # = typecheck + lint + test
bun run format   # = auto-format via Biome
bun run build    # = production build
```

### Stack assumptions
- **Bun** as package manager and runtime
- **TypeScript** for all code
- **Biome** for linting + formatting
- **bun:test** for test runner
- Non-Bun projects: skills document fallback commands (npm, pytest, cargo, go)

### When to load skills
- **interview-me** — load when user's ask is underspecified. Phase 0 before spec.
- **spec-driven-development** — load at project start. Reference during implementation.
- **planning-and-task-breakdown** — load after spec is approved. Produces task list.
- **incremental-implementation** — load during implementation phase. Enforces thin-slice discipline.
- **factory-mode** — load ONCE when setting up a project. Don't keep loaded afterward.
- **eval-harness** — should be loaded persistently. If not, load at session start for development sessions.
- **test-driven-development** — load when implementing features/bug fixes.
- **code-review-and-quality** — load before merge/PR review.
- **requesting-code-review** — load before commit with 2+ file changes.
- **context-compaction** — load alongside eval-harness. Fires automatically before large payloads.
- **agent-observability** — load at session start for production work. Emit outcome after each task.
- **rho-retrospective-harness-optimization** — load weekly or after user correction. Scans past sessions.

## Pitfalls

- **Don't modify configs without asking** — user enforces this. factory-mode audits first, proposes changes, gets confirmation.
- **Respect tsconfig.json** — never change `include`/`exclude` without explicit permission.
- **Biome 2.5 schema** — `files.ignore` removed in 2.x. Use `vcs.useIgnoreFile: true`.
- **Monorepo awareness** — if `web/` has its own package.json, configs go there.
- **eval-harness + requesting-code-review overlap** — eval-harness is per-action, requesting-code-review is pre-commit with security scan + independent reviewer. Both can and should be active.
- **TDD strictness** — if agent writes code before test, delete and restart. No exceptions.
- **spec-driven gating** — don't proceed to PLAN until spec is human-reviewed.
- **inline-planning** — for multi-step tasks, emit a lightweight PLAN before executing (30-second investment, 30-minute savings).
- **context-compaction ≠ lossy** — don't over-filter. Missing a critical error line costs more than tokens saved.
- **RHO threshold** — one failure isn't a pattern. Wait for 2+ occurrences before proposing a patch.
- **Observability without action is waste** — if not using metrics to improve (RHO), don't track them.
- **Scope discipline** — incremental-implementation Rule 0.5: touch only what the task requires. Note unrelated improvements, don't fix them.
- **interview-me gating** — don't produce spec/plan/tasks before explicit yes from user. "Whatever you think" is NOT yes.
