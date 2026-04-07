# AI ORCHESTRATOR

## How to Use

Point any AI at this file using its absolute path on your machine (e.g. `/Users/you/ai-orchestrator/system/ORCHESTRATOR.md`). It is the single entry point for the entire system. Read it fully before taking any action. All other files in the system are referenced explicitly — the AI will know exactly which file to load at each step.

**Path convention:** `@ai-orchestrator/` is a self-referential prefix used internally between files — it resolves relative to wherever you installed this tool. You never type it yourself; just use the absolute path to this file when starting a session.

**Setup:** Clone this repo once to a stable location on your machine (e.g. `~/ai-orchestrator/`). Point your AI at the absolute path to this file at the start of any session. The tool works across all your projects without being copied into each one.

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

Every runtime file in this system and when to load it:

| File | Role | Load when |
| ---- | ---- | --------- |
| `@ai-orchestrator/skills/architect.md` | Frames problem, defines scope, proposes approach | FEATURE_MODE or BUGFIX_MODE — first step |
| `@ai-orchestrator/skills/review-validator.md` | Validates review consistency, removes weak findings | REVIEW_MODE — first step; BUGFIX_MODE — alternative to Architect |
| `@ai-orchestrator/system/SKILL_DISCOVERY.md` | Selects relevant skills for the problem domain | All modes — after problem framing, or as first step in PERFORMANCE_MODE |
| `@ai-orchestrator/skills-mapping.md` | Maps problem domains to skill names | Used by SKILL_DISCOVERY |
| `@ai-orchestrator/system/SKILL_EXECUTOR.md` | Executes one skill and converts output to findings | Called per skill by SKILL_DISCOVERY |
| `@ai-orchestrator/system/FINDINGS_AGGREGATOR.md` | Merges, deduplicates, and normalizes all findings | All modes — after all skills run |
| `@ai-orchestrator/skills/prioritizer.md` | Assigns P0-P3 priority and effort estimate | All modes — after Findings Aggregation |
| `@ai-orchestrator/skills/epic-generator.md` | Groups findings into independently valuable epics | All modes — after Prioritizer |
| `@ai-orchestrator/skills/ticket-splitter.md` | Splits one epic into atomic implementation tickets | All modes — after Epic Generator |
| `@ai-orchestrator/prompts/task-executor.md` | Executes one approved ticket | Ticket execution phase |
| `@ai-orchestrator/prompts/reviewer.md` | Reviews implementation for correctness and safety | After each ticket execution |
| `@ai-orchestrator/prompts/plan-reviewer.md` | Reviews plan structure before execution — checks goal alignment, ticket completeness, dependencies | All modes — after Ticket Splitter, before planning approval |
| `@ai-orchestrator/approval-template.md` | Approval checkpoint format | All 4 approval gates |
| `@ai-orchestrator/system/DELEGATION.md` | Multi-agent delegation rules (orchestrator + specialist) | When delegating work to a sub-agent |
| `@ai-orchestrator/system/WORKSPACE.md` | Per-feature workspace contract — naming, schemas, lifecycle, resume logic | Step 0 of every pipeline |

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

Approval gates use `@ai-orchestrator/approval-template.md`. Load the template at each gate.

### Step 0: Workspace (all modes)

Load `@ai-orchestrator/system/WORKSPACE.md` → create or resume `.ai-orchestrator/<slug>/`, write `context.md`

### Phase 1: Framing (mode-specific)

| Mode | Load | Notes |
| ---- | ---- | ----- |
| `REVIEW_MODE` | `@ai-orchestrator/skills/review-validator.md` | Validate and normalize the review |
| `FEATURE_MODE` | `@ai-orchestrator/skills/architect.md` | Frame problem, define scope |
| `BUGFIX_MODE` | `@ai-orchestrator/skills/architect.md` OR `@ai-orchestrator/skills/review-validator.md` | Choose based on input type |
| `PERFORMANCE_MODE` | *(skip — no framing)* | Jump directly to Phase 2 |

→ **[Checkpoint 1: after framing]** human approval (skip for PERFORMANCE_MODE)

### Phase 2: Analysis (all modes)

1. Load `@ai-orchestrator/system/SKILL_DISCOVERY.md` → select relevant skills (consults `@ai-orchestrator/skills-mapping.md`)
2. Load `@ai-orchestrator/system/SKILL_EXECUTOR.md` → run each selected skill
3. Load `@ai-orchestrator/system/FINDINGS_AGGREGATOR.md` → merge and normalize all findings

**PERFORMANCE_MODE only:** after aggregation, load `@ai-orchestrator/skills/architect.md` → propose architectural approach (used as directional context for prioritization, not as a finding).

**BUGFIX_MODE note:** if `architect` was used in Phase 1 and no skills were run, convert the architect's `Scope` and `Risks` output into a minimal Finding artifact (`severity: medium`, `evidence: architect analysis`) before passing to aggregation.

→ **[Checkpoint 2: after normalized findings]** human approval

### Phase 3: Planning (all modes)

1. Load `@ai-orchestrator/skills/prioritizer.md` → assign priority and effort
2. Load `@ai-orchestrator/skills/epic-generator.md` → group into epics (skip if work fits one independently shippable group)
3. Load `@ai-orchestrator/skills/ticket-splitter.md` → split into tickets
4. Load `@ai-orchestrator/prompts/plan-reviewer.md` → review plan structure
   - If plan-reviewer outputs REVISE: return concerns to ticket-splitter, re-split affected tickets, re-run plan-reviewer (max 1 revision cycle). If still blocked: present at Checkpoint 3 with blocker flag.

