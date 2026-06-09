---
name: systematic-debugging
description: Use when encountering any bug, test failure, or unexpected behavior, before proposing fixes
---

# Systematic Debugging

**Core principle: Find root cause before attempting fixes. Symptom fixes are failure.**

## Iron Law

1. **No automatic fixes.** Never implement a fix without the user's explicit approval.
2. **No fixes without root cause investigation first.** If you haven't completed Phase 1, you cannot propose fixes.

## When to Use

Use for ANY technical issue: test failures, bugs, unexpected behavior, performance problems, build failures.

**Especially when:** under time pressure, "just one quick fix" seems obvious, you've already tried multiple fixes, previous fix didn't work, you don't fully understand the issue.

**Don't skip when:** issue seems simple, you're in a hurry, someone wants it fixed NOW (systematic is faster than thrashing).

## The Five Phases

Complete each phase before proceeding to the next.

### Phase 1: Root Cause Investigation

1. **Read error messages carefully** — stack traces, line numbers, error codes. Don't skip past warnings.
2. **Reproduce consistently** — exact steps. If not reproducible, gather more data; don't guess.
3. **Check recent changes** — git diff, recent commits, new dependencies, config changes.
4. **Trace data flow** — where does the bad value originate? Trace backward through the call stack until you find the source. Fix at source, not at symptom.
5. **Multi-component systems** — for each component boundary, add diagnostic instrumentation to log what enters and exits, then run once to see where it breaks.

### Phase 2: Pattern Analysis

1. **Find working examples** — locate similar working code in the same codebase.
2. **Compare against references** — if implementing a pattern, read the reference implementation completely.
3. **Identify differences** — list every difference between working and broken, however small.
4. **Understand dependencies** — what other components, configs, or assumptions does this depend on?

### Phase 3: Hypothesis and Testing

1. **Form a single hypothesis** — "I think X is the root cause because Y." Be specific.
2. **Test minimally** — smallest possible change, one variable at a time.
3. **Verify** — did it work? Yes → Phase 4. No → form NEW hypothesis. Don't pile on more fixes.
4. **When you don't know** — say "I don't understand X." Don't pretend.

### Phase 4: Present Before Acting

**Do not implement any fix yet.** After root cause is confirmed, present to the user:

1. **Conclusion** — What is the root cause, with clear evidence.
2. **Fix plan** — What specific changes will be made, where, and why.
3. **Impact assessment** — What else might be affected by this change.

**Wait for the user's explicit approval.** Only proceed to Phase 5 after the user says "go ahead", "approved", "implement it", or similar.

### Phase 5: Implementation (only after user approval)

1. **Create a failing test** that reproduces the bug (use the `test-driven-development` skill for the bug-fix TDD flow).
2. **Implement a single fix** — address the root cause, one change at a time. No "while I'm here" extras.
3. **Verify** — test passes? No other tests broken? Issue resolved?
4. **If fix doesn't work** — stop. Count attempts:
   - < 3: Return to Phase 1 with new information.
   - **≥ 3: STOP. Question the architecture.** Each fix revealing new problems in different places signals a fundamental architectural issue. Discuss with the user before attempting more fixes.

## Red Flags — Return to Phase 1

If you catch yourself thinking any of these, stop and return to root cause investigation:

- "Quick fix for now, investigate later" — random fixes create new bugs
- "Just try changing X and see" — guessing is not debugging
- "It's probably X, let me fix that" — probability is not root cause
- "Let me just fix it directly" — never implement a fix without presenting the conclusion and fix plan to the user first
- "One more fix attempt" (after 2+ failures) — 3+ failures signals an architectural problem, not a debugging one

## When Process Reveals No Root Cause

If systematic investigation reveals the issue is truly environmental, timing-dependent, or external (rare: ~5% of cases):
1. Document what you investigated.
2. Implement appropriate handling (retry, timeout, error message).
3. Add monitoring/logging for future investigation.

## After Debugging

Once the root cause is identified and the fix is verified, return to the workflow step that was interrupted. If debugging was triggered by a subagent failure in `executing-plans`, re-dispatch the implementer with the root cause analysis to apply the fix. If debugging was triggered by a user directly asking about a bug, report the findings and fix.
