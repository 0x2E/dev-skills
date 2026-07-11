---
name: simplifying-architecture
description: "Use when code or design is over-engineered, overly complex, or has unnecessary abstraction layers — scans for over-engineering patterns, validates each via a deletion test, and presents findings. Identifies only; does not implement changes"
---

# Simplifying Architecture

Identify unnecessary architectural complexity. **This skill only identifies — it does not implement changes.**

## Scope

| This skill handles | Not this skill's job |
|---|---|
| Unnecessary structural / architectural complexity | Code style cleanup (→ reviewing-code) |
| Over-abstraction, redundant layers, speculative flexibility | New architecture design (→ brainstorming) |
| Reducing complexity | Restructuring without simplifying (refactoring ≠ simplification) |

## Four Phases

Complete each phase before proceeding to the next.

### Phase 1: Determine Input

| Trigger | Input |
|---|---|
| Standalone on code | Source files / directories specified by user |
| Standalone on design | Design doc / spec |
| Design checkpoint | Design documentation output |
| Implementation checkpoint | Implementation diff |

If input is unclear, ask the user what to review.

### Phase 2: Scan

Run **both** methods. Neither alone is sufficient.

**Pattern checklist** — check for each:

- **Premature abstraction** — abstraction created before Rule of Three is met
- **Speculative generality** — flexibility built for hypothetical future needs
- **Pass-through layers** — modules / functions that only forward calls
- **Over-configuration** — values externalized as config that never change
- **Shallow module** — interface complexity ≈ implementation complexity
- **Unused extensibility** — parameters, hooks, options with no consumer
- **Framework-for-one** — general framework built for a single use case

**Friction exploration** — walk the code / design as a new reader. Note where:

- Understanding one concept requires bouncing between 3+ files
- A simple change touches many layers
- The abstraction doesn't match the problem's mental model

This catches over-engineering that doesn't map to a named pattern.

### Phase 3: Deletion Test (Gate)

For each suspected finding from Phase 2, apply this test before flagging:

> Imagine deleting this abstraction entirely — inlining its logic into its callers. Do the callers become harder to understand?
> - **Yes** → the abstraction is hiding complexity. **Keep it.** Do not flag.
> - **No / simpler** → the abstraction is unnecessary. **Flag it.**

**Do not flag any finding that fails this test.** This prevents harmful simplifications.

If everything passes the deletion test: report the architecture is appropriately complex. **Do not manufacture findings.**

### Phase 4: Present Findings

Present each validated finding with:

| Field | Requirement |
|---|---|
| **Location** | `file:line` or design section |
| **Pattern** | which of the seven, or "friction-based" |
| **Evidence** | concrete data — not "looks complex" |
| **Deletion test result** | confirm: inlining makes callers simpler |
| **Proposed simplification** | what removing it looks like |
| **Severity** | High / Medium / Low |

Wait for the user to select which findings to act on.

## Next Step

- **User selects findings to act on** → invoke `planning` immediately with the selected findings.
- **No findings, or user declines** → skill complete.
