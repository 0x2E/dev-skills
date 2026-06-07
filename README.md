# dev-skills

A tightly-coupled set of development workflow skills for AI coding assistants. Inspired by [Superpowers](https://github.com/obra/superpowers).

## Philosophy

- **Concise**: Each skill is focused and minimal — no heavy checklists or verbose gates
- **Self-driving chain**: Skills form an autonomous chain; each skill drives to the next without an orchestrator
- **Manual control**: The entry point is user-triggered, not auto-injected via hooks

## Skills

| Skill | Description |
|-------|-------------|
| `using-devflow` | Entry router — skill index, workflow orchestration, intent routing |
| `brainstorming` | Requirements → design → long-lived docs (Spec/PRD/ADR) |
| `planning` | Design doc → milestone-grouped task list + execution strategy (TDD decisions) |
| `subagent-execution` | Serial milestone execution with checkpoint reviews |
| `finishing-work` | Test verification → merge/PR/keep/discard options → cleanup |
| `code-review` | Dispatch reviewer → structured report + handle feedback |
| `verification-gate` | Run commands → see output → then claim completion |
| `systematic-debugging` | Root cause investigation → hypothesis → minimal fix |
| `tdd` | Red-Green-Refactor cycle, invoked on-demand by subagent-execution |

## Default Workflow

```
using-devflow → brainstorming → planning → subagent-execution → finishing-work
                                              (per-task verification-gate,
                                               per-milestone code-review,
                                               final global code-review)
```

using-devflow auto-drives the chain without per-step confirmations. `systematic-debugging` and `code-review` can also be invoked independently.

