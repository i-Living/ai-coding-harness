# CONTEXT.md — AI Coding Harness Domain Glossary

Shared vocabulary for all 12 harness skills. Use these terms exactly — consistent language is how agents and humans stay aligned.

## Core Concepts

### Harness
The system that builds the system that builds software. The harness is NOT the code — it's the verification gates, quality pipelines, context management, and self-improvement loops that surround the code. The agent generates; the harness verifies, compacts, observes, and improves.

### Gate
A verification checkpoint that blocks progress until passed. Gates are automatic (eval-harness fires after every code change), manual (human review), or pipeline (CI, pre-commit hooks). A green gate = proceed. A red gate = stop and fix.

### Verification
Confirmation that a change meets a standard. Not just "tests pass" — verification includes formatting, linting, type checking, runtime behavior, and manual checks. Every task in `planning-and-task-breakdown` must have a verification step.

### Factory Model
The architectural pattern where AI generates code and the harness verifies it. Named after the factory system that produced interchangeable parts — the harness ensures every output meets the same quality bar, regardless of which agent produced it.

### Pipeline
The end-to-end flow from intent to production:
```
interview → spec → plan → implement → TDD → review → verify → commit
```

### Depth / Shallow
A measure of interface quality. A **deep** module packs a lot of behavior behind a small interface (high leverage). A **shallow** module's interface is nearly as complex as its implementation (low leverage). The `improve-codebase-architecture` skill hunts for shallowness.

### Seam
A place where behavior can be altered without editing in place. Interfaces, dependency injection points, and configuration boundaries are seams. One adapter = hypothetical seam. Two adapters = real seam.

## Workflow Terms

### Vertical Slice
One complete end-to-end path through the stack (DB → API → UI) for a single user action. "User can create a task" is a vertical slice. "Build all database tables" is NOT — that's horizontal slicing (bad). See `planning-and-task-breakdown` §3.

### RED-GREEN-REFACTOR
The TDD cycle: write a failing test (RED), write minimal code to pass (GREEN), clean up (REFACTOR). No code before a failing test. See `test-driven-development`.

### Increment
One unit of work in the cycle: Implement → Test → Verify → Commit → Next. Each increment leaves the system compilable, tested, and working. See `incremental-implementation`.

### Checkpoint
A verification pause after 2-3 tasks. Confirms the system is still in a working state before proceeding. See `planning-and-task-breakdown` §5.

## Quality Terms

### 5-Axis Review
The review framework: Correctness, Readability, Architecture, Security, Performance. Every PR is evaluated on all five axes. See `code-review-and-quality`.

### Definition of Done
The standing, project-wide bar every change must clear. Different from acceptance criteria (which vary per task). DoD is fixed: Correctness → Quality → Integration → Documentation → Ship-readiness. See `code-review-and-quality/references/definition-of-done.md`.

### Pre-commit Pipeline
Security scan → independent reviewer → auto-fix → verified commit. A fresh agent context reviews your changes. See `requesting-code-review`.

## Optimization Terms

### Context Compaction
Reducing token usage without losing information quality. Two sides: **token economics** (grep filters, selective extraction, structured summaries) and **context quality** (hierarchy, trust levels, confusion management). See `context-compaction`.

### RHO (Retrospective Harness Optimization)
The self-improvement loop: DETECT failures → CLASSIFY root cause → PROPOSE harness fix → VALIDATE → OBSERVE. The harness improves itself by analyzing its own failures. See `rho-retrospective-harness-optimization`.

### Agent Observability
Tracking what the agent did, what worked, what failed, and why. Three levels: task outcomes (always), action traces (debugging), aggregate metrics (weekly). Observability feeds RHO with data. See `agent-observability`.

## Skill Categories

| Category | Skills | Purpose |
|----------|--------|---------|
| **Define** | interview-me, spec-driven-development | Clarify intent, write spec |
| **Plan** | planning-and-task-breakdown | Break spec into tasks |
| **Build** | incremental-implementation, test-driven-development | Execute tasks |
| **Review** | code-review-and-quality, requesting-code-review | Verify before merge |
| **Infrastructure** | factory-mode, eval-harness | Set up and run the harness |
| **Optimization** | context-compaction, rho-retrospective-harness-optimization, agent-observability | Improve efficiency and quality over time |

## Conventions

- **Stack:** Bun, TypeScript, Biome (formatting + linting)
- **Verification:** `bun run check` = typecheck + lint + test
- **Test runner:** `bun:test` (auto-discovers `*.test.ts`)
- **Non-Bun projects:** skills document fallbacks (npm, pytest, cargo, go)
