# Per-Feature Workspace Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a per-feature temporary workspace (`.ai/<slug>/`) that lets Claude Code and Codex cooperate across multiple sessions by reading and writing shared context, findings, tickets, and task state.

**Architecture:** A new `system/WORKSPACE.md` file defines the workspace contract. ORCHESTRATOR.md gains a Step 0 (create or resume workspace) and a cleanup step after Checkpoint 4. Seven existing files get targeted System Context updates to reference the workspace files they read or write.

**Tech Stack:** Markdown files only — no code, no dependencies.

---

## Files

| Action | Path | Responsibility |
|--------|------|----------------|
| Create | `ai/system/WORKSPACE.md` | Full workspace contract: naming, schemas, lifecycle, resume logic, deletion |
| Modify | `ai/system/ORCHESTRATOR.md` | System Map updates + Step 0 + CP4 cleanup + Task Execution section |
| Modify | `ai/system/FINDINGS_AGGREGATOR.md` | Add workspace write note to System Context |
| Modify | `ai/skills/epic-generator.md` | Add workspace write note to System Context |
| Modify | `ai/skills/ticket-splitter.md` | Update System Context Outputs to |
| Modify | `ai/prompts/task-executor.md` | Update System Context Uses |
| Modify | `ai/tasks/tasks.md` | Replace with pointer to workspace |
| Modify | `ai/system/CLAUDE_CODE_INTEGRATION.md` | Add `workspace_path` to delegation payload |

---

### Task 1: Create `system/WORKSPACE.md`

**Files:**
- Create: `ai/system/WORKSPACE.md`

- [ ] **Step 1: Verify file does not yet exist**

```bash
ls /Users/vladpopov/Documents/ai/system/WORKSPACE.md 2>&1
```
Expected: `No such file or directory`

- [ ] **Step 2: Create the file**

Write the following content to `/Users/vladpopov/Documents/ai/system/WORKSPACE.md`:

```markdown
## System Context

* **Part of:** `@ai/system/ORCHESTRATOR.md`
* **Used by:** ORCHESTRATOR — Step 0 of every pipeline
* **Uses:** nothing
* **Outputs to:** creates `.ai/<slug>/` workspace in the project root with 4 files

---

# WORKSPACE

## Purpose

Manages per-feature temporary workspaces that enable multi-session and multi-agent cooperation. Every pipeline creates or resumes a workspace and deletes it after the final approval checkpoint.

---

## Location

Lives in the **project being worked on** — not in the `ai/` system folder:

```
<project-root>/
└── .ai/
    └── <feature-slug>/
        ├── context.md    ← goal, scope, status — written once, read by all agents
        ├── findings.md   ← findings and epics — written during planning
        ├── tickets.md    ← approved ticket artifacts — written after Checkpoint 3
        └── tasks.md      ← TODO / IN PROGRESS / DONE board
```

---

## Naming

Derive slug from the feature description:
- lowercase, words joined with `-`
- max 40 characters
- examples: `add-oauth-login`, `fix-payment-timeout`, `refactor-auth-middleware`

---

## Lifecycle

### Step 0: Create or Resume

At the start of every pipeline:

1. Derive slug from the feature description
2. Check if `.ai/<slug>/` exists in the project root
3. **If exists** and `context.md` Status is `planning` or `executing` → resume — read `context.md` and continue from current status. Do not overwrite any existing files.
4. **If not exists** → create `.ai/<slug>/`, write `context.md` with Status `planning`, create empty `findings.md`, `tickets.md`, `tasks.md`

### .gitignore

When creating the first workspace in a project, check for `.ai/` in `.gitignore`. If missing, append `.ai/` to `.gitignore` automatically.

### Status Transitions

Update `context.md` Status field at each transition:

| Status | When |
|--------|------|
| `planning` | From creation through Checkpoint 3 |
| `executing` | After Checkpoint 3, during ticket execution |
| `reviewing` | During Checkpoint 4 review |
| `done` | After Checkpoint 4 approval — triggers deletion |

### Deletion

After Checkpoint 4 is approved:
1. Update `context.md` Status to `done`
2. Delete `.ai/<slug>/` and all its contents
3. If `.ai/` directory is now empty, leave it in place (the `.gitignore` entry remains useful)

---

## File Schemas

### context.md

Written once at creation by the orchestrator. Updated at each status transition. Read by all agents at the start of every session to orient themselves.

```markdown
# Workspace: <feature-slug>

