---
name: orchestrator
description: Universal AI execution workflow for code review, feature development, bug fixing, and performance optimization. Use whenever a task needs structured multi-phase analysis → planning → implementation with human approval gates. Triggers on — "review [module/code/pr]", "add feature", "implement [capability]", "fix bug", "broken [behavior]", "performance issue", "slow [thing]", "optimize [X]", "/orchestrator [task]". ALWAYS use this when the request involves both analysis AND code changes — it prevents unplanned scope creep, enforces human checkpoints, and produces verifiable output through a structured pipeline.
---

# AI Orchestrator

Universal AI workflow with automatic mode selection, skill-based domain analysis, structured artifact pipeline, human approval checkpoints, and multi-agent delegation.

---

## Mode Selection

Detect from the request. Default to `REVIEW_MODE` if unclear and state the assumption.

| Request type | Mode |
|---|---|
| Review, audit, findings | `REVIEW_MODE` |
| New feature, capability | `FEATURE_MODE` |
| Bug, broken behavior | `BUGFIX_MODE` |
| Slowness, latency, performance | `PERFORMANCE_MODE` |

---

## Step 0: Workspace

Derive a slug from the feature description:
1. Lowercase → strip non-alphanumeric except spaces → replace spaces with `-` → truncate to 40 chars at a word boundary

Check `<project-root>/.ai-orchestrator/<slug>/`:
- **Exists, status `planning` or `executing`** → resume. Read `context.md` and continue from current status.
- **Exists, status `done`** → ask whether to start fresh or reuse context.
- **Not exists** → create the directory. Write `context.md` (status: `planning`). Create empty `findings.md`, `tickets.md`, `tasks.md`.

Add `.ai-orchestrator/` to `.gitignore` if missing.

**`context.md` schema:**
```
# Workspace: <slug>
- **Mode:** REVIEW_MODE | FEATURE_MODE | BUGFIX_MODE | PERFORMANCE_MODE
- **Goal:** one sentence
- **Scope:** what is in scope
- **Out of scope:** what is explicitly excluded
- **Constraints:** constraints and assumptions
- **Created:** YYYY-MM-DD
- **Status:** planning | executing | reviewing | done
- **Active agent:** orchestrator | specialist | (none)
```

---

## Phase 1: Framing

| Mode | Skill | Action |
|---|---|---|
| `REVIEW_MODE` | @ai-orchestrator/skills/review-validator.md | Validate and normalize the incoming review. Remove weak/duplicate findings. |
| `FEATURE_MODE` | @ai-orchestrator/skills/architect.md | Define scope, constraints, approach, success criteria. |
| `BUGFIX_MODE` | @ai-orchestrator/skills/architect.md OR @ai-orchestrator/skills/review-validator.md | Use @ai-orchestrator/skills/architect.md for raw bug reports; @ai-orchestrator/skills/review-validator.md for existing review input. |
| `PERFORMANCE_MODE` | @ai-orchestrator/skills/performance-optimization.md | Establish baseline metric, identify likely bottleneck, propose measurement approach. |

**@ai-orchestrator/skills/architect.md output:** problem summary, goal, in_scope[], out_of_scope[], constraints[], assumptions[], approach, alternatives_considered[], risks[], success_criteria[]

**@ai-orchestrator/skills/review-validator.md output:** valid findings (normalized), rejected items with reasons, missing areas, advisory priority hints

**@ai-orchestrator/skills/performance-optimization.md output:** symptom, metric, target, gap, likely_bottleneck, bottleneck_evidence, investigation_needed, approach, risks[], success_criteria[], skills_requested[]

→ **[Checkpoint 1: after framing]** — present output, wait for APPROVE / MODIFY / REJECT.

---

## Phase 2: Analysis

### 2a — Skill Discovery

1. Identify problem domains: performance, architecture, state management, networking, database, security, testing, frontend, backend, mobile.
2. Run `npx skills find <query>` for each detected domain (queries defined in `@ai-orchestrator/skills-mapping.md`). If npx unavailable, fall back to inline domain analysis.
3. Select 3–5 relevant skills. Prefer specific over general.
4. Define execution plan: `run_first` (sequential) and `run_in_parallel[]` (concurrent when safe).

**In PERFORMANCE_MODE:** use `skills_requested[]` from Phase 1 as the starting selection; refine with skills-mapping rules.

### 2b — Skill Execution

Run each selected skill. For each skill:
- Execute the skill against the problem context
- Extract concrete findings with evidence
- Normalize to the canonical Finding schema
- Mark weak or uncertain findings explicitly

