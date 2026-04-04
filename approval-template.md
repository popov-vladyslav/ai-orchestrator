## System Context

* **Part of:** `@ai/system/ORCHESTRATOR.md`
* **Used by:** Up to 4 approval checkpoints per mode (REVIEW_MODE, FEATURE_MODE, BUGFIX_MODE: checkpoints 1–4; PERFORMANCE_MODE: checkpoints 2–4)
* **Uses:** artifacts from the preceding step (findings, epics, tickets, or review output)
* **Outputs to:** human decision — APPROVE / REJECT / MODIFY

---

# APPROVAL TEMPLATE

## Purpose

Ensure human control over AI execution at the defined checkpoints.

---

## Approval Required

### Checkpoint

`{{CHECKPOINT_NAME}}`

### Risk Level

`{{LOW|MEDIUM|HIGH}}`

### Summary

{{short summary}}

---

## Structured Output

{{artifact summary with ids}}

---

## Decision Context

* `why_now`: {{reason}}
* `alternatives`: {{alternatives considered}}
* `open questions`: {{questions}}

---

## Risks

* {{risk_1}}
* {{risk_2}}

---

## Review Checklist

* [ ] Matches the goal
* [ ] Scope is controlled
* [ ] Risks are understood
* [ ] Output is specific enough to execute
* [ ] Safe to proceed

---

## Decision

* APPROVE
* REJECT
* MODIFY

---

## Notes

{{your notes}}
