# dev-skills

A tightly-coupled set of structured workflow skills for AI coding assistants — designed for development but applicable to any task requiring analysis, planning, and systematic execution. Inspired by [Superpowers](https://github.com/obra/superpowers).

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
| `executing-plans` | Serial milestone execution with checkpoint reviews |
| `finishing-work` | Integration decision: merge, PR, keep, or discard |
| `reviewing-code` | Dispatch reviewer → structured report + handle feedback |
| `verifying-completion` | Run commands → see output → then claim completion |
| `systematic-debugging` | Root cause investigation → hypothesis → minimal fix |
| `test-driven-development` | Red-Green-Refactor cycle, invoked on-demand by executing-plans |

## Default Workflow

```
using-devflow → brainstorming → planning → executing-plans → finishing-work
                                               (per-task verifying-completion,
                                                per-milestone reviewing-code,
                                                final global reviewing-code)
```

using-devflow auto-drives the chain without per-step confirmations. `systematic-debugging` and `reviewing-code` can also be invoked independently.

