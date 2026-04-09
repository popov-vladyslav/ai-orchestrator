## System Context

* **Part of:** `@ai-orchestrator/system/ORCHESTRATOR.md`
* **Used by:** ORCHESTRATOR — when delegating bounded work to a specialist agent
* **Uses:** `@ai-orchestrator/system/WORKSPACE.md` (shared workspace contract)
* **Outputs to:** nothing — defines the delegation contract, not a pipeline step

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
5. Moves tickets TODO → IN PROGRESS → DONE in `tasks.md`
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

## Invocation Template

When delegating, provide the specialist with:

```
Read @ai-orchestrator/system/ORCHESTRATOR.md

Execute tickets [T-001, T-002] from workspace:
  <workspace_path>/

Read context.md for orientation, tickets.md for full specs.
Update tasks.md and results.md as you work.
```
