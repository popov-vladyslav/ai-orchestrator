# AI ORCHESTRATOR

## How to Use

Point any AI at this file using `@ai/system/ORCHESTRATOR.md`. It is the single entry point for the entire system. Read it fully before taking any action. All other files in the `ai/` directory are referenced explicitly — the AI will know exactly which file to load at each step.

---

## Purpose

Universal AI execution workflow with:

* automatic mode selection
* skill-based routing
* structured artifacts between phases
* human approval at defined checkpoints
* parallel execution when safe
* support for multi-agent delegation

Works with: Codex, Claude, GPT, etc.

---

## System Map

Every runtime file in this system and when to load it (see `docs/` for non-runtime specs and usage examples):

| File | Role | Load when |
| ---- | ---- | --------- |
| `@ai/skills/architect.md` | Frames problem, defines scope, proposes approach | FEATURE_MODE or BUGFIX_MODE — first step |
| `@ai/skills/review-validator.md` | Validates review consistency, removes weak findings | REVIEW_MODE — first step; BUGFIX_MODE — alternative to Architect |
| `@ai/system/SKILL_DISCOVERY.md` | Selects relevant skills for the problem domain | All modes — after problem framing, or as first step in PERFORMANCE_MODE |
| `@ai/skills-mapping.md` | Maps problem domains to skill names | Used by SKILL_DISCOVERY |
| `@ai/system/SKILL_EXECUTOR.md` | Executes one skill and converts output to findings | Called per skill by SKILL_DISCOVERY |
| `@ai/system/FINDINGS_AGGREGATOR.md` | Merges, deduplicates, and normalizes all findings | All modes — after all skills run |
| `@ai/skills/prioritizer.md` | Assigns P0-P3 priority and effort estimate | All modes — after Findings Aggregation |
| `@ai/skills/epic-generator.md` | Groups findings into independently valuable epics | All modes — after Prioritizer |
| `@ai/skills/ticket-splitter.md` | Splits one epic into atomic implementation tickets | All modes — after Epic Generator |
| `@ai/prompts/task-executor.md` | Executes one approved ticket | Ticket execution phase |
| `@ai/prompts/reviewer.md` | Reviews implementation for correctness and safety | After each ticket execution |
| `@ai/prompts/plan-reviewer.md` | Reviews plan structure before execution — checks goal alignment, ticket completeness, dependencies | All modes — after Ticket Splitter, before planning approval |
| `@ai/approval-template.md` | Approval checkpoint format | All 4 approval gates |
| `@ai/tasks/tasks.md` | Pointer to workspace — active boards live in `.ai/<slug>/tasks.md` in your project | Reference only |
| `@ai/system/CLAUDE_CODE_INTEGRATION.md` | Multi-agent delegation rules for Claude Code + Codex | When delegating work to a sub-agent |
| `@ai/system/WORKSPACE.md` | Per-feature workspace contract — naming, schemas, lifecycle, resume logic | Step 0 of every pipeline |

---

## Execution Modes

Select mode from the input:

| Input             | Mode             |
| ----------------- | ---------------- |
| review output     | `REVIEW_MODE`    |
| feature request   | `FEATURE_MODE`   |
| bug report        | `BUGFIX_MODE`    |
| performance issue | `PERFORMANCE_MODE` |

If unclear, default to the smallest safe mode and state the assumption.

Mode does not need to be specified explicitly if the request and context are clear.

Use explicit mode only when:

* the request could fit multiple modes
* you want predictable routing
* you are resuming work from a specific phase
* you want to override a likely but incorrect auto-detection

If the user only references this orchestrator and asks for an outcome, infer the mode and continue.

---

## Optional Context Inputs

The workflow works better when project context is split into stable reference docs.

Useful optional inputs:

* `system audit` — current architecture, stack, boundaries, constraints
* `product requirements` — target behavior and success criteria
* `delta analysis` — repo versus target gaps
* `task plan` — epics, tickets, sequencing
* `ui spec` — design tokens, screens, interactions
* `test plan` — unit, integration, UI, and verification scope

