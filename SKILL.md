---
name: adams
description: Autonomous Diagnostic and Agentic Management System — a reusable workflow for coordinating AI coding agents through cycles of diagnosis, planning, execution, validation, and correction. Use when working with multiple agents, task coordination, or iterative development workflows.
license: Complete terms in LICENSE.txt
---

# ADAMS

Treat this as the technical lead running a multi-agent software project. The client needs work done reliably across parallel agents, with clear ownership, validated results, and no false confidence about what's actually working. Every cycle must produce verifiable evidence, not just claims of completion.

## Ground your work in the project's current state

Before planning or executing, read the project's actual state: source files, tests, configuration, existing issues, and any prior validation results. Do not rely on assumptions from earlier cycles when newer evidence exists. The project's current reality is the only reliable starting point.

Identify the objective clearly before creating tasks. If the brief does not specify what "done" looks like, define it yourself and confirm with the user. Ambiguous objectives produce scattered work.

## Skill discovery

Before creating tasks, scan all recognized skill directories and build a list of available skills with their names and descriptions. Skill directories are:

```
.opencode/skills/<name>/SKILL.md
~/.config/opencode/skills/<name>/SKILL.md
.claude/skills/<name>/SKILL.md
~/.claude/skills/<name>/SKILL.md
.agents/skills/<name>/SKILL.md
~/.agents/skills/<name>/SKILL.md
```

Read each SKILL.md frontmatter to get the `name` and `description`. When creating tasks, match task content against skill descriptions and attach relevant skill names to each task. If no skill matches, use `none`.

Matching heuristics:
- UI/visual/design tasks → `frontend-design`
- Social coding/vit tasks → `using-vit`
- Skill validation tasks → `skill-validator`
- Project-specific skills → match by description keywords
- No relevant skill → `none`

## Workflow principles

The workflow operates in cycles. Each cycle follows:

```
Understand → Diagnose → Plan → Execute → Validate → Evaluate
```

A cycle ends only when validation confirms zero relevant unresolved problems for the current objective. Do not stop because all tasks are marked done — stop because the project actually works.

**State before assumptions.** Use current project evidence to decide what happens next. Outdated assumptions are the most common source of wasted work.

**Planning before execution.** Understand the objective and create bounded, actionable tasks before any implementation begins. A worker receiving a vague task will produce vague results.

**Focused work.** Workers receive specific, scoped tasks. Do not ask a worker to "fix the project" — ask it to "repair the login timeout handling in auth/service.ts."

**Validation before completion.** A worker marking a task DONE is a claim, not proof. The Validator determines what actually happened.

**Explicit state.** All work is visible through the shared `prompt.md`. If it is not in the file, it does not exist for coordination purposes.

**Feedback loops.** Problems found during validation return to planning. The next cycle reflects current reality, not the previous plan.

## Roles

### Manager

The Manager coordinates and reasons. It does not implement — unless only one agent is available.

When multiple agents exist, the Manager delegates to workers. When only one agent exists (single-agent scenario), the Manager implements directly while still following the workflow: create prompt.md, track tasks, validate results.

Responsibilities:
- Understand the project's objective and current state
- Inspect relevant context: code, tests, docs, prior validation
- Diagnose concrete problems
- Discover available skills and attach relevant ones to tasks
- Create bounded, actionable tasks with clear scope
- Assign complexity: High, Medium, or Low
- Determine worker strategy and concurrency
- Check for config (project → global → setup)
- Start workers (or implement directly if single-agent)
- Detect stalled workers (IN PROGRESS but no file edits)
- Reassign or complete stalled tasks
- Use fallback agent when primary fails, times out, or hits quota
- Interpret validation results
- Decide whether another cycle is needed
- Generate final summary when validated work is complete

Rules:
- Do not treat worker claims as proof of completion
- Re-diagnose when problems persist across multiple cycles
- Avoid creating work just to keep workers busy
- Run interactive setup if no config exists anywhere
- Step in directly when workers stall on planning without executing
- Never auto-fill config values — always ask the user
- Never generate models or commands from your own knowledge — ask the user

### Worker

The Worker implements. It claims tasks, works within scope, and reports honestly.

Responsibilities:
- Read `prompt.md`
- Find an available PENDING task
- Claim it: change status to IN PROGRESS, record agent and start time
- If the task lists Skills, read each referenced SKILL.md before starting implementation
- Follow the skill guidance during implementation
- Implement only the requested scope
- Verify the result when practical
- Mark DONE only when genuinely complete
- Report blockers, uncertainty, and partial completion honestly
- Look for additional available work after finishing

