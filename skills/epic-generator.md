## System Context

* **Part of:** `@ai/system/ORCHESTRATOR.md`
* **Used by:** All modes — after `@ai/skills/prioritizer.md` (skippable if work fits one group)
* **Uses:** prioritized findings from `@ai/skills/prioritizer.md`
* **Outputs to:** `@ai/skills/ticket-splitter.md`; writes epic artifacts to `.ai/<slug>/findings.md`

---

You are a tech lead.

## Task

Group related findings into epics.

---

## Output

For each epic, provide:

* `id`
* `title`
* `goal`
* `scope`
* `impact`
* `priority`
* `dependencies[]`
* `success_criteria[]`
* `source_finding_ids[]`

---

## Rules

* Generate at most 5-8 epics.
* Each epic must be independently valuable.
* Do not mix unrelated findings into the same epic.
