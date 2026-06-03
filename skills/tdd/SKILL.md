---
name: tdd
description: Use when writing code with test-driven development — write a failing test first, then minimal implementation, then refactor.
---

# Test-Driven Development

Guide implementation through the Red-Green-Refactor cycle.

## When to Use

**Applicable:**
- Backend logic functions
- Data processing / transformations
- API endpoints / services
- Utility / helper methods
- Frontend core function methods (state management, data processing, utility functions)

**Not applicable:**
- Frontend page components / styles / layouts
- Config changes / dependency upgrades
- Boilerplate / scaffolding

## The Cycle

```
Red:     Write a test → run to confirm it fails (expected failure proves the test is valid)
Green:   Write minimal implementation to pass the test → run to confirm it passes
Refactor: Refactor to improve design → run to confirm tests still pass
```

## Test Writing Principles

- Verify behavior, not implementation details
- One test verifies one thing
- Cover happy paths first, then edge cases, then errors
- Tests should be readable and serve as documentation

## Workflow

1. Read the task description and acceptance criteria
2. Red: Write a test for the first acceptance criterion, run and see it fail
3. Green: Write the minimal code to make it pass, run and see it pass
4. Refactor: Clean up the code while keeping tests green
5. Repeat for each acceptance criterion
6. Run the full test suite to check for regressions

## Tool Mapping

| Generic Term | Description |
|-------------|-------------|
| Run command | Execute terminal commands (test runner, build) |
| Read file | Read source and test files |
| Edit file | Write tests and implementation |

If the test framework or test command is unclear, ask before starting.
