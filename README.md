# C389 CA2: Personal Productivity Agent

Scenario **S5: Notes and Reading Assistant**, built beside the Finance Tracker baseline.

## Repo layout

| Folder | What goes here |
|---|---|
| `workflows/final/` | Exported JSON of every workflow in the merged build, from the nominated account (this is what gets submitted) |
| `workflows/working/` | Work-in-progress exports before the merge, named `<member>-<workflow>.json` |
| `site/` | The chat page that embeds the widget |
| `eval/` | Fixed eval set, our own messages with expected tools, before/after results |
| `docs/` | PRDs, architecture diagram, guardrails |

## Rules

- **Never commit a key, token or Google credential.** Name the credential, never paste it. Check exported JSON before pushing.
- Finance tabs are never renamed: `transactions`, `categories`, `budgets`.
- At most seven tools on the hub.
- Scheduled workflows run once a day, never more often.

## Docs

- [Project PRD (whole build)](docs/PRD-Project.md)
- [Hub agent and merge PRD (Member 1)](docs/PRD-Member1-Hub-Agent.md)
