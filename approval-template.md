## System Context

* **Part of:** `@ai-orchestrator/system/ORCHESTRATOR.md`
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

## Checkpoint-Specific Focus

Adapt your review focus based on which checkpoint you are presenting:

* **Checkpoint 1 (Framing):** Is the problem correctly understood? Is the scope right-sized? Are constraints realistic?
* **Checkpoint 2 (Findings):** Are findings evidence-based? Are priorities defensible? Is anything missing?
* **Checkpoint 3 (Planning):** Are tickets atomic and scoped? Are dependencies correct? Is parallelization safe?
* **Checkpoint 4 (Execution):** Does the implementation match the tickets? Do quality checks pass? Are there regressions?

---

## Decision

* APPROVE
* REJECT
* MODIFY

---

## Notes

{{your notes}}
