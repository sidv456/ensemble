# Personas

Each persona file defines one agent's identity in the Ensemble swarm. The Orchestrator reads these at dispatch time, hands the matching persona to each spawned agent, and (when Switchboard is wired) registers the persona for live A2A consults.

## File naming

`personas/<codename>.yaml` — lowercase codename, dashes for multi-word.

Examples in this directory:

- `_template-ic.yaml` — bare template; copy and rename
- `example-frontend-ic.yaml` — concrete IC, frontend surface
- `example-backend-ic.yaml` — concrete IC, backend surface
- `example-pm.yaml` — Project Manager
- `example-qa.yaml` — QA Lead (Audit role)

`_template-ic.yaml` starts with an underscore so directory listings sort it first; remove the underscore when you copy it.

## Required fields

| Field | Description |
|---|---|
| `codename` | The agent's name. Single word, theme-aligned. |
| `role` | One of: `IC`, `Project Manager`, `QA Lead`. |
| `function` | One of: `eng`, `qa`, `comms`. |
| `expertise` | 1-3 short tags describing this agent's surface. Used for consult routing. |
| `scope` | List of file/path globs this agent owns. Used for boundary enforcement. |
| `dispatch_template` | Path (relative to repo root) to the per-codename prompt prelude. Combined with the per-task spec at dispatch time. |

## Optional fields

| Field | Description |
|---|---|
| `consult_examples` | Sample questions other ICs might ask this persona. Helps Switchboard's persona-shadow mode return useful answers when the agent isn't currently dispatched. |
| `voice_guide` | (Project Manager only) Path to the voice-guide markdown file used when posting status updates. |
| `escalation` | What this agent should do when blocked. |
| `tools` | Tools/MCP servers this agent needs (subset of the global toolset). |

## Writing a new persona

1. Copy `_template-ic.yaml` → `personas/<your-codename>.yaml`
2. Fill in `codename`, `role`, `function`, `expertise`, `scope`
3. Write the prompt prelude in `prompts/personas/<your-codename>.md`
4. Add the codename to your roster in `prompts/orchestrator-dispatch.md`
5. Code-review the persona before first dispatch. Personas drift if not maintained.
