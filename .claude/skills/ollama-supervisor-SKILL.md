---
name: ollama-supervisor
description: Two-tier supervisor-worker orchestration for minimising cost while preserving quality. Primary strategy: Claude model-tier routing (Opus → Sonnet → Haiku). Secondary strategy: Ollama as free local worker for single-file execution. Apply this at the start of every coding session.
---

# Supervisor-Worker Orchestration

Two complementary strategies for cost reduction. Use them in combination: Claude-tier routing is the primary approach; Ollama is the free fallback for bounded execution tasks.

---

## Strategy 1 — Claude Model-Tier Routing (primary)

Route work across Claude models by cognitive demand. Tiering across Opus / Sonnet / Haiku achieves **60–80% cost reduction** compared to running Opus everywhere, while keeping all work at Claude quality.

### Role definitions

| Model | Role | When to use |
| ----- | ---- | ----------- |
| **Opus** | Architect + reviewer | Phase planning, design system decisions, expert panel invocations, security review, cross-file architectural reasoning, judging output from other models |
| **Sonnet** | Implementer | Complex multi-file feature work (booking wizard, search/filter, admin dashboard), anything requiring context across 3+ files simultaneously |
| **Haiku** | Executor | Single-file boilerplate (route files, schema tables, simple form components, type definitions, straightforward refactors) |

### How to switch in Claude Code

Use `/model` at phase boundaries — not mid-file. Switch to Opus at the start of a phase to plan and decompose; hand off to Sonnet/Haiku for execution; bring Opus back to review before closing the phase.

```bash
/model opus    ← planning, design decisions, expert panels, security
/model sonnet  ← complex feature implementation
/model haiku   ← boilerplate, single-file tasks, tests
```

### Routing heuristic

- **Opus if:** the task requires holding the whole system in mind simultaneously, involves security/auth decisions, or you are making a decision that is hard to reverse.
- **Sonnet if:** the task spans 2–5 files and requires understanding how they interact, or the component is complex enough that Haiku would hallucinate interfaces.
- **Haiku if:** the task fits in one file with a clear spec and you could write the prompt in under 50 words.

### Phase routing plan for this project

| Phase | Planning model | Execution model | Review model |
| ----- | -------------- | --------------- | ------------ |
| Marketing strategy | Opus | — | Opus |
| Design system / style guide | Opus | — | Opus |
| Auth + roles | Opus | Sonnet | Opus |
| Public pages (Landing, Search, Detail) | Sonnet | Sonnet / Haiku | Sonnet |
| Booking wizard | Opus | Sonnet | Opus |
| Dosha quiz + consultation | Sonnet | Haiku | Sonnet |
| User dashboard | Sonnet | Haiku | Sonnet |
| Center admin portal | Sonnet | Sonnet | Opus |
| Platform admin | Opus | Sonnet | Opus |
| Deployment | Opus | Haiku | Opus |

---

## Strategy 2 — Ollama Worker (free local execution)

Claude credits are expensive. Ollama runs locally and costs nothing. When a task is bounded enough to route to Haiku, it is usually also bounded enough to route to Ollama — at zero cost.

The default posture for single-file execution: try Ollama first. Escalate to Haiku if Ollama output fails quality checks. Escalate to Sonnet if Haiku output fails.

Research shows this achieves an additional **8–10x cost reduction** on top of model-tier routing for pure execution tasks.

### Ollama role definitions

**Claude (any tier) — supervisor**
Plans, routes, judges output, makes architectural decisions. Never delegates to Ollama anything that crosses file boundaries or requires security reasoning.

**Ollama (qwen3.5 9.7B) — worker**
Executes well-defined, bounded tasks in a single file. Near-Claude-Haiku quality on clearly-specified tasks. Weakness: multi-file context and complex reasoning chains.

### Task routing

#### Route to Ollama (try before Haiku)

