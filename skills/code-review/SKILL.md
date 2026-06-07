---
name: code-review
description: Use before merging or after milestones — two-stage review: spec compliance first, then code quality. Also handles receiving review feedback.
---

# Code Review

Two-stage review after each task or milestone: first verify the code matches the spec, then evaluate code quality. Fixing is delegated back to subagent-execution.

## Triggers

1. **Checkpoint Review** — after each milestone completes (invoked by subagent-execution)
2. **Final Global Review** — after all milestones complete (invoked by subagent-execution)
3. **Manual trigger** — user invokes directly via `/review` or instruction

## Review Target

Always determine the review target explicitly before dispatching reviewers:

| Trigger | Review Target |
|---------|--------------|
| Checkpoint Review | Diff between the Milestone's first and last commit |
| Final Global Review | Diff between feature branch HEAD and base branch (e.g., `main`) |
| Manual: PR review | `git diff base...HEAD` (the PR's changes) |
| Manual: uncommitted work | `git diff` + untracked files |
| Manual: specific file/dir | User specifies; scope is only those files |

If the target is unclear, ask the user: "What should I review — uncommitted changes, a specific branch, or a PR?"

## Two-Stage Review

Execute reviews in strict order. Do NOT start Stage 2 before Stage 1 passes.

### Stage 1: Spec Compliance

Dispatch a spec compliance reviewer. **Only question: does the implementation match the specification?**

```markdown
## Review Scope
{specific files or directories to review}

## Original Specification
{summary of what this code should do, from spec/plan}

## Key Acceptance Criteria
- {critical behaviors to verify}
- {edge cases to check}

## Output
- ✅ Spec compliant: all requirements met, nothing extra
- ❌ Issues:
  - Missing: {what the spec requires but code doesn't do}
  - Extra: {what the code does that the spec doesn't require}
  - Wrong: {code contradicts the spec}
- Assessment: Spec Compliant / Needs Fix
```

**Important**: Spec compliance is binary. If the reviewer finds missing, extra, or wrong behavior, it fails. Do NOT proceed to Stage 2.

### Stage 2: Code Quality

Only after Stage 1 passes, dispatch a code quality reviewer:

```markdown
## Review Scope
{specific files or directories to review}
{git SHAs showing the diff}

## Output
- Strengths: what was done well
- Issues:
  - Critical: blocking (bugs, security, data loss)
  - Important: should fix before merge (code structure, error handling)
  - Minor: suggestions, non-blocking (naming, comments)
- Assessment: Ready / Needs Fix
```

## Review Loop

If either stage finds issues:
1. The implementer fixes all issues from that stage
2. Re-run the same stage's review
3. Repeat until that stage passes, then proceed to the next stage
4. Max one re-review round per stage — if issues persist after re-review, fix what can be fixed, note remaining issues, and proceed

## Feedback Handling

When receiving review feedback — whether from the reviewer subagent or from an external reviewer (PR review, user, teammate):

1. Understand: Restate the feedback in your own words. If anything is unclear, ask before implementing.
2. Verify: Check against the actual codebase. Is it technically correct? Does it break existing functionality?
3. Evaluate: Does it violate architectural decisions? Is it YAGNI?
4. Act:
   - Correct and important → implement (one item at a time, test each)
   - Uncertain → ask the user
   - Wrong or inapplicable → push back with technical reasoning

**No performative agreement.** State what you're fixing, or state why you disagree. Actions over words.

If an external reviewer's feedback conflicts with the user's prior architectural decisions, discuss with the user before implementing.
