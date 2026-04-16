---
name: delegation
description: Defines the contract for delegating bounded work from the orchestrator to specialist agents. Load when Phase 4 execution should be handed to Codex, a parallel agent, or any other specialist. Agent-agnostic — works with Claude Code, Codex, GPT, or any tool that can read workspace files.
---

# DELEGATION

## Purpose

Defines the contract for delegating work from an orchestrator agent to one or more specialist agents. Agent-agnostic — works with any combination of AI tools (Claude Code, Codex, GPT, etc.).

---

## Roles

| Role | Responsibilities |
| ---- | ---------------- |
| **Orchestrator agent** | Owns the pipeline. Manages checkpoints, aggregates results, updates `context.md`, handles approval gates. |
| **Specialist agent** | Executes bounded subtasks (one or more tickets). Reads workspace, updates `tasks.md` and `results.md`. Does NOT update `context.md`. |

---

## Delegation Contract

When delegating work to a specialist agent, always pass:

1. **`workspace_path`** — absolute path to `.ai-orchestrator/<slug>/` so the specialist can read context and write results without needing the full conversation history
2. **`ticket_ids`** — which tickets to execute (e.g., `T-001`, `T-002`)
3. **`orchestrator_path`** — absolute path to `@ai-orchestrator/system/ORCHESTRATOR.md` so the specialist can load task-executor.md and other system files

The specialist agent:

1. Reads `context.md` → orient (goal, scope, constraints)
2. Reads `tickets.md` → full spec for assigned tickets
3. Reads `tasks.md` → check current state
4. Executes each assigned ticket per `@ai-orchestrator/prompts/task-executor.md`
5. Moves ticket `TODO → IN PROGRESS` in `tasks.md`; leaves it `IN PROGRESS` when done — the orchestrator sets `DONE` only after reviewer approval
6. Appends execution summary to `results.md`

---

## Rules

* Delegate only bounded work with clear inputs and expected outputs.
* Pass relevant artifact IDs and file scope.
* Do not delegate the same unresolved task to multiple agents.
* The orchestrator remains responsible for final aggregation and approval handling.
* The specialist does not present approval checkpoints — only the orchestrator does.
* If the specialist encounters a blocker, it logs the blocker to `results.md` and stops. The orchestrator handles escalation.

---

## Parallel Delegation

Multiple specialist agents can work in parallel when:

* their assigned tickets do not modify the same files
* there are no unresolved dependency edges between assigned tickets
* each agent has an isolated ticket set

The orchestrator monitors `tasks.md` to track progress across all specialists.

---

## Delegating to a Specialist Agent

When a ticket is implementation-heavy (effort M or L) and has clear file scope and acceptance criteria, delegate it to a specialist agent rather than executing inline.

Use whatever specialist capability your runtime provides — a subagent, Codex, a parallel Claude agent, or any tool that can read workspace files and follow the contract above.

**Example: Claude Code subagent**
```
Agent({
  prompt: "Execute tickets [T-001] from workspace: <workspace_path>/. Read context.md for orientation, tickets.md for full specs. Move each ticket TODO → IN PROGRESS when you start. Leave IN PROGRESS when done. Append execution summary to results.md."
})
```

**Example: Codex agent** (if `codex:codex-rescue` is available in your setup)
```
Execute tickets [T-001, T-002] from workspace:
  <workspace_path>/

Read context.md for orientation, tickets.md for full specs.
Move each ticket TODO → IN PROGRESS in tasks.md when you start it.
Leave tickets IN PROGRESS when done — the orchestrator sets DONE after review.
Append execution summary to results.md.
Orchestrator reference: <absolute-path>/system/ORCHESTRATOR.md
```

The specialist executes, writes to the workspace, and returns with tickets in `IN PROGRESS`. The orchestrator reads `results.md`, runs the reviewer, and sets each ticket to `DONE` after approval.

---

## Parallel Dispatch with Multiple Agents

When multiple independent tickets can run concurrently, dispatch one specialist agent per ticket or ticket group. Each agent receives its own ticket IDs and the shared workspace path.

Use whatever parallel dispatch capability your runtime provides (e.g., `superpowers:dispatching-parallel-agents` in Claude Code, multiple concurrent `Agent` calls, or any equivalent).

Recommended max concurrency: 2–4 agents. Monitor `tasks.md` to track progress across all specialists.

---

## Delegation Decision Table

| Ticket effort | File conflicts with others | Recommendation |
|---|---|---|
| XS or S | — | Execute inline |
| M or L | No conflicts, no deps | Delegate to Codex |
| M or L | Conflicts or unresolved deps | Execute serially |
| Multiple M/L, all independent | No file conflicts | Parallel Codex dispatch |

---

## Invocation Template

**Generic (any specialist):**

```
Read @ai-orchestrator/system/ORCHESTRATOR.md

Execute tickets [T-001, T-002] from workspace:
  <workspace_path>/

Read context.md for orientation, tickets.md for full specs.
Move each ticket TODO → IN PROGRESS in tasks.md when you start it.
Leave tickets IN PROGRESS when done — the orchestrator sets DONE after review.
Append execution summary to results.md.
```

**Example with a Codex-capable agent:**

```
Execute tickets [T-001, T-002] from workspace:
  /path/to/project/.ai-orchestrator/add-oauth-login/

Read context.md for orientation, tickets.md for full specs.
Move each ticket TODO → IN PROGRESS in tasks.md when you start it.
Leave tickets IN PROGRESS when done — the orchestrator sets DONE after review.
Append execution summary to results.md.
Orchestrator reference: /path/to/ai-orchestrator/system/ORCHESTRATOR.md
```
