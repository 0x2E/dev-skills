# 002 - Skill Authoring Best Practices

Synthesized from [Claude agent-skills best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices) and [agentskills.io best practices](https://agentskills.io/skill-creation/best-practices), adapted for this project's workflow-oriented skills.

## Core Principles

- **Concise is key.** Only add context the agent doesn't already have. Run the **no-op test sentence by sentence, in isolation**: for each sentence ask "If I delete this, does the agent's behavior change?" If no → delete the **whole sentence**, don't rewrite it. Most prose that fails should go, not be trimmed.
- **Match specificity to fragility.** Bridge with cliffs = prescriptive (exact commands, no variation). Open field = flexible (general guidance, trust judgment).
- **Provide defaults, not menus.** Pick the recommended approach. Mention alternatives briefly as escape hatches — don't list them as equal options.
- **Favor procedures over declarations.** Teach *how to approach* a class of problems, not *what to produce* for a single instance.

## Naming Conventions

- Use **gerund form** (verb + -ing): `reviewing-code`, `executing-subagents`, `verifying-work`, `test-driving-development`.
- Consistency across the skill collection makes discovery and reference easier. Prefer full descriptive names (e.g., `test-driven-development` over `tdd`) to improve automatic triggering.

## Invocation Mode

Every skill sits in one of two invocation modes. Decide **before** writing the description — the mode dictates how the description is written.

### The two modes

| Mode | Who can trigger | Resting load |
|---|---|---|
| **Model-invoked** | The agent fires it autonomously when the task fits; users and other skills can also call it by name | **Context load** — the `description` occupies the agent's window every turn |
| **User-invoked** | Intended only for a human typing the skill name at a deliberate moment; the agent should *not* reach for it on its own | **Cognitive load** — the human must remember the skill exists |

No free lunch: a skill either spends the agent's context (always loaded) or the human's memory (must be recalled). Naming both costs lets you choose deliberately.

### How to decide

Ask, in order:

1. **Should the agent fire this on its own when the task fits?** Yes → model-invoked.
2. **Does any other skill need to reach this?** Yes → model-invoked (being reachable counts).
3. Both no → user-invoked.

### How the description differs

- **Model-invoked**: the description is the agent's only hook. Front-load the key terms and **list trigger branches** so it fires reliably ("Use when the user wants…, mentions…, asks for…").
- **User-invoked**: strip trigger lists — keep a one-line human-facing summary. No reason to pay wording that baits an invocation you don't want.

### Caveat — "user-invoked" is a writing convention, not a runtime switch

No mainstream runtime honors a `disable-model-invocation` (or similar) frontmatter flag — **unknown frontmatter fields are silently ignored** (confirmed in opencode; Claude Code lists only `name`/`description`). Writing such a field is a lie that misleads future authors. Real enforcement is one of:

- **Write the description un-baited** (no trigger phrases) — the agent is less likely to auto-fire it. This is the primary lever, and it is *soft*: the skill still appears in the agent's available-skills list.
- **In opencode, gate it via `opencode.json`**: `"permission": { "skill": { "<name>": "ask" } }` forces user approval before the agent loads it — the closest thing to a true switch. (`deny` hides it entirely, but blocks manual calls too.)

> There is **no** mechanism that perfectly means "only the human can start this, the agent never will." `ask` gives the human a veto; an un-baited description makes auto-firing unlikely but not impossible. Treat "user-invoked" as a goal you steer toward, not a binary you flip.

### Worked example (this project)

| Skill | Mode | Why |
|---|---|---|
| `verifying-completion` | model-invoked | Agent should recall "verify before claiming done" on its own, every time |
| `systematic-debugging` | model-invoked | Any failure/crash should auto-trigger it; other skills reference it |
| `test-driven-development` | model-invoked | Invoked internally by `executing-plans` — must be reachable |
| `using-devflow` | user-invoked | Entry router — only the human should start the chain; description kept un-baited |

## When to Split a Skill

**Granularity** — how finely to divide skills — costs one of the two resting loads on every cut, so split only when the cut earns it. Two cuts:

| Cut | Split when… | Cost paid |
|---|---|---|
| **By invocation** | The skill has a distinct leading word that should trigger on its own, **or** another skill must reach it | **Context load** — the new skill's `description` lives in the window every turn |
| **By sequence** | Later steps tempt the agent to rush the current one ([premature completion](#completion-criteria)) | Later steps are hidden, so the agent can't fixate on them |

Decision checklist:

1. Should the agent fire this on its own, or must another skill reach it? → consider **by invocation** (weigh whether independent reach is worth the context load).
2. Does the agent keep rushing ahead to a later step? → consider **by sequence** (hide the tail).
3. Neither → **don't split**.

## Writing Descriptions

The `description` field is critical — it's how the agent decides whether to trigger the skill.

- **Include both what and when.** Describe what the skill does *and* the specific triggers/contexts for activation.
- **Use key terms liberally.** Think about what words will appear in user requests that should route to this skill.

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

### Co-location

Keep a concept's definition, rules, and caveats **under one heading** rather than scattered. Reading one part should bring its neighbours.

### The branch test for disclosure

When a skill serves multiple use cases (**branches**), use them as the cleanest disclosure test:

- **Every branch needs it** → inline in `SKILL.md`.
- **Only some branches need it** → push it behind a context pointer.

Inline what all paths share; disclose what only some paths reach.

## Patterns to Use

### Match the Form to the Failure

Before reaching for a pattern, match it to *why* the agent fails. The wrong form can make things worse.

| Failure type | Right form | Why |
|--------------|-----------|-----|
| Discipline slip — knows better but slips | Flat rule / Red Flags ("don't X", "if you think Y, stop") | Names the lapse; the agent can self-correct |
| Output shape is wrong — structure, detail level, or style | Worked example (input→output pair) | The agent pattern-matches a concrete model, not an abstraction |
| Missing environment-specific fact | Gotcha | The agent can't discover this alone |
| Wrong order of operations | Checklist / Workflow | Sequence is the whole point |

A flat "don't do X" backfires on shape problems: it says what to avoid but not what to aim at, so the next attempt drifts differently. When the problem is *how the output should look*, show one.

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

### Completion Criteria

Every step ends on a **completion criterion** — a checkable condition that tells the agent the step is genuinely done. Without one, the agent drifts into **premature completion**: declaring a step finished because attention has already slid to the next one.

A good criterion is:

- **Checkable** — the agent can tell done from not-done.
- **Exhaustive** — use universal quantifiers ("*every* modified model accounted for", not "produce a change list") to close the loophole that lets one item slip.

| Vague (the agent will hand-wave) | Checkable + exhaustive |
|---|---|
| "List the changed files" | "*Every* changed model appears in the change list" |
| "Write a test" | "A test that fails because of this bug now passes" |
| "Review the code" | "*Every* changed function passes lint" |

Defence against premature completion, in order: sharpen the criterion first (cheap, local); only if it is irreducibly fuzzy *and* you observe the rush, split the sequence (see [When to Split a Skill](#when-to-split-a-skill)).

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

## Failure Modes

Use these named symptoms to diagnose what's wrong with a skill. Each maps to a fix elsewhere in this doc.

| Symptom | What it looks like | Fix |
|---|---|---|
| **Premature completion** | Agent declares a step done while attention has slid to the next one | Sharpen the [completion criterion](#completion-criteria); if irreducibly fuzzy, [split the sequence](#when-to-split-a-skill) |
| **Duplication** | The same meaning in more than one place — costs maintenance and tokens | Collapse into a single source of truth |
| **Sediment** | Stale layers that settle because adding feels safe and removing feels risky | Run the [no-op test](#core-principles) sentence by sentence |
| **Sprawl** | Skill simply too long, even when every line is unique | Push reference down the [progressive disclosure](#progressive-disclosure) ladder; split by branch or sequence |
| **No-op** | A line the agent already obeys by default — paying load to say nothing | Delete the whole sentence |

> The default fate of any skill without a pruning discipline is sediment. Visit these symptoms on every edit.

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
