---
name: subagent-execution
description: Use after planning — execute tasks serially per milestone, dispatching implementer subagents, with checkpoint reviews between milestones.
---

# Subagent Execution

Execute the implementation plan by dispatching implementer subagents for each task, running checkpoint reviews between milestones, and a final global review.

## Preconditions

Before starting, you must have:
1. **Planning output** — milestone-grouped task list with dependency annotations and execution strategy (mode + TDD per task)

## Assumption

By the time execution begins, brainstorming and planning have thoroughly resolved all design decisions and ambiguities. The plan is complete and unambiguous. Implementer subagents should be able to execute each task without requiring human intervention.

## Git Environment Check

Before starting execution, determine the commit strategy to avoid accumulating an uncommitted blob of changes:

1. Is this a git repository? Check: `git rev-parse --git-dir 2>/dev/null`

2. If **not a git repo**: Ask the user: "This is not a git repository. Do you want me to initialize one and commit as we go?"

3. If **is a git repo**, check whether this is a git worktree:
   - Run `git rev-parse --git-path HEAD` — if the `HEAD` file path is outside `.git/`, run `git worktree list` to confirm
   - Or check: `test -f .git && echo "worktree"` (worktrees have `.git` as a file, not a directory)

4. Commit strategy:

   | Environment | Strategy |
   |------------|----------|
   | **Git worktree** | Require a commit after **each task** completes. The implementer subagent must stage and commit their changes before returning. If a milestone groups multiple tasks, also commit after the milestone. |
   | **Main branch** (regular repo) | Ask the user: "You are on `main`. Should I create a feature branch and commit as we go, or proceed without commits?" |
   | **Feature branch** (regular repo, not worktree) | Ask the user: "Should I commit after each task, after each milestone, or do you prefer to commit yourself?" |
   | **Not a git repo** | Follow user's answer from step 2. |

5. **Rationale**: Committing per task or milestone produces a clean, reviewable history. Accumulating all changes into one big commit obscures the progression of the work and makes review and rollback harder.

## Scheduling Rules

Execute milestones serially. For each milestone, execute tasks serially. Run verification-gate after each task, code-review after each milestone, and final code-review after all milestones. Never dispatch multiple implementer subagents in parallel.

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
{Include the tdd skill instructions: red → green → refactor cycle}

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
- **Focused**: one clear task, not "fix everything"
- **Constrained**: specify what NOT to change if relevant
- **Specific output**: tell the subagent exactly what to return

## Handling Implementer Status

When an implementer subagent returns:

| Status | Action |
|--------|--------|
| **Done** | Proceed to verification-gate |
| **Needs context** | Provide the missing info (or point to the file to read) and re-dispatch. Do not pre-read and summarize unless the subagent explicitly needs interpretation |
| **Blocked** | The plan should have resolved all blockers. If genuinely blocked by an unforeseen issue, assess and resolve — provide more context, break the task down, or adjust the approach. Do not escalate to human. |
| **Concerns raised** | Read concerns, address if critical, otherwise note and proceed |

## Checkpoint Review

After a milestone completes, dispatch two-stage code-review:
1. Gather all changed files from the milestone's tasks
2. **Stage 1**: Dispatch spec compliance reviewer (scope = milestone changes + relevant spec)
3. If spec compliance fails → implementer fixes → re-review (max one round). Do NOT proceed to Stage 2 until Stage 1 passes.
4. **Stage 2**: Dispatch code quality reviewer (scope = milestone changes)
5. If code quality fails → implementer fixes → re-review (max one round)
6. If issues remain after re-review on either stage, fix what can be fixed, note remaining issues, and proceed

## Final Global Review

After all milestones complete, dispatch two-stage code-review:
1. Gather all changes across the entire implementation
2. **Stage 1**: Dispatch spec compliance reviewer (scope = full diff + original spec)
3. If spec compliance fails → implementer fixes → re-review (max one round)
4. **Stage 2**: Dispatch code quality reviewer (scope = full diff)
5. If code quality fails → implementer fixes → re-review (max one round)
6. Both stages pass → proceed to final verification

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


