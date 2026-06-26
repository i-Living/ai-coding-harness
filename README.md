# AI Coding Harness

Factory Model harness for AI-assisted development — Hermes Agent skills that automate quality gates, project infrastructure, and the full spec→verify→optimize pipeline.

> Build the system that builds software. The agent generates code; the harness verifies, compacts, observes, and improves.

## Harness Skills (9)

### Infrastructure

| Skill | Role |
|-------|------|
| **factory-mode** | One-command harness setup: audit → configure → verify |
| **eval-harness** | Per-action verification gates after every code change |

### Workflow

| Skill | Role |
|-------|------|
| **spec-driven-development** | SPECIFY → PLAN → TASKS → IMPLEMENT with spec as source of truth |
| **test-driven-development** | RED-GREEN-REFACTOR with `bun test` — no code without failing test |
| **code-review-and-quality** | 5-axis review before merge |
| **requesting-code-review** | Pre-commit pipeline: security scan → independent reviewer → auto-fix |

### Optimization

| Skill | Role |
|-------|------|
| **context-compaction** | Token economics: think-in-code, selective extraction, structured output |
| **rho-retrospective-harness-optimization** | Self-improving harness: analyze failures → auto-patch skills |
| **agent-observability** | Agent traces, health checks, failure metrics — data for RHO |

### How They Work Together

```
         ┌──────────── factory-mode (sets up everything) ────────────┐
         │                                                           │
         ▼                                                           │
spec-driven-dev → TDD → code-review → requesting-code-review → commit
      │              │         │               │
      └──────────────┴─────────┴───────────────┘
                        │
                   eval-harness (per-action gates)
                   context-compaction (token savings)
                        │
                   agent-observability (traces + metrics)
                        │
                   RHO (analyze → improve → validate)
```

## Quick Start

```bash
# Set up a project from scratch:
skill_view(name='factory-mode')

# Write a spec before coding:
skill_view(name='spec-driven-development')

# Full pipeline with optimization:
skill_view(name='factory-mode')
skill_view(name='eval-harness')
skill_view(name='context-compaction')
```

## Structure

```
ai-coding-harness/
├── AGENTS.md                                    # Agent instructions
├── README.md
└── skills/
    ├── eval-harness/SKILL.md                    # Verification gates
    ├── factory-mode/SKILL.md                    # Harness setup
    ├── spec-driven-development/SKILL.md         # Spec-first workflow
    ├── test-driven-development/SKILL.md         # TDD with bun test
    ├── code-review-and-quality/SKILL.md         # 5-axis review
    ├── requesting-code-review/SKILL.md          # Pre-commit pipeline
    ├── context-compaction/SKILL.md              # Token economics
    ├── rho-retrospective-harness-optimization/SKILL.md  # Self-improving harness
    └── agent-observability/SKILL.md             # Traces + metrics
```

## The Pipeline

```
spec → AI generates → context-compaction → eval-harness gates → pre-commit → CI → merge
                                                      │
                                              agent-observability
                                              (logs everything)
                                                      │
                                              RHO
                                              (analyzes → improves)
```

## Principles

1. **Structure scales, vibes don't** — for production, discipline is mandatory
2. **AI amplifies your engineering culture** — harness multiplies both strengths and weaknesses
3. **Generation is solved** — verification, judgment, and direction are the new craft
4. **What gets measured gets improved** — observability feeds RHO feeds harness evolution
5. **Tokens are money** — context compaction is not optional for OpEx control

## Related

- [The New SDLC With Vibe Coding](https://addyosmani.com/blog/agentic-engineering/) — whitepaper by Addy Osmani et al.
- [Awesome Harness Engineering](https://github.com/ai-boost/awesome-harness-engineering) — curated list that inspired this project
- [Hermes Agent](https://github.com/NousResearch/hermes-agent) — the agent platform these skills run on
