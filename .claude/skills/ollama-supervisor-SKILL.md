---
name: ollama-supervisor
description: Orchestration pattern for using Claude as supervisor and Ollama as worker. Apply this whenever building software to minimise Claude credit usage. Claude plans, routes, judges output quality, and handles complex reasoning. Ollama executes code generation, review, testing, and refactoring. Documented to achieve 8-10x cost reduction.
---

# Ollama Supervisor Pattern

Claude credits are expensive. Ollama runs locally and costs nothing. The default posture is: Ollama executes, Claude supervises. Route every task to Ollama first. Escalate to Claude only when Ollama is insufficient or the task genuinely requires deep multi-file reasoning.

Research shows this pattern achieves **8–10x cost reduction** while maintaining output quality on the tasks that matter.

---

## Role definitions

**Claude — supervisor**
Plans the approach, decomposes tasks, routes to Ollama, reads and judges Ollama's output, makes architectural decisions, and synthesises results. Claude's advantage is multi-file context, long-range reasoning, and domain nuance. Use it for those things.

**Ollama — worker**
Executes well-defined, bounded tasks: code generation, file review, bug fixing, refactoring, test writing. Ollama performs at near-Claude level on single-file, clearly-specified tasks. Its weakness is multi-file context and complex reasoning chains — don't give it those.

---

## Task routing

### Route to Ollama (always try first)

| Task | Tool |
| ---- | ---- |
| Generate code for a specific file or function | `ollama_generate_code` / `ollama_generate_code_with_context` |
| Review a file for bugs, correctness, style | `ollama_general_task` with file content in context |
| Fix a specific bug | `ollama_fix_code` |
| Refactor or clean up a module | `ollama_refactor_code` |
| Write tests for a specific function or file | `ollama_write_tests` |
| Explain what a file or function does | `ollama_explain_code` |
| Draft documentation, copy, or content | `ollama_general_task` |
| Structural review (architecture, naming, delivery risk) | `ollama_general_task` |
| Single-file or well-scoped multi-file tasks | `ollama_generate_code_with_context` |

### Keep in Claude (do not delegate)

- Tasks spanning many files where reasoning must hold across all of them simultaneously
- Security decisions (authentication, authorisation, data exposure)
- Domain nuance requiring specialist knowledge (theology, legal, medical, compliance)
- Synthesising results from multiple Ollama outputs into a single coherent decision
- Final judgment on whether Ollama's output is correct, complete, and safe to ship
- Anything the user has explicitly asked Claude to own

**The routing heuristic:** if the task fits in one file with a clear spec, Ollama. If it requires holding the whole system in mind, Claude.

---

## Pre-delegation checklist

Before routing a task to Ollama, confirm it meets this gate. Vague tasks will fail.

- [ ] **Task is specific.** Not "fix the dark mode" but "replace bg-parchment with bg-input-bg in JoinScreen inputs (lines 63, 78) and add dark: color classes to text elements"
- [ ] **Success criteria are explicit.** Define what "done" looks like: "inputs render with proper contrast in both themes" or "config validates without errors"
- [ ] **Constraints are stated.** Style, naming, conventions, forbidden patterns: "no pseudo-classes in color definitions", "match existing component patterns", "follow RFC 7231 for HTTP headers"
- [ ] **Context files are identified.** What does Ollama need to read? List specific files or patterns, not "everything"
- [ ] **Architecture is documented if complex.** For system-level tasks (config, patterns, scaffolding), include key constraints upfront: tech stack, key libraries, naming conventions

**Quick template before delegating:**

```text
TASK: [specific objective]
SUCCESS: [how to verify it works]
CONSTRAINTS: [style/pattern/convention rules]
CONTEXT: [affected files]
OUTPUT: [format/location]
```

If you can't fill all five fields, the task is not ready to delegate. Refine it first.

---

## Hallucination detection

Ollama will sometimes produce output completely unrelated to the task. Detect this immediately and escalate.

**Hallucination checklist (run after Ollama returns output):**

