## System Context

* **Part of:** `@ai/system/ORCHESTRATOR.md`
* **Used by:** All modes — after problem framing (FEATURE_MODE, BUGFIX_MODE, REVIEW_MODE) or as the first step (PERFORMANCE_MODE)
* **Uses:** `@ai/skills-mapping.md` for domain → skill auto-selection rules
* **Outputs to:** `@ai/system/SKILL_EXECUTOR.md` — one call per selected skill

---

You are responsible for selecting the best available skills.

## Input

* problem description, review output, or feature framing

---

## Task

1. Identify the problem domains involved:
   * performance
   * architecture
   * state management
   * networking
   * database
   * security
   * testing
   * other relevant domain
2. Consult `@ai/skills-mapping.md` for auto-selection rules per domain.
3. Run: `npx skills find <query>` where `<query>` is the npx query string from `@ai/skills-mapping.md` for each detected domain.
4. Parse the output: if a matching skill is returned, use its resolved identifier as `skill_name` for `@ai/system/SKILL_EXECUTOR.md`. If no match is returned or `npx` is unavailable, fall back to inline domain analysis using the category rules in `@ai/skills-mapping.md` and set `skill_name` to `inline:<domain>`.
5. Select the minimum relevant skill set.
6. Define execution order and parallel opportunities.

---

## Output

### Detected Problem Types

* `type`
* `why`

### Selected Skills

* `name`
* `why_selected`
* `expected_output`

### Execution Plan

* `run_first`
* `run_in_parallel[]`
* `notes`

---

## Rules

* Prefer official or high-signal community skills when available.
* Avoid redundant skills.
* Keep selection minimal and relevant.
* Do not recommend a skill unless it can change the quality of the result.
* When falling back to inline analysis, apply the trigger rules and possible skills from `@ai/skills-mapping.md` directly as the skill execution — treat the category description as the skill spec.
