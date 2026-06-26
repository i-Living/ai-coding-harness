# AI Coding Harness

Factory Model harness for AI-assisted development — Hermes Agent skills that automate quality gates, project infrastructure, and the full interview→spec→verify→optimize pipeline.

> Build the system that builds software. The agent generates code; the harness verifies, compacts, observes, and improves.

## Harness Skills (13)

### Define (pre-code)

| Skill | Role |
|-------|------|
| **interview-me** | Pre-spec intent extraction: one question at a time until ~95% confidence |
| **spec-driven-development** | SPECIFY → PLAN → TASKS → IMPLEMENT with spec as source of truth |

### Infrastructure

| Skill | Role |
|-------|------|
| **toolchain-discovery** | Detects project stack → produces canonical command map (test, lint, format, build). Makes all other skills stack-agnostic. |
| **factory-mode** | One-command harness setup: audit → configure → verify |
| **eval-harness** | Per-action verification gates after every code change |

### Workflow

| Skill | Role |
|-------|------|
| **planning-and-task-breakdown** | Break specs into ordered, verifiable tasks with vertical slicing |
| **incremental-implementation** | Build in thin slices: implement → test → verify → commit → next |
| **test-driven-development** | RED-GREEN-REFACTOR with `bun test` — no code without failing test |
| **code-review-and-quality** | 5-axis review before merge |
| **requesting-code-review** | Pre-commit pipeline: security scan → independent reviewer → auto-fix |

### Optimization

| Skill | Role |
|-------|------|
| **context-compaction** | Token economics + context quality: think-in-code, selective extraction, confusion management |
| **rho-retrospective-harness-optimization** | Self-improving harness: analyze failures → auto-patch skills |
| **agent-observability** | Agent traces, health checks, failure metrics — data for RHO |

### How They Work Together

```
toolchain-discovery (detect stack → command map)
      │
      ▼
interview-me (intent)
      │
      ▼
spec-driven-dev (spec)
      │
      ▼
planning-and-task-breakdown (tasks)
      │
      ▼
incremental-implementation ──→ TDD ──→ code-review ──→ requesting-code-review ──→ commit
      │                               │         │               │
      └───────────────────────────────┴─────────┴───────────────┘
                                        │
                                   eval-harness (per-action gates)
                                   context-compaction (token savings + quality)
                                        │
                                   agent-observability (traces + metrics)
                                        │
                                   RHO (analyze → improve → validate)
```

## Quick Start

```bash
# Set up a project from scratch:
skill_view(name='factory-mode')

# Extract intent before spec:
skill_view(name='interview-me')

# Write a spec before coding:
skill_view(name='spec-driven-development')

# Break into tasks:
skill_view(name='planning-and-task-breakdown')

# Full pipeline with optimization:
skill_view(name='eval-harness')
skill_view(name='context-compaction')
```

## Structure

```
ai-coding-harness/
├── AGENTS.md                                    # Agent instructions
├── README.md
├── CONTEXT.md                                   # Domain glossary
├── references/                                  # Shared checklists (security, performance, definition of done)
│   ├── security-checklist.md
│   ├── performance-checklist.md
│   └── definition-of-done.md
└── skills/
    ├── toolchain-discovery/SKILL.md              # Stack detection + command map
    ├── interview-me/SKILL.md                    # Pre-spec intent extraction
    ├── spec-driven-development/SKILL.md         # Spec-first workflow
    ├── planning-and-task-breakdown/SKILL.md     # Task decomposition
    ├── incremental-implementation/SKILL.md      # Thin slice execution
    ├── test-driven-development/SKILL.md         # TDD with bun test
    ├── code-review-and-quality/SKILL.md         # 5-axis review
    ├── requesting-code-review/SKILL.md          # Pre-commit pipeline
    ├── factory-mode/SKILL.md                    # Harness setup
    ├── eval-harness/SKILL.md                    # Verification gates
    ├── context-compaction/SKILL.md              # Token economics + quality
    ├── rho-retrospective-harness-optimization/SKILL.md  # Self-improving harness
    └── agent-observability/SKILL.md             # Traces + metrics
```

## The Pipeline

```
toolchain-discovery → interview → spec → plan → implement → TDD → review → verify → commit
         │                                                         │
   (command map                                             eval-harness
    consumed by                                             (per-action gates
    all skills)                                              using toolchain map)
         │                                                         │
   context-compaction ←─────────────────────────────→ RHO ──→ improve
   agent-observability
```

## Principles

1. **Structure scales, vibes don't** — for production, discipline is mandatory
2. **AI amplifies your engineering culture** — harness multiplies both strengths and weaknesses
3. **Generation is solved** — verification, judgment, and direction are the new craft
4. **What gets measured gets improved** — observability feeds RHO feeds harness evolution
5. **Tokens are money** — context compaction is not optional for OpEx control
6. **Intent before spec** — interview-me surfaces what the user actually wants before any code exists

## Related

- [The New SDLC With Vibe Coding](https://addyosmani.com/blog/agentic-engineering/) — whitepaper by Addy Osmani et al.
- [Awesome Harness Engineering](https://github.com/ai-boost/awesome-harness-engineering) — curated list that inspired this project
- [Agent Skills](https://github.com/addyosmani/agent-skills) — upstream source of several skills (interview-me, planning-and-task-breakdown, incremental-implementation)
- [Hermes Agent](https://github.com/NousResearch/hermes-agent) — the agent platform these skills run on
