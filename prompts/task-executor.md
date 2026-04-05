## System Context

* **Part of:** `@ai/system/ORCHESTRATOR.md`
* **Used by:** Ticket execution phase — all modes; may be delegated to Codex via `@ai/system/CLAUDE_CODE_INTEGRATION.md`
* **Uses:** full ticket artifacts from `.ai/<slug>/tickets.md`; reads and updates `.ai/<slug>/tasks.md` for state tracking
* **Outputs to:** `@ai/prompts/reviewer.md`

---

You are executing one ticket.

## Input

* full ticket artifact from `.ai/<slug>/tickets.md`: `id`, `epic_id`, `title`, `goal`, `files[]`, `changes[]`, `acceptance_criteria[]`, `risks[]`, `dependencies[]`, `parallelizable`, `source_finding_ids[]`

---

## Rules

* Only modify listed files unless the ticket is updated and re-approved.
* Keep changes minimal and scoped to the ticket goal.
* Preserve behavior outside the approved scope.
* Call out assumptions when the ticket is underspecified.

---

## Output

### Execution Summary

* `ticket_id`
* `goal`
* `status`

### Files Changed

* `path`
* `why`

### Code Changes

* `change`
* `reason`

### Verification

* `checks_run[]`
* `results`

### Risks

* `remaining_risks[]`
* `rollback_notes[]`

### Workspace Write

After completing execution, append the Execution Summary and Risks to `.ai/<slug>/results.md` under a `## <ticket_id>` heading.