**Inline skill execution (when `skill_name` starts with `inline:`):** Read relevant files, analyze against domain dimensions, output findings in the canonical schema with `source: inline`.

**If a skill produces no output:** log as info-severity finding; do not block the pipeline. Surface the failure at Checkpoint 2.

### 2c — Findings Aggregation

1. Merge duplicate findings (keep strongest evidence).
2. Remove unsupported or low-signal findings.
3. Normalize wording and severity.
4. Assign stable IDs (`F-001`, `F-002`, …).

**BUGFIX_MODE with no skills run:** convert @ai-orchestrator/skills/architect.md's `Scope` + `Risks` into a minimal Finding (severity: medium, evidence: @ai-orchestrator/skills/architect.md analysis) before aggregation.

**Output validation:** every finding needs `id`, `title`, `severity`, `evidence`, `recommended_action`. Drop findings missing required fields; note them in Checkpoint 2.

**Write to:** `.ai-orchestrator/<slug>/findings.md`

→ **[Checkpoint 2: after normalized findings]** — present findings list, wait for APPROVE / MODIFY / REJECT.

---

## Phase 3: Planning

### 3a — @ai-orchestrator/skills/prioritizer.md

Assign to each finding:
- `priority`: P0 (critical/security) → P1 (significant user impact) → P2 (noticeable, non-blocking) → P3 (polish)
- `effort`: XS (<1h) → S (half day) → M (1 day) → L (2–3 days, consider splitting)
- `dependencies[]`, `execution_order`, `why_now`

No more than 30% of findings at P0/P1. If all findings rated the same priority, force-rank.

### 3b — @ai-orchestrator/skills/epic-generator.md

Group related findings into 2–8 independently shippable epics. Each epic must be valuable on its own — it can ship without other epics being complete.

**Epic schema:** `id`, `title`, `goal`, `scope`, `impact`, `priority`, `dependencies[]`, `success_criteria[]`, `source_finding_ids[]`

Skip this step if all work fits one independently shippable group (treat as an implicit single epic).

### 3c — @ai-orchestrator/skills/ticket-splitter.md

Split each epic into atomic tickets touching 1–5 files with clear acceptance criteria. Run once per epic.

**Ticket schema:** `id`, `epic_id`, `title`, `goal`, `files[]`, `changes[]`, `acceptance_criteria[]`, `risks[]`, `dependencies[]`, `parallelizable`, `source_finding_ids[]`

Mark `parallelizable: true` only when there are no file conflicts and no dependency edges.

Skip if exactly one atomic ticket remains after epic generation.

### 3d — Plan Review

Review plan structure before execution:
- Goal alignment — does the plan address the original problem?
- Scope control — any tickets outside approved scope?
- Ticket completeness — every ticket has goal, files, changes, acceptance criteria?
- Dependency correctness — ordered correctly?
- Parallelization safety — no file conflicts on parallel tickets?

If REVISE: return concerns to @ai-orchestrator/skills/ticket-splitter.md, re-split affected tickets, re-run plan review (max 1 revision cycle). If still blocked: present at Checkpoint 3 with blocker flag.

**Output validation:** every ticket needs `id`, `goal`, `files[]`, `changes[]`, `acceptance_criteria[]`. Return incomplete tickets to @ai-orchestrator/skills/ticket-splitter.md for one fix pass.

→ **[Checkpoint 3: after planning artifacts]** — present epics and tickets, wait for APPROVE / MODIFY / REJECT.

After approval: write full ticket artifacts to `tickets.md`. Add ticket IDs to `tasks.md` TODO section. Update `context.md` Status to `executing`.

---

## Phase 4: Execution

For each approved ticket in dependency order:

1. Move ticket `TODO → IN PROGRESS` in `tasks.md`.
2. Execute: implement changes scoped strictly to the ticket spec. Only modify listed files.
3. Review: check correctness, safety, scope control, code quality, performance, and verification evidence.
   - **REVISE:** log revision reason to `results.md`; re-execute with reviewer feedback (max 2 cycles). If still blocked after 2 cycles: escalate to Checkpoint 4 with blocker flag.
   - **APPROVE:** move ticket `IN PROGRESS → DONE` in `tasks.md`. `DONE` must only be set after a successful review.
4. Append execution summary and review output to `results.md` under `## <ticket_id>`.

**Quality gate** (after all tickets executed and reviewed):
1. Run pre-commit hook if present (`.husky/pre-commit`, `.git/hooks/pre-commit`, `lefthook.yml`)
2. Run pre-push hook if present
3. Fallback if no hooks: lint → typecheck (`npx tsc --noEmit`) → tests (affected files)

