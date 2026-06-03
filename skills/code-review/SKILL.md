---
name: code-review
description: Use before merging or after milestones — dispatch a reviewer subagent for pure code review, producing a structured report. Does not fix code.
---

# Code Review

Dispatch a reviewer subagent to evaluate code and produce a structured review report. This skill focuses on review only — fixing is handled by subagent-execution.

## Triggers

1. **Checkpoint Review** — after each milestone completes (invoked by subagent-execution)
2. **Final Global Review** — after all milestones complete (invoked by subagent-execution)
3. **Manual trigger** — user invokes directly via `/review` or instruction

## Reviewer Subagent Prompt

Craft the reviewer prompt with these elements:

```markdown
## Review Scope
{specific files or directories to review}
{base SHA and head SHA if using git diff}

## Original Requirements
{summary of what this code should do, from spec/plan}

## Key Acceptance Points
- {critical behaviors to verify}
- {edge cases to check}

## Output Format
- Strengths: what was done well
- Issues:
  - Critical: blocking, must fix
  - Important: should fix before merge
  - Minor: suggestions, non-blocking
- Assessment: Ready / Needs Fix
```

## Feedback Handling

When receiving review feedback:

1. Verify: Is it technically correct for this codebase?
2. Evaluate: Does it violate architectural decisions? Is it YAGNI?
3. Act:
   - Correct and important → acknowledge and track for fixing
   - Uncertain → ask user
   - Wrong or inapplicable → push back with technical reasoning

## Review Loop

- Subagent-execution dispatches fixes → re-triggers review (max one re-review loop)
- Still failing after re-review → flag for human decision

## Tool Mapping

| Generic Term | Description |
|-------------|-------------|
| Dispatch subagent | Create a reviewer subagent with isolated context |
| Run command | Execute git diff, grep for relevant changes |
| Read file | Read source files for review |
| Search code | Search for patterns, usages, references |
