## System Context

* **Part of:** `@ai-orchestrator/system/ORCHESTRATOR.md`
* **Used by:** `@ai-orchestrator/system/SKILL_DISCOVERY.md` — called once per selected skill
* **Uses:** the externally resolved skill from `npx skills` or inline domain analysis per `@ai-orchestrator/skills-mapping.md` — not the orchestration files in `@ai-orchestrator/skills/`
* **Outputs to:** `@ai-orchestrator/system/FINDINGS_AGGREGATOR.md`

---

You are executing one selected skill.

## Input

* `skill_name`
* `problem_context`
* optional existing findings to refine

---

## Task

1. Run the skill.
2. Extract concrete insights.
3. Convert output into normalized findings.
4. Mark weak or uncertain findings explicitly.

---

## Output

### Findings

For each finding, provide:

* `title`
* `summary`
* `category`
* `severity`
* `evidence`
* `impact`
* `recommended_action`
* `dependencies[]`

### Rejected Or Weak Points

* `statement`
* `reason`

### Priority Hints

* `finding_title`
* `priority_hint`
* `why`

---

## Rules

* Do not modify code.
* Focus on analysis only.
* Prefer evidence over generic best practices.
* Output must be easy to merge into the canonical `Finding` schema.

---

## Error Handling

If skill execution produces no findings or malformed output:

1. Log: "Skill `<skill_name>` produced no usable output"
2. Record as an info-severity finding: `{ title: "<skill_name> — no output", severity: "info", evidence: "no output produced", recommended_action: "manual review of <domain>" }`
3. Continue pipeline — do not block on a single skill failure
4. Surface the failure in Checkpoint 2 summary so the human can decide whether to re-run or accept

If all skills in the execution plan fail:

1. Surface as a Checkpoint 2 blocker
2. Recommend manual analysis or scope reduction
3. Do not proceed past Checkpoint 2 without human approval
