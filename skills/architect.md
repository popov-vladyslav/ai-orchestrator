---
name: architect
description: Frames problems before analysis runs. Use in FEATURE_MODE and BUGFIX_MODE (Phase 1) to define scope, constraints, approach, and success criteria before skill discovery runs. Skip when input already defines scope, constraints, and success criteria explicitly — pass directly to SKILL_DISCOVERY instead.
---

## Overview

Frame the problem before any analysis runs. Defines scope, constraints, approach, and success criteria so that skill discovery and execution operate on a shared understanding — not assumptions.

## When to Use

- Input is a feature request, bug report, or open-ended problem statement
- Scope is unclear or could be interpreted multiple ways
- Constraints or assumptions need to be made explicit before skills run

**When NOT to use:**
- Input already defines scope, constraints, and success criteria explicitly — pass directly to `@ai-orchestrator/system/SKILL_DISCOVERY.md`
- PERFORMANCE_MODE — use `@ai-orchestrator/skills/performance-optimization.md` instead

## Process

1. Understand the goal.
2. Define scope (in and out).
3. State constraints and assumptions.
4. Propose a high-level approach — no implementation details, no ticket-level task breakdown.
5. Identify risks.
6. Define success criteria.

### Output fields

**Problem Understanding**
* `summary`
* `goal`

**Scope**
* `in_scope[]`
* `out_of_scope[]`

**Constraints**
* `constraints[]`
* `assumptions[]`

**Approach**
* `approach`
* `alternatives_considered[]`

**Risks**
* `risks[]`

**Success Criteria**
* `success_criteria[]`

### Example

**Input:** "Add OAuth login with Google and GitHub providers"

**Output (abbreviated):**
* **Problem Understanding:** Users need social login. Goal: add OAuth via Google and GitHub.
* **Scope:** In: OAuth provider config, callback handler, session integration. Out: UI changes, other providers.
* **Constraints:** Must use existing session store. Cannot add new dependencies without approval.
* **Approach:** Add a provider-agnostic OAuth service with adapter pattern. Google and GitHub as first two adapters.
* **Risks:** Token refresh timing, callback URL mismatch across environments.
* **Success Criteria:** User can log in via Google or GitHub; session persists; existing login unaffected.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "The requirements are already clear enough to skip this" | Even clear requirements need documented scope, constraints, and risks before discovery runs. Undocumented assumptions become scope creep during execution. |
| "I'll define scope during execution" | Scope decisions made during execution cause unplanned file changes and regressions. Define it now so the human can approve before any skills run. |
| "This is a small change, architecture framing is overkill" | Small changes have hidden dependencies. The framing step prevents surprise regressions and keeps the human aligned on scope before work starts. |

## Red Flags

- Scope that includes implementation details (file names, function signatures)
- Success criteria that are not measurable ("works correctly", "feels faster")
- Risks that are generic ("may have bugs", "could break something")
- Approach that commits to specific implementation before skills have run

## Verification

After completing this skill:

- [ ] Problem summary is specific — not "improve the feature" or "fix the issue"
- [ ] `in_scope[]` and `out_of_scope[]` are both explicitly listed
- [ ] At least one constraint and one assumption are documented
- [ ] Approach is high-level — no implementation file names or function signatures
- [ ] Each risk is specific and actionable — not "may have edge cases"
- [ ] Success criteria are measurable