These are context artifacts, not workflow steps. Use them as inputs when available, but do not require them for every task.

---

## Pipeline

Approval gates use `@ai/approval-template.md`. Four gates are mandatory for standard work; low-risk single-ticket changes may combine checkpoints 2 and 3. Load the template at each gate.

### REVIEW_MODE

0. Load `@ai/system/WORKSPACE.md` → create or resume `.ai/<slug>/`, write `context.md`
1. Load `@ai/skills/review-validator.md` → validate and normalize the review
2. Load `@ai/approval-template.md` → **[Checkpoint 1: after framing]** human approval
3. Load `@ai/system/SKILL_DISCOVERY.md` → select relevant skills (consults `@ai/skills-mapping.md`)
4. Load `@ai/system/SKILL_EXECUTOR.md` → run each selected skill
5. Load `@ai/system/FINDINGS_AGGREGATOR.md` → merge and normalize all findings
6. Load `@ai/approval-template.md` → **[Checkpoint 2: after normalized findings]** human approval
7. Load `@ai/skills/prioritizer.md` → assign priority and effort
8. Load `@ai/skills/epic-generator.md` → group into epics
9. Load `@ai/skills/ticket-splitter.md` → split into tickets
10. Load `@ai/prompts/plan-reviewer.md` → review plan structure
11. Load `@ai/approval-template.md` → **[Checkpoint 3: after planning artifacts]** human approval
12. Load `@ai/prompts/task-executor.md` → execute each approved ticket
13. Load `@ai/prompts/reviewer.md` → review implementation
14. Load `@ai/approval-template.md` → **[Checkpoint 4: after execution and review]** before merge or final delivery
    → After approval: update `context.md` Status to `done`, delete `.ai/<slug>/`

### FEATURE_MODE

0. Load `@ai/system/WORKSPACE.md` → create or resume `.ai/<slug>/`, write `context.md`
1. Load `@ai/skills/architect.md` → frame problem, define scope
2. Load `@ai/approval-template.md` → **[Checkpoint 1: after framing]** human approval
3. Load `@ai/system/SKILL_DISCOVERY.md` → select relevant skills (consults `@ai/skills-mapping.md`)
4. Load `@ai/system/SKILL_EXECUTOR.md` → run each selected skill
5. Load `@ai/system/FINDINGS_AGGREGATOR.md` → merge and normalize all findings
6. Load `@ai/approval-template.md` → **[Checkpoint 2: after normalized findings]** human approval
7. Load `@ai/skills/prioritizer.md` → assign priority and effort
8. Load `@ai/skills/epic-generator.md` → group into epics
9. Load `@ai/skills/ticket-splitter.md` → split into tickets
10. Load `@ai/prompts/plan-reviewer.md` → review plan structure
11. Load `@ai/approval-template.md` → **[Checkpoint 3: after planning artifacts]** human approval
12. Load `@ai/prompts/task-executor.md` → execute each approved ticket
13. Load `@ai/prompts/reviewer.md` → review implementation
14. Load `@ai/approval-template.md` → **[Checkpoint 4: after execution and review]** before merge or final delivery
    → After approval: update `context.md` Status to `done`, delete `.ai/<slug>/`

### BUGFIX_MODE

0. Load `@ai/system/WORKSPACE.md` → create or resume `.ai/<slug>/`, write `context.md`
1. Load `@ai/skills/architect.md` OR `@ai/skills/review-validator.md` → frame or validate
2. Load `@ai/approval-template.md` → **[Checkpoint 1: after framing]** human approval
3. If non-trivial: load `@ai/system/SKILL_DISCOVERY.md` → select relevant skills
4. If non-trivial: load `@ai/system/SKILL_EXECUTOR.md` → run each selected skill
5. Load `@ai/system/FINDINGS_AGGREGATOR.md` → merge and normalize all findings
   Note: if `architect` was used (not `review-validator`) and no skills were run, first convert the architect's `Scope` and `Risks` output into a minimal Finding artifact (`severity: medium`, `evidence: architect analysis`) before passing to aggregation.
