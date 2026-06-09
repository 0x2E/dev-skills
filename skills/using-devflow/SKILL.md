---
name: using-devflow
description: Entry point for the dev-skills workflow system — provides skill index and routes user intent to the right skill.
---

# Using DevFlow

Entry point for the dev-skills system. Routes user intent to the right skill and ensures the workflow starts without autonomous pre-research gaps.

**Two routing paths:**
- **Structured workflow** — analysis/design, planning, execution, finishing (applies to feature development, system design, architecture review, research, documentation planning, and more)
- **Problem fixing** — bug, test failure, crash, build break, performance issue → `systematic-debugging`

The structured workflow is general-purpose — any task that benefits from requirements clarification, systematic planning, and structured execution can use it. Development is the primary use case, not the only one.

## Skills & Routing

| Skill | Trigger | Purpose |
|-------|---------|---------|
| `brainstorming` | "Design X", "Brainstorm Y", "Analyze X", "Evaluate X", "How should I approach X", "Figure out X", "I want to build X" | Requirements/analysis → design → long-lived docs |
| `planning` | "Create a plan", "Plan this", "Break this down", "What are the steps for X" | Design doc → milestone-grouped task list with execution strategy |
| `subagent-execution` | "Execute the plan", "Start implementing" | Serial milestone execution with checkpoint reviews |
| `finishing-work` | Invoked by subagent-execution | Test verification → merge/PR/keep/discard options → cleanup |
| `code-review` | "Review my code", "/review", "Check this PR" | Dispatch reviewer → structured report + handle feedback |
| `verification-gate` | "Am I done?", "Is this ready?" | Run commands → verify output → claim complete |
| `systematic-debugging` | "This bug", "Fix this error", "Why is this failing", "Tests are failing", "Build broke", "Something crashed", "Debug this", "Performance issue", any error/failure/exception scenario | Root cause investigation → hypothesis → minimal fix |
| `test-driven-development` | "add tests", "write a test", "TDD", any implementation with test coverage | Red-Green-Refactor cycle — also invoked internally by subagent-execution |

When a user request matches the trigger column, load the corresponding skill immediately — no pre-research.

## Red Flags

These thoughts mean STOP — you are about to waste tokens and duplicate work. Load the skill instead.

| Thought | Reality |
|---------|---------|
| "Let me explore the project first" | Skills define what to explore and how. Load the skill first. |
| "Let me gather context before loading the skill" | Each skill handles its own context gathering internally. Pre-gathering duplicates work. |
| "Let me spawn a subagent to understand the project" | Skills define exploration scope. Load the skill first. |
| "I should read some files first to understand" | The skill tells you what to read. Load it now. |
| "This is just a simple question / small task" | Check the routing table. If it matches, load the skill. |
| "I know what brainstorming does, I can summarize it" | Load the current version. Skills evolve. Don't rely on memory. |
| "Something is broken / failing / crashing right now" | Load `systematic-debugging` — do not guess fixes |

## Rules

- When a user request matches a trigger in the table above, load the matching skill **immediately** — no pre-research.
- Do NOT pre-gather project information, spawn subagents, or read files before loading the skill.
- Do NOT describe the workflow chain as a plan. Load the first skill and let it drive.
- When encountering any error, test failure, crash, build break, or unexpected behavior — load `systematic-debugging` immediately, do not attempt ad-hoc fixes.
- The user can interrupt at any time — say "stop", "skip to planning", "just do X", etc.
- **Before modifying any code** — no matter how small the issue, requirement, or change — present the approach/solution plan and wait for user confirmation before executing. The only exception is when the user has explicitly requested or authorized skipping confirmation.
