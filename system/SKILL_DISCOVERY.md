## System Context

* **Part of:** `@ai-orchestrator/system/ORCHESTRATOR.md`
* **Used by:** All modes — after problem framing (Phase 2)
* **Uses:** `@ai-orchestrator/skills-mapping.md` for domain → skill auto-selection rules
* **Outputs to:** `@ai-orchestrator/system/SKILL_EXECUTOR.md` — one call per selected skill

---

You are responsible for selecting the best available skills.

## Input

* problem description, review output, or feature framing
* in PERFORMANCE_MODE: also accepts `skills_requested[]` from `@ai-orchestrator/skills/performance-optimization.md` — use as the starting skill selection, then refine with `@ai-orchestrator/skills-mapping.md`

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
2. Consult `@ai-orchestrator/skills-mapping.md` for auto-selection rules per domain.
3. Run: `npx skills find <query>` where `<query>` is the npx query string from `@ai-orchestrator/skills-mapping.md` for each detected domain.
4. Parse the output: if a matching skill is returned, use its resolved identifier as `skill_name` for `@ai-orchestrator/system/SKILL_EXECUTOR.md`. If no match is returned or `npx` is unavailable, fall back to inline domain analysis using the category rules in `@ai-orchestrator/skills-mapping.md` and set `skill_name` to `inline:<domain>`.
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
* When falling back to inline analysis, apply the trigger rules and possible skills from `@ai-orchestrator/skills-mapping.md` directly as the skill execution — treat the category description as the skill spec.
