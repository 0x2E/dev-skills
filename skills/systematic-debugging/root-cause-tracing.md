# Root Cause Tracing

Bugs surface deep in the call stack but originate at the trigger. The instinct to patch where the error *appears* treats a symptom. **Trace backward through the call chain to the original trigger, then fix at the source.**

Load this technique when Phase 1 (Root Cause Investigation) of `SKILL.md` reaches "the error is deep and I can't tell where the bad value originates."

## When to Use

- Error fires deep in execution, not at the entry point
- Stack trace shows a long call chain
- Unclear which caller produced the invalid value
- A value is wrong but you don't know where it was set

## The Tracing Process

Walk backward one frame at a time. At each step ask: **what value was passed, and who passed it?**

1. **Observe the symptom** — the exact error, file, line.
2. **Find the immediate cause** — the code that directly raised it. What argument did it receive?
3. **Ask what called it** — move up one frame. Why did the caller pass that value?
4. **Keep tracing up** until the value's origin is a literal, a default, or an assignment — not a pass-through. That is the source.
5. **Fix at the source**, not at the symptom. Then consider adding layered validation (see `defense-in-depth.md`).

## Worked Example

Symptom: `git init` runs in the source-code directory instead of a temp dir.

```
1. Symptom:    Error: git init executed in ~/project/packages/core
2. Immediate:  execFile('git', ['init'], { cwd: projectDir })  ← projectDir is bad
3. Up one:     WorktreeManager.createWorktree(projectDir, id)  ← projectDir passed in
4. Up again:   Session.create() passed projectDir from context  ← context.tempDir
5. Source:     const ctx = setupCoreTest();  // ctx.tempDir is '' until beforeEach runs
```

Root cause: a top-level read of `ctx.tempDir` captured `''` before the test fixture populated it. The empty `cwd` resolved to `process.cwd()` — the source tree.

Fix at source: make `tempDir` a getter that throws when read before the fixture sets it. Patching `git init` to reject empty paths would only mask every other caller that passes the same bad value.

## When Manual Tracing Stalls: Add Instrumentation

If you cannot follow the chain by reading code, inject a stack capture right before the dangerous operation:

```typescript
async function gitInit(directory: string) {
  console.error('DEBUG git init', {
    directory,
    cwd: process.cwd(),
    stack: new Error().stack,
  });
  await execFile('git', ['init'], { cwd: directory });
}
```

```bash
npm test 2>&1 | grep 'DEBUG git init'
```

- Log **before** the operation, not after it fails.
- Use `console.error`/`stderr` in tests — application loggers are often suppressed during test runs.
- Look for the test filename and line in the captured stack; that names the triggering caller.

## Finding the Polluting Test

When state corruption appears across tests but you don't know which test introduced it, bisect: run the suite incrementally and stop at the first run that leaves the bad state behind. A one-by-one scan over the test glob pinpoints the polluter.

## Key Principle

Found the immediate cause? Ask whether you can trace one more level up.

- **Yes** → keep tracing.
- **No, this is the source** → fix here, then add validation at each layer.
- **Never** patch only the symptom site. Same bad value will arrive via another path tomorrow.

After fixing at the source, pair with `defense-in-depth.md` to make the bug structurally impossible.