- [ ] Is the output relevant to the stated task?
- [ ] Does it address the specific requirement?
- [ ] Are all stated constraints met?
- [ ] Does it integrate with existing code?

**Escalation thresholds:**

| Symptom | Action |
| --- | --- |
| Minor deviation (e.g., missing optional field) | Retry with clarification |
| Partial failure (e.g., got half the component right) | Retry with more context or stricter spec |
| **Complete hallucination (wrong component, unrelated code)** | **Escalate immediately to Claude** |
| Same failure 3+ times | Escalate + re-evaluate task routing |

---

## Prompt engineering for Ollama

Simple file context is not enough. Include constraints and architecture explicitly.

**Always include:**

```markdown
### Architecture Context
- **Framework:** [e.g., React + Tailwind + TypeScript]
- **Conventions:** [naming, patterns, style rules]
- **Key constraints:** [forbidden patterns, must-respect rules]
- **File structure:** [brief overview if relevant]

### Task
- **What:** [specific objective]
- **Why:** [brief reason/context]
- **Constraints:** [explicit rules and limits]
- **Success:** [how to verify it works]

### Examples (if available)
- Similar working code
- Pattern to follow
```

---

## Retry and escalation thresholds

```text
Attempt 1:  Ollama with the basic prompt
Attempt 2:  Ollama with more context (add relevant files, examples, constraints)
Attempt 3:  Claude generates → Ollama reviews the output
Attempt 4+: Tell the user what Ollama produced and why it's insufficient
```

Do not retry the same prompt more than twice. If Ollama fails twice on the same task, the task is likely too complex or underspecified for Ollama — either tighten the spec or escalate to Claude.

---

## Context management

**Ollama model:** qwen3.5 (9.7B, Q4_K_M quantization)

- **Context window:** 262,144 tokens (massive — larger than Claude's limit)
- **Capabilities:** completion, vision, tools, thinking
- **Practical limit:** For code tasks, aim for 20–50k tokens of context

**Strategy:** Ollama's enormous context window means you can pass entire modules or multiple related files at once without worry. Don't artificially split large tasks into tiny fragments.

**Guideline:** If you're hesitating to pass a file because it's "too much context," pass it. Ollama won't choke on it. The only reason to split is semantic — the task is too broad and needs decomposition first.

---

## Tool rules

**Use `ollama_general_task` not `ollama_review_file`.**
`ollama_review_file` is unreliable on Windows paths. Always pass file content as a string in the `context` field of `ollama_general_task`.

**Never spawn parallel Claude sub-agents for reviews.**
One `ollama_general_task` call does the same job at zero cost. Parallel Claude agents are the single most expensive operation in this workflow — avoid unless the user explicitly requests a multi-perspective review.

**If `ollama_generate_code_with_context` returns empty output:**
This is a tool reliability issue, not a signal to skip Ollama. Generate in Claude, then pass the output to `ollama_general_task` for review.

---

## Verification by language

Run the appropriate check after every Ollama output before shipping:

| Language | Command |
| -------- | ------- |
| TypeScript / JavaScript | `npx tsc --noEmit` |
| Python | `mypy .` or `pyright` |
| Any with tests | `npm test` / `pytest` |
| Web app build | `npm run build` |

---

## Pattern for every coding task

```text
1. Claude: read context, plan the approach, decompose into bounded tasks
2. Claude: route each task — Ollama for execution, Claude for synthesis
3. Ollama: execute (generate / review / fix / test)
4. Claude: read Ollama output, check against acceptance criteria
5. If acceptable: write to disk, run type-check, move on
6. If not acceptable: retry with more context (max 2 retries), then escalate
7. After each phase: one Ollama review pass before starting the next phase
8. Claude: synthesise phase output and plan the next phase
```

---

## When Ollama is unavailable

If Ollama tools return errors or empty output consistently:

1. `ollama list` — confirm Ollama is running and the model is available
2. `ollama run <model> "hello"` — confirm the model responds
3. Fall back to Claude for that session
