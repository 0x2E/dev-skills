# 004 - Curating Docs Skill Design

## Background

dev-skills covers the full development workflow but lacks a capability for **governing existing documentation assets**. Legacy projects accumulate doc chaos: root-level scattered `.md` files, multiple competing `docs/` directories, inconsistent naming, and stale/duplicate/orphan docs coexisting. This spec defines a manually-invoked skill that surveys the current state, recommends an organization convention, codifies it into the AI config file, then reorganizes structure and remediates content.

## Positioning

A standalone, **manually-triggered** skill. Both **structure** and **content** are first-class dimensions. It is NOT wired into the automated workflow (no `using-devflow` routing entry) — trigger keywords live in the skill's `description`.

## Two Core Dimensions

| Dimension | Core question | Diagnosis | Handling |
|---|---|---|---|
| **Structure** | Is the physical organization sound? | scattered files, multiple `docs/`, naming chaos, physical duplication | automated reorg (git safety net) |
| **Content** | Is the content healthy? | stale, duplicate, contradictory, orphaned, missing | per-item human confirmation |

## Five-Phase Flow

### Phase 1: Survey

One scan, two diagnoses.

**Structure signals:**
- Root-level scattered `.md` (excluding convention files: README/LICENSE/CHANGELOG/CONTRIBUTING)
- Multiple doc directories (`docs/`, `doc/`, `documentation/`, `wiki/`)
- Naming chaos (inconsistent case, no prefixes, mixed date/version schemes)
- Physical duplication (same/near-same filename in multiple locations)

**Content signals:**
- **Stale** — apparent from the doc's own content: broken internal cross-references, outdated version/date/deprecated-tool cues, or explicit stale markers (does not cross-check against source code)
- **Duplicate** — overlapping content across documents
- **Contradictory** — conflicting descriptions of the same subject
- **Orphaned** — no inbound links/references
- **Missing** — doc types the convention requires but the project lacks

For large projects, dispatch an `explore` subagent. Output: a survey report with two sections (structure + content).

### Phase 2: Recommend

Load the built-in defaults (see "Built-in Defaults"), identify the project's existing sensible structures as **keep-items**, merge, and present: directory structure + naming convention + doc-type taxonomy. User confirms/adjusts item by item.

### Phase 3: Codify

Detect the AI config file by priority (see "Config File Priority"), and add the confirmed documentation conventions to it. Write only after user confirmation.

### Phase 4: Reorganize (structure dimension)

- **Precondition:** git working tree must be clean; if not, abort and prompt the user (do NOT auto-stash)
- Generate a migration plan: old path → new path map + delete/merge list
- User confirms
- Dispatch an `implementer` subagent to perform moves/renames
- Update cross-references (relative links) between docs
- Verify: `git status` shows only intended changes + link check (no broken links) + no files lost
- Produce a change report

### Phase 5: Remediate (content dimension)

Act on the Phase 1 content diagnosis, confirming each handling action (update/merge/delete/write-missing) with the user item by item. **No batch auto-rewrite** — every content judgment needs user confirmation because content semantics cannot be mechanically verified.

## Built-in Defaults

Default directory/naming skeleton selected by project type, inlined in SKILL.md. Sketch (library / application):

- `README.md` (root, required)
- `docs/specs/` — `NNN-{desc}.md` (specifications)
- `docs/adrs/` — `NNN-{desc}.md` (architecture decisions)
- `docs/design/` (design docs)
- `docs/runbooks/` (ops/playbooks)
- `docs/api/` (API reference)
- `CHANGELOG.md`, `CONTRIBUTING.md` (root)

Survey phase infers project type and picks the matching default.

## Config File Priority

Detection order — write to the **first one that already exists**; if none exist, create `AGENTS.md`:

1. `AGENTS.md`
2. `CLAUDE.md`
3. `.cursor/rules/*.mdc`
4. `GEMINI.md`
5. `.github/copilot-instructions.md`

## Subagent Usage

- **Phase 1 Survey** — `explore` subagent for large-project doc distribution mapping
- **Phase 4 Reorganize** — `implementer` subagent for file moves + link updates (deterministic batch ops)

## Iron Law

1. No file move/delete without a user-confirmed migration plan
2. git working tree must be clean before reorg; if not, abort and prompt (no auto-stash)
3. No batch auto-rewrite in content remediation — per-item user confirmation

## Red Flags

- "Let me also refactor this unrelated old code" — out of doc scope
- "This README looks stale, delete it" — content deletion requires user confirmation
- Skipping the git-clean check before reorg — breaks reversibility

## Deliberate Decisions

1. **Single-file SKILL.md, no reference/template split.** The config priority table and built-in defaults are inlined tightly; report formats per phase are described, not templated. Follows spec 002's concise principle — the user explicitly rejected an over-structured multi-file layout.
2. **Structure + content as twin cores.** Content remediation is Phase 5, a peer action — not an optional tail. But its handling stays human-in-the-loop (per-item confirmation) because it is not mechanically verifiable, whereas structure reorg can run automated under the git safety net.
3. **Boundary between "physical duplication" (structure) and "content duplication" (content).** Same filename scattered in multiple locations → Phase 4. Overlapping semantic content across distinct files → Phase 5. Surfaced explicitly to avoid ambiguity.
4. **Not wired into `using-devflow`.** Per requirement, purely manual; no routing table entry.
5. **No auto-stash.** Abort-and-prompt on a dirty tree avoids hiding the user's uncommitted work.

## Files to Create / Modify

**Create:**
- `skills/curating-docs/SKILL.md`

**Modify:** none (not wired into the workflow; no other skill changes)

## Open Items for Planning

- Concrete entries of the built-in defaults table per project type (library / application / monorepo)
- SKILL.md body wording and length control (<500 lines)
- Phase 1 survey implementation — glob/grep vs. dispatched subagent threshold
