---
name: verifying-completion
description: "Use before claiming any task is complete, fixed, or passing — run actual verification commands and confirm the output before making any success claims."
---

# Verifying Completion

**Iron rule: No completion claim without fresh verification evidence.**

If you haven't run the verification command in this message, you cannot claim it passes.

## The Gate

Before claiming any status:

1. Identify: What command proves this claim?
2. Run: Execute the full command (fresh, complete)
3. Read: Full output, check exit code, count failures
4. Verify: Does output confirm the claim?
   - If NO: State actual status with evidence
   - If YES: State claim with evidence
5. Only then: Make the claim

## Reference Checklist

| Claim | Must Run |
|-------|----------|
| Tests pass | Run full test suite, 0 failures |
| Lint clean | Run lint command, 0 errors |
| Build succeeds | Run build command, exit code 0 |
| Bug fixed | Run original reproduction steps, confirm no longer occurs |

## Red Flags

- Using "should", "probably", "seems to"
- Expressing satisfaction before verification ("Done!", "Fixed!", "Great!")
- Trusting previous runs or extrapolation
- Relying on partial verification

## Gotchas

- **Previous runs are not evidence.** Output from an earlier command does not describe the current code state. Each completion claim must rest on a command run in the *current* message — anything older is stale.
- **Partial suites miss regressions.** Running only the test file you edited is not a green suite. A change can break untouched code via shared dependencies. Run the full suite unless the scope was explicitly narrowed.
- **Trust the exit code, not the prose.** Some tools print "PASS" but exit non-zero (warnings counted as failures); others print errors yet exit 0. Read both the exit code and the failure count, not the success-looking text.
- **Beware watch/cached output.** Output captured from a `--watch` process or a cached run may reflect a prior state. Use a fresh one-shot invocation to verify.
