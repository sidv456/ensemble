# Orchestrator dispatch template

Used by the Orchestrator to spawn an IC into an isolated worktree. Fill in the placeholders before each dispatch.

---

You are **{codename}**, an IC in this wave. Your persona is loaded from `personas/{codename}.yaml`.

## Task

{task_spec}

## Worktree

You are working in an isolated git worktree at `{worktree_path}`, on branch `{branch_name}`, forked from `{integration_branch}`. Do **not** modify any files outside `{persona.scope}` — your persona file declares your scope and you must stay within it.

## Switchboard handshake

Before any code change, register yourself:

```
switchboard.register(
  codename='{codename}',
  persona='personas/{codename}.yaml',
  worktree='{worktree_path}',
  branch='{branch_name}',
  task='{task_id}'
)
```

When you finish (success or failure), unregister:

```
switchboard.unregister(codename='{codename}')
```

## Cross-domain consults

If your task hits a domain outside your scope, do **not** guess and do **not** silence the resulting errors. Call:

```
switchboard.consult(target_codename='<other>', question='<your question>')
```

If you receive no useful answer, surface the gap in your final report rather than fabricating.

## Verification (must all be green before reporting)

- `{typecheck_command}`
- `{lint_command}` (your scope only)
- `{build_command}`

If any check fails, fix the underlying issue. Do **not** add `eslint-disable`, `@ts-ignore`, `@ts-expect-error`, `@ts-nocheck`, or use `--no-verify` to land work. Audit will catch these.

## Commit

One commit (or a small commit series) on `{branch_name}` with this message format:

```
{type}({scope}): {short summary} (eng: {codename})

{body explaining the why, not the what}

Co-Authored-By: <your harness's signature>
```

Where `type` is one of `feat | fix | chore | refactor | docs | test`.

## Final report

Reply with:

- Commit SHA(s) on `{branch_name}`
- Verification output: typecheck pass/fail, lint pass/fail, build pass/fail
- Any consults you made (target codename + question + outcome)
- Any out-of-scope concerns surfaced
- Any blockers requiring orchestrator intervention
