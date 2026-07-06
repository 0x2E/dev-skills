---
name: reviewing-code
description: "Use before merging or after milestones — two-stage review: spec compliance first, then code quality. Also handles receiving review feedback."
---

# Reviewing Code

Two-stage review after each task or milestone: first verify the code matches the spec, then evaluate code quality. Fixing is delegated back to executing-plans.

## Triggers

1. **Checkpoint Review** — after each milestone completes (invoked by executing-plans)
2. **Final Global Review** — after all milestones complete (invoked by executing-plans)
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

## Dispatch Discipline

When dispatching either reviewer subagent, the prompt may contain **only** facts: the review scope, the spec/plan, and the diff (or SHAs). It must never carry the dispatcher's opinion about the work.

**Banned in the dispatch prompt:**
- Pre-rating severity ("this is minor", "at most Important")
- Telling the reviewer to skip or ignore anything ("don't worry about X")
- Answering for the reviewer ("this is fine because…", "I left this unabstracted on purpose")
- Framing the spec loosely so something slips through

A reviewer handed the dispatcher's verdict has no reason to check the work. Keep it adversarial by feeding evidence, never conclusions.

The reviewer is **strictly read-only**: it must not edit files or run `git checkout`/`commit` — a reviewer that runs `git checkout` can orphan later commits.

## Two-Stage Review

Execute reviews in strict order. Do NOT start Stage 2 before Stage 1 passes.

### Stage 1: Spec Compliance

Dispatch a spec compliance reviewer. **Only question: does the implementation match the specification?** Use the prompt in `templates/stage1-spec-compliance.md`.

**Important**: Spec compliance is binary. If the reviewer finds missing, extra, or wrong behavior, it fails. Do NOT proceed to Stage 2.

Each issue must include: file:line reference, what's wrong, what the spec requires, and how to fix.

### Stage 2: Code Quality

Only after Stage 1 passes, dispatch a code quality reviewer using the prompt in `templates/stage2-code-quality.md`.

Stage 2 requires file:line references for every concrete issue. Calibrate severity — not everything is Critical. Acknowledge strengths before listing issues; accurate praise helps the implementer trust the rest of the feedback.

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

### Feedback Decision Table

| Feedback pattern | Don't | Do |
|------------------|-------|-----|
| Reviewer flags a real bug with `file:line` | Argue defensively | Fix it — one item, tested |
| Reviewer says "this is wrong" without evidence | Agree reflexively to seem responsive | Verify against the code first; implement only if correct |
| Suggestion that adds scope beyond the spec | Implement to please the reviewer | Push back: out of scope / YAGNI; ask the user if unsure |
| External feedback conflicts with a prior decision | Silently override the decision | Surface the conflict to the user before acting |
| Feedback you don't understand | Guess at the intent and change something | Restate it in your own words and ask before acting |

## Gotchas

- **Resist "Critical" inflation.** Tagging style nits as Critical drowns out the real must-fix issues. Reserve Critical for bugs, security holes, data loss, and broken functionality.
- **Large diffs get shallow reviews.** If the milestone diff is too large for one pass, review per-task or split the scope explicitly rather than letting the reviewer skim.