Rules:
- Never work on a task marked IN PROGRESS or DONE
- Never begin implementation before claiming the task
- Preserve existing behavior unless change is required
- Avoid unrelated refactoring
- Report uncertainty rather than guessing

### Validator

The Validator verifies. It is skeptical of completion claims.

Responsibilities:
- Inspect the resulting project, not just worker reports
- Run tests and verification commands
- Verify that requested changes actually work
- Look for regressions
- Detect incomplete work
- Identify newly relevant problems
- Record unresolved problems in `prompt_done.md`
- Provide the Manager with information for the next cycle

Rules:
- Worker reports are evidence, not proof
- Evaluate the project itself, not the claims about it

## Single-agent vs multi-agent

Before starting work, detect whether this is a single-agent or multi-agent scenario:

1. Check if `adams.config.json` exists (project or global)
2. If it exists, read the workers config
3. If only one worker is configured → single-agent mode
4. If multiple workers are configured → multi-agent mode

**Single-agent mode:** The Manager implements directly. Still create prompt.md, track tasks, and validate results. The workflow is the same, but there's no delegation.

**Multi-agent mode:** The Manager delegates to workers via the configured agents. Workers claim tasks from prompt.md and implement them.

## Task model

Every task has:
- Task ID (TSK-01, TSK-02, ...)
- Description
- Complexity (High, Medium, Low)
- Skills (comma-separated skill names, or `none`)
- Status (PENDING, IN PROGRESS, DONE)
- Agent owner (when claimed)
- Start time (when claimed)
- Completion time (when done)

Complexity expresses relative difficulty. It does not permanently bind a task to a specific model or agent. Use it to guide worker priority:

```
Strong worker:  High → Medium → Low
Standard worker: Medium → Low → High
Light worker:   Low → Medium → High
```

These are priority strategies, not restrictions. A worker may still perform any suitable task when the project state requires it.

## Configuration

ADAMS reads `adams.config.json` to know which agents are available, their complexity assignments, and verification commands.

### Config resolution order

When the Manager starts, check for config in this order:

```
1. Project root: ./adams.config.json
      ↓ (missing)
2. Global: ~/.config/adams/adams.config.json
      ↓ (missing)
3. Run interactive setup
```

Project config overrides global config. Use project-specific config when different projects need different agents or models.

### Interactive setup

When no config exists anywhere, the Manager runs interactive setup. **Do not auto-fill, do not suggest models, do not generate from knowledge. Ask the user for everything.**

1. **Ask location:**
   - "Should the config be **global** (saved at `~/.config/adams/adams.config.json`) or **workspace** (saved at `./adams.config.json`)?"

2. **Ask about each worker — one at a time:**

   For **High** complexity:
   - "What agent name? (e.g., opencode, agy, cursor)"
   - "What model? (e.g., gemini-3.8-flash-high)"
   - "What command runs this agent? (e.g., `opencode run`, `agy -p`)"
   - "What priority order? (e.g., High → Medium → Low)"
   - "What fallback agent? (used if primary fails)"
   - "What fallback model?"
   - "What fallback command?"

   For **Medium** complexity:
   - "What agent name?"
   - "What model?"
   - "What command runs this agent?"
   - "What priority order?"
   - "What fallback agent?"
   - "What fallback model?"
   - "What fallback command?"

   For **Low** complexity:
   - "What agent name?"
   - "What model?"
   - "What command runs this agent?"
   - "What priority order?"
   - "What fallback agent?"
   - "What fallback model?"
   - "What fallback command?"

3. **Ask about validation:**
   - "What agent for validation?"
   - "What model for validation?"
   - "What fallback agent for validation?"
   - "What fallback model for validation?"

4. **Ask about verification commands:**
   - "What build command? (e.g., npm run build)"
   - "What test command? (e.g., npm run test)"
   - "What lint command? (e.g., npm run lint)"
   - "What typecheck command? (e.g., npm run typecheck)"

5. **Ask about concurrency:**
   - "How many workers in parallel? (default: 3)"

6. **Confirm** the config with the user before saving

7. **Save** to chosen location

8. **Continue workflow**

### Config structure