If quality checks fail: auto-fix lint when safe; attempt manual fixes for type/test failures (max 2 cycles). If still failing: present at Checkpoint 4 with blocker flag.

→ **[Checkpoint 4: after execution, review, and quality gate]** — present results, wait for APPROVE.

After approval: update `context.md` Status to `done`. Ask user: keep or delete `.ai-orchestrator/<slug>/`.

---

## Checkpoint Format

At each checkpoint, present using this structure:

```
## Checkpoint N — [Framing | Findings | Planning | Execution]

**Risk level:** LOW | MEDIUM | HIGH

**Summary:** [what was produced]

**Structured output:** [artifacts with IDs]

**Decision context:**
- Why now: [reason]
- Alternatives: [considered]
- Open questions: [if any]

**Risks:**
- [risk_1]

**Review checklist:**
- [ ] Matches the goal
- [ ] Scope is controlled
- [ ] Risks are understood
- [ ] Output is specific enough to execute
- [ ] Safe to proceed

**Decision:** APPROVE / MODIFY / REJECT + notes
```

---

## Artifact Schemas

### Finding
`id` · `title` · `summary` · `category` · `severity` · `evidence` · `impact` · `recommended_action` · `dependencies[]`

### Epic
`id` · `title` · `goal` · `scope` · `impact` · `priority` · `dependencies[]` · `success_criteria[]` · `source_finding_ids[]`

### Ticket
`id` · `epic_id` · `title` · `goal` · `files[]` · `changes[]` · `acceptance_criteria[]` · `risks[]` · `dependencies[]` · `parallelizable` · `source_finding_ids[]`

---

## Multi-Agent Delegation

### When to delegate to Codex

Use `codex:codex-rescue` when a ticket is implementation-heavy (effort M or L) and has clear file scope and acceptance criteria. Codex handles the implementation; the orchestrator handles checkpoints and aggregation.

**Delegation template:**
```
Execute tickets [T-001, T-002] from workspace:
  <workspace_path>/

Read context.md for orientation, tickets.md for full specs.
Update tasks.md (TODO → IN PROGRESS → DONE) and results.md as you work.
```

Pass: `workspace_path`, `ticket_ids[]`, and `orchestrator_path` (absolute path to this plugin's `system/ORCHESTRATOR.md` for reference).

The specialist:
1. Reads `context.md` → orient
2. Reads `tickets.md` → full spec
3. Reads `tasks.md` → check current state
4. Executes each ticket
5. Updates `tasks.md` and `results.md`

The orchestrator remains responsible for final aggregation and all approval checkpoints.

### Parallel dispatch

Use `superpowers:dispatching-parallel-agents` when multiple tickets:
- Do NOT modify the same files
- Have no unresolved dependency edges between them
- Can be reviewed independently

Dispatch 2–4 tickets in parallel maximum. Monitor `tasks.md` to track progress across all agents.

**Decision table:**

| Ticket effort | File conflicts | Recommendation |
|---|---|---|
| XS or S | — | Execute inline |
| M or L | No conflicts, no deps | Delegate to Codex |
| M or L | Conflicts or deps | Execute serially |
| Multiple M/L tickets, no conflicts | Independent | Parallel Codex dispatch |

---

## Lightweight Path

For single-ticket changes with no domain ambiguity and ≤5 files:

1. Step 0: Workspace
2. Phase 1: Framing → Checkpoint 1
3. Skip Skill Discovery and Epic Generator
4. Ticket Splitter → Plan Review → combined Checkpoint 2+3
5. Execute → Review → Quality Gate → Checkpoint 4

---

## Skip Rules

| Skip when | Instead |
|---|---|
| Input already defines scope, constraints, success criteria | Pass directly to Skill Discovery |
| Review is already deduplicated and prioritized | Pass directly to Findings Aggregator |
| Trivial, single-surface change, no domain ambiguity | Skip Skill Discovery |
| All findings fit one independently shippable group | Skip Epic Generator; treat as single epic |
| Exactly one atomic ticket after epic generation | Skip Ticket Splitter; use epic goal as ticket goal |

---

## Pipeline Limits

- **Max 15 findings** at Checkpoint 2. If exceeded: prioritize top by severity, present, note deferred count.
- **Max 5 epics, max 15 tickets** at Checkpoint 3. Same: top items first, defer rest.
- **Max 20 ticket executions** per session. If reached: pause, present progress summary, let user decide.

---

## Safety Rules

Never:
- Refactor large areas without an approved plan
- Modify files outside the ticket spec
- Introduce breaking changes silently
- Skip a checkpoint without explicit human approval
- Claim completion without verification evidence
- Change database or storage behavior without a migration strategy
