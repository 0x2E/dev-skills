---
name: test-driven-development
description: "Use when implementing features or fixing bugs that need test coverage — Red (write failing test first) → Green (minimal code to pass) → Refactor. Use for any task where tests prevent regressions."
---

# Test-Driven Development

Guide implementation through the Red-Green-Refactor cycle.

## The Core Rule

```
NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST
```

If you wrote code before the test, delete it and start over. Watching the test fail proves it actually tests something.

## The Cycle

```
Red:     Write a test → run to confirm it fails (expected failure proves the test is valid)
Green:   Write minimal implementation to pass the test → run to confirm it passes
Refactor: Refactor to improve design → run to confirm tests still pass
```

### RED — Write Failing Test

Write one test for one behavior.

**Good test** — clear name, tests real behavior:
```typescript
test('retries failed operations 3 times', async () => {
  let attempts = 0;
  const operation = () => { attempts++; if (attempts < 3) throw new Error(); return 'ok'; };
  expect(await retryOperation(operation)).toBe('ok');
  expect(attempts).toBe(3);
});
```

**Bad test** — vague name, tests mock not code:
```typescript
test('retry works', async () => {
  const mock = jest.fn().mockRejectedValueOnce(new Error()).mockResolvedValueOnce('ok');
  await retryOperation(mock);
  expect(mock).toHaveBeenCalledTimes(2);
});
```

### Verify RED — Watch It Fail

**Mandatory. Never skip.**

Run the test and confirm:
- Test fails (not errors — failure means the assertion ran but the feature is missing)
- Failure message describes the expected behavior
- Fails for the right reason (feature missing, not typo or setup error)

Test passes immediately? It's testing existing behavior — fix the test.
Test errors out? Fix the error, re-run until it fails correctly.

### GREEN — Write Minimal Code

Write the simplest code to pass the test. Nothing more.

Don't add features, refactor other code, or "improve" beyond what the test requires. That's for the refactor step.

### Verify GREEN — Watch It Pass

**Mandatory.**

Run the test and confirm:
- Test passes
- Other tests still pass
- No warnings or errors in output

### REFACTOR — Clean Up

After green only:
- Remove duplication
- Improve names
- Extract helpers
- Simplify structure

Keep tests green throughout. Don't add new behavior during refactor.

## Test Writing Principles

- Verify behavior, not implementation details
- One test verifies one thing (if the name contains "and", split it)
- Cover happy paths first, then edge cases, then errors
- Prefer real code over mocks — mock only external dependencies that are slow, flaky, or unavailable
- Tests should be readable and serve as documentation

## Workflow

1. Read the task description and acceptance criteria
2. Red: Write a test for the first acceptance criterion, run and see it fail
3. Green: Write the minimal code to make it pass, run and see it pass
4. Refactor: Clean up the code while keeping tests green
5. Repeat for each acceptance criterion
6. Run the full test suite to check for regressions

## Bug Fix TDD

1. Write a failing test that reproduces the bug
2. Verify the test fails (confirms the bug exists)
3. Write the fix to make the test pass
4. Verify all tests pass
5. Refactor if needed

Never fix a bug without a test that reproduces it first.

## When Stuck

| Problem | Solution |
|---------|----------|
| Don't know how to test | Write the wished-for API first, then the assertion. Ask for help. |
| Test too complicated | The design is too complicated. Simplify the interface. |
| Must mock everything | Code is too tightly coupled. Consider dependency injection. |
| Test setup is huge | Extract test helpers. Still complex? Simplify the design. |
| Can't isolate the unit | Test at a slightly higher level (integration-style). Don't skip testing. |
| Test framework unclear | Check package.json scripts, Makefile, or ask before starting. |

## Testing Anti-Patterns

| Anti-Pattern | Problem | Fix |
|-------------|---------|-----|
| Testing mock behavior | Your test proves the mock works, not the code | Use real code, mock only external deps |
| Test-only methods in production | Adding public methods just for testing | Test through the public API |
| Over-mocking | Mocking every dependency makes tests brittle | Mock boundaries (DB, network), not internals |
| Testing implementation details | Tests break on harmless refactors | Test behavior (inputs → outputs), not how it's done |
| Giant test functions | Hard to understand what failed | One assertion per test, or group related assertions clearly |

If the test framework or test command is unclear, ask before starting.
