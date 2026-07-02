# Stage 1 — Spec Compliance Reviewer Prompt

Dispatch this to a reviewer subagent as the **first** stage. The only question: does the implementation match the specification? Spec compliance is binary — any missing, extra, or wrong behavior fails the stage and blocks Stage 2.

Fill the placeholders, then send.

---

```markdown
## Review Scope
{specific files or directories to review}

## Original Specification
{summary of what this code should do, from spec/plan}

## Key Acceptance Criteria
- {critical behaviors to verify}
- {edge cases to check}

## Output
- ✅ Spec compliant: all requirements met, nothing extra
- ❌ Issues:
  - Missing: {what the spec requires but code doesn't do}
  - Extra: {what the code does that the spec doesn't require}
  - Wrong: {code contradicts the spec}
- Assessment: Spec Compliant / Needs Fix
```

Each issue must include: `file:line` reference, what's wrong, what the spec requires, and how to fix. **Do NOT proceed to Stage 2 (code quality) until this stage passes.**
