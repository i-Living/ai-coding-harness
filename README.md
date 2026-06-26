# AI Harness

Factory Model harness for AI-assisted development — Hermes Agent skills that automate quality gates and project infrastructure.

## What is this?

Two skills that form the backbone of disciplined AI-assisted development, following the [Factory Model](https://addyosmani.com/blog/factory-model/) paradigm:

> Build the system that builds software. The agent generates code; the harness verifies it.

## Skills

### 1. eval-harness

**Automatic verification gates** — fires after every code change:

- **Code gates** — `npx biome format` + `bun run check` after any file change
- **File gates** — verify file exists, non-empty, valid format after write
- **Git gates** — verify branch state, no pending changes
- **Wiki gates** — index.md, log.md, frontmatter integrity
- **Commit gates** — full lint + build + test before commit

### 2. factory-mode

**One-command harness setup** — audits a project and configures the full pipeline:

| Phase | What it does |
|-------|-------------|
| Audit | Check AGENTS.md, Biome, tests, pre-commit, CI, check script |
| Configure | Install and set up missing components |
| Verify | Run `check` + `build` to confirm pipeline works |
| Document | Update AGENTS.md with conventions, pitfalls, verification steps |

### What gets set up

```
spec → AI generates → bun run check (typecheck + lint + test) → pre-commit hook → push → CI gate → merge
```

Components:
- **AGENTS.md** — project conventions for AI agents
- **Biome** — formatter + linter
- **bun:test** — test runner with smoke test
- **Husky + lint-staged** — pre-commit formatting
- **GitHub Actions** — CI pipeline on push/PR
- **`bun run check`** — unified verification command

## Quick Start

```bash
# In your Hermes session:
skill_view(name='factory-mode')
# → audits the current project, asks what to set up, configures everything

# Or load eval-harness for per-action verification:
skill_view(name='eval-harness')
# → after every code change, auto-runs format + check
```

## Structure

```
ai-harness/
├── skills/
│   ├── eval-harness/
│   │   └── SKILL.md          # Verification gates skill
│   └── factory-mode/
│       └── SKILL.md          # Harness setup skill
└── README.md                  # This file
```

## Principles

1. **Structure scales, vibes don't** — for production, discipline is mandatory
2. **AI amplifies your engineering culture** — harness multiplies both strengths and weaknesses
3. **Generation is solved** — verification, judgment, and direction are the new craft

## Related

- [The New SDLC With Vibe Coding](https://addyosmani.com/blog/agentic-engineering/) — whitepaper by Addy Osmani et al.
- [Hermes Agent](https://github.com/NousResearch/hermes-agent) — the agent platform these skills run on
- [LLM Wiki](https://github.com/i-Living) — knowledge base for AI-assisted development concepts
