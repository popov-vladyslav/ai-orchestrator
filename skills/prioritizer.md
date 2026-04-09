## System Context

* **Part of:** `@ai-orchestrator/system/ORCHESTRATOR.md`
* **Used by:** All modes — after `@ai-orchestrator/system/FINDINGS_AGGREGATOR.md`
* **Uses:** normalized findings from `@ai-orchestrator/system/FINDINGS_AGGREGATOR.md`; in PERFORMANCE_MODE also accepts the framing output from `@ai-orchestrator/skills/performance-optimization.md` as directional context (proposed bottleneck and approach used to align priorities — not treated as a finding)
* **Outputs to:** `@ai-orchestrator/skills/epic-generator.md`

---

## Overview

Assign priority and effort to each finding so that planning and execution focus on what matters most, in the right order.

## When to Use

- Always — runs after findings aggregation in every mode

**When NOT to use:**
- Never skipped. If the findings list is empty, surface that as a blocker before proceeding.

## Process

1. Assign priority (`P0`–`P3`) to each finding based on severity and business impact.
2. Estimate effort (`XS`–`L`).
3. Define dependency edges between findings.
4. Suggest execution order optimizing for early value and risk reduction.

**Priority scale:**
* `P0` — Critical: data loss, security breach, broken core flow. Fix before anything else.
* `P1` — High: significant user impact, degrades key flows. Fix in current cycle.
* `P2` — Medium: noticeable but not blocking. Fix in next cycle.
* `P3` — Low: polish, nice-to-have. Fix when capacity allows.

**Effort scale:**
* `XS` — < 1 hour
* `S` — half day
* `M` — 1 day
* `L` — 2–3 days (consider splitting)

### Output fields

For each item:
* `id`
* `title`
* `priority`
* `effort`
* `dependencies[]`
* `execution_order`
* `why_now`

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "Everything is critical, I can't rank them" | If everything is P0, nothing is. Force-rank by user impact and blast radius. The human can override at Checkpoint 3. |
| "I'll let the human decide priorities" | Arriving at planning without recommendations means rework. Provide defensible priorities even if overridden. |
| "This is a small fix so it's low priority" | Effort and priority are independent. A 30-minute fix can be P0 if it's a security issue. |

## Red Flags

- All items assigned the same priority
- Effort estimates missing or uniformly `M`
- `execution_order` that doesn't reflect `dependencies[]`
- `why_now` that is generic ("important for the product") rather than specific to the finding

## Verification

After completing this skill:

- [ ] At least one P0 exists if any critical or high severity findings are in the input
- [ ] No more than 30% of findings are at P0 or P1
- [ ] All dependency edges are reflected in `execution_order`
- [ ] Each `why_now` is specific to its finding
