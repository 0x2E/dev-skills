---
name: verification-gate
description: Use before claiming any task is complete, fixed, or passing — run actual verification commands and confirm the output before making any success claims.
---

# Verification Gate

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

## Tool Mapping

| Generic Term | Description |
|-------------|-------------|
| Run command | Execute terminal commands (tests, lint, build) |
