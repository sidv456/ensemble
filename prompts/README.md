# Prompts

Prompt templates the Orchestrator (and the agents it dispatches) use during a wave. Customize each one for your project's voice, tooling, and conventions.

## Files

- `orchestrator-dispatch.md` — Template the Orchestrator uses to spawn an IC. Includes the per-task spec, persona reference, worktree path, and Switchboard registration handshake.
- `ic-prelude.md` — Boilerplate every IC gets prepended to its task spec. Covers expectations, tooling, and reporting format.
- `pm-voice-guide.md` — One-page guide for the Project Manager agent: how status updates should sound. Replace with your team's leadership voice.
- `audit-checklist.md` — What the QA Lead (Audit) verifies on every wave's commits.
- `personas/<codename>.md` — Per-codename prompt preludes (one file per persona). Loaded by the Orchestrator at dispatch time, combined with `ic-prelude.md` and the task spec.

## Adopting

1. Copy this directory into your project.
2. Rewrite `pm-voice-guide.md` to match your team's leadership voice.
3. For each persona in `personas/<codename>.yaml`, write a matching `personas/<codename>.md` prompt prelude.
4. Adjust `orchestrator-dispatch.md` references if your project tracker / build commands differ.