6. Load `@ai/approval-template.md` → **[Checkpoint 2: after normalized findings]** human approval
7. Load `@ai/skills/prioritizer.md` → assign priority and effort
8. Load `@ai/skills/epic-generator.md` → group into epics (skip if work fits one independently shippable group)
9. Load `@ai/skills/ticket-splitter.md` → split into tickets
10. Load `@ai/prompts/plan-reviewer.md` → review plan structure
11. Load `@ai/approval-template.md` → **[Checkpoint 3: after planning artifacts]** human approval
12. Load `@ai/prompts/task-executor.md` → execute each approved ticket
13. Load `@ai/prompts/reviewer.md` → review implementation
14. Load `@ai/approval-template.md` → **[Checkpoint 4: after execution and review]** before merge or final delivery
    → After approval: update `context.md` Status to `done`, delete `.ai/<slug>/`

### PERFORMANCE_MODE

> Note: PERFORMANCE_MODE starts with skill discovery — there is no framing step, so Checkpoint 1 is not applicable. Checkpoints 2, 3, and 4 are mandatory.

0. Load `@ai/system/WORKSPACE.md` → create or resume `.ai/<slug>/`, write `context.md`
1. Load `@ai/system/SKILL_DISCOVERY.md` → select relevant skills (consults `@ai/skills-mapping.md`)
2. Load `@ai/system/SKILL_EXECUTOR.md` → run each selected skill
3. Load `@ai/system/FINDINGS_AGGREGATOR.md` → merge and normalize findings
4. Load `@ai/approval-template.md` → **[Checkpoint 2: after normalized findings]** human approval
5. Load `@ai/skills/architect.md` → propose architectural approach
6. Load `@ai/skills/prioritizer.md` → assign priority and effort
7. Load `@ai/skills/epic-generator.md` → group into epics (skip if work fits one independently shippable group)
8. Load `@ai/skills/ticket-splitter.md` → split into tickets
9. Load `@ai/prompts/plan-reviewer.md` → review plan structure
10. Load `@ai/approval-template.md` → **[Checkpoint 3: after planning artifacts]** human approval
11. Load `@ai/prompts/task-executor.md` → execute each approved ticket
12. Load `@ai/prompts/reviewer.md` → review implementation
13. Load `@ai/approval-template.md` → **[Checkpoint 4: after execution and review]** before merge or final delivery
    → After approval: update `context.md` Status to `done`, delete `.ai/<slug>/`

---

## Multi-Agent Model

See `@ai/system/CLAUDE_CODE_INTEGRATION.md` for the full delegation contract.

Recommended setup:

* `Claude Code` acts as the primary orchestrator.
* `Codex` acts as a delegated execution or analysis agent.

Typical responsibility split:

* `Claude Code` — interpret user intent, choose mode, run orchestration steps, decide where to delegate, collect outputs and present the final result
* `Codex` — perform focused code review, inspect specific files or flows, execute one ticket, validate implementation risks

Delegation rules:

* Delegate bounded work with clear inputs and expected outputs.
* Pass the relevant artifact ids and file scope.
* Do not delegate the same unresolved task to multiple agents.
* The orchestrator remains responsible for final aggregation and approval handling.

---

## Step Skip Rules

Skip steps only when the reason is explicit in the output.

* Skip `Architect` if the input already defines scope, constraints, and success criteria.
* Skip `Review Validator` if the review has already been de-duplicated and prioritized.
* Skip `Skill Discovery` only for trivial, single-surface changes with no domain ambiguity.
* Skip `Epic Generator` when the work fits one independently shippable task group.
* Skip `Ticket Splitter` when exactly one atomic ticket remains.

---

## Skill Discovery