- **Mode:** FEATURE_MODE | REVIEW_MODE | BUGFIX_MODE | PERFORMANCE_MODE
- **Goal:** one sentence description
- **Scope:** what is in scope
- **Out of scope:** what is explicitly excluded
- **Constraints:** known constraints or assumptions
- **Created:** YYYY-MM-DD
- **Status:** planning | executing | reviewing | done
- **Active agent:** Claude Code | Codex | (none)
```

### findings.md

Written by `@ai/system/FINDINGS_AGGREGATOR.md` (findings section) and `@ai/skills/epic-generator.md` (epics section). Read by `@ai/skills/prioritizer.md` and `@ai/skills/ticket-splitter.md`.

```markdown
# Findings

## Epics
[epic artifacts — id, title, goal, scope, impact, priority, dependencies[], success_criteria[], source_finding_ids[]]

## Normalized Findings
[finding artifacts — id, title, summary, category, severity, evidence, impact, recommended_action, dependencies[]]

## Passthrough
[missing_areas[], priority_hints[] forwarded from upstream]
```

### tickets.md

Written by the orchestrator after Checkpoint 3 approval (from ticket-splitter output). Read by `@ai/prompts/task-executor.md`.

```markdown
# Tickets

## <ticket-id>: <title>

- **epic_id:** E-001
- **goal:** one sentence
- **files:** [list]
- **changes:** [list]
- **acceptance_criteria:** [list]
- **risks:** [list]
- **dependencies:** [ticket ids]
- **parallelizable:** true | false
- **source_finding_ids:** [finding ids]
```

### tasks.md

Written by the orchestrator after Checkpoint 3. Updated by `@ai/prompts/task-executor.md` as tickets move through states. Intentionally lightweight — full artifact is in `tickets.md`.

```markdown
# Tasks

## TODO
- T-001 — Add OAuth provider config

## IN PROGRESS
- T-002 — Implement callback handler

## DONE
- T-003 — Update environment variables
```

---

## Multi-Agent Cooperation

When delegating to Codex via `@ai/system/CLAUDE_CODE_INTEGRATION.md`, always pass `workspace_path` so Codex can read context and write results without needing the full conversation history.

```
Claude Code (session 1)
  → creates .ai/auth-feature/
  → runs planning pipeline
  → writes findings.md, tickets.md, tasks.md
  → delegates T-001 to Codex

Codex (session 2)
  → reads .ai/auth-feature/context.md    (orient)
  → reads .ai/auth-feature/tickets.md    (full spec)
  → reads .ai/auth-feature/tasks.md      (check state)
  → executes T-001
  → moves T-001 from TODO → DONE in tasks.md

Claude Code (session 3)
  → reads .ai/auth-feature/tasks.md      (resume)
  → reviews T-001 output
  → continues with T-002
```
```

- [ ] **Step 3: Verify the file was created**

```bash
grep -c "Create or Resume" /Users/vladpopov/Documents/ai/system/WORKSPACE.md
```
Expected: `1`

- [ ] **Step 4: Commit**

```bash
cd /Users/vladpopov/Documents/ai && git add system/WORKSPACE.md && git commit -m "feat: add workspace contract"
```

---

### Task 2: Update `ORCHESTRATOR.md` — System Map

**Files:**
- Modify: `ai/system/ORCHESTRATOR.md`

- [ ] **Step 1: Verify WORKSPACE.md is not yet in the System Map**

```bash
grep "WORKSPACE" /Users/vladpopov/Documents/ai/system/ORCHESTRATOR.md
```
Expected: no output

- [ ] **Step 2: Add WORKSPACE.md row to System Map**

In the System Map table, after the line:
```
| `@ai/system/CLAUDE_CODE_INTEGRATION.md` | Multi-agent delegation rules for Claude Code + Codex | When delegating work to a sub-agent |
```

Add:
```
| `@ai/system/WORKSPACE.md` | Per-feature workspace contract — naming, schemas, lifecycle, resume logic | Step 0 of every pipeline |
```

- [ ] **Step 3: Update tasks.md row in System Map**

Find:
```
| `@ai/tasks/tasks.md` | Lightweight execution board for approved tickets | During ticket execution |
```

Replace with:
```
| `@ai/tasks/tasks.md` | Pointer to workspace — active boards live in `.ai/<slug>/tasks.md` in your project | Reference only |
```

- [ ] **Step 4: Verify both changes**

```bash
grep "WORKSPACE.md" /Users/vladpopov/Documents/ai/system/ORCHESTRATOR.md
grep "Pointer to workspace" /Users/vladpopov/Documents/ai/system/ORCHESTRATOR.md
```
Expected: one line each

- [ ] **Step 5: Commit**

```bash
cd /Users/vladpopov/Documents/ai && git add system/ORCHESTRATOR.md && git commit -m "feat: add workspace to system map"
```

---

### Task 3: Update `ORCHESTRATOR.md` — Pipelines (Step 0 + CP4 cleanup)

**Files:**
- Modify: `ai/system/ORCHESTRATOR.md`

- [ ] **Step 1: Add Step 0 to REVIEW_MODE pipeline**

