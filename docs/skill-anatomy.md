# Skill Anatomy

Format specification for all skill files in `skills/`.

## File Format

Every skill file follows this structure, in this order:

```
## System Context      ← required wiring block
---
## Overview
## When to Use
## Process
## Common Rationalizations
## Red Flags
## Verification
```

## Section Rules

**System Context** — Required in every skill. Documents the wiring: which mode/file loads this skill, what it reads, and where its output goes. Never remove or abbreviate.

**Overview** — 1-2 sentences maximum. Answers: what does this skill do, and why does it exist in this pipeline?

**When to Use** — Bullet list of triggering conditions. Include a "When NOT to use" subsection when meaningful exclusions exist.

**Process** — The core of the skill. Numbered steps. All output field definitions live here. Add an `### Example` subsection only when a concrete example materially aids understanding — not by default.

**Common Rationalizations** — Table of excuses the AI uses to skip or shortcut this step, paired with factual rebuttals. Every skip-worthy step needs a counter-argument here.

**Red Flags** — Observable warning signs that this skill is being violated. Used for self-monitoring during execution and review.

**Verification** — Checklist of exit criteria. Every item must be verifiable with evidence. "Seems correct" is not evidence.

## Cross-Skill References

Reference other skills by their file path:

```
See `@ai-orchestrator/skills/architect.md` for problem framing guidance.
```

Do not duplicate content between skills — reference instead.

## Artifact Schemas

Output field definitions are documented in each skill's `## Process` section. The canonical artifact schemas (Finding, Epic, Ticket) are defined in `@ai-orchestrator/system/ORCHESTRATOR.md`.
