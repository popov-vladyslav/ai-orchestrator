## System Context

* **Part of:** `@ai/system/ORCHESTRATOR.md`
* **Used by:** `@ai/prompts/task-executor.md` — reads full ticket artifacts for execution
* **Uses:** approved tickets from `@ai/skills/ticket-splitter.md`
* **Outputs to:** `@ai/prompts/reviewer.md` (completed items trigger review)

---

# Tasks

Lightweight execution board for approved tickets. Each entry must contain the full ticket artifact — not just `ticket_id - title`.

## Rules

* Each task must include all ticket fields from `@ai/skills/ticket-splitter.md` output schema.
* Do not add work that has not passed the planning checkpoint (Checkpoint 3).
* Move items across sections instead of duplicating them.
* The executor (`@ai/prompts/task-executor.md`) reads the full artifact from the IN PROGRESS section.

---

## TODO

```
ticket_id: T-001
epic_id: E-001
title: Example ticket
goal: One sentence goal
files: [path/to/file.ts]
changes: [description of change]
acceptance_criteria: [what done looks like]
risks: [any risks]
dependencies: []
parallelizable: false
```

## IN PROGRESS

(move full artifact block here when starting)

## DONE

(move full artifact block here when complete)
