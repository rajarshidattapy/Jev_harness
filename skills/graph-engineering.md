# Graph Engineering

## Purpose

This skill helps design, implement, debug, and evaluate multi-agent and multi-step systems where explicit control flow connects specialized execution units.

Graph engineering is the layer above individual agent loops.

A graph wires together nodes such as:

- Agent loops
- Tools
- Deterministic functions
- Validators
- Human approval steps
- Routers
- State transformations

Edges define how control and state move between those nodes.

---

## Core Mental Model

A graph is:

```text
START
  ↓
NODE
  ├──→ NODE
  └──→ NODE
          ↓
       MERGE
          ↓
        END
```

Each node can perform one bounded unit of work.

Importantly, a node may itself contain an autonomous loop:

```text
Graph
 ├── Researcher Loop
 ├── Retriever Loop
 ├── Analysis Loop
 └── Reviewer Loop
```

Therefore:

**Loop engineering = engineering an autonomous execution cycle.**

**Graph engineering = engineering how multiple execution cycles and other nodes coordinate.**

---

## When to Use a Graph

Use graph orchestration when one loop is no longer enough to hold the workflow together.

Strong signals:

### 1. Parallel work

Independent tracks can execute simultaneously.

```text
              ┌→ Research Loop ─┐
START ────────┤                  ├→ Merge
              └→ Retrieval Loop ─┘
```

Do not serialize independent work unnecessarily.

### 2. Specialized agents

Different tasks need different expertise, tools, memory, or prompts.

Examples:

```text
Researcher
Retriever
Coder
Tester
Reviewer
Planner
```

Each can own a smaller loop or deterministic task.

### 3. Different permissions

Some nodes may require:

- Read-only access
- Repository write access
- Production access
- Database access
- External API credentials
- Human approval

The graph can make these boundaries explicit.

### 4. Different contexts or memory

A single giant context is often the wrong abstraction.

Different nodes can receive only the state relevant to their role.

### 5. Explicit branching

The workflow needs deterministic routing:

```text
if tests_pass:
    deploy
else:
    debugger
```

### 6. Human-in-the-loop control

A graph makes approval points explicit:

```text
Agent → Review → Human Approval → Deploy
```

### 7. Auditable control flow

For high-value or production systems, you may need to answer:

- Which node ran?
- Why did it run?
- What state did it receive?
- What decision caused the next transition?
- Which branch was taken?
- Where did the final result originate?

Explicit graph state makes this easier to inspect.

---

## Graph Building Blocks

### Nodes

A node should have a clear responsibility.

Examples:

```text
research()
retrieve()
generate()
validate()
review()
approve()
deploy()
```

Avoid nodes that secretly perform the entire workflow. That recreates a monolithic loop inside the graph.

### Edges

Edges define transitions.

Types include:

- Sequential
- Conditional
- Parallel
- Fan-out
- Fan-in
- Retry
- Escalation
- Human approval

### State

The graph needs an explicit state model.

Example:

```python
state = {
    "goal": ...,
    "research": ...,
    "retrieval": ...,
    "analysis": ...,
    "validation": ...,
    "artifacts": ...,
}
```

State should be structured and intentional.

Do not pass an uncontrolled blob of conversation history between every node.

---

## Fan-Out and Fan-In

One of the most important graph patterns:

```text
             ┌→ Researcher ─┐
START ───────┤               ├→ Reviewer
             └→ Retriever ──┘
```

The branches run independently, then merge.

Fan-out is useful when work is independent.

Fan-in is useful when a later node needs the combined result.

---

## Conditional Routing

Graphs become useful when control flow itself is part of the problem.

Example:

```text
                 ┌→ Fixer
Validator ───────┤
                 └→ Reviewer
```

The transition should be based on explicit state:

```python
if validation.failed:
    next_node = "fixer"
else:
    next_node = "reviewer"
```

Do not hide important routing decisions inside vague agent reasoning when deterministic rules are sufficient.

---

## Loops Inside Graphs

A graph can contain cycles:

```text
Coder
  ↓
Tester
  ↓
  ├── pass → Reviewer
  └── fail → Coder
```

Here, the graph controls the workflow while individual nodes may themselves be autonomous loops.

This is the key relationship:

```text
Loop = local autonomy
Graph = global coordination
```

---

## Graph Design Principles

### 1. Start from the workflow, not the framework

First identify:

```text
goal
↓
units of work
↓
dependencies
↓
parallelism
↓
state
↓
routing
↓
verification
↓
termination
```

Only then choose an orchestration framework.

Framework choice is implementation detail. Control-flow design is architecture.

### 2. Keep nodes narrow

