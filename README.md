# Ensemble

*An AI agent swarm orchestration platform.*

> One human at the top. Everything below is AI: an Orchestrator dispatching individual contributor agents (ICs) into isolated git worktrees, with a Project Manager agent handling stakeholder communication, a QA Lead verifying compliance, and a Switchboard sidecar enabling live agent-to-agent coordination via MCP.

This document describes the platform — its architecture, cadence, codename system, and adoption pattern — so a team can stand up their own Ensemble instance for their project.

---

## What Ensemble is

A multi-agent orchestration framework for AI swarms doing real engineering. Not a single long-context agent. Not full autonomy. A coordinated team of dispatched, role-typed agents working in parallel, each with their own filesystem isolation and a shared communication substrate.

The unit of work is a **Wave** — a batched dispatch of N independent contributor agents (ICs), each owning one task spec, each in their own git worktree. Waves close with verification (typecheck, build, lint), an audit pass, and a status update from the Project Manager agent.

**The bet:** parallelism comes from work isolation, not agent hierarchy. Strip middle tiers. Run wide and flat.

## Core principles

1. **One human in the chain.** A single person sets direction and owns final approval. Everything below is AI. The human's job is to define waves, approve scope, and intervene when the swarm is stuck.
2. **Worktree isolation per IC.** Every individual contributor agent operates in a real git worktree. No clobbering. No commit-lock contention. No incentive to bypass pre-commit hooks.
3. **Codename attribution.** Every commit message ends `(eng: <Codename>)`. Every project-tracker comment is signed by the agent that produced it. Traceability survives even when the dispatch tree is flatter than the documented org.
4. **Project tracker as source of truth.** Status updates, ticket comments, initiative-level rollups all live in the project tracker (Linear, Jira, GitHub Issues, etc.). Ensemble's Project Manager agent writes them in a stylistic voice consistent with the team's actual leadership.
5. **Plan files as canonical specs.** Every IC dispatch references a per-task spec file. Reduces re-explanation, keeps the orchestrator's prompts small.
6. **Honest accounting.** Documented hierarchy and executing hierarchy are reconciled in retrospectives. If only N of M codenames mapped to live agents, say so.
7. **Iterate the platform with the work.** Each Wave delivers product *and* a refined version of Ensemble. Wave-end retrospectives explicitly address what to keep, change, add, drop.

## Architecture

```
Human (direction-setter; final approval)
│
└── Orchestrator — Engineering Manager — top-level AI session
    │   (wave dispatch, merges, verification)
    │
    ├── Switchboard — A2A sidecar — MCP server
    │   (registry, consult router, audit log)
    │
    ├── IC × N — individual contributors
    │   (each in own git worktree, one task spec each)
    │
    ├── Audit — QA Lead — read-only commit verification
    │   (pre-commit hook compliance, suppression-pragma audits)
    │
    └── Project Manager — comms agent
        (project-tracker status updates in team-leadership voice;
         per-IC ticket comments; wave-end rollups)
```

The structural shape — Orchestrator + Switchboard + flat IC pool + Audit + Project Manager — is invariant. The IC roster is project-specific.

### Roles

| Role | Responsibility | Lifetime |
|---|---|---|
| **Human** | Direction, scope, final approval | Permanent |
| **Orchestrator** | Wave dispatch, merge, verification, conflict resolution | Per session |
| **IC** | One task spec, one worktree, one branch, one commit | Per task |
| **Audit** | Read-only verification of pre-commit hooks; suppression-pragma audits; hook-bypass detection | Per wave |
| **Project Manager** | Project-tracker writes (status updates, ticket comments, initiative rollups) in team-leadership voice | Per wave |
| **Switchboard** | A2A consult routing, persona registry, dispatch state, audit log | Per session |

ICs coordinate via Switchboard. The Orchestrator is the hub for dispatch and merge; Switchboard is the hub for inter-IC consults. An IC asking "are you currently touching `<file>`?" of a sibling routes through Switchboard's MCP tool surface.

## The wave cadence

A wave is a single batched dispatch with a defined scope, run end-to-end:

1. **Plan.** Orchestrator writes per-IC task specs. Specs name the IC codename, the surface area, the success criteria, and any cross-IC dependencies.
2. **Dispatch.** Orchestrator spawns N ICs in parallel, each in an isolated worktree, each handed exactly its task spec.
3. **Build.** ICs work in parallel. Each produces a single commit (or small commit series) on their branch with attribution `(eng: <Codename>)`.
4. **Report.** ICs report back: commit SHA, branch name, verification output (typecheck, lint, build), any escalations.
5. **Merge.** Orchestrator merges branches into the integration branch. Sequential when files overlap; octopus when they don't.
6. **Verify.** Orchestrator runs final typecheck, lint, build on the integration branch.
7. **Audit.** Audit agent runs read-only verification of the wave's commits — hook-bypass detection, silenced-error pragma detection, config-loosening detection.
8. **Rollup.** Project Manager writes a wave-end status update on the project tracker (initiative + sub-initiative / project) in the team's leadership voice.
9. **Retro.** Orchestrator captures lessons in the platform retro doc. Adjustments roll into the next wave's dispatch template.

Indicative wall clock: ~25 minutes for ~15-20 atomic tasks.

## The codename system

Ensemble uses **per-agent codenames** instead of role-numbered identifiers (`Eng-1.1.4`). Codenames are:

