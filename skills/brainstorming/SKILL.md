---
name: brainstorming
description: Use before any creative, analytical, or design work — explore requirements, clarify scope, produce design, and save long-lived documentation (Spec, PRD, ADR, or analysis report).
---

# Brainstorming

Turn ideas into fully-formed designs or analysis through collaborative dialogue, producing long-lived project documentation.

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

Decide the output type based on the design's scope and expected lifespan:

| Output | Criteria |
|--------|----------|
| **Long-lived doc** (Spec / PRD / ADR) | Enduring truths about the project — module architecture, core protocols, technical decisions with lifetime relevance. These outlive any single task. |
| **Plan-attached design** | Feature work or optimization with substantive design, but the design serves the current task only. No long-term reference value after implementation. |
| **Context-only** (inline) | Bug fix, small refactor, config change, trivial change — minimal design needed. |

### Long-lived doc (Spec / PRD / ADR)

**First, check AGENTS.md** for existing doc storage conventions. If AGENTS.md defines location and naming, follow it directly — do not re-ask the user.

**If AGENTS.md has no doc conventions**, present the built-in defaults as a recommendation:

| Doc Type | Default Location | Naming |
|----------|-----------------|--------|
| Spec / PRD | `docs/specs/` | `NNN-{description}.md` (e.g., `001-devflow-system-design.md`) |
| ADR | `docs/adrs/` | `NNN-{description}.md` (same format, independent directory) |

Rules:
- Three-digit sequential numbering, incremented independently per directory
- Description in lowercase English, words separated by hyphens
- Scan the target directory for existing files; use max existing number + 1

Ask the user to confirm or adjust the default scheme. Once confirmed, write the conventions to AGENTS.md so future sessions won't need to re-ask.

If the project has no AGENTS.md, suggest creating one.

Then:
- Generate the long-lived doc: **Spec** (feature specifications), **PRD** (product requirements), or **ADR** (architecture decisions) depending on context
- Only save after user confirmation

### Plan-attached design

When the design has substance but is task-scoped (not an enduring project truth):
- Summarize the confirmed design in the conversation
- **Tell the user why**: "This is a task-scoped design. It will be attached to the implementation plan rather than saved as a standalone spec."
- Pass the design forward to planning — it will become the **Design** section at the top of the plan document (see `planning` skill for output format)

### Context-only (inline)

For minimal-design tasks (bug fix, config change, trivial):
- Summarize the confirmed design in the conversation
- **Tell the user why**: briefly explain the reason for not persisting (e.g., "This is a small bug fix, so the design will stay in conversation context rather than writing a spec file.")
- Pass the summary forward to planning as an inline description (no file written)
- The design lives in the session's conversation history

## Key Principles

- One question at a time
- YAGNI ruthlessly — remove unnecessary features
- Incremental validation — confirm before moving on
- Stay focused — scope creep is the enemy

## Next Step

After the design is confirmed and documentation is produced (whether persisted to file or kept in conversation context), invoke the `planning` skill immediately. Do NOT start implementing. Do NOT explore the codebase further. The only valid next action after completing brainstorming is loading the planning skill.
