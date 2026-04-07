## System Context

* **Part of:** `@ai-orchestrator/system/ORCHESTRATOR.md`
* **Used by:** ORCHESTRATOR — Step 0 of every pipeline
* **Uses:** nothing
* **Outputs to:** creates `.ai-orchestrator/<slug>/` workspace in the project root with 5 files

---

# WORKSPACE

## Purpose

Manages per-feature temporary workspaces that enable multi-session and multi-agent cooperation. Every pipeline creates or resumes a workspace and deletes it after the final approval checkpoint.

---

## Location

Lives in the **project being worked on** — not in the `ai-orchestrator/` system folder:

```
<project-root>/
└── .ai-orchestrator/
    └── <feature-slug>/
        ├── context.md    ← goal, scope, status — written once, read by all agents
        ├── findings.md   ← findings and epics — written during planning
        ├── tickets.md    ← approved ticket artifacts — written after Checkpoint 3
        ├── tasks.md      ← TODO / IN PROGRESS / DONE board
        └── results.md    ← execution summaries and review output (created on first ticket completion)
```

---

## Naming

Derive a deterministic slug from the feature description using this exact algorithm:
1. Lowercase the description
2. Strip all characters except letters, numbers, and spaces
3. Replace spaces with `-`
4. Truncate to 40 characters at a word boundary
5. If a workspace with the same slug already exists for a different feature, append `-2`, `-3`, etc.

Examples:
- "Add OAuth login" → `add-oauth-login`
- "Fix payment timeout bug" → `fix-payment-timeout-bug`
- "Refactor auth middleware" → `refactor-auth-middleware`
- "Fix bug #123 in the API layer!" → `fix-bug-123-in-the-api-layer`

---

## Lifecycle

### Step 0: Create or Resume

At the start of every pipeline:

0. Identify the project root — the directory containing the codebase being worked on. Use `git rev-parse --show-toplevel` if inside a git repo, otherwise use the current working directory. This is provided by the orchestrator.
1. Derive slug from the feature description
2. Check if `.ai-orchestrator/<slug>/` exists in the project root
3. **If exists** and `context.md` Status is `planning` or `executing` → resume — read `context.md` and continue from current status. Do not overwrite any existing files.
   - If Status is `reviewing` → resume at Checkpoint 4 review step
   - If Status is `done` → the workspace should have been deleted; treat as not exists and create fresh
4. **If not exists** → create `.ai-orchestrator/<slug>/`, write `context.md` with Status `planning`, create empty `findings.md`, `tickets.md`, `tasks.md`. Note: `results.md` is not created at init — it is created by `@ai-orchestrator/prompts/task-executor.md` when the first ticket completes.

### Recovery

If `.ai-orchestrator/<slug>/` exists but `context.md` is missing or has no valid Status field:
1. Log a warning: "workspace found but context.md is missing or invalid"
2. Treat as not exists — re-create context.md with Status `planning`
3. Do not overwrite `findings.md`, `tickets.md`, or `tasks.md` if they exist

### .gitignore

When creating the first workspace in a project, check for `.ai-orchestrator/` in `.gitignore`. If missing, append `.ai-orchestrator/` to `.gitignore` automatically.

### Status Transitions

Update `context.md` Status field at each transition:

| Status | When |
|--------|------|
| `planning` | From creation through Checkpoint 3 |
| `executing` | After Checkpoint 3, during ticket execution |
| `reviewing` | During Checkpoint 4 review |
| `done` | After Checkpoint 4 approval |

### Completion

After Checkpoint 4 is approved:
1. Update `context.md` Status to `done`
2. Ask the user: "Would you like to keep the workspace `.ai-orchestrator/<slug>/` for reference, or delete it?"
   - **Keep**: leave all files in place. The workspace serves as documentation of what was done (findings, tickets, results).
   - **Delete**: remove `.ai-orchestrator/<slug>/` and all its contents. If `.ai-orchestrator/` directory is now empty, leave it in place (the `.gitignore` entry remains useful).
3. Do not delete automatically — the user decides.

---

## File Schemas

### context.md

Written once at creation by the orchestrator. Updated at each status transition. Read by all agents at the start of every session to orient themselves.

```
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

Only the orchestrator (`@ai-orchestrator/system/ORCHESTRATOR.md`) updates `context.md`. Other agents (task-executor, reviewer) read it but do not write to it.

### findings.md

Written by `@ai-orchestrator/system/FINDINGS_AGGREGATOR.md` (findings section) and `@ai-orchestrator/skills/epic-generator.md` (epics section). Read by `@ai-orchestrator/skills/prioritizer.md` and `@ai-orchestrator/skills/ticket-splitter.md`.

```
# Findings

## Epics
[epic artifacts — id, title, goal, scope, impact, priority, dependencies[], success_criteria[], source_finding_ids[]]

## Normalized Findings
[finding artifacts — id, title, summary, category, severity, evidence, impact, recommended_action, dependencies[]]

## Passthrough
[missing_areas[], priority_hints[] forwarded from upstream]
```

### tickets.md

Written by the orchestrator after Checkpoint 3 approval (from ticket-splitter output). Read by `@ai-orchestrator/prompts/task-executor.md`.

```
# Tickets

## T-001: <title>

- **id:** T-001
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

Ticket IDs use `T-NNN` format (T-001, T-002, etc.) to match the `tasks.md` board.

### tasks.md

Written by the orchestrator after Checkpoint 3. Updated by `@ai-orchestrator/prompts/task-executor.md` as tickets move through states. Intentionally lightweight — full artifact is in `tickets.md`.

```
# Tasks

## TODO
- T-001 — Add OAuth provider config

## IN PROGRESS
- T-002 — Implement callback handler

## DONE
- T-003 — Update environment variables
```

### results.md

Appended by `@ai-orchestrator/prompts/task-executor.md` (execution summary) and `@ai-orchestrator/prompts/reviewer.md` (review output) after each ticket completes. Read by any agent resuming a session to understand what has already been executed and reviewed.

```
# Results

## T-001

### Execution
- **status:** done | failed
- **files_changed:** [list of paths]
- **verification:** [checks run and results]
- **remaining_risks:** [list]

### Review
- **decision:** APPROVE | REVISE
- **blockers:** [list or none]
- **issues:** [list or none]
```

---

## Multi-Agent Cooperation

When delegating to Codex via `@ai-orchestrator/system/DELEGATION.md`, always pass `workspace_path` so Codex can read context and write results without needing the full conversation history.

```
Claude Code (session 1)
  → creates .ai-orchestrator/auth-feature/
  → runs planning pipeline
  → writes findings.md, tickets.md, tasks.md
  → delegates T-001 to Codex

Codex (session 2)
  → reads .ai-orchestrator/auth-feature/context.md    (orient)
  → reads .ai-orchestrator/auth-feature/tickets.md    (full spec)
  → reads .ai-orchestrator/auth-feature/tasks.md      (check state)
  → executes T-001
  → moves T-001 from TODO → DONE in tasks.md
  → appends execution summary to results.md

Claude Code (session 3)
  → reads .ai-orchestrator/auth-feature/tasks.md      (resume)
  → reviews T-001 output
  → continues with T-002
```
