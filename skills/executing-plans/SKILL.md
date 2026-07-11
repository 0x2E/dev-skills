---
name: executing-plans
description: "Use after planning — execute tasks serially per milestone, dispatching implementer subagents, with checkpoint reviews between milestones."
---

# Executing Plans

Execute the implementation plan by dispatching implementer subagents for each task, running checkpoint reviews between milestones, and a final global review.

## Preconditions

Before starting, you must have:
1. **Planning output** — milestone-grouped task list with dependency annotations and execution strategy (mode + TDD per task)

## Git Environment Check

Before starting execution, determine the commit strategy to avoid accumulating an uncommitted blob of changes:

1. Is this a git repository? Check: `git rev-parse --git-dir 2>/dev/null`

2. If **not a git repo**: Ask the user: "This is not a git repository. Do you want me to initialize one and commit as we go?"

3. If **is a git repo**, check whether this is a git worktree:
   - `test -f .git && echo "worktree"` (worktrees have `.git` as a file, not a directory)

4. Commit strategy:

   | Environment | Strategy |
   |------------|----------|
   | **Git worktree** | Require a commit after **each task** completes. The implementer subagent must stage and commit their changes before returning. If a milestone groups multiple tasks, also commit after the milestone. |
   | **Main branch** (regular repo) | Ask the user: "You are on `main`. Should I create a feature branch and commit as we go, or proceed without commits?" |
   | **Feature branch** (regular repo, not worktree) | Ask the user: "Should I commit after each task, after each milestone, or do you prefer to commit yourself?" |
   | **Not a git repo** | Follow user's answer from step 2. |

## Plan Pre-flight

Before dispatching the first task, scan the plan once. Catch problems that are cheap to find now and expensive to hit mid-run:

- **Internal conflicts** — a task contradicts a Global Constraint, or two tasks specify incompatible interfaces (`[produces]`/`[consumes]` don't line up).
- **Defective asks** — something the plan asks for that a reviewer would flag (e.g., a task adds a dependency the Global Constraints forbid).
- **Broken dependencies** — a task's `[depends on]` points to nothing, or the ordering is impossible.
- **Missing acceptance** — a task with no testable success condition.

Raise everything found **at once** to the user. Fix the plan (or get an explicit decision) before starting — do not begin execution and stumble into a conflict partway through. If the scan is clean, proceed.

## Scheduling Rules

Execute milestones serially. For each milestone, execute tasks serially. Run verifying-completion after each task, reviewing-code after each milestone, and final reviewing-code after all milestones. Never dispatch multiple implementer subagents in parallel.

If a task fails verification after 3 fix attempts, invoke `systematic-debugging` to diagnose the root cause before further retries. Do not keep retrying without root cause analysis.

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
{Include the test-driven-development skill instructions: red → green → refactor cycle}

## Commit Instructions (if enabled by git environment check)
{If commits are required: Stage all changed files, write a concise commit message summarizing the task, and commit before returning. Follow the existing commit message convention (check `git log --oneline` for the repo's style).}

## Output Requirements
- Complete the implementation
- Run verification (tests, lint, build)
- Commit changes (if required)
- Return: summary of changes + issues encountered + verification result + commit SHA (if committed)
```

### Prompt Crafting Principles

- **Essential context upfront**: subagent receives task description, design context, file paths, and acceptance criteria. It does NOT need to read the full plan/spec.
- **Self-gather for details**: the subagent should read source files, explore the codebase, and look up conventions on its own as needed. The parent does not need to pre-package every file.

## Handling Implementer Status

When an implementer subagent returns:

| Status | Action |
|--------|--------|
| **Done** | Proceed to verifying-completion |
| **Needs context** | Provide the missing info (or point to the file to read) and re-dispatch. Do not pre-read and summarize unless the subagent explicitly needs interpretation |
| **Blocked** | The plan should have resolved all blockers. If genuinely blocked by an unforeseen issue, assess and resolve — provide more context, break the task down, or adjust the approach. Do not escalate to human. |
| **Concerns raised** | Read concerns, address if critical, otherwise note and proceed |

## Review Gates

After each milestone and after all milestones complete, dispatch two-stage reviewing-code. The only difference between checkpoint and final review is scope:

| Gate | Scope |
|------|-------|
| **Checkpoint** (per milestone) | Diff between milestone's first and last commit |
| **Final global** (all milestones) | Diff between feature branch HEAD and base branch |

If reviewing-code reports issues, dispatch an implementer to fix them, then re-run reviewing-code. Proceed to the next milestone (checkpoint) or to final verification (final) only after reviewing-code passes.

## Over-Engineering Check

After the final global review passes, invoke `simplifying-architecture` to check the implementation for over-engineering.

If no findings, proceed to Final Verification. If findings are presented, the user selects which to act on per the skill's process, then invoke `planning` for the selected findings.

## Final Verification

After the final global review passes, run the full verification suite before claiming completion:

1. Run the full suite: tests, lint, and build
2. If **all pass**: implementation is complete. Report the results to the user.
3. If **any fail**: dispatch a fix subagent with the specific failure output. The fix subagent must:
   - Understand and fix the root cause of the failure
   - Run the failing verification command to confirm the fix
   - Return a summary of changes and the verification result
4. Re-run the full verification suite after the fix.
5. If still failing after **3 fix cycles**: invoke `systematic-debugging` to diagnose the root cause. Do not continue retrying without root cause analysis. After debugging, re-dispatch the implementer with the root cause findings.

## Complete

After final verification passes, report completion to the user with a summary of what was implemented and the verification results.

## Next Step

After final verification passes and completion is reported, invoke the `finishing-work` skill immediately. This skill handles the integration decision: merge, PR, keep, or discard. Do NOT skip this step — the workflow is not complete until the branch is integrated or explicitly set aside.


