## System Context

* **Part of:** `@ai-orchestrator/system/ORCHESTRATOR.md`
* **Used by:** `@ai-orchestrator/system/SKILL_DISCOVERY.md` — consulted during domain identification
* **Uses:** nothing
* **Outputs to:** `@ai-orchestrator/system/SKILL_EXECUTOR.md` (via skill selection in SKILL_DISCOVERY)

---

# SKILL MAPPING SYSTEM

## Purpose

Select relevant skills based on problem context.

---

## Discovery Command

`npx skills find`

---

## Auto-Selection Rules

### Frontend (React Native / Expo)

npx query: `npx skills find react-native`

Trigger when:

* `.tsx` files are in scope
* screens or components are mentioned
* UI behavior or rendering issues are reported

Possible skills:

* `react-native`
* `frontend-performance`
* `re-renders`
* `component-architecture`

### Backend (Express / API)

npx query: `npx skills find backend-validation`

Trigger when:

* routes, controllers, or middleware are involved
* validation, auth, or error behavior is in scope

Possible skills:

* `backend-validation`
* `api-design`
* `security`
* `error-handling`

### Database

npx query: `npx skills find database-performance`

Trigger when:

* schema, queries, migrations, or indexes are involved

Possible skills:

* `database-performance`
* `indexing`
* `schema-design`
* `transactions`

### Performance

npx query: `npx skills find performance`

Trigger when:

* re-renders are mentioned
* UI is slow
* repeated fetching or cache misses are present

Possible skills:

* `performance`
* `caching`
* `memoization`
* `state-optimization`

### Architecture

npx query: `npx skills find architecture`

Trigger when:

* files are large
* boundaries are unclear
* ownership or scaling concerns appear

Possible skills:

* `architecture`
* `modularization`
* `separation-of-concerns`

---

## Rules

* Select at most 3-5 skills.
* Prefer specific skills over general skills.
* Include performance-related skills when evidence supports it.
* Document why each skill was selected.

---

## Output Format

* `selected_skills[]`
* `reason_per_skill`
* `expected_value_per_skill`
