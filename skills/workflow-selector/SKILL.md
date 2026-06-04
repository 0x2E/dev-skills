---
name: workflow-selector
description: Use after planning — analyze task complexity and present TDD + Review workflow combination options. User chooses before execution begins.
---

# Workflow Selector

Analyze the task list and present workflow combination options so the user can choose how to execute before any code is written.

## Process

### 1. Read Input
Consume the planning output — milestone-grouped task list with dependency annotations.

### 2. Analyze Complexity
For each task, classify its nature:

| Signal | Type |
|--------|------|
| Database schema, ORM models, data access | Backend Logic |
| REST/GraphQL endpoints, services | Backend Logic |
| Auth, validation, business rules | Backend Logic |
| Utility functions, data transforms | Logic / Utility |
| React/Vue components, pages, layouts | Frontend UI |
| CSS/styling changes | Frontend UI |
| Config files, dependency updates | Config / Infra |
| Refactoring without behavior change | Refactor |

### 3. Present Options
Generate 2-3 workflow combos and present with a recommendation:

```
Option A: Full TDD (Recommended)
- TDD for all backend logic and utility tasks
- Milestone checkpoint reviews
- Final global review

Option B: Backend TDD Only
- TDD for backend logic and utility tasks
- No TDD for frontend UI tasks
- Milestone checkpoint reviews
- Final global review

Option C: Review Only (No TDD)
- No TDD on any task
- Milestone checkpoint reviews
- Final global review
```

The user can also specify a custom combination.

### 4. Record Decision
Document the chosen workflow. This will be consumed by subagent-execution as configuration:
- Which tasks/milestones use TDD?
- Review strategy (already fixed: checkpoint + final)

## TDD Applicability

See the `tdd` skill for detailed guidance on when TDD applies. Use that reference to classify each task as TDD-enabled or TDD-skipped.

## Tool Mapping

| Generic Term | Description |
|-------------|-------------|
| Read file | Read planning output |
