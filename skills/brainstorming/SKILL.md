---
name: brainstorming
description: Use before any creative or feature work — explore requirements, produce design, and save long-lived documentation (Spec, PRD, or ADR).
---

# Brainstorming

Turn ideas into fully-formed designs through collaborative dialogue, producing long-lived project documentation.

## Process

### 1. Understand Context
Check the current project state: files, docs, recent commits, existing conventions.

### 2. Clarify Requirements
Ask questions one at a time to refine the idea. Focus on: purpose, constraints, success criteria. Prefer multiple-choice when possible.

### 3. Explore Approaches
Propose 2-3 approaches with trade-offs and a recommendation. Lead with the recommended option and explain why.

### 4. Design Confirmation
Present the design to the user section by section. After each section, check if it looks right. Be ready to revise.

### 5. Self Review
Before finalizing, review the design internally:
- Any placeholders, TODOs, or vague requirements?
- Any internal contradictions?
- Is the scope focused enough?
- Could any requirement be interpreted two ways? If so, make it explicit.

Fix issues before presenting to the user.

### 6. Produce Documentation
When the design is confirmed:

- Ask: **Where should the doc be saved?** and **What naming convention to use?**
- If AGENTS.md doesn't define doc location/naming, suggest: "Should I add these conventions to AGENTS.md for future sessions?"
- Generate the long-lived doc: **Spec** (feature specifications), **PRD** (product requirements), or **ADR** (architecture decisions) depending on context
- Only save after user confirmation

### 7. Transition
Ask: "Proceed to planning?" Do NOT force transition.

## Doc Type Selection

| Type | When to use |
|------|-------------|
| Spec | New feature or module, detailed technical specification |
| PRD | Product-level feature, user-facing requirements |
| ADR | Architectural decision, trade-off documentation |

## Key Principles

- One question at a time
- YAGNI ruthlessly — remove unnecessary features
- Incremental validation — confirm before moving on
- Stay focused — scope creep is the enemy

## Tool Mapping

| Generic Term | Description |
|-------------|-------------|
| Read file | Read project files, AGENTS.md, existing docs |
| Search code | Find patterns, conventions, existing implementations |
| Write file | Save the produced spec/PRD/ADR document |
