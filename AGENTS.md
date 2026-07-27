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

A skill guides action. Author for unhobbling: every line must change behavior.

### Content types — a skill may contain only these four

| Type | Cures | What you write |
|------|-------|----------------|
| **Cue** | Model knows a concept but won't activate it unprompted | Name it as a requirement ("apply Occam's razor") — never define it |
| **Procedure** | Model can do each step, but would pick an unstable sequence or approach | Fix the steps and the choices ("decompose → subagent → verify → next") |
| **Conventions** | Model can't derive project/harness-specific facts | State verbatim (paths, commands, naming, dispatch protocols, output formats) |
| **Gate** | Model knows what to do but skips it under momentum | Block forward progress until a checkable condition is met with evidence |

**Forbidden: Explanation.** Re-teaching general knowledge cures nothing. Knowledge gaps are solved by external fetch (MCP, docs, other skills), never by in-skill re-teaching. A project-local term the model can't know is Conventions, not Explanation.

### Authoring rules

1. **Failure-mode filter.** For every line ask: *which failure mode does it cure?* None → delete. That includes empty opinion, generic "best practices" a strong model already applies, and any line that would not change behavior given the skill's other lines and normal repo context. Prefer cutting over clarifying.

2. **Absolute language — narrow use.** Use must/never/only for: Gates; safety or irreversibility; and this skill's sequence invariants (e.g. "do not start Stage 2 before Stage 1"). For style and exploration defaults, prefer judgement Cues ("match surrounding code", "follow the repo"). Gates must be binary, checkable, and independent of model/harness (no `ultrathink`-style tokens, no harness-specific frontmatter assumptions).

3. **Self-contained phase; no cross-skill coupling.**
   - **Inline** every constraint this phase needs when loaded alone (Gates, this flow's Procedure). Do not defer to another skill for a rule your flow still requires if that skill isn't loaded.
   - **Do not** restate another skill's internals, anticipate its steps, or rephrase the same collection-wide preference in different words across skills.
   - **Do not** invent user-repo style preferences (comment policy, formatting taste, doc voice) that collide with the user's AGENTS.md / CLAUDE.md / equivalent — use "match the repo" Cues instead. Workflow cadence (e.g. when to commit during execution) is Procedure, not user style.
   - **Handoff** names the successor skill only; it does not prescribe the successor's internals.
   - **One skill, one phase.** Open a new skill only when an existing one cannot carry the work without polluting its phase.

4. **Interfaces over examples.** Prefer enums, status tables, binary gates, and fill-in templates over narrative transcripts that pin exploration. Shape the decision surface; do not demo one path.

5. **Rich references for done-checks.** Acceptance criteria, review targets, and completion checks should point to inspectable evidence (tests, commands, `file:line`, rubric templates, reference implementations) — not open-ended adjectives ("robust", "clean", "good UX").

6. **Descriptions** carry what + when — front-load trigger terms; name specific activation contexts.

7. **Progressive disclosure** — keep SKILL.md focused; push detail to a reference file one level deep, never chained. Load the slice the phase needs; do not pre-load every practice the agent might need.

8. **Naming** — prefer gerunds (`reviewing-code`, `executing-plans`, `verifying-completion`); keep established domain terms in canonical form (`test-driven-development`).
