# Lightweight Development Skills System Design

## 1. Background & Motivation

Superpowers is a comprehensive AI Coding workflow Skills collection, but it has the following issues:

- **Over-engineering**: Skill files are verbose, with excessive checklists and gates, lacking flexibility
- **Harness coupling**: Tightly bound to specific tool names and call patterns, difficult to adapt across AI Coding platforms
- **Poor model adaptability**: Workflows are too heavy to keep pace with rapid model capability upgrades

**Goal**: Learn from Superpowers' core ideas and build a lightweight, loosely-coupled, harness-agnostic software development Skills system.

## 2. Design Principles

1. **Neutral terminology** — Use generic terms in skill bodies, avoiding harness-specific tool names. Each Skill includes a "Tool Mapping" glossary section at the end
2. **Loose coupling + chainable** — Each Skill can be invoked independently or chained through the entry point Skill into a full workflow
3. **Conciseness first** — Each SKILL.md stays at a reasonable length, retaining only essential logic
4. **Fully manual trigger** — The entry point Skill is not auto-injected via hooks; users trigger it on demand
5. **Docs on demand** — Long-lived docs (Spec/PRD/ADR) are saved only after user confirmation; no auto-generated temporary plans

## 3. Skills Inventory

| # | Skill | Filename | Role | Responsibility |
|---|-------|----------|------|----------------|
| 0 | Entry Router | `dev-flow/SKILL.md` | Entry | Skill index + workflow orchestration + routing |
| 1 | Brainstorming | `brainstorming/SKILL.md` | Core | Requirements discussion → design confirmation → long-lived docs |
| 2 | Planning | `planning/SKILL.md` | Core | Design doc → milestone-grouped task list |
| 3 | Workflow Selector | `workflow-selector/SKILL.md` | Core | Analyze complexity, present TDD+Review combos for user selection |
| 4 | Subagent Execution | `subagent-execution/SKILL.md` | Core | Serial milestone execution with checkpoint reviews |
| 5 | Code Review | `code-review/SKILL.md` | Core | Pure review (output report, no fixing) |
| 6 | Verification Gate | `verification-gate/SKILL.md` | Checkpoint | Run commands, see output, then claim completion |
| 7 | TDD | `tdd/SKILL.md` | Sub-flow | Red-Green-Refactor cycle, invoked on-demand by subagent-execution |

## 4. Default Workflow Chain

```
User manually triggers dev-flow
    ↓
brainstorming → planning → workflow-selector → subagent-execution → code-review → verification-gate
    ↓                ↓                      ↓                    ↓                    ↓
Discuss reqs /    Break into             Present TDD+Review   Serial execution +    Final global review   Run commands
design → produce  milestone groups       combos for user      milestone             (pure review,          to verify
long-lived docs   (with deps)            selection             checkpoint review     no fixing)
(Spec/PRD/ADR)
                                              ↓ (if TDD enabled)
                                            tdd (red-green-refactor cycle)
```

Each Skill can be invoked independently via slash command or direct instruction, bypassing the entry point.

## 5. Detailed Skill Designs

### 5.1 Entry Router (dev-flow)

**Responsibility**: Skill index + workflow orchestration + routing.

**Content structure**:
- Skill inventory table (name + one-liner only, no detailed capabilities)
- Default workflow diagram (simplified)
- Independent invocation guide
- Tool mapping table

**Key difference from Superpowers**:
- Not auto-injected via hooks; fully manual trigger
- Routing only, never duplicates sub-skill content (avoids drift)

---

### 5.2 Brainstorming (brainstorming)

**Responsibility**: Discuss requirements with user, form design, produce long-lived project docs (Spec / PRD / ADR).

**Flow**:
1. Understand project context
2. Ask questions one at a time — clarify requirements, constraints, success criteria
3. Propose 2-3 approaches with trade-offs and a recommendation
4. Design confirmation: present design to user → Self Review (check placeholders/contradictions/scope/ambiguity) → user confirmation
5. Produce docs:
   - Ask for **storage path** and **naming convention**
   - If AGENTS.md doesn't define these, suggest writing them into AGENTS.md
   - Generate long-lived doc after user confirmation
6. Ask user whether to proceed to planning

**Key differences from Superpowers**:
- Visual Companion removed
- Docs not forced to a fixed path; saved on demand
- Doc type not limited to Spec; supports PRD / ADR
- Self Review retained; dual-round user review removed
- Does not force transition to planning (loose coupling)

---

### 5.3 Planning (planning)

**Responsibility**: Transform design docs into a milestone-grouped, executable task list.

**Output structure example**:
```markdown
### Milestone 1: Data Layer
- Task 1.1: Create database schema [depends on: none]
- Task 1.2: Implement User model [depends on: Task 1.1]
- Task 1.3: Implement data access layer [depends on: Task 1.2]

### Milestone 2: API Layer
- Task 2.1: Implement REST routes [depends on: Milestone 1]
- Task 2.2: Implement auth middleware [depends on: Task 2.1]

### Milestone 3: Frontend
- Task 3.1: Implement login page [depends on: Milestone 2]
- Task 3.2: Implement Dashboard [depends on: Task 3.1]
```

**Content structure**:
1. Read input: consume design docs from brainstorming
2. Decompose modules, analyze dependencies, annotate milestones
3. Output structured task list
4. Ask user whether to proceed to workflow-selector
5. Tool mapping

---

### 5.4 Workflow Selector (workflow-selector)

**Responsibility**: Analyze task complexity and present TDD + Review workflow combination options to the user.

