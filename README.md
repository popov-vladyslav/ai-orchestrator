# AI Orchestrator

A universal AI execution workflow for code review, feature development, bug fixes, and performance work. Works with Claude Code, Codex, GPT, and any AI that can read markdown files.

## Setup

1. **Clone this repo once** to a stable location on your machine:
   ```bash
   git clone https://github.com/your-username/ai-orchestrator ~/ai-orchestrator
   ```
   Or anywhere you prefer — `~/Documents/ai-orchestrator`, `/opt/ai-orchestrator`, etc. You install it once and use it across all your projects.

2. **Point your AI at the entry point** using the absolute path to `ORCHESTRATOR.md`:
   ```
   /Users/you/ai-orchestrator/system/ORCHESTRATOR.md
   ```
   In Claude Code: `@/Users/you/ai-orchestrator/system/ORCHESTRATOR.md`

3. **`@ai-orchestrator/` in the system files** is a self-referential prefix — it resolves relative to wherever you installed the tool. You never need to type it yourself; it's used internally between files so the AI knows how to load each step.

4. **Two separate folders — don't confuse them:**
   - `ai-orchestrator/` — this tool. Lives once on your machine, outside any project.
   - `.ai-orchestrator/` — temporary workspaces, one per active feature, inside each project. Gitignored by default.

5. **Prerequisites:** Any AI assistant that can read files — Claude Code, Codex, GPT with file access, etc. No installs required beyond the markdown files.

## Example

```
# In Claude Code (adjust path to your install location):
@/Users/you/ai-orchestrator/system/ORCHESTRATOR.md

Review the authentication module in src/auth/ — check for security issues,
missing error handling, and anything that doesn't match our current patterns.
```

The AI will:
1. Create a workspace at `.ai-orchestrator/review-auth-module/`
2. Load `@ai-orchestrator/skills/review-validator.md`, run relevant domain skills
3. Surface findings with P0-P3 priority
4. Ask for your approval before creating tickets
5. Execute approved tickets and clean up the workspace when done

## How to Use

Point any AI at the entry point using the absolute path to `ORCHESTRATOR.md` on your machine:

```
/Users/you/ai-orchestrator/system/ORCHESTRATOR.md
```

The AI reads the system map, selects the right mode, and self-orchestrates through the full pipeline — loading the appropriate files at each step, pausing for human approval at defined checkpoints, and tracking work in a per-feature workspace.

## Modes

| Request type | Mode |
|---|---|
| Review output | `REVIEW_MODE` |
| Feature request | `FEATURE_MODE` |
| Bug report | `BUGFIX_MODE` |
| Performance issue | `PERFORMANCE_MODE` |

## What It Does

1. **Frames the problem** — architect or review validator defines scope and constraints
2. **Discovers and runs skills** — selects relevant domain skills, extracts findings
3. **Aggregates findings** — merges, deduplicates, assigns stable IDs
4. **Plans** — prioritizes findings, groups into epics, splits into tickets
5. **Reviews the plan** — tech lead review before execution begins
6. **Human approval** — 4 checkpoints gate every stage
7. **Executes** — ticket by ticket, with code review after each
8. **Cleans up** — per-feature workspace deleted on completion

## File Structure

```
ai-orchestrator/
├── system/
│   ├── ORCHESTRATOR.md          ← entry point
│   ├── WORKSPACE.md             ← per-feature workspace contract
│   ├── SKILL_DISCOVERY.md       ← selects skills for the problem domain
│   ├── SKILL_EXECUTOR.md        ← runs one skill, outputs findings
│   ├── FINDINGS_AGGREGATOR.md   ← merges and normalizes all findings
│   └── DELEGATION.md             ← multi-agent delegation rules
├── skills/
│   ├── architect.md             ← problem framing
│   ├── review-validator.md      ← validates review output
│   ├── prioritizer.md           ← P0-P3 priority + effort
│   ├── epic-generator.md        ← groups findings into epics
│   └── ticket-splitter.md       ← splits epics into tickets
├── prompts/
│   ├── plan-reviewer.md         ← reviews plan before execution
│   ├── task-executor.md         ← executes one ticket
│   └── reviewer.md              ← reviews implementation
├── skills-mapping.md            ← domain → skill auto-selection rules
└── approval-template.md         ← human approval checkpoint format
```

## Multi-Agent Support

One agent acts as the orchestrator (e.g. Claude Code), another as a specialist (e.g. Codex). The specialist can be delegated bounded subtasks — reviewing a folder, executing a single ticket, validating output. All agents cooperate through a shared per-feature workspace at `.ai-orchestrator/<feature-slug>/` in your project.

## Per-Feature Workspace

Each feature gets a temporary workspace in your project:

```
your-project/
└── .ai-orchestrator/
    └── add-oauth-login/
        ├── context.md    ← goal, scope, status
        ├── findings.md   ← findings and epics
        ├── tickets.md    ← approved ticket artifacts
        ├── tasks.md      ← TODO / IN PROGRESS / DONE
        └── results.md    ← execution summaries and review output
```

The workspace is created automatically, survives across sessions, and is deleted after the final approval checkpoint. `.ai-orchestrator/` is gitignored by default.

