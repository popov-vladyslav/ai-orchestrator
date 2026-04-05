# Spec: Per-Feature Workspace

**Date:** 2026-04-04
**Status:** Approved

---

## Context

The orchestration system currently uses a single `@ai-orchestrator/tasks/tasks.md` file as the execution board. This breaks down when work spans multiple sessions or multiple agents (Claude + Codex): a new session has no context on what was planned, and Codex has no shared location to read tickets from or write results to.

This spec introduces a per-feature workspace — a temporary folder in the project being worked on — that gives all agents a shared, persistent context for the duration of a feature. It is created automatically, used throughout the pipeline, and deleted after the final approval checkpoint.

---

## Design

### Location

```
<project-root>/
└── .ai-orchestrator/
    └── <feature-slug>/
        ├── context.md
        ├── findings.md
        ├── tickets.md
        └── tasks.md
```

`.ai-orchestrator/` lives in the project root, not in the `ai-orchestrator/` system folder. It is gitignored by default.

### Naming

The orchestrator derives a slug from the feature description:
- lowercase, words joined with `-`
- max 40 characters
- examples: `add-oauth-login`, `fix-payment-timeout`, `refactor-auth-middleware`

### Lifecycle

1. Orchestrator receives new work → checks if `.ai-orchestrator/<slug>/` already exists
2. If exists and `context.md` Status is `planning` or `executing` → **resume** (do not overwrite)
3. If not exists → create folder, write `context.md`, create empty `findings.md`, `tickets.md`, `tasks.md`
4. Pipeline runs, agents read/write workspace files at each phase
5. After Checkpoint 4 approved → orchestrator deletes `.ai-orchestrator/<slug>/`

### `.gitignore` handling

When creating the first workspace in a project, the orchestrator checks for `.ai-orchestrator/` in the project's `.gitignore`. If missing, it appends `.ai-orchestrator/` automatically.

---

## File Schemas

### `context.md`

Written once at creation by the orchestrator. Read by all agents at session start to orient themselves.

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

The `Status` field is updated by the orchestrator as the pipeline progresses. `Active agent` is optional — helps avoid two agents stepping on the same work.

### `findings.md`

Written by `FINDINGS_AGGREGATOR` (findings section) and `epic-generator` (epics section). Read by `prioritizer` and `ticket-splitter`.

```markdown
# Findings

## Epics
[epic artifacts — id, title, goal, scope, impact, priority, dependencies, success_criteria, source_finding_ids]

## Normalized Findings
[finding artifacts — id, title, summary, category, severity, evidence, impact, recommended_action, dependencies]

## Passthrough
[missing_areas, priority_hints forwarded from review-validator / skill-executor]
```

### `tickets.md`

Written after Checkpoint 3 approval by the orchestrator (from ticket-splitter output). Read by `task-executor`.

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

### `tasks.md`

Written by the orchestrator after Checkpoint 3. Updated by `task-executor` as tickets move through states. Read by any agent checking progress.

```markdown
# Tasks

## TODO
- T-001 — Add OAuth provider config
- T-002 — Implement callback handler

## IN PROGRESS
- T-003 — Write integration tests

## DONE
- T-004 — Update environment variables
```

`tasks.md` is intentionally lightweight (id + title only). Full artifact is in `tickets.md`.

---

## System Changes

### New file: `system/WORKSPACE.md`

Defines workspace contract, naming convention, file schemas, lifecycle rules, resume logic, and deletion trigger. Referenced from ORCHESTRATOR.md.

### `system/ORCHESTRATOR.md`

- Add `@ai-orchestrator/system/WORKSPACE.md` to System Map table
- Update `@ai-orchestrator/tasks/tasks.md` row to note it is now a pointer
- Add **Step 0** to all 4 pipelines:
  ```
  0. Load @ai-orchestrator/system/WORKSPACE.md → create or resume .ai-orchestrator/<slug>/
  ```
- Add cleanup step after Checkpoint 4 in all pipelines:
  ```
  → delete .ai-orchestrator/<slug>/ after human approval
  ```
- Update Task Execution section to reference workspace files

### `system/FINDINGS_AGGREGATOR.md`

Add to System Context: writes normalized findings to `.ai-orchestrator/<slug>/findings.md`.

### `skills/epic-generator.md`

Add to System Context: writes epic artifacts to epics section of `.ai-orchestrator/<slug>/findings.md`.

### `skills/ticket-splitter.md`

Update System Context `Outputs to`: writes ticket artifacts to `.ai-orchestrator/<slug>/tickets.md` after Checkpoint 3.

### `prompts/task-executor.md`

Update `Uses`: reads full ticket artifacts from `.ai-orchestrator/<slug>/tickets.md`, updates `.ai-orchestrator/<slug>/tasks.md`.

### `tasks/tasks.md`

Replace content with pointer: active workspaces live in `.ai-orchestrator/<slug>/tasks.md` in your project. See `@ai-orchestrator/system/WORKSPACE.md`.

### `system/CLAUDE_CODE_INTEGRATION.md`

Add `workspace_path` to the Delegation Contract payload:
- `workspace_path`: `.ai-orchestrator/<slug>/` — the path where Codex should read context, tickets, and write results.

---

## Multi-Agent Cooperation Pattern

```
Claude Code (session 1)
  → creates .ai-orchestrator/auth-feature/
  → runs planning pipeline
  → writes findings.md, tickets.md, tasks.md
  → requests Codex to execute T-001

Codex (session 2)
  → reads .ai-orchestrator/auth-feature/context.md    (orient)
  → reads .ai-orchestrator/auth-feature/tickets.md    (get full spec)
  → reads .ai-orchestrator/auth-feature/tasks.md      (check state)
  → executes T-001
  → moves T-001 from TODO → DONE in tasks.md

Claude Code (session 3)
  → reads .ai-orchestrator/auth-feature/tasks.md      (resume)
  → reviews T-001 output
  → continues with T-002
```

---

## Completion

After Checkpoint 4 (human approves execution and review):

1. Orchestrator updates `context.md` Status to `done`
2. Orchestrator deletes `.ai-orchestrator/<slug>/`
3. If `.ai-orchestrator/` is now empty, it may be left in place (the `.gitignore` entry remains useful)
