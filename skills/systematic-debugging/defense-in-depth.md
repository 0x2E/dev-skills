# Defense-in-Depth Validation

A single validation at the fix point feels sufficient, but it is bypassable: a different code path, a refactor, or a mock can route the same bad value past that one check. **Validate at every layer data passes through, so the bug becomes structurally impossible to reproduce.**

Load this technique after `root-cause-tracing.md` identifies where the bad value originates and the fix is about to be applied.

## Why Multiple Layers

- One check: "we fixed this occurrence."
- Layered checks: "this class of bug can no longer happen here."

Each layer catches what another misses — entry validation stops naive callers, environment guards stop context-specific hazards, debug logging catches the case that somehow still slips through.

## The Four Layers

### Layer 1 — Entry-point validation
Reject obviously invalid input at the public API boundary.

```typescript
function createProject(name: string, dir: string) {
  if (!dir || dir.trim() === '') {
    throw new Error('dir cannot be empty');
  }
  if (!existsSync(dir) || !statSync(dir).isDirectory()) {
    throw new Error(`dir is not a directory: ${dir}`);
  }
  // proceed
}
```

### Layer 2 — Business-logic validation
Ensure data makes sense *for this operation* (not just "is a string").

```typescript
function initializeWorkspace(projectDir: string) {
  if (!projectDir) {
    throw new Error('projectDir required to initialize a workspace');
  }
  // proceed
}
```

### Layer 3 — Environment guards
Refuse dangerous operations in specific contexts.

```typescript
async function gitInit(directory: string) {
  if (process.env.NODE_ENV === 'test') {
    const target = resolve(directory);
    if (!target.startsWith(resolve(tmpdir()))) {
      throw new Error(`Refusing git init outside tmpdir during tests: ${directory}`);
    }
  }
  // proceed
}
```

### Layer 4 — Debug instrumentation
Capture context for forensics, in case the earlier layers are somehow bypassed.

```typescript
async function gitInit(directory: string) {
  console.error('git init', { directory, cwd: process.cwd(), stack: new Error().stack });
  // proceed
}
```

## Applying the Pattern

1. **Trace the data flow** — where the bad value originates, every checkpoint it passes through.
2. **Map the layers** — list each point data crosses: entry → business logic → environment → the operation itself.
3. **Add validation at each layer** — entry, business, environment, debug.
4. **Verify each layer independently** — deliberately bypass layer 1 and confirm layer 2 still catches it.

## When One Layer Is Enough

Layering has a cost: more code, more noise, more assertions to maintain. Don't add all four layers reflexively.

- **One layer suffices** for a self-contained function with a single caller and no external input.
- **Layer aggressively** when the value crosses a trust boundary, flows through many callers, or the failure mode is destructive (file deletion, data loss, git operations in the wrong directory).

Match the depth to the blast radius.

## Worked Example

Bug: empty `projectDir` caused `git init` to run in the source tree.

Data flow: `setupCoreTest()` returns `tempDir: ''` → `Project.create(name, '')` → `WorkspaceManager.create('')` → `git init` in `process.cwd()`.

Layers added:

- Layer 1: `Project.create()` rejects empty/nonexistent/non-directory paths.
- Layer 2: `WorkspaceManager` rejects empty `projectDir`.
- Layer 3: `gitInit` refuses to run outside `tmpdir()` under `NODE_ENV=test`.
- Layer 4: stack-trace logging before every `git init`.

Result: the original bug is unreachable, and any future caller passing an empty directory fails loudly at the first layer it hits.
