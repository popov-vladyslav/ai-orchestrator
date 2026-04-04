## System Context

* **Part of:** `@ai/system/ORCHESTRATOR.md`
* **Used by:** ORCHESTRATOR — when delegating work to a sub-agent at any pipeline step
* **Uses:** any step file from `@ai/skills/` or `@ai/prompts/` may be delegated
* **Outputs to:** guides the Claude Code ↔ Codex hand-off throughout the workflow

---

# CLAUDE CODE INTEGRATION

## Purpose

Use `Claude Code` as the orchestrator and `Codex` as a delegated specialist for analysis or implementation.

---

## Operating Model

### Claude Code

Owns:

* mode selection
* workflow progression
* skill selection
* artifact consolidation
* approval checkpoints
* final user communication

### Codex

Owns delegated tasks such as:

* reviewing a bounded code change
* summarizing a specific folder or subsystem
* validating one review output
* implementing one approved ticket
* performing a focused regression or risk check

---

## Delegation Contract

When `Claude Code` delegates to `Codex`, include:

* `mode`
* `task_type`
* `goal`
* `scope`
* `inputs`
* `required_output`
* `constraints`

### Recommended Delegation Payload

* `mode`: `REVIEW_MODE`, `FEATURE_MODE`, `BUGFIX_MODE`, or `PERFORMANCE_MODE`
* `task_type`: `analysis`, `review`, `implementation`, or `validation`
* `goal`: one sentence objective
* `scope`: files, folders, PR, ticket ids, or epic ids
* `inputs`: review text, findings, epics, tickets, or source files
* `required_output`: expected artifact shape
* `constraints`: do not edit code, edit only listed files, preserve behavior, and so on
* `workspace_path`: `.ai/<slug>/` — path where Codex reads context, tickets, and writes task state

---

## Good Delegation Examples

### Example: Review Validation

Ask `Codex` to:

* read review output
* validate technical accuracy using `@ai/skills/review-validator.md`
* remove weak points
* return normalized findings

### Example: Ticket Execution

Ask `Codex` to:

* read context from `.ai/<slug>/context.md` to orient itself
* read the full ticket artifact from `.ai/<slug>/tickets.md`
* execute one ticket using `@ai/prompts/task-executor.md`
* touch only approved files
* update `.ai/<slug>/tasks.md` (move ticket from TODO → DONE)
* return execution summary, verification, and residual risks

### Example: Folder Analysis

Ask `Codex` to:

* inspect one folder
* summarize current flows
* identify missing pieces, risks, and inconsistencies

---

## Anti-Patterns

Do not ask `Codex` to:

* "fix the whole project"
* infer missing scope from a large ambiguous request
* plan and implement multiple unrelated streams at once
* bypass the canonical artifact contracts

---

## Hand-Off Pattern

1. `Claude Code` frames the task.
2. `Claude Code` delegates one bounded subtask to `Codex`.
3. `Codex` returns structured output.
4. `Claude Code` aggregates the result into the main workflow.
5. `Claude Code` decides whether approval is needed before the next step.

---

## Output Expectations By Task Type

### Analysis

Return:

* summary of current structure
* findings
* risks
* recommended next actions

### Review

Return:

* normalized findings
* weak or rejected points
* priority hints

### Implementation

Return:

* execution summary
* files changed
* verification
* residual risks

### Validation

Return:

* accepted points
* rejected points
* missing areas
* prioritization hints
