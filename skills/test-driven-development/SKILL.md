---
name: test-driven-development
description: "Use when implementing features or fixing bugs that need test coverage — Red (write failing test first) → Green (minimal code to pass) → Refactor. Use for any task where tests prevent regressions."
---

# Test-Driven Development

## The Core Rule

```
NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST
```

If you wrote code before the test, delete it and start over. Watching the test fail proves it actually tests something.

## The Cycle

### RED — Write Failing Test

Write one test for one behavior.

### Verify RED — Watch It Fail

**Mandatory. Never skip.**

Run the test and confirm:
- Test fails (not errors — failure means the assertion ran but the feature is missing)
- Fails for the right reason (feature missing, not typo or setup error)

Test passes immediately? It's testing existing behavior — fix the test.

### GREEN — Write Minimal Code

Write the simplest code to pass the test. Nothing more. Don't add features, refactor other code, or "improve" beyond what the test requires — that's for the refactor step.

### Verify GREEN — Watch It Pass

**Mandatory.**

Run the test and confirm: test passes, other tests still pass, no warnings or errors.

### REFACTOR — Clean Up

After green only: remove duplication, improve names, extract helpers, simplify structure. Keep tests green throughout. Don't add new behavior during refactor.

## Workflow

1. Red: Write a test for the first acceptance criterion, run and see it fail
2. Green: Write the minimal code to make it pass, run and see it pass
3. Refactor: Clean up while keeping tests green
4. Repeat for each acceptance criterion
5. Run the full test suite to check for regressions

## Bug Fix TDD

1. Write a failing test that reproduces the bug
2. Verify the test fails (confirms the bug exists)
3. Write the fix to make the test pass
4. Verify all tests pass

Never fix a bug without a test that reproduces it first.
