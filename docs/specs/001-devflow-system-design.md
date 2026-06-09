# Lightweight Development Skills System Design

## 1. Background & Motivation

Superpowers is a comprehensive AI Coding workflow Skills collection, but it has the following issues:

- **Over-engineering**: Skill files are verbose, with excessive checklists and gates, lacking flexibility
- **Harness coupling**: Tightly bound to specific tool names and call patterns, difficult to adapt across AI Coding platforms
- **Poor model adaptability**: Workflows are too heavy to keep pace with rapid model capability upgrades

**Goal**: Learn from Superpowers' core ideas and build a lighter, tightly-coupled software development Skills system.

## 2. Design Principles

1. **Tightly-coupled skills with self-driving chain** — Each skill explicitly names its successor (Next Step), forming an autonomous chain: brainstorming → planning → executing-plans → finishing-work → complete. No intermediate orchestrator describes the plan; each skill drives to the next automatically.
2. **Conciseness first** — Each SKILL.md stays at a reasonable length, retaining only essential logic
3. **Fully manual trigger** — The entry point Skill is not auto-injected via hooks; users trigger it on demand
4. **Docs on demand** — Long-lived docs (Spec/PRD/ADR) are saved only after user confirmation; no auto-generated temporary plans

## 3. Skills Inventory

| # | Skill | Filename | Role | Responsibility |
|---|-------|----------|------|----------------|
| 0 | Entry Router | `using-devflow/SKILL.md` | Entry | Skill index + workflow orchestration + routing |
| 1 | Brainstorming | `brainstorming/SKILL.md` | Core | Requirements discussion → design confirmation → long-lived docs |
| 2 | Planning | `planning/SKILL.md` | Core | Design doc → milestone-grouped task list + execution strategy |
| 3 | Executing Plans | `executing-plans/SKILL.md` | Core | Serial milestone execution with checkpoint reviews |
| 4 | Finishing Work | `finishing-work/SKILL.md` | Core | Integration decisions: merge, PR, keep, or discard |
| 5 | Reviewing Code | `reviewing-code/SKILL.md` | Core | Dispatch reviewer → structured report + handle feedback |
| 6 | Verifying Completion | `verifying-completion/SKILL.md` | Checkpoint | Run commands, see output, then claim completion |
| 7 | Systematic Debugging | `systematic-debugging/SKILL.md` | Standalone | Root cause investigation → hypothesis → minimal fix |
| 8 | TDD | `test-driven-development/SKILL.md` | Sub-flow | Red-Green-Refactor cycle, invoked on-demand by executing-plans |

## 4. Default Workflow Chain

```
User manually triggers using-devflow
    ↓
brainstorming → planning → executing-plans → finishing-work
    ↓                ↓                    ↓                    ↓
Discuss reqs /    Break into             Serial execution +   Verify tests →
design → produce  milestone groups       milestone            present merge/PR/
long-lived docs   + execution strategy    checkpoint review   keep/discard
(Spec/PRD/ADR)   (with deps, mode, TDD)                       options → cleanup
                                            ↓ (if TDD enabled)
                                          test-driven-development (red-green-refactor cycle)

Standalone skills (on-demand, not in chain):
  systematic-debugging   → triggered by: bug, test failure, unexpected behavior
```

Each Skill except standalone diagnostics drives the chain via its own Next Step section.

## 5. Detailed Skill Designs

### 5.1 Entry Router (using-devflow)

**Responsibility**: Skill index + routing + behavioral guardrails. Not an orchestrator — does not describe a multi-step plan.

**Content structure**:
- Skill inventory table (name + one-liner only, no detailed capabilities)
- Routing table — maps user intent to skill load action
- **Red Flags table** — blocks autonomous behaviors that waste tokens: exploring the project before loading a skill, spawning subagents to pre-gather context, reading files before checking which skill to load
- Rules — load the matching skill immediately, no pre-research

**Key difference from Superpowers**:
- Not auto-injected via hooks; fully manual trigger by the user
- Red Flags borrowed from using-superpowers, preventing the AI from filling gaps between skill invocations with autonomous research

---

### 5.2 Brainstorming (brainstorming)

**Responsibility**: Discuss requirements with user, form design, produce long-lived project docs (Spec / PRD / ADR).

**Flow**:
1. Understand **relevant** context — focus only on modules/files related to the feature being discussed, not the entire project
2. Ask questions one at a time — clarify requirements, constraints, success criteria
3. Propose 2-3 approaches with trade-offs and a recommendation
4. Design confirmation: present design to user section by section, be ready to revise
5. Self Review: check for placeholders, contradictions, scope creep, ambiguity — fix before presenting to user
6. Produce docs:
   - Ask for **storage path** and **naming convention**
   - If AGENTS.md doesn't define these, suggest writing them into AGENTS.md
   - Generate long-lived doc after user confirmation

