# DevFlow Workflow Design

## 1. Motivation

DevFlow is a minimal skill chain for development work — built in
reaction to heavy, Superpowers-style bundles that re-teach what strong
models already know. The authoring philosophy lives in `AGENTS.md`;
this document describes only how the chain is structured and how it runs.

## 2. System Design Principles

- **Self-driving chain** — each skill names its successor via a Next Step
  section; no external orchestrator.
- **Manual entry** — the chain is user-triggered (`using-devflow`), not
  auto-injected by hooks.
- **Stability from gates, not bulk** — checkpoints placed where the
  agent reliably slips (verify, review, debug, plan-confirm); minimal
  instruction elsewhere.
- **Unhobble over constrain** — prefer judgement Cues and interfaces
  over absolute style rules; avoid conflicting preferences across skills
  and with the user's project config. Authoring detail lives in `AGENTS.md`.
- **Loose coupling** — each skill owns its phase; one can be rewritten
  without re-architecting the chain.

## 3. Skills

| Skill | Role |
|-------|------|
| `using-devflow` | Entry router — index + intent routing |
| `brainstorming` | Requirements → design → long-lived docs |
| `planning` | Design → milestone-grouped task list + execution strategy |
| `executing-plans` | Serial milestone execution + checkpoint reviews |
| `finishing-work` | Integration decision: merge / PR / keep / discard |
| `reviewing-code` | Two-stage review (spec compliance → code quality) |
| `verifying-completion` | Run commands, see output, then claim done |
| `systematic-debugging` | Root cause before any fix |
| `test-driven-development` | Red-Green-Refactor, invoked on-demand |

## 4. Workflow

Main chain (each skill drives the next via its own Next Step):

```
using-devflow → brainstorming → planning → executing-plans → finishing-work
```

During `executing-plans`, these fire on schedule:

```
per task              → verifying-completion
[tdd: yes] tasks      → test-driven-development
per milestone         → reviewing-code (checkpoint)
all milestones done   → reviewing-code (final)
3 failed fix attempts → systematic-debugging
```

Standalone (on-demand, outside the chain):

- `systematic-debugging` — any failure, crash, or unexpected behavior
- `reviewing-code` — manual `/review` anytime

## 5. Gates

| Gate | When | Purpose |
|------|------|---------|
| Plan confirm | After planning, before execution | Catch design gaps while cheap |
| Per-task verify | After each implementer task | No "done" without fresh evidence |
| Checkpoint review | After each milestone | Spec compliance + code quality |
| Final review | After all milestones | Global spec + quality pass |
| Debug escalation | After 3 failed fixes | Force root cause before more retries |
