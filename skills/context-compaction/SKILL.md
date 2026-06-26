---
name: context-compaction
description: "Minimize token usage when sending context to LLMs: think-in-code pattern, selective extraction, structured output."
version: 1.0.0
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [context, token-economics, optimization, op-ex, compaction]
    category: ship
    related_skills: [eval-harness, factory-mode, context-engineering]
---

# Context Compaction — Token Economics

Reduce token consumption in AI agent interactions. Every token costs money (OpEx) and dilutes signal. This skill teaches patterns that cut token usage without losing information quality.

**Principle:** The agent's context window is expensive real estate. Don't dump — extract.

## When to Apply

**Always** before sending large payloads to an LLM:
- Diff > 500 lines → compact first
- Source file > 300 lines → extract relevant sections
- Multiple files to review → batch with summaries
- Web content > 5000 chars → summarize before embedding
- Log output > 200 lines → grep for errors only

## Patterns

### Pattern 1: Think in Code

Instead of dumping data into the prompt, write a script that processes it:

```bash
# BAD: dump 2000 lines of build output into context
terminal("bun run build 2>&1")  # → 2000 lines in context

# GOOD: write a filter script, get only relevant output
terminal("bun run build 2>&1 | grep -iE 'error|fail|warning' | tail -20")
```

```bash
# BAD: read entire large file into context
read_file("server/handler.ts")  # → 800 lines in context

# GOOD: extract only the function you need
terminal("sed -n '/export async function handleRequest/,/^}/p' server/handler.ts")
```

**Rule of thumb:** if output > 100 lines, pipe through `grep`/`sed`/`awk` first.

### Pattern 2: Selective Extraction

Read only what you need, not the whole file:

```bash
# BAD
read_file("src/large-module.ts")  # 600 lines

# GOOD
search_files(pattern="export function", path="src/large-module.ts", output_mode="content")
# → finds only function signatures

# GOOD
read_file("src/large-module.ts", offset=120, limit=40)  # just the section you need
```

### Pattern 3: Structured Summaries

When you must pass context from one agent to another (delegate_task), use structured summaries:

```python
# BAD
delegate_task(
    goal="Review this code",
    context=f"Here's the full output:\n{full_output}"  # dumps 5000 chars
)

# GOOD
delegate_task(
    goal="Review this code",
    context=f"""Files changed: {len(files)}
Key logic changes:
- {changes_summary[:500]}
Test results: {test_summary[:200]}
Full diff available on request."""
)
```

### Pattern 4: Batch Independent Operations

Don't serialize reads — batch them:

```bash
# BAD: 3 separate read_file calls → 3 LLM roundtrips
read_file("file1.ts")
read_file("file2.ts")
read_file("file3.ts")

# GOOD: one terminal call, grep for what you need
terminal("grep -n 'export' file1.ts file2.ts file3.ts")
```

### Pattern 5: Progressive Disclosure

Don't load full context upfront — disclose as needed (already core to Hermes skills system):

```
Level 1: Skill metadata (name + description) — 50 tokens
Level 2: Full skill instructions — loaded on task match — 500 tokens
Level 3: Reference material — loaded on explicit call — 2000 tokens
```

## When NOT to Compact

- **User explicitly asks for full output** — respect the request
- **Debugging unknown error** — first investigation needs full context
- **Small payloads** — < 50 lines not worth the grep overhead
- **Single-line results** — "Tests: 3 pass, 0 fail" doesn't need compaction

## Compaction Checklist

Before sending content to an LLM, ask:
- [ ] Is this entire output needed, or just key parts?
- [ ] Can I filter with `grep`/`sed`/`awk` instead of full dump?
- [ ] Is this > 100 lines? If yes, consider summarization.
- [ ] Can I use `read_file(offset, limit)` instead of full file?
- [ ] Would a structured summary (changed files, key functions) suffice?

## Token Savings Reference

| Pattern | Typical Reduction | When |
|---------|------------------|------|
| grep filter | 80-95% | Logs, build output |
| offset/limit read | 70-90% | Large source files |
| Structured summary | 60-85% | delegate_task context |
| Batch grep | 50-70% | Multi-file exploration |
| Progressive disclosure | 40-60% | Skill loading |

## Pitfalls

- **Don't over-filter** — missing a relevant error line costs more than the tokens saved
- **Don't compact user-facing output** — user gets full result, compaction is for LLM context only
- **Grep on Cyrillic** — `grep -P` for Unicode, or `grep -i` for case-insensitive
- **Offset/limit on changing files** — if file was edited between reads, offsets shift
