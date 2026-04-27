# Ensemble Setup

Quick start for adopting Ensemble on your project.

## 1. Pull the template

```
git clone <ensemble-repo-url> ensemble
cp -r ensemble/personas ./
cp -r ensemble/prompts ./
cp ensemble/.mcp.json.example ./.mcp.json
cp ensemble/SETUP.md ./SETUP.md  # optional, for your team's reference
```

You should now have:

```
your-project/
├── personas/         # IC roster (edit these)
├── prompts/          # Dispatch templates (customize)
├── .mcp.json         # MCP wiring (fill in API keys)
└── ...your code
```

## 2. Define your roster

Pick 8-20 codenames for your project's surfaces. Theme them to taste (craftsperson, mythic, noir, astronomical, etc.).

For each codename:

1. Copy `personas/_template-ic.yaml` → `personas/<codename>.yaml`
2. Fill in `expertise`, `scope`, `dispatch_template`
3. Write a matching `prompts/personas/<codename>.md` prelude

Reuse `personas/example-pm.yaml` and `personas/example-qa.yaml` as starting points for your Project Manager and QA Lead.

## 3. Set the Project Manager voice

Replace `prompts/pm-voice-guide.md` with your team's actual voice guide. The Project Manager agent reads this when posting on your project tracker — without it, updates sound generic.

## 4. Wire MCP

Edit `.mcp.json` to fill in API keys for your project tracker (Linear, Jira, etc.), docs platform (Notion, Confluence), and comms platform (Slack, Discord). Switchboard's defaults assume Linear + Notion + Slack; swap as needed.

## 5. Confirm tooling

Make sure your typecheck, lint, and build commands run cleanly on a fresh clone. Update the placeholders in `prompts/orchestrator-dispatch.md` (`{typecheck_command}`, `{lint_command}`, `{build_command}`) to match.

## 6. Run a small wave

Start with 3-5 ICs on low-risk surfaces. Validate the cadence end-to-end:

1. Orchestrator dispatches ICs (per `prompts/orchestrator-dispatch.md`)
2. ICs work in worktrees, register with Switchboard
3. ICs commit + report
4. Orchestrator merges to integration branch
5. Audit verifies (per `prompts/audit-checklist.md`)
6. Project Manager rolls up status to tracker (per `prompts/pm-voice-guide.md`)

If anything breaks, capture in a retro doc and adjust the dispatch template before the next wave.

## 7. Iterate

Each wave delivers product *and* a refined Ensemble. Wave-end retros explicitly address what to keep, change, add, drop. Update personas, prompts, and `.mcp.json` as you learn.

---

See `README.md` for the full architectural overview, principles, and anti-patterns.