A node should have one meaningful responsibility.

Bad:

```text
SuperAgent
  research
  retrieve
  code
  test
  deploy
  review
```

Better:

```text
Researcher
Retriever
Coder
Tester
Reviewer
```

### 3. Minimize shared state

Shared state is powerful but creates coupling.

Prefer:

```text
Node A → relevant output → Node B
```

over:

```text
Every node can mutate everything.
```

### 4. Make transitions observable

Every edge should be understandable.

Record:

```text
source_node
target_node
condition
state_delta
timestamp
run_id
```

### 5. Make failures local where possible

A failure in one branch should not necessarily destroy unrelated branches.

Use:

- Branch-level retries
- Timeouts
- Fallback nodes
- Dead-letter / failure states
- Human escalation

### 6. Verify at boundaries

Important graph boundaries should have validation.

```text
Research → validate research
Generation → validate artifact
Execution → validate result
Merge → validate combined state
```

---

## Common Graph Patterns

### Parallel specialists

```text
START
 ├── Research
 ├── Retrieval
 └── Analysis
       ↓
     Merge
```

### Planner → workers → reviewer

```text
Planner
   ↓
 ┌─┼────────┐
 ↓ ↓        ↓
A  B        C
 └─┼────────┘
   ↓
Reviewer
```

### Generator → validator → fixer

```text
Generator
    ↓
Validator
 ┌──┴──┐
pass  fail
 ↓      ↓
END   Fixer
        ↓
     Validator
```

### Approval gate

```text
Agent
  ↓
Review
  ↓
Human Approval
  ├── approve → Execute
  └── reject  → Revise
```

---

## Graph Failure Modes

### Unnecessary orchestration

A simple sequential task is turned into ten nodes.

Result:

- More latency
- More state management
- More failure modes
- Harder debugging
- No meaningful capability gain

Fix: collapse the graph back into a loop or simpler workflow.

### Shared-state chaos

Every node can mutate every field.

Fix:

- Typed state
- Ownership rules
- Explicit state transitions
- Narrow node interfaces

### Hidden loops

A node silently retries forever.

Fix:

- Bound node-level loops
- Expose retry counts
- Propagate terminal states

### Graph dead ends

A branch has no valid successor.

Fix:

- Validate graph topology
- Define failure states
- Define terminal states explicitly

### Merge conflicts

Parallel branches produce incompatible state.

Fix:

- Define merge semantics before implementation
- Use deterministic reducers
- Validate merged state

### Over-agentization

A deterministic function becomes an LLM agent for no reason.

Fix:

Use the simplest reliable primitive:

```text
deterministic function > tool > model > autonomous agent
```

when the task allows it.

---

## Observability

Production graph systems should expose:

```text
run_id
node_id
parent_node
transition
branch
state_in
state_out
tool_calls
latency
tokens
cost
verification
failure
termination_reason
```

A useful trace should let an engineer reconstruct the execution path.

Example:

```text
START
 ↓
planner
 ↓
researcher [parallel]
 ↓
retriever  [parallel]
 ↓
merge
 ↓
reviewer
 ↓
END
```

---

## Frameworks

Graph engineering is a pattern, not a specific framework.

Common implementations include:

- LangGraph
- Microsoft AutoGen
- Google ADK
- Custom state-machine/orchestration systems

Do not assume a framework is required. A small explicit state machine may be more appropriate for a bounded workflow.

---

## Graph Engineering Checklist

Before shipping, ask:

- Why is this a graph instead of one loop?
- Which work is genuinely independent?
- Which nodes require different context or permissions?
- What state crosses each edge?
- Which transitions are deterministic?
- Which transitions require model reasoning?
- Where are the verification boundaries?
- What happens when one branch fails?
- How are parallel results merged?
- Can every node terminate?
- Are retries bounded?
- Can the entire execution path be reconstructed from telemetry?
- Could any node simply be a function or tool instead of an agent?

If these answers are unclear, the graph is probably under-specified.

---

## Loop vs Graph

Do not treat them as competing architectures.

A loop is a local autonomous cycle.

A graph is a coordination structure that can contain many loops.

```text
             GRAPH
       ┌───────┼────────┐
       ↓       ↓        ↓
    Loop A   Tool     Loop B
       ↓                ↓
       └──────→ Merge ←─┘
```

Start with a loop when one agent can own the task end to end.

Introduce graph orchestration when the work requires:

- Parallel execution
- Specialized agents
- Different contexts or memory
- Different permissions
- Explicit branching
- Human approval
- Auditable state transitions

The goal is not to "graduate" from loops to graphs.

The goal is to use the smallest architecture that makes the workflow reliable and understandable.
