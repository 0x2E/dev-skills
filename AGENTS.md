# AGENTS.md

## Project Nature

This is a **skills project** — it produces skill definitions consumed by end users. When questions mention skill issues, optimizations, or improvements, the context is about how these skills behave when used by end users in their own projects, not about operating on this project itself. Design discussions should always consider the end-user perspective.

### "Skill" always means THIS project's skills

When a user refers to a "skill" (e.g. "brainstorming skill", "fix a skill bug", "improve a skill", "the planning skill"), they mean the skills defined in **this repo's `skills/` directory**. Always read and edit skill source files at `skills/<name>/SKILL.md` (relative to the repo root).

The runtime may surface installed copies of these skills under `~/.agents/skills/`. Those are not source — **never read from or edit skills outside this repo's `skills/` directory**.

## Spec file naming

Spec files are stored in `docs/specs/` with the format `NNN-xxxx.md` (e.g., `001-devflow-system-design.md`).

## Language

All project files, skills, and documentation are written in English.

## Skill Authoring

A skill guides action; it never explains concepts the model already knows.

### Content types — a skill may contain only these four

Each cures one failure mode — test every line: *which failure mode does it cure?* None → delete.

| Type | Cures | What you write |
|------|-------|----------------|
| **Cue** | Model knows a concept but won't activate it unprompted | Name it as a requirement ("apply Occam's razor") — never define it |
| **Procedure** | Model can do each step, but would pick an unstable sequence or approach | Fix the steps and the choices ("decompose into independent units → subagent → verify → next") |
| **Conventions** | Model can't derive project/harness-specific facts | State verbatim (paths, commands, naming, dispatch protocols, output formats) |
| **Gate** | Model knows what to do but skips it under momentum | Block forward progress until a checkable condition is met with evidence |

**Forbidden: Explanation.** Re-teaching what the model already knows cures
no failure mode and changes no behavior. Knowledge gaps are solved by
external fetch (MCP, knowledge base, other skills), never by in-skill
re-teaching. Defining a term the model can't know (a project-local
concept) is Conventions, not Explanation — the ban targets re-teaching
*general* knowledge the model already has.

### Authoring rules

- Gates must be checkable and unambiguous — binary conditions independent
  of model and harness (no `ultrathink`-style tokens, no harness-specific
  frontmatter assumptions). Vague gates diverge across models and runs.
- **Each skill answers for itself alone** — its content covers its own
  phase, not how other skills in the chain behave. Do not restate,
  anticipate, or coordinate with other skills' content; cross-skill
  coupling makes the collection unmaintainable. State every constraint
  your own flow needs inline — deferring to another skill breaks when it
  isn't loaded. A handoff names the successor; it does not prescribe the
  successor's internals.
- Descriptions carry what + when — front-load trigger terms so the right
  skill fires; name the specific activation contexts.
- Progressive disclosure — keep SKILL.md focused; push detail to a
  reference file one level deep, never chained.
- Naming — prefer gerunds (`reviewing-code`, `executing-plans`,
  `verifying-completion`); keep established domain terms in their
  canonical form (`test-driven-development`).
