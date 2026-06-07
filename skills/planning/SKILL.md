---
name: planning
description: Use after design confirmation — transform design docs into a milestone-grouped, dependency-annotated task list with execution strategy, ready for implementation.
---

# Planning

Transform design documents into a structured, executable task list with milestone grouping, dependency annotations, and execution strategy.

## Process

### 1. Read Input

Consume the design. Two paths are supported:

**Path A — From file** (preferred when available):
- If a design doc exists (Spec / PRD / ADR), read it directly
- Ask the user for the file path if not obvious

**Path B — From conversation context** (when no file exists):
- If the design was kept inline by brainstorming (no persisted file), extract it from the current session's conversation history
- Confirm with the user: "I'll extract the design from our conversation. Let me know if I miss anything."
- Summarize what you extracted before proceeding to decomposition

If neither a file nor conversation context contains a design:
- Ask the user: "What should this implementation achieve? Describe the scope, key features, and constraints."

### 2. Decompose
Break the design into modules and tasks. Identify natural grouping boundaries (e.g., data layer → API layer → frontend). These become milestones.

### 3. Analyze Dependencies
For each task, determine:
- What must be completed before this can start?
- Can tasks within a milestone be ordered logically?
- Which milestones depend on previous milestones?

### 4. Determine Execution Strategy

For each task, classify its nature and determine the execution strategy:

| Signal | Type | TDD? | Execution Mode |
|--------|------|------|----------------|
| Database schema, ORM models, data access | Backend Logic | Yes | Subagent |
| REST/GraphQL endpoints, services | Backend Logic | Yes | Subagent |
| Auth, validation, business rules | Backend Logic | Yes | Subagent |
| Utility functions, data transforms | Logic / Utility | Yes | Subagent |
| React/Vue components, pages, layouts | Frontend UI | No | Subagent |
| CSS/styling changes | Frontend UI | No | Subagent |
| Config files, dependency updates | Config / Infra | No | Main Session |
| Refactoring without behavior change | Refactor | No | Main Session |
| Small bug fix or trivial change | Trivial | No | Main Session |

**Execution mode guidance:**
- **Subagent**: Tasks that involve multiple files, require TDD, or benefit from isolated context. Dispatched via subagent-execution.
- **Main Session**: Simple tasks (config changes, small fixes, trivial updates) that are faster to do directly in the main conversation.

Present the proposed execution strategy to the user for confirmation. The user can adjust individual task strategies.

### 5. Output
Produce a structured task list:

```markdown
## Implementation Plan

### Execution Strategy
- Execution mode: Subagent / Main Session / Mixed
- TDD: Enabled for backend logic and utility tasks
- Review: Milestone checkpoint reviews + final global review

### Milestone 1: {name}
- Task 1.1: {description} [depends on: none] [mode: subagent] [tdd: yes]
- Task 1.2: {description} [depends on: Task 1.1] [mode: subagent] [tdd: no]

### Milestone 2: {name}
- Task 2.1: {description} [depends on: Milestone 1] [mode: subagent] [tdd: yes]
- Task 2.2: {description} [depends on: Task 2.1] [mode: main-session] [tdd: no]
```

Each task should:
- Be small enough to complete in one session
- Have clear acceptance criteria (implied or explicit)
- Note dependencies clearly
- Note execution mode and TDD applicability

## Milestone Grouping Principles

- Group by architectural layer (data, API, UI)
- Group by functional module (auth, billing, dashboard)
- Each milestone should produce a testable, reviewable increment
- 2-5 tasks per milestone is a good range

## Next Step

After the implementation plan is confirmed by the user, invoke the `subagent-execution` skill immediately. Do NOT start implementing. The only valid next action after completing planning is loading the subagent-execution skill.
