---
description: "Project structure, build commands, test commands, directory layout"
applyTo: "**"
---
# Project — Structure & Build

## Directory Layout
- `.github/` — Brain directives (instructions, skills, agents, prompts)
- `.vscode/` — Workspace settings and MCP config
- `Plans/` — Active task plans

## Build & Test
- **myBrain:** No build step — Markdown/YAML directives only
- **pokemon-tcg-tracker (backend):** `pip install -r requirements.txt` → `python scripts/run_api.py`
- **pokemon-tcg-tracker (frontend):** `cd frontend && npm install` → `npm run build` / `npm run test`
- See `pokemon-tcg-tracker.instructions.md` for full command reference

## Cross-References
- **Full project guide:** `project.detail.instructions.md`
