---
name: brainstorming
description: "Use before any creative, analytical, or design work — explore requirements, clarify scope, produce design, and save long-lived documentation (Spec, PRD, ADR, or analysis report)."
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
Ask questions one at a time, waiting for feedback before the next. Prefer multiple-choice, and attach your recommended answer to each question.

- **Look up facts, ask decisions**: anything discoverable in the codebase or existing docs, find it directly. Only put genuine decisions (choices the user must make) to the user.
- **Drill into dependencies, don't pre-enumerate**: during clarification, if the current question depends on another not-yet-settled decision or fact, go resolve that dependency first, then return to the original question. You don't need to list all decisions upfront — drill in only when you hit a dependency.
- **Don't let vagueness pass**: when an answer contains vague terms ("roughly", "flexible", "it depends") or words open to multiple readings, keep probing until concrete — specific values, scenarios, counterexamples, boundaries. Never silently substitute your own assumption for a decision the user hasn't made. There is no objective "done" threshold here — keep probing until the user confirms a shared understanding of the requirements.

Focus on: purpose, constraints, success criteria. Apply YAGNI ruthlessly — cut anything not essential to the core goal.

### 3. Explore Approaches
Propose 2-3 approaches with trade-offs and a recommendation. Lead with the recommended option and explain why.

### 4. Design Confirmation
Present the design to the user section by section. After each section, check if it looks right. Be ready to revise.

### 5. Self Review
Before finalizing, review the design internally:
- Resolve placeholders, TODOs, and vague requirements.
- Resolve internal contradictions.
- Tighten scope to what is essential.
- Disambiguate any requirement open to two readings.
- Remove transient content (implementation progress, todos, status tracking, phase plans).

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

**Content discipline.** Long-lived docs hold only enduring truths: system behavior, decision rationale, requirements, constraints, interfaces. Transient content — implementation progress, TODO/task lists, status checkboxes, phase or milestone plans, next steps — belongs in the plan or issue tracker, not here. Reject trigger words: `Phase 1`, `TODO`, `✓ done`, task list.

Then:
- Generate the long-lived doc: **Spec** (feature specifications), **PRD** (product requirements), or **ADR** (architecture decisions) depending on context
- Only save after user confirmation

### Plan-attached design

When the design has substance but is task-scoped (not an enduring project truth):
- Summarize the confirmed design in the conversation
- **Tell the user why**: "This is a task-scoped design — it won't be saved as a standalone spec."
- Pass the design forward to the planning skill

### Context-only (inline)

For minimal-design tasks (bug fix, config change, trivial):
- Summarize the confirmed design in the conversation
- **Tell the user why**: briefly explain the reason for not persisting (e.g., "This is a small bug fix, so the design will stay in conversation context rather than writing a spec file.")
- Pass the summary forward to planning as an inline description (no file written)

## Next Step

After the design is confirmed and documentation is produced (whether persisted to file or kept in conversation context), invoke the `planning` skill immediately. Do NOT start implementing. Do NOT explore the codebase further. The only valid next action after completing brainstorming is loading the planning skill.