```json
{
  "workers": {
    "high": {
      "agent": "agy",
      "model": "gemini-3.8-flash-high",
      "command": "agy --model gemini-3.8-flash-high -p",
      "priority": "High → Medium → Low",
      "description": "Strong worker for complex tasks",
      "fallback": {
        "agent": "opencode",
        "model": "openrouter/nvidia/nemotron-3-super-120b-a12b:free",
        "command": "opencode run -m openrouter/nvidia/nemotron-3-super-120b-a12b:free"
      }
    },
    "medium": {
      "agent": "agy",
      "model": "gemini-3.8-flash-medium",
      "command": "agy --model gemini-3.8-flash-medium -p",
      "priority": "Medium → Low → High",
      "description": "Standard worker for moderate tasks",
      "fallback": {
        "agent": "opencode",
        "model": "opencode/mimo-v2.5-free",
        "command": "opencode run -m opencode/mimo-v2.5-free"
      }
    },
    "low": {
      "agent": "agy",
      "model": "gemini-3.8-flash-low",
      "command": "agy --model gemini-3.8-flash-low -p",
      "priority": "Low → Medium → High",
      "description": "Light worker for simple tasks",
      "fallback": {
        "agent": "opencode",
        "model": "opencode/ling-3.0-flash-fin-free",
        "command": "opencode run -m opencode/ling-3.0-flash-fin-free"
      }
    }
  },
  "validator": {
    "agent": "opencode",
    "model": "opencode/mimo-v2.5-free",
    "command": "opencode run -m opencode/mimo-v2.5-free",
    "description": "Runs tests, checks build, verifies changes",
    "fallback": {
      "agent": "opencode",
      "model": "openrouter/nvidia/nemotron-3.5-lightning:free",
      "command": "opencode run -m openrouter/nvidia/nemotron-3.5-lightning:free"
    }
  },
  "verification": {
    "build": "npm run build",
    "test": "npm run test",
    "lint": "npm run lint",
    "typecheck": "npm run typecheck"
  },
  "concurrency": {
    "max_workers": 3,
    "avoid_parallel": ["same file edits", "database migrations", "dependent tasks"]
  }
}
```

Each worker and validator has a `fallback` object with an alternative agent, model, and command. Use the fallback when the primary agent fails, times out, or hits quota.

Update the config when switching projects or changing available agents.

### Updating config

To update an existing config:
1. Read the current config
2. Ask the user what they want to change
3. Generate updated config
4. Save to the same location (project or global)

## Concurrency

Multiple workers may use the same `prompt.md` concurrently. Independent tasks may run in parallel.

Avoid concurrent execution when tasks:
- Modify the same sensitive area
- Depend directly on each other
- Can overwrite each other's work
- Require a known order

```
Independent work → parallel when useful
Dependent work  → ordered when necessary
Conflicting work → coordinated
```

## Worker prompts

Use these prompts when spawning workers. Each prompt tells the agent exactly how to behave.

### Strong worker (High → Medium → Low)

```
Read prompt.md, strictly follow the rules by immediately changing a task's status to IN PROGRESS before starting work and setting it to DONE when finished (never touching 'IN PROGRESS' tasks); start with the 'High' complexity tasks first, then proceed to the others. If a task lists Skills, read each referenced SKILL.md and follow its guidance. You must edit actual files — planning without executing is not allowed. If you mark a task IN PROGRESS, you must complete the implementation before moving on.
```

### Light worker (Low → Medium → High)

```
Read prompt.md, strictly follow the rules by immediately changing a task's status to IN PROGRESS before starting work and setting it to DONE when finished (never touching 'IN PROGRESS' tasks); start with the 'Low' complexity tasks first, then proceed to the others. If a task lists Skills, read each referenced SKILL.md and follow its guidance. You must edit actual files — planning without executing is not allowed. If you mark a task IN PROGRESS, you must complete the implementation before moving on.
```

### Handling stalled workers

If a worker marks a task IN PROGRESS but does not produce file edits within a reasonable time, the Manager should:

1. Check if the worker is stuck in planning mode
2. Reassign the task to a different worker
3. If no workers available, the Manager completes the task directly
4. Log the stall in `prompt_done.md` under "Regressions"

Signs of a stalled worker:
- Task is IN PROGRESS but no files were edited
- Worker output contains only plans or descriptions, no implementation
- Worker is iterating on approach without committing to edits

## Key files

### prompt.md

The shared working document for the current cycle. Contains project context, instructions, task descriptions, task prompts, and task status. Multiple agents can run this file in parallel — each picks one available task, claims it, executes, and finishes before picking another.

Template:

