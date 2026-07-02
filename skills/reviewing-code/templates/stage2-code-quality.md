# Stage 2 — Code Quality Reviewer Prompt

Dispatch this to a reviewer subagent **only after Stage 1 (spec compliance) passes**. This stage evaluates quality, not spec adherence.

Fill the placeholders, then send.

---

```markdown
## Review Scope
{specific files or directories to review}
{git SHAs showing the diff}

## Output

### Strengths
[What's well done? Be specific. Acknowledge strengths before issues — accurate praise builds trust in the rest of the feedback.]

### Issues

#### Critical (Must Fix)
[Bugs, security, data loss, broken functionality]

#### Important (Should Fix)
[Architecture problems, missing features, poor error handling, test gaps]

#### Minor (Nice to Have)
[Code style, optimization opportunities, documentation polish]

For each issue:
- File:line reference
- What's wrong
- Why it matters
- How to fix (if not obvious)

### Assessment

**Ready to merge: Yes | No | With fixes**

**Reasoning:** [1-2 sentence technical assessment]
```

Calibrate severity — not everything is Critical. Every concrete issue needs a `file:line` reference.
