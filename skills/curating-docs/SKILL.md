---
name: curating-docs
description: "Use to tame documentation chaos in a project — surveys scattered/stale/duplicate/orphaned docs, recommends an organization convention from built-in best practices, codifies it into the AI config file (AGENTS.md/CLAUDE.md/etc.), reorganizes the file structure, and remediates content health."
---

# Curating Docs

Governs a project's documentation assets across two first-class dimensions — **structure** (physical organization) and **content** (health). Use when docs have accumulated chaos: scattered root files, multiple competing `docs/` directories, inconsistent naming, stale/duplicate/orphaned docs.

## Iron Law

1. **No file move/delete without a user-confirmed migration plan.**
2. **git working tree must be clean before reorg.** If dirty, abort and prompt — do NOT auto-stash.
3. **No batch auto-rewrite in content remediation.** Every content judgment needs per-item user confirmation.

## Red Flags

Stop if you catch yourself thinking:
- "Let me also refactor this unrelated old code while I'm here" — out of doc scope.
- "This README looks stale, delete it" — content deletion always needs user confirmation.
- "Working tree is dirty but I'll just proceed" — breaks reversibility.

## Process

Run the phases in order. Each phase gates the next.

### Phase 1: Survey

One scan, two diagnoses. Output a survey report with two sections.

**Structure signals:**
- Root-level scattered `.md` (excluding convention files: `README`, `LICENSE`, `CHANGELOG`, `CONTRIBUTING`)
- Multiple doc directories (`docs/`, `doc/`, `documentation/`, `wiki/`)
- Naming chaos (inconsistent case, no prefixes, mixed date/version schemes)
- **Physical duplication** — same/near-same filename in multiple locations

**Content signals:**
- **Stale** — apparent from the doc's own content: broken internal cross-references, outdated version/date/deprecated-tool cues, or explicit stale markers
- **Duplicate** — overlapping content across distinct documents
- **Contradictory** — conflicting descriptions of the same subject
- **Orphaned** — no inbound links/references
- **Missing** — doc types the convention requires but the project lacks

For large projects, dispatch an `explore` subagent to map doc distribution. Bound the scan to documentation files (`.md` primarily; `.rst`/`.adoc` if present) — do not scan source code in depth.

**Boundary rule:** physical duplication (same file scattered in multiple locations) → handled in Phase 4. Content duplication (overlapping semantic content across distinct files) → handled in Phase 5.

### Phase 2: Recommend

Load the **Built-in Defaults** below, identify the project's existing sensible structures as **keep-items**, merge, and present: directory structure + naming convention + doc-type taxonomy. The user confirms or adjusts item by item.

Lead with the recommendation; do not present a menu of equal options. Note where the project's existing structure deviates and whether to preserve or migrate it.

### Phase 3: Codify

Detect the AI config file by **Config File Priority** below — write to the first one that already exists; if none exist, create `AGENTS.md`. Add the confirmed documentation conventions to the file. Write only after user confirmation.

Match the existing file's format and conventions. Do not overwrite existing content.

### Phase 4: Reorganize (structure)

- **Precondition:** `git status` must be clean. If dirty, abort and tell the user to commit/stash first.
- Generate a migration plan: `old path → new path` map + delete/merge list.
- User confirms the plan.
- Dispatch an `implementer` subagent to perform moves/renames.
- Update cross-references (relative links) between docs.
- Verify: `git status` shows only intended changes + link check (no broken links) + no files lost.
- Produce a change report (what moved, what merged, what deleted, links updated).

### Phase 5: Remediate (content)

Act on the Phase 1 content diagnosis. For each item, present the finding + recommended action (update/merge/delete/write-missing) and **confirm with the user before acting**. Group by category (stale → duplicate → contradictory → orphaned → missing) for review efficiency, but each action remains individually confirmed.

No batch rewriting. Content semantics cannot be mechanically verified — the user is the safety net here, just as git is the safety net for structure.

## Built-in Defaults

Default directory/naming skeleton, selected by inferred project type. Present the matching default in Phase 2, then adjust to the project.

**Library / application:**

| Path | Purpose | Naming |
|------|---------|--------|
| `README.md` (root) | Project overview, required | — |
| `docs/specs/` | Specifications | `NNN-{desc}.md` |
| `docs/adrs/` | Architecture decisions | `NNN-{desc}.md` |
| `docs/design/` | Design docs | `{desc}.md` |
| `docs/runbooks/` | Ops / playbooks | `{desc}.md` |
| `docs/api/` | API reference | `{desc}.md` |
| `CHANGELOG.md`, `CONTRIBUTING.md` (root) | Convention files | — |

`NNN-` numbering: three-digit sequential, incremented independently per directory; description in lowercase English, hyphen-separated.

**Monorepo:** apply the library/application layout per package, or hoist shared specs/ADRs to the repo root `docs/`. Infer from the existing structure.

These are defaults, not dogma — if the project already has a coherent convention, preserve it and only fill gaps.

## Config File Priority

Write to the **first existing** file; if none exist, create `AGENTS.md`:

1. `AGENTS.md`
2. `CLAUDE.md`
3. `.cursor/rules/*.mdc`
4. `GEMINI.md`
5. `.github/copilot-instructions.md`

## Subagent Usage

- **Phase 1 Survey** — `explore` subagent for large-project doc distribution mapping. Include all scan context in the prompt (subagents have no conversation history).
- **Phase 4 Reorganize** — `implementer` subagent for file moves + link updates (deterministic batch ops). Pass the full migration plan in the prompt.

## Gotchas

- Root-level convention files (`README`, `LICENSE`, `CHANGELOG`, `CONTRIBUTING`, `CODE_OF_CONDUCT`) must not be moved into `docs/`.
- Link updates are the most error-prone step — relative links break when files move. Re-check after reorg.
- Multiple AI config files may coexist; write to only one (the highest-priority existing one) to avoid divergence.
- A doc with zero inbound links isn't always orphaned — entry points (README, indexes) are legitimately link-free. Check whether it's referenced from source/config/comments before flagging.

After Phase 5, report the outcome. No further skill is invoked.
