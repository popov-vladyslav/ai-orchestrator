---
name: orchestrator
description: Universal AI execution workflow for code review, feature development, bug fixing, and performance optimization. Use whenever a task needs structured multi-phase analysis → planning → implementation with human approval gates. Triggers on — "review [module/code/pr]", "add feature", "implement [capability]", "fix bug", "broken [behavior]", "performance issue", "slow [thing]", "optimize [X]", "/orchestrator [task]". ALWAYS use this when the request involves both analysis AND code changes — it prevents unplanned scope creep, enforces human checkpoints, and produces verifiable output through a structured pipeline.
---

Read `@ai-orchestrator/system/ORCHESTRATOR.md` in full before taking any action.
That file is the authoritative pipeline spec. Follow it exactly from Step 0.
Do not proceed from memory or summaries of this skill file.

**Plan mode note:** If running in Claude Code plan mode (file writes are blocked):
defer workspace file creation to the start of Phase 4 execution. Use
in-conversation structured output as workspace substitutes during planning.
At Phase 4 start, create the workspace directory and write all accumulated
artifacts before executing any ticket.