| Task | Tool |
| ---- | ---- |
| Generate code for a specific file or function | `ollama_generate_code` / `ollama_generate_code_with_context` |
| Fix a specific bug in one file | `ollama_fix_code` |
| Refactor or clean up a module | `ollama_refactor_code` |
| Write tests for a specific function | `ollama_write_tests` |
| Review a file for bugs or style | `ollama_general_task` with file content |
| Draft copy or documentation | `ollama_general_task` |

#### Keep in Claude (Sonnet or Opus — never Ollama)

- Tasks spanning many files where reasoning must hold across all of them
- Security decisions (auth, authorisation, data exposure)
- Synthesising results from multiple Ollama outputs
- Final judgment on whether output is correct, complete, and safe

### Pre-delegation checklist

Before routing a task to Ollama, confirm it meets this gate:

- [ ] Task is specific — not "fix the search" but "add a `dosha` filter param to `/api/retreats` GET handler in `backend/routers/retreats.py` line 34"
- [ ] Success criteria are explicit — define what done looks like
- [ ] Constraints are stated — naming, patterns, forbidden approaches
- [ ] Context files are identified — list specific files, not "everything"

**Quick template:**

```text
TASK: [specific objective]
SUCCESS: [how to verify it works]
CONSTRAINTS: [style/pattern/convention rules]
CONTEXT: [affected files]
OUTPUT: [format/location]
```

### Hallucination detection

Run after every Ollama output:

- [ ] Is the output relevant to the stated task?
- [ ] Does it address the specific requirement?
- [ ] Are all constraints met?
- [ ] Does it integrate with existing code?

**Escalation thresholds:**

| Symptom | Action |
| --- | --- |
| Minor deviation | Retry with clarification |
| Partial failure | Retry with more context |
| Complete hallucination (wrong component, unrelated code) | Escalate to Haiku immediately |
| Same failure 3+ times | Escalate to Sonnet + re-evaluate spec |

### Retry ladder

```text
Attempt 1:  Ollama — basic prompt
Attempt 2:  Ollama — more context, stricter spec
Attempt 3:  Haiku — Claude-native with same spec
Attempt 4:  Sonnet — if Haiku also fails, task is too complex for the spec
```

Do not retry the same prompt more than twice at any tier.

---

## Combined pattern for every coding task

```text
1. Opus:   read context, plan the phase, decompose into bounded tasks
2. Opus:   route each task — Ollama/Haiku for execution, Sonnet for complex, Opus for synthesis
3. Worker: execute (Ollama first, escalate up the ladder as needed)
4. Opus:   read output, check against acceptance criteria
5.         If acceptable → write to disk, run type-check, move on
6.         If not → retry with more context (max 2 retries per tier), then escalate
7. Sonnet: one review pass after each phase before Opus closes it
8. Opus:   synthesise phase output, plan the next phase
```

---

## Verification after every worker output

Run the appropriate check before treating output as done:

| Language | Command |
| -------- | ------- |
| TypeScript / JavaScript | `npx tsc --noEmit` |
| Python | `mypy .` or `pyright` |
| Any with tests | `npm test` / `pytest` |
| Web app build | `npm run build` |

---

## Tool rules (Ollama-specific)

**Use `ollama_general_task` not `ollama_review_file`.**
`ollama_review_file` is unreliable on Windows paths. Always pass file content as a string in the `context` field of `ollama_general_task`.

**Never spawn parallel Claude sub-agents for reviews.**
One `ollama_general_task` call does the same job at zero cost.

**If `ollama_generate_code_with_context` returns empty output:**
Generate in Claude (Haiku), then pass the output to `ollama_general_task` for review.

**Ollama context window:** qwen3.5 has 262k tokens. Pass entire modules freely — split only when the task is semantically too broad, not because you're worried about context size.

---

## When Ollama is unavailable

1. `ollama list` — confirm Ollama is running and the model is available
2. `ollama run qwen3.5 "hello"` — confirm the model responds
3. Fall back to Haiku for that session; note which tasks are pending so the pattern resumes when Ollama is restored
