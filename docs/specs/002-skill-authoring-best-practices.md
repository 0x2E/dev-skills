# 002 - Skill Authoring Best Practices

Synthesized from [Claude agent-skills best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices) and [agentskills.io best practices](https://agentskills.io/skill-creation/best-practices), adapted for this project's workflow-oriented skills.

## Core Principles

- **Concise is key.** Only add context the agent doesn't already have. Challenge each sentence: "Would the agent get this wrong without this instruction?" If no, cut it.
- **Match specificity to fragility.** Bridge with cliffs = prescriptive (exact commands, no variation). Open field = flexible (general guidance, trust judgment).
- **Provide defaults, not menus.** Pick the recommended approach. Mention alternatives briefly as escape hatches — don't list them as equal options.
- **Favor procedures over declarations.** Teach *how to approach* a class of problems, not *what to produce* for a single instance.

## Naming Conventions

- Use **gerund form** (verb + -ing): `reviewing-code`, `executing-subagents`, `verifying-work`, `test-driving-development`.
- Consistency across the skill collection makes discovery and reference easier. Prefer full descriptive names (e.g., `test-driven-development` over `tdd`) to improve automatic triggering.

## Writing Descriptions

The `description` field is critical — it's how the agent decides whether to trigger the skill.

- **Always use third person.** "Facilitates requirements exploration…" not "I can help you…" or "You can use this…"
- **Include both what and when.** Describe what the skill does *and* the specific triggers/contexts for activation.
- **Use key terms liberally.** Think about what words will appear in user requests that should route to this skill.
- **For internally-invoked skills** (not triggered by users directly), state who invokes them and under what conditions.

**Bad:**
```yaml
description: Use when writing code with test-driven development
```
**Good:**
```yaml
description: Enforces Red-Green-Refactor cycle for implementation tasks. Invoked internally by executing-plans on tasks marked [tdd: yes]. No production code without a failing test.
```

## Progressive Disclosure

- Keep `SKILL.md` body under 500 lines and ~5000 tokens. Move detailed reference material to separate files.
- **Reference files must be one level deep from SKILL.md.** Never chain references (A → B → C). All files should link directly from SKILL.md.
- For reference files longer than 100 lines, include a table of contents at the top.
- Use descriptive filenames: `reference/schema.md`, `examples/review-output.md`, `templates/implementer-prompt.md` — not `docs/file1.md`.
- Tell the agent *when* to load each file: "Read `reference/api-errors.md` if the API returns a non-200 status" beats "see references/ for details."

## Patterns to Use

### Gotchas Section (highest-value content)

List environment-specific facts that defy reasonable assumptions — corrections the agent won't discover on its own:

```markdown
## Gotchas

- Subagents have NO access to conversation history. Include ALL context in the task prompt.
- `gh pr create` fails silently if GitHub CLI isn't authenticated. Run `gh auth status` first.
- The `/health` endpoint returns 200 even when the DB is down. Use `/ready` for full health.
```

When an agent makes a mistake you have to correct, add it to the gotchas section. This is the most direct way to improve a skill iteratively.

### Checklists for Multi-Step Workflows

```markdown
## Workflow

Progress:
- [ ] Step 1: Analyze the form (run `scripts/analyze_form.py`)
- [ ] Step 2: Create field mapping (edit `fields.json`)
- [ ] Step 3: Validate mapping (run `scripts/validate_fields.py`)
- [ ] Step 4: Apply changes (run `scripts/fill_form.py`)
- [ ] Step 5: Verify output (run `scripts/verify_output.py`)
```

### Templates for Output Format

Provide concrete templates. Agents pattern-match better against structures than prose descriptions:

```markdown
## Report Structure

Use this template, adapting sections as needed:

```markdown
# [Title]

## Summary
[One-paragraph overview]

## Findings
- Finding with supporting data

## Recommendations
1. Specific actionable recommendation
```
```

### Examples (Input/Output Pairs)

For skills where output quality depends on style and detail level, include concrete examples:

```markdown
## Commit Message Format

**Example:**
Input: Added user authentication with JWT tokens
Output:
```
feat(auth): implement JWT-based authentication

Add login endpoint and token validation middleware
```
```

### Validation Loops

Instruct the agent to validate before proceeding. Pattern: do → validate → fix → repeat until pass.

```markdown
## Workflow

1. Make your edits
2. Run validation: `python scripts/validate.py output/`
3. If validation fails: review errors → fix → re-run validation
4. Only proceed when validation passes
```

### Plan-Validate-Execute

For batch or destructive operations: create a plan in a structured format → validate against source of truth → execute only after validation passes.

### Feedback Loops

For quality-critical tasks: implementer works → reviewer checks → implementer fixes → re-review. Cap retry rounds (e.g., max 2 re-review rounds, then escalate).

### Iron Law / Red Flags

For behaviors that must never happen, use strong, unmissable language:

```markdown
## Iron Law

1. **No fixes without root cause investigation first.**
2. **No completion claims without fresh verification evidence.**

## Red Flags

If you catch yourself thinking any of these, stop:
- "Quick fix for now, investigate later"
- "It's probably X, let me fix that"
```

## Patterns to Avoid

- **Too many options.** Present a clear default; mention alternatives as fallbacks only.
- **Time-sensitive information.** Put deprecated/versioned content in collapsible "Old patterns" sections, not inline.
- **Inconsistent terminology.** Pick one term and use it throughout (e.g., always "endpoint", never mix "URL"/"route"/"path").
- **Deeply nested references.** All reference files must link directly from SKILL.md, never through intermediate files.
- **Explaining what the agent already knows.** Skip definitions of common concepts (PDFs, HTTP, databases).
- **Windows-style paths.** Always use forward slashes: `reference/guide.md` not `reference\guide.md`.

## Scripts and Executable Code

When bundling scripts:

- **Solve, don't punt.** Handle error conditions in scripts; don't leave the agent to figure out failures.
- **Document magic numbers.** Every constant needs a justification comment.
- **Make execution intent clear.** "Run `analyze.py` to extract fields" (execute) vs "See `analyze.py` for the algorithm" (read as reference).
- **Prefer scripts for deterministic operations.** More reliable than generated code and saves tokens.

## Testing and Iteration

- **Build evaluations first.** Create 3 scenarios per skill, document expected behavior, test before writing extensive instructions.
- **Observe real execution traces.** Watch what the agent actually does — not just final output. Note unproductive paths, missed connections, ignored content.
- **Iterate from observations.** When an agent struggles, return to the skill author with specifics: "The agent forgot to filter by date for Q4. Should we add a section about date filtering?"
- **Test with all target models.** A skill that works for Opus may need more detail for Haiku.
