---
name: subagent-execution
description: Use after workflow selection — execute tasks serially per milestone, dispatching implementer subagents, with checkpoint reviews between milestones.
---

# Subagent Execution

Execute the implementation plan by dispatching implementer subagents for each task, running checkpoint reviews between milestones, and a final global review.

## Preconditions

Before starting, you must have:
1. **Planning output** — milestone-grouped task list with dependency annotations
2. **Workflow-selector decision** — which tasks use TDD, which don't

## Scheduling Rules

**All tasks execute serially. Never dispatch multiple implementer subagents in parallel.**

```
For each Milestone (in order):
  │
  ├─ For each Task within Milestone (in order):
  │    │
  │    ├─ 1. Dispatch implementer subagent
  │    │    ├─ If TDD enabled for this task → include tdd skill instructions in prompt
  │    │    └─ If TDD not enabled → instruct to implement directly
  │    │
  │    ├─ 2. Implementer completes → run verification-gate
  │    │    ├─ Tests/lint/build pass → proceed
  │    │    └─ Failures → implementer fixes → re-verify (one retry, then flag)
  │    │
  │    └─ 3. Task marked complete → next task
  │
  ├─ Milestone fully complete → checkpoint code-review
  │    ├─ Review passes → next Milestone
  │    └─ Review has issues:
  │         ├─ Dispatch implementer to fix
  │         ├─ Re-review (max one re-review loop)
  │         └─ Still failing → flag for human decision
  │
  └─ All Milestones complete → final global code-review
```

## Implementer Subagent Prompt

Craft the implementer prompt with these elements:

```markdown
## Task
{task description with full context from plan}

## Relevant Context
{summary of the design/spec this task implements}
{paths to relevant existing files}
{codebase conventions or patterns to follow}

## Acceptance Criteria
- {criterion 1}
- {criterion 2}

## TDD Instructions (if enabled)
{Include the tdd skill instructions: red → green → refactor cycle}

## Output Requirements
- Complete the implementation
- Run verification (tests, lint, build)
- Return: summary of changes + issues encountered + verification result
```

### Prompt Crafting Principles

- **Self-contained**: subagent gets all context it needs; it does NOT read the plan file
- **Focused**: one clear task, not "fix everything"
- **Constrained**: specify what NOT to change if relevant
- **Specific output**: tell the subagent exactly what to return

## Handling Implementer Status

When an implementer subagent returns:

| Status | Action |
|--------|--------|
| **Done** | Proceed to verification-gate |
| **Needs context** | Provide missing info and re-dispatch |
| **Blocked** | Assess: missing context? task too large? plan wrong? — resolve or escalate |
| **Concerns raised** | Read concerns, address if critical, otherwise note and proceed |

Never ignore an implementer saying they are blocked. Something needs to change.

## Checkpoint Review

After a milestone completes:
1. Gather all changed files from the milestone's tasks
2. Dispatch code-review with scope = milestone changes + relevant spec
3. Review report: pass → next milestone; issues → fix loop (max one re-review)

## Final Global Review

After all milestones complete:
1. Gather all changes across the entire implementation
2. Dispatch code-review with scope = full diff + original spec
3. Review report: pass → ready to merge; issues → fix loop (max one re-review)

## Continuous Execution

Do not pause between tasks to check in with the user. Execute all tasks without stopping. Only pause when:
- BLOCKED status you cannot resolve
- Ambiguity that genuinely prevents progress
- All tasks complete

## Key Rules

- **Never parallel dispatch** implementer subagents
- **Always verification-gate** after each task before claiming complete
- **Never skip review** — checkpoint or final
- **Max one re-review loop** — flag for human after that
- **Subagents never read the plan** — provide full context in the prompt

## Tool Mapping

| Generic Term | Description |
|-------------|-------------|
| Dispatch subagent | Create an implementer subagent with isolated context |
| Task list | Manage structured to-do items (mark tasks complete) |
| Run command | Execute verification commands |
| Read file | Read plan file, design docs, existing source files |
| Edit file | N/A — subagent handles implementation |
