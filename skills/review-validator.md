## System Context

* **Part of:** `@ai-orchestrator/system/ORCHESTRATOR.md`
* **Used by:** REVIEW_MODE (Phase 1), BUGFIX_MODE (Phase 1 — alternative to Architect)
* **Uses:** nothing
* **Outputs to:** `@ai-orchestrator/system/FINDINGS_AGGREGATOR.md`

---

## Overview

Validate an incoming review for consistency, accuracy, and signal quality before it enters the pipeline. Removes noise so downstream planning is based on defensible findings only.

## When to Use

- Input is an existing code review, audit output, or list of findings
- REVIEW_MODE — always the first step
- BUGFIX_MODE — when input is a review rather than a raw bug report

**When NOT to use:**
- Input is a feature request or bug report without an existing review — use `@ai-orchestrator/skills/architect.md` instead
- The review has already been deduplicated and prioritized — pass findings directly to `@ai-orchestrator/system/FINDINGS_AGGREGATOR.md`

## Process

1. Validate consistency — check that findings don't contradict each other.
2. Validate technical accuracy — reject findings that are factually incorrect.
3. Remove weak or duplicate findings — keep only evidence-backed items.
4. Identify missing areas — note gaps the review didn't cover.
5. Normalize valid findings into the canonical Finding schema.

### Output fields

**Valid Findings** — normalized to the canonical `Finding` schema (`@ai-orchestrator/system/ORCHESTRATOR.md`):
* `title`
* `summary`
* `category`
* `severity`
* `evidence`
* `impact`
* `recommended_action`
* `dependencies[]`

**Weak or Rejected Points** — for each rejected item:
* `statement`
* `reason`

**Missing Areas** — for each gap:
* `area`
* `why_missing`

**Final Priorities** — advisory hints only; definitive priority is set by `@ai-orchestrator/skills/prioritizer.md`:
* `finding_title`
* `priority_hint`
* `why`

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "All the findings look valid to me" | Reviews commonly contain duplicates, contradictions, and unsupported claims. Apply critical judgment — not acceptance. |
| "I'll keep borderline findings to be safe" | Borderline findings dilute priority and waste ticket budget on weak signals. Reject and document the reason. |
| "I don't have enough context to reject a finding" | Lack of supporting evidence IS a rejection reason. Unsupported findings don't become more valid downstream. |

## Red Flags

- Valid findings list contains items with no `evidence` field
- Rejected items list is empty after reviewing a substantive code review
- Missing areas section is empty — real reviews always have gaps
- Priority hints that don't reference a specific finding or justify the "why"

## Verification

After completing this skill:

- [ ] Every valid finding has an `evidence` field — not "seems problematic"
- [ ] Every rejected item has a documented `reason`
- [ ] `missing_areas[]` is populated even if the review was thorough
- [ ] Priority hints reference specific findings by title
