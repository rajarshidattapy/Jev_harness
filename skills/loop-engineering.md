# Loop Engineering

## Purpose

This skill helps design, implement, debug, and evaluate autonomous agent loops.

A loop is one autonomous execution cycle owned by an agent:

**Plan → Act → Verify → Repeat → Stop**

The core idea is not simply "run an LLM repeatedly." Loop engineering is about making that cycle reliable, observable, bounded, and capable of recovering from failure.

---

## Core Mental Model

A loop should have five explicit concerns:

1. **Plan** — decide what to do next.
2. **Act** — execute tools, code, APIs, or other actions.
3. **Verify** — determine whether the action produced the intended result.
4. **Repeat / Recover** — revise the plan when verification fails.
5. **Stop** — terminate when a clear success, failure, budget, or safety condition is reached.

A canonical loop:

```text
START
  ↓
PLAN
  ↓
ACT
  ↓
VERIFY
  ├── done → STOP
  └── not done → PLAN
```

The loop should not depend on a human manually prompting the agent between iterations.

---

## When to Use a Loop

Start with a loop when:

- One agent can own the task end to end.
- The task has one coherent goal.
- The agent has a meaningful way to verify its work.
- There is a clear stopping condition.
- The work does not require independent parallel tracks.
- The same context, memory, tools, and permissions are sufficient throughout the task.

Examples:

- Debugging a failing test suite.
- Researching a bounded question and verifying sources.
- Writing code, running tests, and fixing failures.
- Iteratively improving an artifact against a specification.
- Autonomous bug triage.

Do not introduce a graph merely because the workflow contains several steps. Sequential steps can still belong inside one loop.

---

## Loop Design Principles

### 1. Define the goal precisely

A loop without a measurable goal becomes an endless agent conversation.

Bad:

> "Improve the code."

Better:

> "Make all existing tests pass without changing the public API."

The goal should define what "done" means.

### 2. Make verification explicit

Verification is the most important part of the loop.

Possible verifiers:

- Unit tests
- Type checking
- Linting
- Schema validation
- Evaluation models
- Browser checks
- API responses
- Compilation
- Human approval
- Deterministic invariants

Avoid:

```text
Agent thinks it is done → STOP
```

Prefer:

```text
Agent claims done
      ↓
Independent verification
      ↓
Pass → STOP
Fail → retry with evidence
```

### 3. Bound the loop

Every autonomous loop needs limits.

Useful budgets:

- Maximum iterations
- Maximum tokens
- Maximum execution time
- Maximum tool calls
- Maximum cost
- Maximum retries
- Maximum destructive actions

A stopping condition should exist even when the task cannot be completed.

### 4. Preserve the right state

Decide what survives between iterations.

Typical state:

```text
goal
current_plan
actions_taken
tool_results
verification_results
errors
attempt_count
artifacts
working_memory
```

Do not blindly carry the entire conversation forever. Persistent context should be intentional.

### 5. Treat failure as information

A failed verification should produce structured evidence that changes the next iteration.

```text
ACT
 ↓
VERIFY
 ↓
failure + evidence
 ↓
update plan
 ↓
ACT again
```

A loop that simply repeats the same action is not autonomous recovery; it is repetition.

### 6. Separate action from verification

The agent that performs an action does not necessarily need to be the sole authority deciding that the action succeeded.

For important tasks, use independent or deterministic verification.

---

## Reliability Patterns

### Retry with changed strategy

Do not retry identically after failure.

```text
attempt 1 → failure
attempt 2 → diagnose failure
attempt 3 → change strategy
```

### Escalation

After repeated failure:

```text
Loop
 ↓
retry budget exhausted
 ↓
escalate / request human input / fail safely
```

### Checkpointing

Persist important intermediate state so the loop can resume after interruption.

### Idempotency

Repeated execution should not corrupt state.

Prefer actions that are safe to retry or have explicit deduplication.

### Tool failure handling

Distinguish:

- Agent reasoning failure
- Tool failure
- External system failure
- Verification failure
- Budget exhaustion

These should not all produce the same recovery behavior.

---

## Observability

A production loop should expose enough information to answer:

- What was the agent trying to accomplish?
- Which iteration is running?
- What did it plan?
- Which tools did it call?
- What changed?
- What did verification find?
- Why did it retry?
- Why did it stop?
- How much time/tokens/cost were consumed?

Useful telemetry:

```text
run_id
iteration
plan
action
tool
tool_latency
tool_result
verification
failure_reason
state_transition
tokens
cost
termination_reason
```

The goal is not logging everything. The goal is making loop behavior explainable.

---

## Loop Failure Modes

### Infinite loops

Cause:

- No stopping condition
- Verification never reaches success
- Retry logic does not change strategy

Fix:

- Hard iteration budget
- Explicit terminal states
- Progress detection

### False completion

Cause:

- Agent self-reports success
- Weak verifier
- Verification checks the wrong property

Fix:

- Stronger independent verification
- Deterministic checks
- Test against the actual goal

### Repeated failure

Cause:

- Same failed strategy is repeated

Fix:

- Feed failure evidence into planning
- Force strategy variation
- Escalate after a budget

### Context bloat

Cause:

- Entire history is carried through every iteration

Fix:

- Summarize state
- Persist structured state
- Keep only decision-relevant context

### Tool thrashing

Cause:

- Agent repeatedly calls tools without measurable progress

Fix:

- Tool budgets
- Progress checks
- Better planning
- Explicit failure states

---

## Minimal Implementation Shape

Framework-independent pseudocode:

```python
state = initialize(goal)

for iteration in range(MAX_ITERATIONS):
    plan = agent.plan(state)

    action_result = agent.act(plan)

    verification = verify(
        goal=state.goal,
        result=action_result,
        state=state,
    )

    state.update(
        plan=plan,
        action=action_result,
        verification=verification,
        iteration=iteration,
    )

    if verification.success:
        return success(state)

    if verification.fatal:
        return failure(state)

return budget_exhausted(state)
```

The exact framework is secondary. The control semantics are the important part.

---

## Engineering Checklist

Before shipping a loop, ask:

- What exactly is the goal?
- What proves success?
- What happens after failure?
- Does the next iteration actually learn from the failure?
- What is the maximum number of iterations?
- What is the cost/time/tool budget?
- What state persists?
- Which actions are safe to retry?
- What happens when a tool fails?
- What happens when verification is ambiguous?
- What is the terminal state?
- Can I explain why the loop stopped?

If these answers are unclear, the loop is not engineered yet.

---

## Loop vs Graph

A graph is not the opposite of a loop.

A loop can be a node inside a graph.

Use a loop when one autonomous cycle can own the task. Introduce graph orchestration when the work requires multiple specialized execution cycles, parallel branches, different contexts or permissions, or explicit control over how state moves between them.

**Default: start with the smallest loop that can reliably solve the task.**
