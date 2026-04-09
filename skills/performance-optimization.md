## System Context

* **Part of:** `@ai-orchestrator/system/ORCHESTRATOR.md`
* **Used by:** PERFORMANCE_MODE — Phase 1 (frames the performance problem before skill discovery runs)
* **Uses:** performance problem description, available profiling data, metrics, or user-reported symptoms
* **Outputs to:** `@ai-orchestrator/system/SKILL_DISCOVERY.md`; output serves as framing context for Checkpoint 1

---

## Overview

Frame a performance problem by establishing a measurable baseline and identifying the most likely bottleneck before any analysis skills run. Optimization without measurement is guessing.

## When to Use

- Input describes measurable slowness: latency, load time, re-renders, CPU or memory spikes
- Performance degradation is reported after a recent change
- Profiling data, traces, or benchmarks are available as input
- A performance SLA or budget exists that is not being met

**When NOT to use:**
- The problem is a bug or incorrect behavior — use BUGFIX_MODE
- Performance is a secondary concern in a feature request — use FEATURE_MODE and note the constraint in the architect framing

## Process

### Step 1: Establish the baseline

Document what is currently known before proposing anything:

* `symptom` — what the user observes ("dashboard takes 5 seconds to load")
* `metric` — the measured value if available ("LCP: 4.8s on 4G", "p95 API latency: 2.3s")
* `target` — the acceptable value ("LCP ≤ 2.5s", "p95 ≤ 200ms")
* `gap` — the difference between metric and target
* `profiling_data` — any traces, flame graphs, or query logs provided as input

If no metric is available, document what measurement is needed before optimization can begin. Include it as a prerequisite finding so it appears in Checkpoint 1.

### Step 2: Identify the likely bottleneck

Use the symptom to narrow the investigation:

```
What is slow?
├── First page load → check bundle size, TTFB, render-blocking resources
├── Interaction feels sluggish → check main thread, re-renders, long tasks (>50ms)
├── Page after navigation → check API response times, N+1 fetches, waterfall requests
└── Backend / API → check query plans, indexes, connection pool, caching
```

Output:
* `likely_bottleneck` — one primary hypothesis ("N+1 queries in the task list endpoint")
* `bottleneck_evidence` — what supports this hypothesis ("query log shows 47 queries for 10 tasks")
* `investigation_needed` — what must be confirmed before fixing ("verify with EXPLAIN ANALYZE on the task query")

### Step 3: Propose the optimization approach

* `approach` — high-level fix aligned with the bottleneck ("batch task owner queries using a single JOIN")
* `alternatives_considered[]` — other approaches and why they were deprioritized
* `risks[]` — what could go wrong ("JOIN may be slower than N+1 on small datasets — verify with profiling")
* `success_criteria[]` — measurable outcomes ("p95 API latency ≤ 200ms", "LCP ≤ 2.5s on simulated 4G")

### Step 4: Identify skills needed for Phase 2

List the domain skills to request from `@ai-orchestrator/system/SKILL_DISCOVERY.md`. Common combinations:

* Frontend slowness → `frontend-performance`, `re-renders`, `web-performance`
* Backend latency → `database-performance`, `indexing`, `caching`
* React Native → `react-native`, `frontend-performance`
* Bundle size → `web-performance`, `frontend`

Output:
* `skills_requested[]` — list of skill names to pass to `@ai-orchestrator/system/SKILL_DISCOVERY.md`

### Example

**Input:** "The task list page takes 4.8s to load on mobile. API logs show the /tasks endpoint averages 1.2s."

**Output:**
* **Baseline:** symptom: "task list page slow on mobile", metric: "LCP 4.8s, API p95 1.2s", target: "LCP ≤ 2.5s, API p95 ≤ 200ms", gap: "LCP +2.3s, API +1.0s"
* **Bottleneck:** likely_bottleneck: "N+1 queries in /tasks endpoint", bottleneck_evidence: "1.2s API time with 10 tasks suggests per-task queries", investigation_needed: "Run EXPLAIN ANALYZE on task list query"
* **Approach:** "Batch owner lookups with a single JOIN. Verify with profiling before and after."
* **Success Criteria:** ["API p95 ≤ 200ms", "LCP ≤ 2.5s on simulated 4G"]
* **Skills Requested:** ["database-performance", "indexing"]

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "The bottleneck is obvious, I don't need to measure" | Without a baseline, there is no way to confirm the fix worked or quantify the improvement. Measure first. |
| "We'll optimize after launch" | Performance debt compounds. N+1 queries and missing indexes are cheaper to fix before data scales. |
| "It's fast enough on my machine" | Developer machines are not representative. Profile on realistic hardware and network conditions. |
| "The framework handles performance" | Frameworks prevent some issues but cannot fix N+1 queries, oversized bundles, or missing indexes. |
| "100ms doesn't matter to users" | Research consistently shows 100ms delays affect conversion and perceived quality. |

## Red Flags

- No baseline metric — optimization is proposed without a current measurement
- Bottleneck is assumed, not supported by evidence (profiling data, logs, traces)
- Success criteria are vague ("make it faster", "improve performance")
- Approach addresses multiple bottlenecks at once — fix one, measure, then the next
- No investigation step before fixing (proposing a solution without first confirming the diagnosis)

## Verification

After completing this skill:

- [ ] Baseline metric is documented with a specific number
- [ ] Target is documented with a specific number
- [ ] Likely bottleneck is stated with supporting evidence (not assumed)
- [ ] At least one investigation step is defined before fixing
- [ ] Success criteria are measurable — not "faster" or "better"
- [ ] Approach is high-level — no implementation details before skill discovery runs
