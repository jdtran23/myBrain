---
description: "PowerShell scripting standards, cmdlets, naming, error handling, modules"
applyTo: "**/*.{ps1,psm1,psd1}"
---
# PowerShell — Scripting Standards

1. **Verb-Noun naming** — Follow approved verbs (`Get-`, `Set-`, `New-`, `Remove-`)
2. **PascalCase** for functions and parameters
3. **Error handling** — Use `try/catch` with `-ErrorAction Stop`
4. **Strict mode** — Start scripts with `Set-StrictMode -Version Latest`
5. **Pipeline** — Prefer pipeline over loops where possible

## Cross-References
- **Full PowerShell guide:** `powershell.detail.instructions.md`