**Key differences from Superpowers**:
- Visual Companion removed
- Docs not forced to a fixed path; saved on demand
- Doc type not limited to Spec; supports PRD / ADR
- Self Review retained; dual-round user review removed
- Context gathering is focused, not exhaustive — avoids waste when jumping into feature work

---

### 5.3 Planning (planning)

**Responsibility**: Transform design docs into a milestone-grouped, executable task list with execution strategy.

**Output structure example**:
```markdown
### Execution Strategy
- Execution mode: Subagent / Main Session / Mixed
- TDD: Enabled for backend logic and utility tasks
- Review: Milestone checkpoint reviews + final global review

### Milestone 1: Data Layer
- Task 1.1: Create database schema [depends on: none] [mode: subagent] [tdd: yes]
- Task 1.2: Implement User model [depends on: Task 1.1] [mode: subagent] [tdd: yes]
- Task 1.3: Implement data access layer [depends on: Task 1.2] [mode: subagent] [tdd: yes]

### Milestone 2: API Layer
- Task 2.1: Implement REST routes [depends on: Milestone 1] [mode: subagent] [tdd: yes]
- Task 2.2: Implement auth middleware [depends on: Task 2.1] [mode: subagent] [tdd: yes]

### Milestone 3: Frontend
- Task 3.1: Implement login page [depends on: Milestone 2] [mode: subagent] [tdd: no]
- Task 3.2: Implement Dashboard [depends on: Task 3.1] [mode: subagent] [tdd: no]
```

**Content structure**:
1. Read input: consume design docs from brainstorming
2. Decompose modules, analyze dependencies, annotate milestones
3. Determine execution strategy: classify each task (TDD applicability, subagent vs main session mode)
4. Present proposed strategy to user for confirmation
5. Output structured task list with execution strategy

**Execution mode guidance**:
- **Subagent**: Tasks that involve multiple files, require TDD, or benefit from isolated context
- **Main Session**: Simple tasks (config changes, small fixes, trivial updates) that are faster to do directly

---

### 5.4 Finishing Work (finishing-work)

**Responsibility**: Guide completion of development work after all tasks are done — verify tests, present structured integration options, and execute the chosen workflow.

**Flow**:
1. Verify tests pass
2. Detect environment (normal repo, worktree, detached HEAD)
3. Determine base branch
4. Present 4 options: Merge locally / Push and create PR / Keep as-is / Discard
5. Execute chosen option (verify tests on merge, proper cleanup)
6. Clean up worktree if applicable (only for merge and discard)

**Key differences from Superpowers**:
- Simpler cleanup logic (does not require provenance tracking since user manages worktree creation manually)

---

### 5.5 Executing Plans (executing-plans)

**Responsibility**: Execute tasks serially per milestone, with checkpoint reviews between milestones.

**Core assumption**: By the time execution begins, brainstorming and planning have thoroughly resolved all design decisions and ambiguities. Execution proceeds autonomously without requiring human intervention.

**Scheduling rules**:
```
Read planning output (includes execution strategy)
    ↓
For each Milestone:
  ├─ For each Task within Milestone (serial):
  │    ├─ Check execution mode (subagent vs main session)
  │    ├─ If TDD enabled → invoke test-driven-development skill (red-green-refactor)
  │    ├─ If TDD not enabled → implement directly
  │    ├─ After implementation → verifying-completion (run tests/build)
  │    ├─ If failed → fix → retry
  │    ├─ If still failing after 3 fix attempts → invoke systematic-debugging (do not retry without root cause analysis)
  │    └─ Move to next Task
  │
  ├─ Milestone fully complete → invoke reviewing-code (checkpoint, two-stage: spec compliance → code quality)
  │    ├─ Stage 1 (spec) passes → proceed to Stage 2 (code quality)
  │    ├─ Stage 1 fails → implementer fixes → re-review Stage 1 (max one round)
  │    ├─ Stage 2 passes → next Milestone
  │    └─ Stage 2 has issues → dispatch implementer to fix → re-review Stage 2 (max one round)
  │         └─ Still failing → fix what can be fixed, note remaining issues, proceed
  │
  └─ All Milestones complete → invoke reviewing-code (final global review, two-stage)
        ├─ Stage 1 (spec) passes → Stage 2 (code quality)
        ├─ Stage 2 passes → proceed to final verification
        └─ Either stage has issues → fix loop (max one re-review per stage)

Final Verification:
  ├─ Run full verification suite (tests, lint, build)
  ├─ All pass → implementation complete, report to user
  ├─ Any fail → dispatch fix subagent → re-verify (max 3 fix cycles)
  └─ Still failing after 3 cycles → invoke systematic-debugging, re-dispatch with root cause findings
```

