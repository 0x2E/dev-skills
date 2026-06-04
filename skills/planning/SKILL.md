---
name: planning
description: Use after design confirmation — transform design docs into a milestone-grouped, dependency-annotated task list ready for execution.
---

# Planning

Transform design documents into a structured, executable task list with milestone grouping and dependency annotations.

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

### 4. Output
Produce a structured task list:

```markdown
## Implementation Plan

### Milestone 1: {name}
- Task 1.1: {description} [depends on: none]
- Task 1.2: {description} [depends on: Task 1.1]

### Milestone 2: {name}
- Task 2.1: {description} [depends on: Milestone 1]
- Task 2.2: {description} [depends on: Task 2.1]
```

Each task should:
- Be small enough to complete in one session
- Have clear acceptance criteria (implied or explicit)
- Note dependencies clearly

### 5. Transition
Ask: "Proceed to workflow selection?" Do NOT force transition.

## Milestone Grouping Principles

- Group by architectural layer (data, API, UI)
- Group by functional module (auth, billing, dashboard)
- Each milestone should produce a testable, reviewable increment
- 2-5 tasks per milestone is a good range

## Tool Mapping

| Generic Term | Description |
|-------------|-------------|
| Read file | Read design doc file (if persisted) |
| Conversation context | Extract design from current session history (if inline) |
