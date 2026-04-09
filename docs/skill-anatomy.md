# Skill Anatomy

Format specification for all skill files in `skills/`.

Note: Existing skills in `skills/` are being migrated to this format. New skills must follow this structure from the start.

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

**System Context** — Required in every skill. Documents the wiring: which mode/file loads this skill, what it reads, and where its output goes. Never remove or abbreviate. Separated from the rest of the skill body by `---`.

**Overview** — 1-2 sentences maximum. Answers: what does this skill do, and why does it exist in this pipeline?

**When to Use** — Bullet list of triggering conditions. Include a "When NOT to use" subsection when meaningful exclusions exist.

**Process** — The core of the skill. Numbered steps. All output field definitions live here. Include an `### Example` subsection when the output format is non-obvious. Omit only when the Process steps are self-explanatory.

**Common Rationalizations** — Table of excuses the AI uses to skip or shortcut this step, paired with factual rebuttals. Every skip-worthy step needs a counter-argument here.

Example:
| Rationalization | Reality |
|---|---|
| "I'll define scope during execution" | Scope decisions made during execution cause unplanned changes. Define it now. |

**Red Flags** — Observable warning signs that this skill is being violated. Used for self-monitoring during execution and review.

Example: `- Success criteria that are not measurable ("works correctly", "feels faster")`

**Verification** — Checklist of exit criteria. Every item must be verifiable with evidence. "Seems correct" is not evidence.

## Cross-Skill References

Reference other skills by their file path:

```
See `@ai-orchestrator/skills/architect.md` for problem framing guidance.
```

Do not duplicate content between skills — reference instead.

## Artifact Schemas

The Process section must enumerate all output fields the agent will populate for that skill. The canonical type definitions for Finding, Epic, and Ticket are in `@ai-orchestrator/system/ORCHESTRATOR.md` — reference them by name, do not redefine the schema.