Load `@ai/system/SKILL_DISCOVERY.md` to run this step.

It will:

* identify the relevant domain
* consult `@ai/skills-mapping.md` for auto-selection rules
* run `npx skills find <query>` for external skills (query strings defined in `@ai/skills-mapping.md`)
* select the minimum useful skill set
* explain why each selected skill is needed

Skill Discovery is mandatory for non-trivial work.

---

## Skill Triggers

Trigger skills automatically when relevant. See `@ai/skills-mapping.md` for full domain → skill rules.

| Condition           | Skill Category               |
| ------------------- | ---------------------------- |
| React Native screen | frontend-performance         |
| Large component     | component-splitting          |
| API routes          | backend-validation           |
| DB schema           | database-performance         |
| Re-fetch patterns   | caching or state-optimization |

---

## Artifact Contract

Every phase must emit structured artifacts using stable identifiers.

### Finding

* `id`
* `title`
* `summary`
* `category`
* `severity`
* `evidence`
* `impact`
* `recommended_action`
* `dependencies[]`

### Epic

* `id`
* `title`
* `goal`
* `scope`
* `impact`
* `priority`
* `dependencies[]`
* `success_criteria[]`
* `source_finding_ids[]`

### Ticket

* `id`
* `epic_id`
* `title`
* `goal`
* `files[]`
* `changes[]`
* `acceptance_criteria[]`
* `risks[]`
* `dependencies[]`
* `parallelizable`
* `source_finding_ids[]`

---

## Findings Aggregation

Load `@ai/system/FINDINGS_AGGREGATOR.md` to run this step.

After running one or more skills:

* merge duplicate findings
* remove weak or unsupported points
* normalize into the `Finding` schema
* assign stable identifiers before prioritization

No downstream phase should consume raw skill prose directly.

---

## Parallel Execution

Parallel work is allowed only when:

* tickets do not modify the same files
* tickets have no unresolved dependency edge
* rollback and review remain isolated

Recommended concurrency: 2-4 tickets

Do not force parallelization. Serial work is valid when dependencies require it.

---

## Human Approval

Load `@ai/approval-template.md` at each checkpoint.

Approval is mandatory at these checkpoints:

1. After problem framing — `Architect` or `Review Validator`
2. After normalized findings — post-skill aggregation
3. After planning artifacts — epics and tickets
4. After execution and review — before merge or final delivery

Low-risk single-ticket changes may combine checkpoints 2 and 3.

---

## Review Requirements

Load `@ai/prompts/reviewer.md` after every ticket implementation.

Every implemented ticket must be reviewed for:

* correctness
* scope control
* regressions
* performance impact
* migration or rollout impact when relevant
* verification evidence

---

## Task Execution

Load `@ai/prompts/task-executor.md` to execute an approved ticket.
Track progress in `.ai/<slug>/tasks.md` in your project (see `@ai/system/WORKSPACE.md`).

Before execution begins, write the full approved ticket artifact (all fields: `id`, `epic_id`, `title`, `goal`, `files[]`, `changes[]`, `acceptance_criteria[]`, `risks[]`, `dependencies[]`, `parallelizable`) into `.ai/<slug>/tickets.md`. Add the ticket id and title to the TODO section of `.ai/<slug>/tasks.md`. The executor reads the full artifact from `tickets.md` and tracks state in `tasks.md`.

---

## Safety Rules

Never:

* refactor large areas without an approved plan
* modify unrelated files
* introduce breaking changes silently
* skip validation or review
* change storage or database behavior without migration strategy
* claim completion without verification evidence

---

## Decision Heuristics

Before solving:

1. Can an existing skill materially improve the result? If yes, use it.
2. Is the problem already well structured? If yes, skip `Architect`.
3. Is the work small enough for a single atomic ticket? If yes, skip epics.

---

## Principles

* small steps over big rewrites
* clarity over cleverness
* real evidence over generic advice
* performance is a first-class concern
* structured outputs over free-form summaries
