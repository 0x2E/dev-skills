# dev-skills

A lightweight, loosely-coupled, harness-agnostic set of development workflow skills for AI coding assistants. Inspired by [Superpowers](https://github.com/obra/superpowers).

## Philosophy

- **Concise**: Each skill is focused and minimal — no heavy checklists or verbose gates
- **Harness-agnostic**: Uses neutral terminology with a tool mapping glossary per skill
- **Orchestrated or standalone**: devflow drives the full chain reliably; skills also work independently
- **Manual control**: The entry point is user-triggered, not auto-injected via hooks

## Skills

| Skill | Description |
|-------|-------------|
| `devflow` | Entry router — skill index, workflow orchestration, intent routing |
| `brainstorming` | Requirements → design → long-lived docs (Spec/PRD/ADR) |
| `planning` | Design doc → milestone-grouped task list + execution strategy |
| `subagent-execution` | Serial milestone execution with checkpoint reviews |
| `code-review` | Pure review — dispatch reviewer, produce structured report |
| `verification-gate` | Run commands → see output → then claim completion |
| `tdd` | Red-Green-Refactor cycle, invoked on-demand |

## Default Workflow

```
devflow → brainstorming → planning → subagent-execution
                                              (per-task verification-gate,
                                               per-milestone code-review,
                                               final global code-review)
```

devflow auto-drives the chain without per-step confirmations. All skills can also be invoked independently.


