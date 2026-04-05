# Spec: ORCHESTRATOR as Single Source of Truth

**Date:** 2026-04-04
**Status:** Implemented

---

## Context

The `ai-orchestrator/` system is a multi-file orchestration framework for AI-assisted code review, feature development, bugfix analysis, and performance work. It supports delegation between Claude Code (orchestrator) and Codex (specialist agent).

Previously, `ORCHESTRATOR.md` described the workflow in abstract terms but did not reference the actual files that implement each step. Sub-files had no awareness of their role in the larger system. This made the system hard to adopt: pointing an AI at `ORCHESTRATOR.md` left it guessing which files to load and when.

**Goal:** Make ORCHESTRATOR.md the single entry point — an AI loading it knows exactly what to load at every step. Every sub-file knows its place in the system. The reference graph is complete and bidirectional.

---

## Design: Bidirectional Reference Network

### Pattern

Every file in `ai-orchestrator/` participates in a two-way reference graph:

- **ORCHESTRATOR.md → sub-files:** System Map table lists every runtime file with its role and when to load it. Each pipeline step has explicit `→ load @ai-orchestrator/...` annotations.
- **Sub-files → ORCHESTRATOR.md and peers:** Every file has a `## System Context` header with `Part of`, `Used by`, `Uses`, and `Outputs to` fields.

This means any file loaded in isolation — by a user, an AI, or a plugin — can orient itself and navigate to the right next step.

### Why bidirectional over one-way hub

A one-way hub (ORCHESTRATOR → files only) breaks when a sub-file is loaded directly (e.g. `@ai-orchestrator/skills/architect.md` without the orchestrator). With bidirectional references, each file carries enough context to work independently and still plug back into the full pipeline.

---

## Files Changed

| File | Change |
|------|--------|
| `system/ORCHESTRATOR.md` | Added "How to Use", System Map table, per-step `→ load` annotations on every pipeline step |
| `skills/architect.md` | Added `## System Context` header |
| `skills/review-validator.md` | Added `## System Context` header |
| `skills/prioritizer.md` | Added `## System Context` header |
| `skills/epic-generator.md` | Added `## System Context` header |
| `skills/ticket-splitter.md` | Added `## System Context` header |
| `system/SKILL_DISCOVERY.md` | Added `## System Context` header; added step to consult `skills-mapping.md` |
| `system/SKILL_EXECUTOR.md` | Added `## System Context` header |
| `system/FINDINGS_AGGREGATOR.md` | Added `## System Context` header |
| `system/CLAUDE_CODE_INTEGRATION.md` | Added `## System Context` header; updated delegation examples to reference file paths |
| `prompts/task-executor.md` | Added `## System Context` header |
| `prompts/reviewer.md` | Added `## System Context` header |
| `prompts/plan-reviewer.md` | New file — reviews plan structure before execution |
| `skills-mapping.md` | Added `## System Context` header and npx query strings per domain |
| `approval-template.md` | Added `## System Context` header |
| `tasks/tasks.md` | Added `## System Context` header |

---

## Reference Graph

```
ORCHESTRATOR.md (entry point)
├── skills/architect.md              → SKILL_DISCOVERY.md (or prioritizer in PERFORMANCE_MODE)
├── skills/review-validator.md       → FINDINGS_AGGREGATOR.md
├── system/SKILL_DISCOVERY.md
│   ├── uses: skills-mapping.md
│   └── → SKILL_EXECUTOR.md (one call per skill)
├── system/SKILL_EXECUTOR.md         → FINDINGS_AGGREGATOR.md
├── system/FINDINGS_AGGREGATOR.md    → skills/prioritizer.md
├── skills/prioritizer.md            → skills/epic-generator.md
├── skills/epic-generator.md         → skills/ticket-splitter.md
├── skills/ticket-splitter.md        → prompts/plan-reviewer.md → approval-template.md
├── prompts/plan-reviewer.md         → approval-template.md (checkpoint 3)
├── prompts/task-executor.md         → prompts/reviewer.md
├── prompts/reviewer.md              → approval-template.md (checkpoint 4)
├── approval-template.md             → human decision
├── tasks/tasks.md                   → (pointer only — active boards in .ai-orchestrator/<slug>/tasks.md)
└── system/CLAUDE_CODE_INTEGRATION.md (delegation layer, applies throughout)
```

---

## Usage

Point any AI at the system with:

```
@ai-orchestrator/system/ORCHESTRATOR.md
```

It will:
1. Read the System Map to understand all available files
2. Infer the execution mode from your request
3. Load files step by step as the pipeline progresses
4. Present approval checkpoints using `@ai-orchestrator/approval-template.md`
5. Track execution in the per-feature workspace (see workspace spec)

---

## Future: Plugin / npx Integration

The system is designed to be published as a standalone package. The `ORCHESTRATOR.md` entry point and explicit file references make it compatible with future `npx skills` or plugin discovery mechanisms — any tool that can load a markdown file and follow `@`-references can drive the full workflow.
