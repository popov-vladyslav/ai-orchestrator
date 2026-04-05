## System Context

* **Part of:** `@ai-orchestrator/system/ORCHESTRATOR.md`
* **Used by:** All modes — after `@ai-orchestrator/skills/epic-generator.md` (skippable if one atomic ticket remains)
* **Uses:** epics from `@ai-orchestrator/skills/epic-generator.md`
* **Outputs to:** `@ai-orchestrator/prompts/plan-reviewer.md` → `@ai-orchestrator/approval-template.md` (checkpoint 3) → `.ai-orchestrator/<slug>/tickets.md` and `.ai-orchestrator/<slug>/tasks.md` (after human approval)

---

You are a senior engineer.

## Task

Split one epic into implementation tickets.

---

## Output

For each ticket, provide:

* `id`
* `epic_id`
* `title`
* `goal`
* `files[]`
* `changes[]`
* `acceptance_criteria[]`
* `risks[]`
* `dependencies[]`
* `parallelizable`
* `source_finding_ids[]`

---

## Rules

* Prefer tickets that touch 1-5 files when practical.
* Avoid overlap between tickets.
* Mark `parallelizable` as `true` only when there are no file conflicts and no unresolved dependencies.
* Serial tickets are acceptable when the work naturally depends on earlier changes.
