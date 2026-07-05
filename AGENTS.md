# AGENTS.md

## Project Nature

This is a **skills project** — it produces skill definitions consumed by end users. When questions mention skill issues, optimizations, or improvements, the context is about how these skills behave when used by end users in their own projects, not about operating on this project itself. Design discussions should always consider the end-user perspective.

When a user refers to "skill" or "skills", they mean the skills defined **in this project** (under `skills/`), not the skills or agents configured in the runtime environment (e.g., opencode's own skill system or agent configurations). If the user says "fix a skill bug" or "improve a skill", the target is always this project's skill definitions.

## Spec file naming

Spec files are stored in `docs/specs/` with the format `NNN-xxxx.md` (e.g., `001-devflow-system-design.md`).

## Language

All project files, skills, and documentation are written in English.

## Skill Authoring

### Core Philosophy

- **Skills guide action, not explain knowledge.** A skill is a set of
  explicit process, norms, and constraints that direct the agent's
  actions. Never explain concepts the agent already knows (e.g., what
  TDD is, how to debug, what makes a good commit message).
- **Naming a concept as a requirement is not explaining it.** "Apply
  YAGNI", "follow Red-Green-Refactor" are constraints — keep. "YAGNI
  means...", "Red-Green-Refactor is a cycle where..." are explanations
  — cut.
- **A skill contains only two things:** (1) *discipline* — gates the
  agent would otherwise skip, plus known concepts it wouldn't activate
  unprompted, surfaced as requirements; (2) *glue* — workflow handoffs,
  output formats, and dispatch protocols the agent can't infer.
  Everything else is re-teaching and gets deleted.
- **Per-line test:** does this line guide an action, or explain a
  concept? Only guiding survives.

### Best Practices

- **Gates must be checkable and unambiguous** — binary conditions
  independent of model and harness (no `ultrathink`-style tokens, no
  harness-specific frontmatter assumptions). Vague gates diverge across
  models and runs.
- **Conciseness is mandatory.** Every sentence must change the agent's
  behavior. Run the no-op test sentence by sentence; delete what fails.
- **Descriptions carry what + when.** Front-load trigger terms so the
  right skill fires; name the specific activation contexts.
- **Progressive disclosure.** Keep SKILL.md focused; push detail to
  reference files one level deep, never chained.
- **Gerund naming**, consistent across the collection (`reviewing-code`,
  `executing-plans`, `verifying-completion`).
