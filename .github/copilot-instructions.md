---
description: "Workspace brain — repo overview, scope routing, directive index"
applyTo: "**"
---
# myBrain — Copilot Workspace Brain

## Scope Guard ⚠️
ALL brain updates go in `.github/` in THIS workspace. NEVER edit global instructions.
- ✅ Edit: `.github/instructions/*.instructions.md`
- ✅ Edit: `.github/skills/*/SKILL.md`
- ❌ NEVER: `%APPDATA%\Code\User\prompts\`
- ❌ NEVER: `~/.copilot/instructions/`

## Architecture Principle: Generic vs Repo-Specific Separation
All directives (instructions, skills, agents, knowledge docs) MUST separate **generic** knowledge from **repo-specific** knowledge:
- **Generic** — Reusable across any repo (language standards, design patterns, general workflows). Lives in clearly labeled generic files.
- **Repo-Specific** — Unique to one codebase (project paths, team conventions, repo architecture, domain logic). Lives in clearly labeled repo-specific files.

When creating or updating directives, always ask: "Would this apply to ANY repo, or just THIS one?" Split accordingly. This keeps generic knowledge portable and repo-specific knowledge focused.

## Directive Index
- **Instructions:** `.github/instructions/` — coding standards, team identity, execution rules
- **Skills:** `.github/skills/` — active workflows (brain-manager, plan-management, orchestration-evolution)
- **Agents:** `.github/agents/` — specialist team: mewtwo, uxie, porygon, absol, metagross, cresselia
- **Prompts:** `.github/prompts/` — reusable /slash-command templates (e.g., /orchestrate)
- **Plans:** `Plans/` — active task plans (archived when done)
- **TSG:** `.github/tsg.instructions.md` — troubleshooting & lessons learned

## Quick Reference
- "update your brain" → brain-manager skill updates workspace directives
- "create a plan" → plan-management skill creates validated plans
- "brain health" → brain-manager scans and reports issues
- "evolve agents" / "orchestration check" → orchestration-evolution skill reviews readiness