Find the REVIEW_MODE pipeline opening:
```
### REVIEW_MODE

1. Load `@ai/skills/review-validator.md` → validate and normalize the review
```

Replace with:
```
### REVIEW_MODE

0. Load `@ai/system/WORKSPACE.md` → create or resume `.ai/<slug>/`, write `context.md`
1. Load `@ai/skills/review-validator.md` → validate and normalize the review
```

Repeat the same Step 0 addition for FEATURE_MODE, BUGFIX_MODE, and PERFORMANCE_MODE — each pipeline's first numbered step becomes step 1, preceded by step 0.

- [ ] **Step 2: Add cleanup instruction after CP4 in all 4 pipelines**

In each pipeline, find the final Checkpoint 4 line:
```
14. Load `@ai/approval-template.md` → **[Checkpoint 4: after execution and review]** before merge or final delivery
```
(PERFORMANCE_MODE uses step 13)

After each one, add:
```
    → After approval: update `context.md` Status to `done`, delete `.ai/<slug>/`
```

- [ ] **Step 3: Update Task Execution section**

Find:
```
Load `@ai/prompts/task-executor.md` to execute an approved ticket.
Track progress in `@ai/tasks/tasks.md`.
```

Replace with:
```
Load `@ai/prompts/task-executor.md` to execute an approved ticket.
Track progress in `.ai/<slug>/tasks.md` in your project (see `@ai/system/WORKSPACE.md`).
```

- [ ] **Step 4: Verify**

```bash
grep -c "WORKSPACE.md.*create or resume" /Users/vladpopov/Documents/ai/system/ORCHESTRATOR.md
```
Expected: `4` (one per pipeline)

```bash
grep -c "delete.*ai.*slug" /Users/vladpopov/Documents/ai/system/ORCHESTRATOR.md
```
Expected: `4`

- [ ] **Step 5: Commit**

```bash
cd /Users/vladpopov/Documents/ai && git add system/ORCHESTRATOR.md && git commit -m "feat: add workspace step 0 and cleanup to all pipelines"
```

---

### Task 4: Update `FINDINGS_AGGREGATOR.md` and `epic-generator.md`

**Files:**
- Modify: `ai/system/FINDINGS_AGGREGATOR.md`
- Modify: `ai/skills/epic-generator.md`

- [ ] **Step 1: Update FINDINGS_AGGREGATOR.md System Context**

Find:
```
* **Outputs to:** `@ai/skills/prioritizer.md`
```

Replace with:
```
* **Outputs to:** `@ai/skills/prioritizer.md`; writes normalized findings to `.ai/<slug>/findings.md`
```

- [ ] **Step 2: Update epic-generator.md System Context**

Find:
```
* **Outputs to:** `@ai/skills/ticket-splitter.md`
```

Replace with:
```
* **Outputs to:** `@ai/skills/ticket-splitter.md`; writes epic artifacts to `.ai/<slug>/findings.md`
```

- [ ] **Step 3: Verify**

```bash
grep "findings.md" /Users/vladpopov/Documents/ai/system/FINDINGS_AGGREGATOR.md
grep "findings.md" /Users/vladpopov/Documents/ai/skills/epic-generator.md
```
Expected: one matching line in each file

- [ ] **Step 4: Commit**

```bash
cd /Users/vladpopov/Documents/ai && git add system/FINDINGS_AGGREGATOR.md skills/epic-generator.md && git commit -m "feat: add workspace write notes to aggregator and epic-generator"
```

---

### Task 5: Update `ticket-splitter.md` and `task-executor.md`

**Files:**
- Modify: `ai/skills/ticket-splitter.md`
- Modify: `ai/prompts/task-executor.md`

- [ ] **Step 1: Update ticket-splitter.md System Context**

Find:
```
* **Outputs to:** `@ai/prompts/plan-reviewer.md` → `@ai/approval-template.md` (checkpoint 3) → `@ai/tasks/tasks.md` (after human approval)
```

Replace with:
```
* **Outputs to:** `@ai/prompts/plan-reviewer.md` → `@ai/approval-template.md` (checkpoint 3) → `.ai/<slug>/tickets.md` and `.ai/<slug>/tasks.md` (after human approval)
```

- [ ] **Step 2: Update task-executor.md System Context**

Find:
```
* **Uses:** approved tickets from `@ai/tasks/tasks.md`
```

Replace with:
```
* **Uses:** full ticket artifacts from `.ai/<slug>/tickets.md`; reads and updates `.ai/<slug>/tasks.md` for state tracking
```

- [ ] **Step 3: Verify**

```bash
grep "slug.*tickets.md" /Users/vladpopov/Documents/ai/skills/ticket-splitter.md
grep "slug.*tickets.md" /Users/vladpopov/Documents/ai/prompts/task-executor.md
```
Expected: one matching line in each file

