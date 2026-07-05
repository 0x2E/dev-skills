---
name: planning
description: "Use after design confirmation — transform design docs into a milestone-grouped, dependency-annotated task list with execution strategy, ready for implementation."
---

# Planning

Transform design documents into a structured, executable task list with milestone grouping, dependency annotations, and execution strategy.

## Process

### 1. Read Input

Consume the design. Three paths are supported:

**Path A — From file** (preferred when available):
- If a design doc exists (Spec / PRD / ADR), read it directly
- Ask the user for the file path if not obvious

**Path B — From plan-attached design** (when brainstorming passed a task-scoped design):
- The design was confirmed during brainstorming but marked as task-scoped (not a standalone spec file)
- Extract the key design decisions, constraints, and approach from conversation context
- These will form the **Design** section at the top of the plan document
- Summarize what you extracted before proceeding to decomposition

**Path C — From minimal context** (bug fix, small refactor, config change):
- Extract the essential goal from context — no structured Design section needed
- Confirm with the user: "I'll extract the design from our conversation. Let me know if I miss anything."

If neither a file nor conversation context contains a design:
- Ask the user: "What should this implementation achieve? Describe the scope, key features, and constraints."

### 2. Decompose
Break the design into modules and tasks. Identify natural grouping boundaries (e.g., data layer → API layer → frontend). These become milestones. Each milestone should produce a testable, reviewable increment. Aim for 2-5 tasks per milestone.

### 3. Analyze Dependencies
For each task, determine:
- What must be completed before this can start?
- Can tasks within a milestone be ordered logically?
- Which milestones depend on previous milestones?

### 4. Determine Execution Mode

**Subagent** is the default for all tasks. Use **Main Session** only for trivial single-line changes or purely conversational tasks.

### 5. Determine Testing Strategy

Apply TDD to tasks with testable logic (backend, utilities, data transforms). Skip TDD for pure UI, styling, config, refactoring without behavior change, and trivial changes.

### 6. Commit Discipline

After each milestone is fully complete (all tasks pass), commit the changes before starting the next milestone.

Present the proposed execution strategy to the user for confirmation. The user can adjust individual task strategies.

### 7. Output
Produce a structured task list:

```markdown
## Design (if applicable)

[Key design decisions, constraints, and approach extracted from brainstorming — included when the design was task-scoped and marked as plan-attached]

## Implementation Plan

### Global Constraints
[Rules binding EVERY task — version floors, dependency limits, naming, error/copy conventions, exact values. Copy verbatim so they reach every downstream implementer/reviewer; do not paraphrase. Omit when none apply.]
- {e.g. Node >= 18; all error responses shaped {error:{code,message}}; no new deps without approval}

### Execution Strategy
- Execution mode: Subagent (default) / Mixed
- TDD: See task annotations
- Commit: After each milestone completes, commit before proceeding to the next
- Review: Milestone checkpoint reviews + final global review

### Milestone 1: {name}
- Task 1.1: {description} [depends on: none] [mode: subagent] [tdd: yes] [produces: {contract}] [consumes: {contract}]
- Task 1.2: {description} [depends on: Task 1.1] [mode: subagent] [tdd: no]

### Milestone 2: {name}
- Task 2.1: {description} [depends on: Milestone 1] [mode: subagent] [tdd: yes]
- Task 2.2: {description} [depends on: Task 2.1] [mode: main-session] [tdd: no]
```

Each task should:
- Be sized to earn its own test cycle and a reviewer's pass — fold setup, config, and docs into the task that needs them rather than splitting them into standalone tasks
- Have clear acceptance criteria (implied or explicit)
- Note dependencies clearly
- Note execution mode and TDD applicability
- When a task exposes a contract a neighbor relies on, annotate it with `[produces: ...]`; when it depends on a neighbor's contract, `[consumes: ...]` (omit on trivial tasks)

## Next Step

After the implementation plan is confirmed by the user, invoke the `executing-plans` skill immediately. Do NOT start implementing. The only valid next action after completing planning is loading the executing-plans skill.
