## System Context

* **Part of:** `@ai-orchestrator/system/ORCHESTRATOR.md`
* **Used by:** All modes — after `@ai-orchestrator/skills/epic-generator.md` (skippable if one atomic ticket remains)
* **Uses:** epics from `@ai-orchestrator/skills/epic-generator.md`
* **Outputs to:** `@ai-orchestrator/prompts/plan-reviewer.md` → `@ai-orchestrator/approval-template.md` (checkpoint 3) → `.ai-orchestrator/<slug>/tickets.md` and `.ai-orchestrator/<slug>/tasks.md` (after human approval)

---

You are a senior engineer.

## Task

Split one epic into implementation tickets.

---

## Output

For each ticket, provide:

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

## Rules

* Prefer small, focused tickets over large multi-concern tickets.
* Prefer tickets that touch 1-5 files when practical.
* Avoid overlap between tickets.
* Mark `parallelizable` as `true` only when there are no file conflicts and no unresolved dependencies.
* Serial tickets are acceptable when the work naturally depends on earlier changes.

---

## Example

**Input:** Epic E-001 "Add OAuth login" with findings F-001 (no provider config), F-002 (no callback handler)

**Output (abbreviated):**

* **T-001:** id: T-001, epic_id: E-001, title: "Add OAuth provider config", goal: "Create provider configuration for Google and GitHub", files: [src/auth/providers.ts], changes: ["Add OAuthProvider interface", "Add Google and GitHub configs"], acceptance_criteria: ["Provider configs load from env vars", "Invalid config throws at startup"], risks: ["Env var mismatch across environments"], dependencies: [], parallelizable: true, source_finding_ids: [F-001]
* **T-002:** id: T-002, epic_id: E-001, title: "Implement callback handler", goal: "Handle OAuth callback, exchange code for token, create session", files: [src/auth/callback.ts, src/auth/session.ts], changes: ["Add /auth/callback route", "Exchange auth code for token", "Create session from OAuth profile"], acceptance_criteria: ["Callback exchanges code for token", "Session created on success", "Error redirects to login with message"], risks: ["Token exchange timeout"], dependencies: [T-001], parallelizable: false, source_finding_ids: [F-002]
