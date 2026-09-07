# ADAMS

## Autonomous Diagnostic and Agentic Management System

ADAMS is a reusable workflow for AI coding agents.

Its purpose is to automate a development process that would normally require a developer to repeatedly organize AI agents by hand: understand the project, break the work into tasks, let different agents work on those tasks, check the results, and continue until the important remaining problems have been resolved.

ADAMS turns that process into a repeatable skill that an AI agent can follow.

## What ADAMS Is For

When working on a software project with AI, it is common to do the same coordination work again and again:

- explain the project and the objective
- decide what needs to be done
- split the work into manageable tasks
- choose which agents should work on difficult or simple tasks
- start several agents
- tell them how to coordinate
- check what they actually changed
- identify what is still wrong
- start another round of work

ADAMS exists to make that workflow repeatable.

Instead of manually explaining the process every time, an agent can use the ADAMS skill and follow the same working method from project to project.

## The Core Idea

ADAMS separates the work into three responsibilities:

**Manager** — understands the overall objective, organizes the work, and decides what should happen next.

**Workers** — perform the implementation work on focused tasks.

**Validator** — checks the resulting project and determines what is actually complete and what still needs attention.

The workflow is:

```text
Understand
    ↓
Plan
    ↓
Work
    ↓
Validate
    ↓
Re-plan when necessary
    ↓
Repeat until complete
```

## The Shared Project File

ADAMS uses a single shared file called `prompt.md` for the current work cycle.

The file contains the project context, instructions, task descriptions, task complexity, and task status.

Workers read the same file and claim available tasks from it.

A task normally moves through:

```text
PENDING → IN PROGRESS → DONE
```

A worker must claim a task before working on it and must not deliberately work on a task that another worker has already marked `IN PROGRESS` or `DONE`.

The file therefore gives the agents a common view of the work instead of requiring a separate file for every task.

## Complexity-Based Work

Every task has one of three complexity levels:

- **High**
- **Medium**
- **Low**

Complexity describes the difficulty of a task. It does not permanently assign a task to a particular model.

Different workers can use different priority orders. For example:

```text
Strong worker:
High → Medium → Low

Standard worker:
Medium → Low → High

Light worker:
Low → Medium → High
```

This makes it possible to use stronger agents where they provide the most value while allowing lighter agents to handle simpler work.

## Working With Different AI Agents

ADAMS is intended to be independent from any single AI coding tool.

Today, a project may use agents such as OpenCode or Google Antigravity. Later, the same ADAMS workflow can be used with another coding agent without changing the underlying method.

The workflow should describe **what the agents need to do**, not depend on one specific vendor or application.

## First-Time Setup

When ADAMS is used in a new environment, the agent should help the user configure the available workers for that environment.

The setup should identify things such as:

- which coding agents are available
- which agent or model should be preferred for High-complexity work
- which should be preferred for Medium-complexity work
- which should be preferred for Low-complexity work
- which tool should be used for validation, when applicable

That configuration should be stored separately from the core ADAMS workflow so the skill itself remains reusable.

The workflow can then be reused without asking the user to repeat the same configuration every time.

## How a Typical ADAMS Run Works

A normal run looks like this:

```text
User provides objective
        ↓
ADAMS Manager understands the project
        ↓
Manager creates or updates prompt.md
        ↓
Workers are started
        ↓
Workers claim available tasks
        ↓
Workers implement and verify their work
        ↓
Validator checks the project
        ↓
Remaining problems are identified
        ↓
Manager creates the next work cycle
        ↓
Repeat
```

The process ends when validation shows that there are no relevant unresolved problems.

## Why ADAMS Matters

ADAMS is designed to make AI-assisted development more consistent and easier to repeat.

Instead of treating every AI session as a separate conversation, it provides a shared way of working:

**diagnose → organize → execute → verify → correct**

The value of ADAMS is not a particular model or coding tool. The value is the workflow itself.

## The Purpose of the Skill

The ADAMS skill teaches an AI agent how to use this workflow.

It gives the agent the rules for:

- understanding the project
- creating and maintaining `prompt.md`
- organizing tasks
- using complexity to guide worker priorities
- coordinating multiple workers
- validating results
- handling incomplete work and newly discovered problems
- deciding when to start another cycle
- deciding when the work is complete

The skill is meant to be reusable across projects and compatible with different agent tools.

## Project Files Used by the Workflow

The workflow uses a small set of project-level state files:

```text
prompt.md
prompt_done.md
final_summary.md
```

`prompt.md` describes the current work cycle.

`prompt_done.md` records the result of validation and the problems that remain.

`final_summary.md` records the final validated outcome.

These files make the workflow visible and understandable to both the developer and the agents.

## The Long-Term Vision

ADAMS is intended to become a reusable skill that can be carried from one AI coding environment to another.

The goal is simple: configure the available agents once, give the Manager an objective, and let the ADAMS workflow organize the rest of the work in a consistent way.

The underlying method should remain stable even as the AI tools used to perform the work change.
