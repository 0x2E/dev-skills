---
name: finishing-work
description: "Use when implementation is complete, all tests pass, and you need to decide how to integrate the work — presents structured options for merge, PR, or cleanup"
---

# Finishing Work

Guide completion of development work by verifying tests, presenting clear options, and executing the chosen workflow.

## Preconditions

Before starting, you must have:
1. All implementation tasks complete (executing-plans has finished)
2. Final verification has passed (tests, lint, build)

## Process

### Step 1: Verify Tests

Run the project's test suite before offering options:

```bash
npm test / cargo test / pytest / go test ./...
```

If tests fail: report failures. Cannot proceed with merge/PR until tests pass. Stop.

If tests pass: continue to Step 2.

### Step 2: Detect Environment

Determine the workspace state:

```bash
git rev-parse --git-dir 2>/dev/null && git branch --show-current
```

| State | Menu | Cleanup needed |
|-------|------|----------------|
| Normal repo, named branch | 4 options | No worktree to clean |
| Git worktree, named branch | 4 options | Clean up worktree after merge/discard |
| Detached HEAD | 3 options (no merge) | Externally managed |

### Step 3: Determine Base Branch

```bash
git merge-base HEAD main 2>/dev/null || git merge-base HEAD master 2>/dev/null
```

If neither works, ask: "Which branch did this split from?"

### Step 4: Present Options

**For normal repo and named-branch worktree (4 options):**

```
Implementation complete. What would you like to do?

1. Merge back to <base-branch> locally
2. Push and create a Pull Request
3. Keep the branch as-is (I'll handle it later)
4. Discard this work

Which option?
```

**For detached HEAD (3 options — no local merge):**

```
Implementation complete. You're on a detached HEAD.

1. Push as new branch and create a Pull Request
2. Keep as-is (I'll handle it later)
3. Discard this work

Which option?
```

### Step 5: Execute Choice

#### Option 1: Merge Locally

Return to main repo root, checkout <base-branch>, pull, merge <feature-branch>. Verify tests on the merged result. Only after merge succeeds: clean up worktree (if applicable), then `git branch -d <feature-branch>`.

#### Option 2: Push and Create PR

`git push -u origin <feature-branch && gh pr create`. Do NOT clean up worktree — the user needs it for PR iteration.

#### Option 3: Keep As-Is

Report: "Keeping branch <name>. No changes made."

#### Option 4: Discard

Ask for confirmation first. If confirmed: return to main repo root, checkout <base-branch>, `git branch -D <feature-branch>`, clean up worktree (if applicable).

### Step 6: Clean Up Worktree (Options 1 & 4 only, if applicable)

Only if in a git worktree: return to main repo root, run `git worktree remove <path> && git worktree prune`. Do NOT run worktree remove from inside the worktree itself.

## Quick Reference

| Option | Merge | Push | Clean up worktree |
|--------|-------|------|-------------------|
| 1. Merge locally | yes | - | yes |
| 2. Create PR | - | yes | no |
| 3. Keep as-is | - | - | no |
| 4. Discard | - | - | yes |

## Red Flags

- Never proceed with failing tests.
- Never merge without verifying tests on the merged result.
- Never delete work without typed "discard" confirmation.
- Never clean up worktree for Option 2 (user needs it for PR iteration).
- Never run `git worktree remove` from inside the worktree being removed.

This is the terminal phase of the workflow chain. After completion, report the outcome to the user. No further skill is invoked.