- [ ] **Step 4: Commit**

```bash
cd /Users/vladpopov/Documents/ai && git add skills/ticket-splitter.md prompts/task-executor.md && git commit -m "feat: point ticket-splitter and task-executor to workspace files"
```

---

### Task 6: Replace `tasks/tasks.md` with pointer

**Files:**
- Modify: `ai/tasks/tasks.md`

- [ ] **Step 1: Replace file contents**

Replace the entire content of `/Users/vladpopov/Documents/ai/tasks/tasks.md` with:

```markdown
## System Context

* **Part of:** `@ai/system/ORCHESTRATOR.md`
* **Used by:** reference only — active execution boards live in the per-feature workspace
* **Uses:** nothing
* **Outputs to:** nothing

---

# Tasks

Active task boards live in `.ai/<slug>/tasks.md` inside your project — one per active feature.

See `@ai/system/WORKSPACE.md` for the full workspace contract, file schemas, and lifecycle rules.
```

- [ ] **Step 2: Verify**

```bash
grep "WORKSPACE.md" /Users/vladpopov/Documents/ai/tasks/tasks.md
```
Expected: `See \`@ai/system/WORKSPACE.md\``

- [ ] **Step 3: Commit**

```bash
cd /Users/vladpopov/Documents/ai && git add tasks/tasks.md && git commit -m "feat: replace tasks.md with workspace pointer"
```

---

### Task 7: Update `CLAUDE_CODE_INTEGRATION.md` — delegation payload

**Files:**
- Modify: `ai/system/CLAUDE_CODE_INTEGRATION.md`

- [ ] **Step 1: Add `workspace_path` to Delegation Contract**

Find the Recommended Delegation Payload section:
```
* `constraints`: do not edit code, edit only listed files, preserve behavior, and so on
```

Add after it:
```
* `workspace_path`: `.ai/<slug>/` — path where Codex reads context, tickets, and writes task state
```

- [ ] **Step 2: Update Ticket Execution example**

Find:
```
### Example: Ticket Execution

Ask `Codex` to:

* execute one ticket using `@ai/prompts/task-executor.md`
* touch only approved files
* return execution summary, verification, and residual risks
```

Replace with:
```
### Example: Ticket Execution

Ask `Codex` to:

* read context from `.ai/<slug>/context.md` to orient itself
* read the full ticket artifact from `.ai/<slug>/tickets.md`
* execute one ticket using `@ai/prompts/task-executor.md`
* touch only approved files
* update `.ai/<slug>/tasks.md` (move ticket from TODO → DONE)
* return execution summary, verification, and residual risks
```

- [ ] **Step 3: Verify**

```bash
grep "workspace_path" /Users/vladpopov/Documents/ai/system/CLAUDE_CODE_INTEGRATION.md
```
Expected: one line

- [ ] **Step 4: Commit**

```bash
cd /Users/vladpopov/Documents/ai && git add system/CLAUDE_CODE_INTEGRATION.md && git commit -m "feat: add workspace_path to delegation contract"
```

---

### Task 8: Initialize git repo and final verification

**Files:**
- No file changes — repo setup and full verification

- [ ] **Step 1: Initialize git repo if not already done**

```bash
cd /Users/vladpopov/Documents/ai && git init && git add -A && git commit -m "chore: initial commit — AI orchestration system with workspace support"
```

If repo already initialized, skip `git init` and just verify with `git status`.

- [ ] **Step 2: Verify all workspace references are consistent**

```bash
grep -r "WORKSPACE.md" /Users/vladpopov/Documents/ai/system/ /Users/vladpopov/Documents/ai/skills/ /Users/vladpopov/Documents/ai/prompts/ /Users/vladpopov/Documents/ai/tasks/
```
Expected: lines in ORCHESTRATOR.md (multiple), task-executor.md, tasks/tasks.md, CLAUDE_CODE_INTEGRATION.md

```bash
grep -r "\.ai/<slug>" /Users/vladpopov/Documents/ai/system/ /Users/vladpopov/Documents/ai/skills/ /Users/vladpopov/Documents/ai/prompts/
```
Expected: lines in WORKSPACE.md, ORCHESTRATOR.md, FINDINGS_AGGREGATOR.md, epic-generator.md, ticket-splitter.md, task-executor.md, CLAUDE_CODE_INTEGRATION.md

- [ ] **Step 3: Verify README is present**

```bash
ls /Users/vladpopov/Documents/ai/README.md
```
Expected: file exists

- [ ] **Step 4: Final commit if any files were missed**

```bash
cd /Users/vladpopov/Documents/ai && git status
```

If any untracked or modified files remain, add and commit them:
```bash
git add -A && git commit -m "chore: finalize workspace implementation"
```