**Flow**:
```
Read planning output (milestone task list)
    ↓
Analyze: backend logic? frontend UI? refactoring? config changes?
    ↓
Generate 2-3 workflow combos with a recommendation:
  A. Full TDD + Subagent execution + Milestone Review + Final Review
  B. Backend TDD + frontend core-methods TDD + Subagent execution + Milestone Review + Final Review
  C. No TDD + Subagent execution + Milestone Review + Final Review
    ↓
User selects or manually specifies combo
```

**TDD applicability reference**:
- Backend logic / utility functions / data processing → recommend TDD
- Frontend page components / styles / layouts → recommend skip TDD
- Config changes / dependency upgrades → no TDD needed

---

### 5.5 Subagent Execution (subagent-execution)

**Responsibility**: Execute tasks serially per milestone, with checkpoint reviews between milestones.

**Scheduling rules**:
```
Read planning output + workflow-selector conclusion
    ↓
For each Milestone:
  ├─ For each Task within Milestone (serial):
  │    ├─ If TDD enabled → invoke tdd skill (red-green-refactor)
  │    ├─ If TDD not enabled → implement directly
  │    ├─ After implementation → verification-gate (run tests/build)
  │    ├─ If failed → fix → retry
  │    └─ Move to next Task
  │
  ├─ Milestone fully complete → invoke code-review (checkpoint)
  │    ├─ Review passed → next Milestone
  │    └─ Review has issues → dispatch implementer to fix → re-review (max one round)
  │         └─ Still failing → flag issues for human decision
  │
  └─ All Milestones complete → invoke code-review (final global review)
```

**Implementer subagent prompt template (concise)**:
```markdown
## Task
{task description}

## Context
{relevant design/spec summary}
{relevant file paths}

## Acceptance Criteria
- {criterion 1}
- {criterion 2}

## Output Requirements
- Complete implementation and run verification
- Return: change summary + issues encountered + verification result
```

**Key design decisions**:
- **All serial**: No parallel implementer dispatch to avoid scheduling chaos
- **Review fix loop max one round**: fix → re-review once → still failing? flag for human
- **TDD optional**: Enabled/disabled based on workflow-selector result

---

### 5.6 Code Review (code-review)

**Responsibility**: Pure review — dispatch a reviewer subagent to produce a structured review report. Does NOT fix code (fixing is delegated back to subagent-execution).

**Trigger timing**:
1. **Checkpoint Review** — after each Milestone completes, invoked by subagent-execution
2. **Final Global Review** — after all Milestones complete, invoked by subagent-execution
3. **User manual trigger** — anytime via slash command or direct instruction

**Reviewer subagent prompt template (concise)**:
```markdown
## Review Scope
{specific files/directories}

## Original Requirements
{design doc summary}

## Key Acceptance Points
- {key acceptance point}

## Output
- Strengths: what was done well
- Issues:
  - Critical: blocking issues
  - Important: should fix before merge
  - Minor: suggestions, non-blocking
- Assessment: Ready / Needs Fix
```

**Feedback handling principles**:
1. Verify feedback is correct and applicable to the current project
2. Check if it violates existing architectural decisions or YAGNI
3. Correct & important → accept; uncertain → ask user; wrong/inapplicable → push back with reasoning

---

### 5.7 Verification Gate (verification-gate)

**Responsibility**: Before claiming "done", force-run verification commands and obtain passing evidence.

**Iron rule**: No completion claim without verification.

**Reference checklist**:

| Claim | Must Run |
|-------|----------|
| Tests pass | Run full test suite, 0 failures |
| Lint clean | Run lint command, 0 errors |
| Build succeeds | Run build command, exit code 0 |
| Bug fixed | Run original reproduction steps, confirm no longer occurs |

**Trigger timing**:
- After each implementer subagent completes a task, before reporting back
- Final check before merge
- User manual trigger

---

### 5.8 TDD (tdd)

**Responsibility**: Guide implementer subagents through the Red-Green-Refactor cycle.

**Positioning**: A sub-flow skill invoked on-demand by subagent-execution. Does not independently chain into the main workflow. Only activated when workflow-selector enables TDD.

**Flow**:
```
Red:     Write test first → run to confirm failure (expected failure proves test is valid)
Green:   Write minimal implementation to pass the test → run to confirm pass
Refactor: Refactor code to optimize design → run to confirm still passes
```

**Applicability reference**:
- Backend logic functions / data processing / API endpoints / utility methods → TDD applicable
- Frontend page components / styles / layouts / config changes → TDD not applicable
- Frontend core function methods (state management, data processing, utility functions) → TDD applicable

**Test writing principles**:
- Verify behavior, not implementation details
- One test verifies one thing
- Cover happy paths first, then edge cases and errors

**Key differences from Superpowers**:
- Not mandatory for all tasks; enabled by workflow-selector on demand
- No frontend page component tests required
- Simplified flow description, no heavy checklists

---

## 6. Tool Mapping

Each Skill includes a glossary section at the end, listing generic terms used in that Skill and their corresponding harness tool names. Individual harnesses map these to their own tool sets:

| Generic Term | Description |
|-------------|-------------|
| Dispatch subagent | Create a subagent with isolated context to execute a specific task |
| Task list | Tool for managing structured to-do items |
| Read file | Tool for reading file contents |
| Search code | Tool for pattern-matching file names or content |
| Run command | Tool for executing terminal commands |
| Edit file | Tool for modifying file contents |

---

## 7. File Structure

```
skills/
├── dev-flow/SKILL.md
├── brainstorming/SKILL.md
├── planning/SKILL.md
├── workflow-selector/SKILL.md
├── subagent-execution/SKILL.md
├── code-review/SKILL.md
├── verification-gate/SKILL.md
├── tdd/SKILL.md
docs/
│   └── specs/
│       └── 001-lightweight-dev-skills-design.md
README.md
LICENSE
```
