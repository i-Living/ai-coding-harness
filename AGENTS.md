# AI Coding Harness — Agent Instructions

Factory Model harness for AI-assisted development. Nine skills that automate quality gates, project infrastructure, optimization, and the full spec→verify→improve pipeline.

## Quick Install

```bash
# Clone into Hermes skills directory
git clone https://github.com/i-Living/ai-coding-harness.git ~/.hermes/skills/ai-coding-harness

# Or install skill-by-skill via Hermes:
hermes skills install https://raw.githubusercontent.com/i-Living/ai-coding-harness/main/skills/eval-harness/SKILL.md
hermes skills install https://raw.githubusercontent.com/i-Living/ai-coding-harness/main/skills/factory-mode/SKILL.md
hermes skills install https://raw.githubusercontent.com/i-Living/ai-coding-harness/main/skills/spec-driven-development/SKILL.md
hermes skills install https://raw.githubusercontent.com/i-Living/ai-coding-harness/main/skills/test-driven-development/SKILL.md
hermes skills install https://raw.githubusercontent.com/i-Living/ai-coding-harness/main/skills/code-review-and-quality/SKILL.md
hermes skills install https://raw.githubusercontent.com/i-Living/ai-coding-harness/main/skills/requesting-code-review/SKILL.md
hermes skills install https://raw.githubusercontent.com/i-Living/ai-coding-harness/main/skills/context-compaction/SKILL.md
hermes skills install https://raw.githubusercontent.com/i-Living/ai-coding-harness/main/skills/rho-retrospective-harness-optimization/SKILL.md
hermes skills install https://raw.githubusercontent.com/i-Living/ai-coding-harness/main/skills/agent-observability/SKILL.md
```

After install: `/reload-skills` or restart session.

## Available Skills

### Infrastructure
| Skill | When to use |
|-------|-------------|
| **factory-mode** | User says "настрой harness", "добавь CI", "стандартизируй проект". Audits → configures → verifies. |
| **eval-harness** | After EVERY code change — auto-format + check. No user trigger needed — fires as reflex. |

### Workflow
| Skill | When to use |
|-------|-------------|
| **spec-driven-development** | New feature/project, ambiguous requirements. SPECIFY → PLAN → TASKS → IMPLEMENT. |
| **test-driven-development** | Any logic change, bug fix. RED → GREEN → REFACTOR with `bun test`. |
| **code-review-and-quality** | Before merge. 5-axis review: correctness, readability, architecture, security, performance. |
| **requesting-code-review** | Before commit. Security scan → independent reviewer → auto-fix → verified commit. |

### Optimization
| Skill | When to use |
|-------|-------------|
| **context-compaction** | Before sending large payloads to LLM. Token economics: think-in-code, selective extraction, structured output. |
| **rho-retrospective-harness-optimization** | After failures or user corrections. Analyze past sessions → detect patterns → auto-patch skills/AGENTS.md. |
| **agent-observability** | During and after tasks. Structured traces, health checks, metrics — feeds data into RHO. |

## The Pipeline

```
spec-driven-dev → TDD → code-review → requesting-code-review → [verified] commit
      │             │        │               │
      └─────────────┴────────┴───────────────┘
                       │
                  eval-harness (per-action gates)
                  context-compaction (token savings)
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
- **factory-mode** — load ONCE when setting up a project. Don't keep loaded afterward.
- **eval-harness** — should be loaded persistently. If not, load at session start for development sessions.
- **context-compaction** — load alongside eval-harness. Fires automatically before large payloads.
- **agent-observability** — load at session start for production work. Emit outcome after each task.
- **rho-retrospective-harness-optimization** — load weekly or after user correction. Scans past sessions.
- **spec-driven-development** — load at project start. Reference during implementation.
- **test-driven-development** — load when implementing features/bug fixes.
- **code-review-and-quality** — load before merge/PR review.
- **requesting-code-review** — load before commit with 2+ file changes.

## Pitfalls

- **Don't modify configs without asking** — user enforces this. factory-mode audits first, proposes changes, gets confirmation.
- **Respect tsconfig.json** — never change `include`/`exclude` without explicit permission.
- **Biome 2.5 schema** — `files.ignore` removed in 2.x. Use `vcs.useIgnoreFile: true`.
- **Monorepo awareness** — if `web/` has its own package.json, configs go there.
- **eval-harness + requesting-code-review overlap** — eval-harness is per-action, requesting-code-review is pre-commit with security scan + independent reviewer. Both can and should be active.
- **TDD strictness** — if agent writes code before test, delete and restart. No exceptions.
- **spec-driven gating** — don't proceed to PLAN until spec is human-reviewed.
- **context-compaction != lossy** — don't over-filter. Missing a critical error line costs more than tokens saved.
- **RHO threshold** — one failure isn't a pattern. Wait for 2+ occurrences before proposing a patch.
- **Observability without action is waste** — if not using metrics to improve (RHO), don't track them.