→ **[Checkpoint 3: after planning artifacts]** human approval

After approval: write full ticket artifacts to `.ai-orchestrator/<slug>/tickets.md`. Add ticket ids to the TODO section of `.ai-orchestrator/<slug>/tasks.md`.

### Phase 4: Execution (all modes)

For each approved ticket:

1. Load `@ai-orchestrator/prompts/task-executor.md` → execute the ticket
2. Load `@ai-orchestrator/prompts/reviewer.md` → review implementation
   - If reviewer outputs REVISE: log revision reason to `results.md`, re-execute with reviewer feedback as additional input, re-review (max 2 revision cycles). If still blocked after 2 cycles: escalate to Checkpoint 4 with blocker flag.
   - If reviewer outputs APPROVE: continue to next ticket.

→ **[Quality Gate: after all tickets executed and reviewed]**

Run the project's quality checks before presenting Checkpoint 4. Detect and run in order:

1. **Pre-commit hook** — if `.husky/pre-commit`, `.git/hooks/pre-commit`, or `lefthook.yml` exists, run it (e.g., `npx lefthook run pre-commit` or the hook script directly). Do not actually commit.
2. **Pre-push hook** — if `.husky/pre-push`, `.git/hooks/pre-push`, or `lefthook.yml` exists, run it. Do not actually push.
3. **Fallback** — if no hooks are detected, run these checks individually:
   - Lint: `yarn lint` / `npm run lint` / project equivalent
   - TypeScript: `yarn typecheck` / `npx tsc --noEmit`
   - Tests: `yarn test` (for affected files if possible, e.g., `yarn test --changedSince=HEAD`)

If quality checks fail:
- Fix lint/formatting issues automatically when safe (e.g., `yarn lint:fix`)
- For type errors or test failures: log the failure, attempt to fix within the ticket scope, re-run checks (max 2 fix cycles)
- If still failing after 2 cycles: present at Checkpoint 4 with blocker flag and the failing output

Only present Checkpoint 4 after quality checks pass (or are escalated as blockers).

→ **[Checkpoint 4: after execution, review, and quality gate]** before merge or final delivery
  → After approval: update `context.md` Status to `done`, ask the user whether to keep or delete `.ai-orchestrator/<slug>/`

### Lightweight Path

For single-surface changes with clear scope and no domain ambiguity, use this shortened pipeline:

0. Load `@ai-orchestrator/system/WORKSPACE.md` → create or resume workspace
1. Phase 1: Framing (as above, based on mode)
   → **[Checkpoint 1: after framing]**
2. Skip Skill Discovery and Epic Generator
3. Load `@ai-orchestrator/skills/ticket-splitter.md` → produce a single ticket
4. Load `@ai-orchestrator/prompts/plan-reviewer.md` → review plan
   → **[Checkpoint 2+3: combined findings and planning approval]**
5. Load `@ai-orchestrator/prompts/task-executor.md` → execute
6. Load `@ai-orchestrator/prompts/reviewer.md` → review
   → **[Quality Gate]** run project quality checks (see standard pipeline)
   → **[Checkpoint 4: after execution, review, and quality gate]**
   → After approval: ask user whether to keep or delete workspace

Use the lightweight path when:
* the work is clearly a single-ticket scope
* no domain ambiguity exists (no skill discovery needed)
* the change touches 1-5 files

---

## Multi-Agent Model

See `@ai-orchestrator/system/DELEGATION.md` for the full delegation contract.

Delegation rules:

* Delegate bounded work with clear inputs and expected outputs.
* Pass the relevant artifact ids and file scope.
* Do not delegate the same unresolved task to multiple agents.
* The orchestrator remains responsible for final aggregation and approval handling.

---

## Step Skip Rules

Skip steps only when the reason is explicit in the output.

* **Skip `Architect`** if the input already defines scope, constraints, and success criteria. Pass the input directly to `@ai-orchestrator/system/SKILL_DISCOVERY.md`.
* **Skip `Review Validator`** if the review has already been de-duplicated and prioritized. Pass findings directly to `@ai-orchestrator/system/FINDINGS_AGGREGATOR.md`.
* **Skip `Skill Discovery`** only for trivial, single-surface changes with no domain ambiguity. Proceed directly to `@ai-orchestrator/system/FINDINGS_AGGREGATOR.md` with any available findings.
* **Skip `Epic Generator`** when the work fits one independently shippable task group. Pass the prioritized findings directly to `@ai-orchestrator/skills/ticket-splitter.md` — treat the top finding group as an implicit single epic.
* **Skip `Ticket Splitter`** when exactly one atomic ticket remains after epic generation. Pass the epic directly to `@ai-orchestrator/prompts/plan-reviewer.md` — treat the epic goal as the single ticket goal.

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

## Parallel Execution

Parallel work is allowed only when:

* tickets do not modify the same files
* tickets have no unresolved dependency edge
* rollback and review remain isolated

Recommended concurrency: 2-4 tickets

Do not force parallelization. Serial work is valid when dependencies require it.

---

## Human Approval

Load `@ai-orchestrator/approval-template.md` at each checkpoint.

Approval is mandatory at these checkpoints:

1. After problem framing — `Architect` or `Review Validator`
2. After normalized findings — post-skill aggregation
3. After planning artifacts — epics and tickets
4. After execution and review — before merge or final delivery

Low-risk single-ticket changes may combine checkpoints 2 and 3.

---

## Review Requirements

Load `@ai-orchestrator/prompts/reviewer.md` after every ticket implementation.

Every implemented ticket must be reviewed for:

* correctness
* scope control
* regressions
* performance impact
* migration or rollout impact when relevant
* verification evidence

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
