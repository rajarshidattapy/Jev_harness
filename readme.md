# Jev Agent Harness

Link: https://x.com/sydneyrunkle/status/2100754364545761643

A lightweight harness for running and evaluating coding agents with **Jev**.

The harness provides a controlled environment where an agent can:

* inspect a repository
* plan and execute tasks
* use tools
* modify code
* run tests
* observe results
* iterate until the task is complete

## Architecture

```text
Task
  ↓
Harness
  ↓
Jev Agent
  ├── Tools
  ├── Repository
  ├── Execution Environment
  └── Test Runner
  ↓
Observations
  ↓
Agent Loop
```

## Goal

Build a minimal, reproducible environment for experimenting with agent behavior, tool use, and task completion.

## Status

🚧 Early development
