---
name: "brain-manager"
description: "Update, create, and health-check brain directives (instructions, skills, agents, TSG)"
---
# brain-manager — Master Skill

> The master skill for updating, creating, and health-checking brain directives.

## Triggers
- "update brain" / "update your brain"
- "add rule" / "add instruction"
- "teach brain" / "teach brain about [X]"
- "brain health" / "fix brain"
- "compress instructions"
- "check brain"

## Core Logic: Detect → Route → Create/Update

### Step 1: Detect the Topic
Determine the domain from: current file type, recent conversation, explicit request.

### Step 2: Route to the Right File
Look in `.github/instructions/` for a matching instruction file:
- C# → `csharp.instructions.md`
- Python → `python.instructions.md`
- TypeScript → `typescript.instructions.md`
- PowerShell → `powershell.instructions.md`
- Build/test → `project.instructions.md`
- New topic → create new category

### Step 3: Create or Update
- **Existing stub** → Add rule to stub (if <20 lines) or create/update detail file
- **New topic** → Create both stub + detail file pair

### Step 4: Confirm
Show the user what was created/updated.

## File Templates

### New Stub Template (`[topic].instructions.md`)
```yaml
---
description: "[Topic] [keyword1], [keyword2], [keyword3]"
applyTo: "**"
---
# [Topic] — [Title]

1. [Rule 1]
2. [Rule 2]
3. [Rule 3]

## Cross-References
- **Full guide:** `[topic].detail.instructions.md`
```

### New Detail Template (`[topic].detail.instructions.md`)
```yaml
---
description: "[Topic] deep reference, [keyword1], [keyword2], [keyword3], [keyword4], [keyword5]"
---
# [Topic] — Detailed Reference

## [Section 1]
[Detailed content, examples, edge cases]

## [Section 2]
[More detailed content]
```

**CRITICAL:** Detail files must NOT have `applyTo` — only `description` with keywords for lazy-loading.

## Generic vs Repo-Specific Separation
When creating or updating ANY directive, always split:
- **Generic** (reusable) — Language standards, design patterns, general workflows. File prefix or section clearly labeled. Example: `csharp.instructions.md` has generic C# rules.
- **Repo-Specific** — Project paths, team conventions, repo architecture, domain models. Example: `project.instructions.md` has THIS repo's build commands and layout.

If a file mixes both, split it. If a skill applies to any repo, keep repo-specific config in a separate companion file. Ask: "Would this apply to ANY repo, or just THIS one?"

## Rules
- Stubs: MAX 20 lines, 3-5 core rules, `applyTo: "**"` or glob pattern
- Details: No line limit, keyword-rich `description`, NO `applyTo`
- NEVER auto-fix health issues without showing options first
- When splitting bloated stubs: stub keeps rules, detail gets examples/explanations

## Health Check
When user says "brain health":
1. Scan all `.github/instructions/*.instructions.md` files
2. Check for: valid frontmatter, `description` + `applyTo` fields, file size (<150 lines for stubs)
3. Report: total files, compliance %, issues found
4. Offer to fix each issue (never auto-fix)

## Workspace Scope Guard ⚠️
ALL brain updates go in `.github/` in THIS workspace.
- ✅ `.github/instructions/` and `.github/skills/`
- ❌ NEVER `%APPDATA%\Code\User\prompts\`
- ❌ NEVER `~/.copilot/instructions/`