```markdown
# [Project Name] — Project Context & Task Tracker

## Project Summary & Specifications
* **Business Goal:** [What the project is, who it is for, and its primary objective]
* **Core Features:** [3-5 major features]
* **Tech Stack:**
  * Frontend: [e.g., React 19 + TypeScript, Next.js, Tailwind CSS]
  * Backend: [e.g., Node.js, Express, Python FastAPI, Laravel]
  * Database: [e.g., PostgreSQL, MongoDB, Firebase]

---

## System Instructions for AI Agent ([Current Phase])

> **Multi-Agent Compatible:** Multiple agents can run this file in parallel. Each agent picks one available task, claims it, executes, and finishes before picking another. Never touch a task that is already `IN PROGRESS` or `DONE`. **Planning without executing is not allowed** — you must edit actual files to complete a task.

### Workflow
1. **Context Initialization:** Read all project `.md` files and review relevant specifications.
2. **Task Selection:** Find the first task with the status `PENDING`.
3. **Atomic Start (Claim):** Change its status to `IN PROGRESS`, write your Agent identifier, and log the start time.
4. **Skill Loading:** If the task lists Skills, read each referenced SKILL.md and follow its guidance.
5. **Execution:** Build/refactor the component following the task's instructions. **Edit actual files.**
6. **Verification:** Run the verification command specified in the task.
7. **Completion:** Mark the status as `DONE`, log the completion time, and output a short summary report.

---

## Prompts Stock ([Current Phase])

### Prompt for [TSK-XX]: [Task Title]
**Context:** [Why this task is needed and how it fits into the user journey]
**Skills:** [comma-separated skill names, e.g., `frontend-design`, `vit`, or `none`]
**Task:**
1. [Step 1: Specific instruction on what to build or fix]
2. [Step 2: Specific constraints, design requirements, or edge cases]
3. [Step 3: Required testing, responsiveness, or integration details]

### Prompt for [TSK-XX]: [Task Title]
**Context:** [Context description]
**Skills:** [skill names or `none`]
**Task:**
1. [Step 1]
2. [Step 2]
3. [Step 3]

*(Copy and paste the task block for as many tasks as needed)*

---

## Status Table

| Task ID | Component / Task Description | Complexity | Skills | Status | Agent | Start Time | Completion Time |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **TSK-01** | [Task Description] | High | [skills] | PENDING | - | - | - |
| **TSK-02** | [Task Description] | Medium | [skills] | PENDING | - | - | - |
| **TSK-03** | [Task Description] | Low | [skills] | PENDING | - | - | - |
```

### prompt_done.md

The validated outcome of the current cycle. Created by the Validator after inspecting the project. Must contain a section named `Problems Not Solved` — this is the primary input to the next Manager cycle.

Template:

```markdown
# Validation Results — [Date]

## Completed
- [Task ID] [what was done and how it was verified]

## Failed
- [Task ID] [what was attempted but did not work, and why]

## Remaining
- [Task ID] [partial work or skipped steps]

## Regressions
- [new problems introduced by this cycle's changes]

## Verification Commands Run
- `[command]` → [result]

# Problems Not Solved
- [description of each unresolved problem]
- [why it remains]
- [what should be attempted next]
```

### final_summary.md

Generated after successful validation confirms zero relevant problems. This is the closing record of the work.

Template:

```markdown
# Final Summary — [Project Name]

## What Was Wrong
[Original problems that triggered the ADAMS workflow]

## What Was Changed
[Summary of all changes made across cycles]

## What Was Verified
[Tests run, validations performed, evidence of correctness]

## Important Decisions
[Key choices made during the workflow and why]

## Future Considerations
[Known limitations, technical debt, or follow-up work]
```

## Iteration

After validation:

```
prompt_done.md → Manager evaluation → Problems Not Solved?
                                          /          \
                                        YES           NO
                                         ↓             ↓
                                    new prompt.md   final_summary.md
                                         ↓             ↓
                                      next cycle      stop
```

When problems remain:
1. Read `prompt_done.md`
2. Identify unresolved problems
3. Understand why they remain
4. Re-diagnose when necessary
5. Create bounded tasks
6. Assign complexity
7. Determine worker strategy
8. Start next cycle

When no problems remain:
1. Confirm validation supports completion
2. Avoid unnecessary additional implementation
3. Generate `final_summary.md`
4. Stop

## Preventing repetitive failure

Do not blindly retry the same failed approach. When a problem survives multiple cycles, reconsider:
- The original diagnosis
- The task definition
- The worker choice
- Hidden dependencies
- Project assumptions
- Ambiguous requirements
- The implementation strategy

Repeated failure triggers re-diagnosis, not retry.

## Context priority

When information conflicts:
1. Current validated project state
2. Current `prompt.md`
3. Current task specification
4. Project requirements / architecture
5. Older assumptions

Current evidence overrides stale assumptions.

## Completion

The workflow is complete when validation supports zero relevant unresolved problems for the current objective.

Do not stop because:
- All workers reported DONE
- The original task list was processed
- One test passed
- The Manager believes the project is probably correct

Completion must be supported by validation.
