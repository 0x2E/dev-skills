# dev-skills

A lightweight, loosely-coupled, harness-agnostic set of development workflow skills for AI coding assistants. Inspired by [Superpowers](https://github.com/obra/superpowers).

## Philosophy

- **Concise**: Each skill is focused and minimal — no heavy checklists or verbose gates
- **Harness-agnostic**: Uses neutral terminology with a tool mapping glossary per skill
- **Chainable or standalone**: Skills form a default workflow chain but each works independently
- **Manual control**: The entry point is user-triggered, not auto-injected via hooks

## Skills

| Skill | Description |
|-------|-------------|
| `dev-flow` | Entry router — skill index, workflow orchestration, intent routing |
| `brainstorming` | Requirements → design → long-lived docs (Spec/PRD/ADR) |
| `planning` | Design doc → milestone-grouped task list |
| `workflow-selector` | Analyze complexity → present TDD + Review combos |
| `subagent-execution` | Serial milestone execution with checkpoint reviews |
| `code-review` | Pure review — dispatch reviewer, produce structured report |
| `verification-gate` | Run commands → see output → then claim completion |
| `tdd` | Red-Green-Refactor cycle, invoked on-demand |

## Default Workflow

```
dev-flow → brainstorming → planning → workflow-selector → subagent-execution → code-review → verification-gate
```

Each step asks before proceeding to the next. All skills can also be invoked independently.


