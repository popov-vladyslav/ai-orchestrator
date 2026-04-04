## System Context

* **Part of:** `@ai/system/ORCHESTRATOR.md`
* **Used by:** All modes — after `@ai/system/SKILL_EXECUTOR.md` output and/or `@ai/skills/review-validator.md` output is available
* **Uses:** findings from `@ai/skills/review-validator.md` and/or `@ai/system/SKILL_EXECUTOR.md`
* **Outputs to:** `@ai/skills/prioritizer.md`; writes normalized findings to `.ai/<slug>/findings.md`

---

You are consolidating outputs from review validation and skill execution.

## Input

* one or more finding lists from `Review Validator` and/or `Skill Execution`

---

## Task

1. Merge duplicate findings.
2. Remove unsupported or low-signal findings.
3. Normalize wording and severity.
4. Assign stable identifiers.
5. Produce one canonical findings list for downstream planning.

---

## Output

### Findings

For each finding, provide:

* `id`
* `title`
* `summary`
* `category`
* `severity`
* `evidence`
* `impact`
* `recommended_action`
* `dependencies[]`

### Merged Items

* `new_finding_id`
* `source_items[]`

### Rejected Items

* `source_item`
* `reason`

### Passthrough Fields

Forward these as-is from upstream — do not merge or deduplicate:

* `missing_areas[]` — from `@ai/skills/review-validator.md`
  * `area`
  * `why_missing`
* `priority_hints[]` — from `@ai/skills/review-validator.md` and/or `@ai/system/SKILL_EXECUTOR.md`
  * `finding_title`
  * `priority_hint`
  * `why`

---

## Rules

* Do not pass raw duplicated prose downstream.
* Preserve the strongest evidence when merging.
* If severity is uncertain, choose the lower defensible value and note the uncertainty.
