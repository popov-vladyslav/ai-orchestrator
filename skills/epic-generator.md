## System Context

* **Part of:** `@ai-orchestrator/system/ORCHESTRATOR.md`
* **Used by:** All modes — after `@ai-orchestrator/skills/prioritizer.md` (skippable if work fits one group)
* **Uses:** prioritized findings from `@ai-orchestrator/skills/prioritizer.md`
* **Outputs to:** `@ai-orchestrator/skills/ticket-splitter.md`; writes epic artifacts to `.ai-orchestrator/<slug>/findings.md`

---

## Overview

Group related findings into independently valuable epics so that planning produces shippable increments, not a single all-or-nothing delivery.

## When to Use

- Always — runs after prioritization in every mode
- Skip only when all findings fit one independently shippable group (treat as an implicit single epic and pass directly to `@ai-orchestrator/skills/ticket-splitter.md`)

## Process

1. Review prioritized findings and identify natural groupings by domain, risk, or user impact.
2. Group related findings into epics — each epic must be independently valuable.
3. Assign priority and dependencies across epics.
4. Define success criteria per epic.

### Output fields

Output one `Epic` per group (canonical schema: `@ai-orchestrator/system/ORCHESTRATOR.md`):
* `id`
* `title`
* `goal`
* `scope`
* `impact`
* `priority`
* `dependencies[]`
* `success_criteria[]`
* `source_finding_ids[]`

### Rules

* Generate at most 5–8 epics.
* Each epic must be independently valuable — it can ship without all other epics being complete.
* Do not mix unrelated findings into the same epic.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "I'll put everything in one epic to keep it simple" | One epic means no shippable increment until all work is done. Group by independent value so partial delivery is possible. |
| "These two epics overlap slightly, I'll merge them" | Overlap in scope is acceptable if the goals are independent. Merging delays value delivery unnecessarily. |
| "This is too small to be its own epic" | An independently valuable and deployable unit of work is always worth its own epic, regardless of size. |

## Red Flags

- Fewer than 2 epics when 8+ findings exist
- An epic that depends on all other epics (not independently shippable)
- `success_criteria[]` that are not measurable ("works better", "feels right")
- `source_finding_ids[]` that reference findings not present in the input

## Verification

After completing this skill:

- [ ] Each epic can ship independently — it does not require all other epics to be complete
- [ ] No epic mixes findings from unrelated domains without explicit justification
- [ ] Total epic count is between 2 and 8 (unless input findings justify fewer)
- [ ] Every `source_finding_id` references an actual finding from the aggregator output
- [ ] `success_criteria[]` are measurable for every epic