- **Distinctive.** Each codename maps to one agent identity across waves. If a codename shows up in one wave handling a domain and again in a later wave doing a related fix, that's the same logical agent — the persona is consistent.
- **Themed.** Pick a theme that suits your project (craftsperson, mythic, noir, astronomical, etc.). The convention is what matters, not the specific theme.
- **Load-bearing.** Codename attribution survives in commit messages, tracker comments, status updates, and Switchboard's persona registry. Every line of code traces back to a specific agent identity.

A typical codename has:

- **A name** (single word, theme-aligned)
- **A role** (IC, QA Lead, Project Manager)
- **A function** (eng, qa, comms)
- **An expertise tag** (1-3 words describing their surface)

Switchboard's persona registry (`personas/<codename>.yaml`) makes this structured and code-reviewable.

### Persona file template

```yaml
codename: <Name>
role: IC               # IC | QA Lead | Project Manager
function: eng          # eng | qa | comms
expertise:             # 1-3 short tags describing surface area
  - <tag-1>
  - <tag-2>
scope:                 # files / directories the persona owns
  - <path-glob-1>
  - <path-glob-2>
dispatch_template:     # path to per-codename prompt prelude
  prompt: personas/<name>.prompt.md
```

## Tooling integrations

Ensemble assumes a working environment with:

- **Git** with worktree support
- **Project tracker** (Linear, Jira, GitHub Issues, etc.) — accessed via MCP
- **Docs platform** (Notion, Confluence, etc.) — accessed via MCP, optional
- **Comms platform** (Slack, Discord, etc.) — accessed via MCP, optional
- **An agent harness** with subagent dispatch and tool calls (e.g. Claude Code)
- **A typecheck / lint / build pipeline** — language-specific

Switchboard is itself an MCP server, plugged in alongside the others. The full MCP stack lets the Project Manager post updates, Audit replay verification, ICs read tickets — all through standard tool calls, no bespoke transport.

### Switchboard tool surface

```
switchboard.register(codename, persona, worktree, branch, task)
   — IC announces dispatch
switchboard.consult(target_codename, question)
   — sibling consult; routes to live IC if dispatched, else persona-shadow
switchboard.list_active()
   — discover currently-live ICs
switchboard.get_state(codename)
   — current branch / dirty files for an IC
switchboard.audit(filter)
   — review of consult log
switchboard.unregister(codename)
   — IC announces completion
```

## Adopting Ensemble for your project

1. **Define your roster.** Pick 8-20 codenames mapped to your project's surfaces. Write `personas/<codename>.yaml` files with name, expertise, scope, file ownership. Code-reviewable; check them in.
2. **Pick a Project Manager voice.** The Project Manager agent posts in the voice of your actual team leadership. Write a one-page voice guide so updates feel native to your team.
3. **Write the dispatch template.** Orchestrator's prompt for spawning an IC: codename, persona reference (loads from registry), task spec, worktree path, success criteria, Switchboard registration handshake.
4. **Wire MCP.** Add Switchboard, your project tracker, your docs platform, your comms platform to `.mcp.json`. Make sure Audit can read git history; make sure the Project Manager can write tracker comments.
5. **Run a small wave first.** 3-5 ICs on low-risk surfaces. Validate the cadence end-to-end before scaling.
6. **Capture the retro.** What broke? What got skipped? Adjust the dispatch template before the next wave.

## Anti-patterns

- **Single long-context agent doing everything.** Not Ensemble. The dispatch tree is the point.
- **Concurrent agents on the same git tree.** Causes branch flips, lost commits, hook bypasses. Use worktrees.
- **Manager tier above ICs.** EMs, TLs, and similar middle layers add bookkeeping without adding information. The plan file is the manager.
- **Aspirational hierarchy claims.** Don't document an org chart larger than the swarm can actually execute. Reconcile documented and executing structure in every retro.
- **Ad-hoc consults via prompt embedding.** Encoding one IC's expertise inside every other IC's prompt is stale on day one. Use Switchboard's persona registry as the source of truth.
- **Suppression-pragma cleanup later.** ICs should not silence lint/typecheck errors with `eslint-disable` / `@ts-ignore` / `--no-verify` to land work. Audit's job is to flag these immediately, not at end-of-wave.

## What Ensemble is not

- **Not a single long-context agent.** A "do everything in one chat" loop is not Ensemble.
- **Not full autonomy.** A human still owns scope, approval, and final intervention. Ensemble accelerates execution; it does not replace direction.
- **Not a substitute for engineering judgment.** ICs are subject to the same code review, typecheck, and audit gates a human contributor would face.
- **Not platform-locked.** Designed around standard primitives (git, MCP, project trackers). Should run on any team's stack with minor adaptation.

## Naming glossary

- **Ensemble** — the orchestration platform (this doc)
- **Switchboard** — the A2A sidecar component of Ensemble
- **Project Manager** — comms agent inside Ensemble (typically given a codename per the team's theme)
- **Audit** — read-only QA Lead inside Ensemble (typically given a codename per the team's theme)
- **IC** — individual contributor agent
- **Wave** — a batched dispatch of N ICs run end-to-end
- **Persona registry** — `personas/<codename>.yaml` defining each IC's identity and scope

---

*Template. Drop in your project's roster, voice, and tracker config to instantiate.*
