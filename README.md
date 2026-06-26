# AI Coding Harness

Factory Model harness for AI-assisted development — Hermes Agent skills that automate quality gates, project infrastructure, and the full spec→verify pipeline.

> Build the system that builds software. The agent generates code; the harness verifies it.

## Harness Skills (6)

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
| **code-review-and-quality** | 5-axis review before merge — correctness, readability, architecture, security, performance |
| **requesting-code-review** | Pre-commit pipeline: security scan → independent reviewer → auto-fix |

### How They Work Together

```
spec-driven-dev → TDD → code-review → requesting-code-review → commit
      │              │         │               │
      └──────────────┴─────────┴───────────────┘
                        │
                   eval-harness
                 (fires after every action)
                        │
                   factory-mode
                (sets up the whole thing)
```

## Quick Start

```bash
# In your Hermes session:

# Set up a project from scratch:
skill_view(name='factory-mode')

# Write a spec before coding:
skill_view(name='spec-driven-development')

# Enforce TDD discipline:
skill_view(name='test-driven-development')

# Review before merge:
skill_view(name='code-review-and-quality')

# Full pre-commit pipeline:
skill_view(name='requesting-code-review')
```

## Structure

```
ai-coding-harness/
├── skills/
│   ├── eval-harness/SKILL.md              # Verification gates
│   ├── factory-mode/SKILL.md              # Harness setup
│   ├── spec-driven-development/SKILL.md   # Spec-first workflow
│   ├── test-driven-development/SKILL.md   # TDD with bun test
│   ├── code-review-and-quality/SKILL.md   # 5-axis review
│   └── requesting-code-review/SKILL.md    # Pre-commit pipeline
└── README.md
```

## The Pipeline

```
spec → AI generates → bun run check (typecheck + lint + test) → pre-commit hook → push → CI gate → merge
```

Components set up by **factory-mode**:
- **AGENTS.md** — project conventions for AI agents
- **Biome** — formatter + linter  
- **bun:test** — test runner with smoke test
- **Husky + lint-staged** — pre-commit formatting
- **GitHub Actions** — CI pipeline on push/PR
- **`bun run check`** — unified verification command

## Principles

1. **Structure scales, vibes don't** — for production, discipline is mandatory
2. **AI amplifies your engineering culture** — harness multiplies both strengths and weaknesses
3. **Generation is solved** — verification, judgment, and direction are the new craft

## Related

- [The New SDLC With Vibe Coding](https://addyosmani.com/blog/agentic-engineering/) — whitepaper by Addy Osmani et al.
- [Hermes Agent](https://github.com/NousResearch/hermes-agent) — the agent platform these skills run on
- [LLM Wiki](https://github.com/i-Living) — knowledge base for AI-assisted development concepts
