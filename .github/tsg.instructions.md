---
description: "Troubleshooting guide, fixes, lessons learned, debugging tips, solutions"
---
# Troubleshooting & Lessons Learned

> This file grows over time as the AI records fixes and lessons learned.
> Add entries with date, problem, root cause, and fix.

<!-- Example entry:
## 2026-03-30 — Build fails after NuGet update
- **Problem:** dotnet build fails with missing assembly reference
- **Root Cause:** NuGet package version mismatch after update
- **Fix:** Run `dotnet restore --force` then rebuild
-->

## 2026-03-31 — SKILL.md files require YAML frontmatter
- **Problem:** VS Code flags "Skill must provide a name" and "Skill must provide a description" on SKILL.md files
- **Root Cause:** SKILL.md files were created without YAML frontmatter. VS Code requires `name` and `description` fields, same as `.agent.md` files.
- **Fix:** Add frontmatter block: `---\nname: "skill-name"\ndescription: "What the skill does"\n---`

## 2026-03-30 — shadcn/ui v4 uses @base-ui/react, not Radix
- **Problem:** `SheetTrigger asChild` prop caused type errors and runtime failures
- **Root Cause:** shadcn/ui v4 migrated from Radix to `@base-ui/react`. The `asChild` pattern doesn't exist in base-ui components.
- **Fix:** Remove `asChild` prop. Apply className directly to the trigger component. Check shadcn docs for v4-specific API differences.

## 2026-03-30 — Background terminals always start from workspace root
- **Problem:** `cd frontend && npm run build` in a background terminal failed — CWD was workspace root, not where the foreground terminal had `cd`'d to
- **Root Cause:** Background terminals (`isBackground: true`) spawn a new shell at workspace root regardless of prior CWD in foreground terminals
- **Fix:** Always include `cd <target>` in background terminal commands, or use foreground terminals for short-lived commands

## 2026-03-30 — TypeScript 6 deprecated baseUrl in tsconfig
- **Problem:** Build warnings about `baseUrl` being deprecated
- **Root Cause:** TypeScript 6 no longer needs `baseUrl` when using `paths` with `bundler` moduleResolution
- **Fix:** Remove `baseUrl` from tsconfig.json entirely. Path aliases (`@/*`) work without it.

## 2026-03-30 — Vite scaffolding has interactive prompts
- **Problem:** `npm create vite@latest` hangs during automated execution — waiting for interactive framework selection
- **Root Cause:** The scaffolding CLI requires user input for template selection
- **Fix:** Skip the scaffolding tool. Manually create `package.json`, install deps, create `vite.config.ts`, and write config files directly.

## 2026-03-30 — useState inside TanStack Table column cell renders
- **Problem:** `useState` called inside a column `cell` arrow function
- **Root Cause:** While TanStack Table's `flexRender` happens to treat cell functions like components, this violates Rules of Hooks. If `columns` is ever moved inside the component, all state resets on every render.
- **Fix:** Extract cell content into a proper named component (e.g., `ImageCell`) and reference it from the column def.

## 2026-03-30 — Absol review skipped in commit pipeline
- **Problem:** Phase 3 was reported as "done" without running Absol code review — process violation
- **Root Cause:** Treating Absol as an optional/separate step rather than an integrated part of the definition of done
- **Fix:** Updated Mewtwo's agent definition with a mandatory "Commit Gate" section. Pipeline is now: build → test → @absol → fix → re-verify → done. Memory note also recorded.

## 2026-03-30 — decodeURIComponent crashes on malformed URL params
- **Problem:** Navigating to `/cards/%ZZ` would crash the entire React component tree with uncaught `URIError`
- **Root Cause:** `decodeURIComponent` throws on invalid percent-encoding sequences
- **Fix:** Wrap in try/catch, falling back to raw string: `try { decodeURIComponent(v) } catch { v }`

## 2026-03-30 — Link wrapping Button is invalid HTML
- **Problem:** React Router `<Link>` (renders `<a>`) wrapping shadcn `<Button>` (renders `<button>`) creates invalid HTML nesting
- **Root Cause:** HTML spec forbids interactive content inside `<a>`. Screen readers and click handlers behave inconsistently.
- **Fix:** Use `useNavigate()` hook with `<Button onClick={() => navigate("/path")}>` instead of nesting.