**Next Step**: After final verification passes and completion is reported to the user, invoke the `finishing-work` skill to handle the integration decision.

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

## TDD Instructions (if enabled)
{Include test-driven-development skill instructions}

## Output Requirements
- Complete implementation and run verification
- Return: change summary + issues encountered + verification result
```

**Key design decisions**:
- **All serial**: No parallel implementer dispatch to avoid scheduling chaos
- **Review fix loop max one round**: fix → re-review once → still failing? fix what can be fixed, note remainder, proceed
- **TDD and execution mode from planning**: Planning determines which tasks use TDD and which execution mode
- **Autonomous execution**: No human intervention during execution — all decisions resolved in brainstorming and planning
- **Systematic debugging on persistent failure**: After 3 verification retries fail, invokes systematic-debugging instead of retrying blindly

---

### 5.6 Reviewing Code (reviewing-code)

**Responsibility**: Two-stage review — spec compliance first, then code quality. Also handles receiving review feedback. Does NOT fix code (fixing is delegated back to executing-plans).

**Trigger timing**:
1. **Checkpoint Review** — after each Milestone completes, invoked by executing-plans
2. **Final Global Review** — after all Milestones complete, invoked by executing-plans
3. **User manual trigger** — anytime via slash command or direct instruction

**Two-stage process (strict order)**:
1. **Stage 1 — Spec Compliance**: Does the implementation match the specification? Checks for missing features, extra features, and incorrect behavior. Binary: pass or fail. Do NOT proceed to Stage 2 until Stage 1 passes.
2. **Stage 2 — Code Quality**: Is the code well-built? Checks for bugs, code structure, error handling, edge cases. Output: Strengths + Issues (Critical / Important / Minor) + Assessment.

Each stage dispatches a separate reviewer subagent with a focused prompt template.

**Review loop**: If either stage finds issues → implementer fixes → same stage re-reviewed (max one round per stage). If issues persist after re-review, fix what can be fixed, note remainder, and proceed.

**Feedback handling**:
1. Understand feedback fully before acting — ask if anything is unclear
2. Verify against the actual codebase — is it technically correct? Does it break existing functionality?
3. Evaluate — does it violate architectural decisions? Is it YAGNI?
4. Act — implement one item at a time (test each), or push back with technical reasoning if wrong
5. No performative agreement — state what you're fixing, or state why you disagree

**Key differences from Superpowers**:
- Feedback handling merged into this skill
- Two-stage review follows the same pattern as Superpowers (spec compliance → code quality)

---

### 5.7 Verifying Completion (verifying-completion)

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

### 5.8 Systematic Debugging (systematic-debugging)

**Responsibility**: Guide through systematic debugging when encountering bugs, test failures, or unexpected behavior. Enforces root cause investigation before any fix attempt.

**Four phases**:
1. **Root Cause Investigation** — read errors, reproduce, check recent changes, trace data flow
2. **Pattern Analysis** — find working examples, compare, identify differences
3. **Hypothesis and Testing** — form single hypothesis, test minimally, verify
4. **Implementation** — create failing test, single fix, verify. 3+ failed fixes → question architecture.

**Iron Law**: No fixes without root cause investigation first.

**Key differences from Superpowers**:
- Simplified to core 4-phase structure (Superpowers has separate sub-documents for root-cause-tracing, defense-in-depth, condition-based-waiting)
- No "your human partner's signals" section (covered by Red Flags)
- No "Real-World Impact" section
- Concise Red Flags bullet list instead of full rationalization table

---

### 5.9 TDD (test-driven-development)

**Responsibility**: Guide implementer subagents through the Red-Green-Refactor cycle.

**Positioning**: A sub-flow skill invoked on-demand by executing-plans. Does not independently chain into the main workflow. Only activated when planning enables TDD for a task. TDD applicability (which tasks use TDD) is determined solely by planning — this skill only covers how to do TDD.

**Core rule**: No production code without a failing test first.

**Flow**: Red-Green-Refactor cycle with test writing principles, anti-patterns, and troubleshooting guidance.

**Key differences from Superpowers**:
- Not mandatory for all tasks; enabled by planning on demand
- No frontend page component tests required
- Less dogmatic tone — practical guidance over rigid rules

---

## 6. File Structure

```
skills/
├── using-devflow/SKILL.md
├── brainstorming/SKILL.md
├── planning/SKILL.md
├── executing-plans/SKILL.md
├── finishing-work/SKILL.md
├── reviewing-code/SKILL.md
├── verifying-completion/SKILL.md
├── systematic-debugging/SKILL.md
├── test-driven-development/SKILL.md
docs/
│   └── specs/
│       └── 001-devflow-system-design.md
README.md
LICENSE
```
