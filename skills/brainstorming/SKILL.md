---
name: brainstorming
description: Use before any creative or feature work — explore requirements, produce design, and save long-lived documentation (Spec, PRD, or ADR).
---

# Brainstorming

Turn ideas into fully-formed designs through collaborative dialogue, producing long-lived project documentation.

## Process

### 1. Understand Relevant Context
Focus only on modules and files related to the feature/change being discussed. Do NOT scan the entire project. Check:
- Existing code in the target module area
- Related design docs or specs (if any)
- Conventions visible in neighboring files (not project-wide)

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

First, decide whether the design warrants a persisted document:

| Persist to file | Keep in context only |
|----------------|---------------------|
| New feature or module | Bug fix (even with design discussion) |
| Architectural change (ADR) | Small refactor with limited scope |
| Multi-session or multi-person work | Single-session work |
| Product requirement (PRD) | Configuration or dependency change |
| Complex design with multiple approaches | Trivial or obvious implementation |

If persisting is appropriate:

- Ask: **Where should the doc be saved?** and **What naming convention to use?**
- If AGENTS.md doesn't define doc location/naming, suggest: "Should I add these conventions to AGENTS.md for future sessions?"
- Generate the long-lived doc: **Spec** (feature specifications), **PRD** (product requirements), or **ADR** (architecture decisions) depending on context
- Only save after user confirmation

If keeping in context only:
- Summarize the confirmed design in the conversation
- **Tell the user why**: briefly explain the reason for not persisting (e.g., "This is a small bug fix, so the design will stay in conversation context rather than writing a spec file.")
- Pass the summary forward to planning as an inline description (no file written)
- The design lives in the session's conversation history

## Doc Type Selection

| Type | When to use |
|------|-------------|
| Spec | New feature or module, detailed technical specification |
| PRD | Product-level feature, user-facing requirements |
| ADR | Architectural decision, trade-off documentation |
| Inline (no file) | Bug fix, small refactor, config change — design kept in conversation context |


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
