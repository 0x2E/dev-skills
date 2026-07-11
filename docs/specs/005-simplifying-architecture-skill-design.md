# 005 - Simplifying Architecture Skill Design

## Background

dev-skills covers the full development workflow (brainstorming -> planning -> executing -> finishing) with quality gates (reviewing-code, systematic-debugging, verifying-completion). However, there is no dedicated capability for **identifying and removing unnecessary architectural complexity** — over-engineering.

brainstorming applies YAGNI during design, and reviewing-code's Stage 2 catches code quality issues including some architecture problems. But neither performs a **systematic scan for over-engineering patterns** with a **validation gate** to prevent harmful simplifications. When a user faces code or a design that feels over-engineered, there is no skill to call.

## Positioning

A **standalone + workflow-checkpoint** skill. Triggered three ways:

1. **Standalone** — user invokes directly ("simplify this", "is this over-engineered?")
2. **Design checkpoint** — brainstorming asks the user whether to run an over-engineering check after design documentation is produced
3. **Implementation checkpoint** — executing-plans asks the user whether to run an over-engineering check after the final review passes

The skill **only identifies** — it does not implement changes. After presenting findings, it chains to `planning` if the user selects findings to act on.

## Scope Boundary

| This skill handles | Not this skill's job |
|---|---|
| Unnecessary structural/architectural complexity | Code style cleanup (-> reviewing-code Stage 2) |
| Over-abstraction, redundant layers, speculative flexibility | New architecture design (-> brainstorming) |
| Reducing complexity | Restructuring without simplifying (refactoring != simplification) |

## Core Design: Pattern Checklist + Deletion Test

### Why not a theoretical framework

Matt Pocock's `improve-codebase-architecture` uses Ousterhout's "deep modules" theory. This requires Explanation content (defining the vocabulary), which violates this project's authoring rules — skills guide action, they don't re-teach concepts the model already knows. Instead, the skill uses a **pattern checklist as Cues** (naming patterns the model already knows, activating them as check requirements) and a **deletion test as a Gate** (a binary validation that prevents harmful simplifications).

### Pattern Checklist (Cues)

Seven over-engineering patterns, each named as a check requirement — not defined:

1. **Premature abstraction** — abstraction created before Rule of Three
2. **Speculative generality** — flexibility for hypothetical future needs
3. **Pass-through layers** — modules/functions that only forward calls
4. **Over-configuration** — values externalized as config that never change
5. **Shallow modules** — interface complexity approximately equals implementation complexity
6. **Unused extensibility** — parameters, hooks, options with no consumer
7. **Framework-for-one** — general framework built for a single use case

### Friction Exploration

Complements the checklist: walk the code/design as a new reader, noting where understanding one concept requires bouncing between 3+ files, where a simple change touches many layers, where abstraction doesn't match the problem's mental model. Catches over-engineering that doesn't map to a named pattern.

### Deletion Test (Gate)

The core validation that distinguishes this skill from unstructured "simplify this" requests. For each suspected finding:

> Imagine deleting this abstraction entirely — inlining its logic into its callers. Do the callers become harder to understand?
> - **Yes** — the abstraction is hiding complexity. Keep it. Do not flag.
> - **No / simpler** — the abstraction is unnecessary. Flag it.

If everything passes the deletion test: report the architecture is appropriately complex. **Do not manufacture findings.**

## Four-Phase Process

### Phase 1: Determine Input

| Trigger | Input |
|---|---|
| Standalone on code | Source files / directories specified by user |
| Standalone on design | Design doc / spec |
| Design checkpoint | Brainstorming output |
| Implementation checkpoint | Implementation diff |

### Phase 2: Scan

Both methods required:
- **Pattern checklist** — check for each of the seven patterns
- **Friction exploration** — walk as a new reader, note friction points

### Phase 3: Deletion Test (Gate)

Validate each suspected finding before flagging. Block flagging unless the deletion test confirms the abstraction is unnecessary.

### Phase 4: Present Findings

Each finding includes: location, pattern matched, evidence (concrete data, not "looks complex"), deletion test result, proposed simplification, severity (High / Medium / Low). Wait for user to select which to act on.

## Chaining

After Phase 4, if the user selected findings to act on, invoke `planning` immediately. If no findings or user declines: skill is complete.

## Workflow Integration

### using-devflow routing table

Add entry with triggers: "Simplify this", "Is this over-engineered?", "Too complex", "Reduce complexity", "Too many layers/abstractions".

### brainstorming checkpoint

After design documentation is produced (Step 6), before Next Step: automatically invokes `simplifying-architecture`. Not a vague "if the design is complex" condition, not an opt-in question — a fixed review step that always runs.

### executing-plans checkpoint

After final global review passes, before Final Verification: automatically invokes `simplifying-architecture`. A fixed review step that always runs — not an opt-in question.

### reviewing-code: no change

reviewing-code already handles code quality in its review. Over-engineering systematic analysis is simplifying-architecture's job. The checkpoint is placed in executing-plans (which controls the flow) rather than reviewing-code (which is invoked at both milestone and final points — running at every milestone would be too frequent).

## Deliberate Decisions

1. **Identification only, no implementation.** Phase 5 (Simplify) was removed to avoid overlap with planning/executing-plans. The skill presents findings and chains to planning. This keeps the skill focused and avoids duplicating the implementation workflow.
2. **Automatic checkpoints, not vague conditions or opt-in questions.** Original proposal had "if the design has significant complexity, optionally invoke..." — this is not a checkable gate (model judgment varies). Changed to: automatically invoke `simplifying-architecture` as a fixed review step. The user does not opt in; the check always runs. If no findings, the flow continues uninterrupted.
3. **Pattern checklist over theoretical framework.** Ousterhout's deep modules would require Explanation content. Pattern checklist as Cues activates existing knowledge without re-teaching. The deletion test provides the validation rigor that the framework would otherwise provide.
4. **Deletion test as the key differentiator.** Without it, the skill would just be "find things that look complex" — subjective and inconsistent. The deletion test turns simplification proposals into binary validated findings. It also prevents the skill's own failure mode: suggesting harmful simplifications that remove useful abstractions.
5. **Checkpoint in executing-plans, not reviewing-code.** reviewing-code fires at both milestone checkpoints and final review. Placing the over-engineering check there would run at every milestone — too frequent. executing-plans controls the flow and can place the check only after the final review.
6. **Do not manufacture findings.** If the deletion test passes for everything, the skill reports the architecture is appropriate. This prevents the skill from justifying its own existence by inventing problems.

## Files to Create / Modify

**Create:**
- `skills/simplifying-architecture/SKILL.md`

**Modify:**
- `skills/using-devflow/SKILL.md` — routing table entry
- `skills/brainstorming/SKILL.md` — design checkpoint (fixed ask)
- `skills/executing-plans/SKILL.md` — implementation checkpoint (fixed ask)
