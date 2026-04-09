## System Context

* **Part of:** `@ai-orchestrator/system/ORCHESTRATOR.md`
* **Used by:** All modes — after `@ai-orchestrator/skills/epic-generator.md` (skippable if one atomic ticket remains)
* **Uses:** epics from `@ai-orchestrator/skills/epic-generator.md`
* **Outputs to:** `@ai-orchestrator/prompts/plan-reviewer.md` → `@ai-orchestrator/approval-template.md` (Checkpoint 3) → `.ai-orchestrator/<slug>/tickets.md` and `.ai-orchestrator/<slug>/tasks.md` (after human approval)

---

## Overview

Split one epic into atomic implementation tickets so that each unit of work has a clear goal, bounded scope, and verifiable acceptance criteria.

## When to Use

- Always — called once per epic after epic generation
- Skip only when exactly one atomic ticket remains after epic generation (treat the epic goal as the ticket goal)

## Process

1. Read the epic goal and scope.
2. Identify the smallest units of work that can be implemented and reviewed independently.
3. Define file scope, changes, and acceptance criteria for each ticket.
4. Mark dependency edges and parallelization safety.

### Output fields

For each ticket:
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

### Rules

* Prefer small, focused tickets over large multi-concern tickets.
* Prefer tickets that touch 1–5 files when practical.
* Avoid overlap between tickets.
* Mark `parallelizable: true` only when there are no file conflicts and no unresolved dependencies.
* Serial tickets are acceptable when the work naturally depends on earlier changes.

### Example

**Input:** Epic E-001 "Add OAuth login" with findings F-001 (no provider config), F-002 (no callback handler)

**Output (abbreviated):**
* **T-001:** id: T-001, epic_id: E-001, title: "Add OAuth provider config", goal: "Create provider configuration for Google and GitHub", files: [src/auth/providers.ts], changes: ["Add OAuthProvider interface", "Add Google and GitHub configs"], acceptance_criteria: ["Provider configs load from env vars", "Invalid config throws at startup"], risks: ["Env var mismatch across environments"], dependencies: [], parallelizable: true, source_finding_ids: [F-001]
* **T-002:** id: T-002, epic_id: E-001, title: "Implement callback handler", goal: "Handle OAuth callback, exchange code for token, create session", files: [src/auth/callback.ts, src/auth/session.ts], changes: ["Add /auth/callback route", "Exchange auth code for token", "Create session from OAuth profile"], acceptance_criteria: ["Callback exchanges code for token", "Session created on success", "Error redirects to login with message"], risks: ["Token exchange timeout"], dependencies: [T-001], parallelizable: false, source_finding_ids: [F-002]

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "This ticket is large but it's one logical unit" | If a ticket touches more than 5 files or has multiple distinct concerns, split it. Reviewers and rollback both require focused scope. |
| "I'll handle the edge case in the same ticket" | Edge cases have their own acceptance criteria and risk profile. Give them their own ticket so they don't get deprioritized. |
| "Splitting this creates artificial overhead" | Smaller tickets mean faster review cycles, safer rollback, and clearer acceptance criteria. The overhead is worth it. |

## Red Flags

- A ticket with `files[]` longer than 5 entries without explicit justification
- `acceptance_criteria[]` that are vague ("works correctly", "no errors")
- `parallelizable: true` on tickets with overlapping `files[]`
- A ticket with no `source_finding_ids[]`

## Verification

After completing this skill:

- [ ] Each ticket touches 1–5 files (exceptions are explicitly flagged with justification)
- [ ] Every acceptance criterion is testable — not "works as expected"
- [ ] `parallelizable: true` only when no file overlap and no dependency edge exists
- [ ] All `source_finding_ids` reference actual findings from upstream
