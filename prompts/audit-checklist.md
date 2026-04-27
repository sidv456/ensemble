# Audit checklist

What the QA Lead (Audit) verifies after every wave. Read-only — Audit never modifies source.

## On every wave

### 1. Identify the commit range

The wave's commits are between the integration branch's previous tip and the new tip after merge. Walk:

```
git log --oneline <prev-integration-tip>..HEAD
```

For each commit, verify it has the expected `(eng: <Codename>)` attribution.

### 2. Verify each commit's claim

For each IC's branch, replay the verification:

- `git show <sha>` — review diff
- Run typecheck, lint, build at that SHA
- Compare against the IC's reported pass/fail status

### 3. Detect suppression pragmas

Scan diffs for any of:

- `// eslint-disable` (any variant — `eslint-disable`, `eslint-disable-next-line`, `eslint-disable-line`)
- `@ts-ignore`, `@ts-expect-error`, `@ts-nocheck`
- `# pylint: disable`, `# noqa`, `# type: ignore`
- `#[allow(...)]` (Rust)
- Language-specific equivalents

For each one found:

- File:line
- Rule silenced
- IC who introduced it
- Whether a justifying comment is present

If a justifying comment is present and narrow (single line, specific reason), accept. If blanket-disable or no justification, escalate.

### 4. Detect config loosening

Diff any config that affects gate strictness:

- `.eslintrc*`, `eslint.config.*`
- `tsconfig*.json`
- `.husky/`, `lint-staged.config.*`
- `.prettierrc*`
- CI workflow files
- Pre-commit hook scripts

If any of these were modified in the wave, surface the diff.

### 5. Detect hook bypasses

Best-effort. Git doesn't reliably record `--no-verify`, but check:

- Commit metadata for unusual signing patterns
- Any `core.hooksPath=/dev/null` or similar in commit messages / config diffs
- Whether commits exist that violate rules the local pre-commit hook would have caught

### 6. Verify the integration branch

Run:

- Typecheck
- Lint (full repo, not just wave's scope)
- Build

Capture exact exit codes and tails of any failures.

## Reporting format

```
## Audit Report — Wave <N>

Branch: <integration-branch> (HEAD <sha>)
Range audited: <prev-tip>..<new-tip> — <N> commits

### HEAD verification
- typecheck: exit <code> — <tail>
- lint: exit <code> — <tail>
- build: exit <code> — <tail>

### Net-new lint regressions
<list, file:line + rule + IC>

### Suppression pragmas added
<list, file:line + rule + IC + justified yes/no>

### Config loosening
<list of diffs, or "none">

### Hook bypass evidence
<list, or "none">

### Verdict
✅ Clean / ⚠️ Escalate / 🔴 Block — <reasoning>

### Recommended actions
<list of follow-ups for the next wave>
```

Audit's job is to surface issues honestly, not to block waves on minor findings. The Orchestrator decides whether to act on Audit's report.
