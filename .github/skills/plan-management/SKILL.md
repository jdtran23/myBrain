---
name: "plan-management"
description: "Create, manage, and track structured plans with AI validation guardrails (Plan-Generate-Verify-Iterate)"
---
# plan-management — AI Validation Guardrail (AI-DLC)

> Enforces the most important rule in AI-assisted work: AI output is untrusted until validated.

## Triggers
- "plan" / "create plan" / "new plan"
- "work on plan [ID]"
- "AI validation" / "verification strategy"

## Workflow

### Creating a Plan
1. User says "create a plan for [feature]"
2. Create numbered plan file: `Plans/NNN-plan-[slug].md`
3. Plan follows the v2 template below
4. Route to @cresselia for plan review (plan-review gate)
5. Address review findings, iterate until APPROVED
6. User says "work on plan NNN" → execute with verification

### Plan File Template (v2)

```markdown
# Plan NNN: [Title]
Status: Draft  <!-- Draft | Active | Blocked | Complete | Archived -->

## Objective
[What success looks like — one paragraph]

## Non-goals
<!-- What we're explicitly NOT doing, even though someone might expect it -->
- [Explicit exclusion 1]
- [Explicit exclusion 2]

## AI Usage Declaration
- **AI will generate:** [list what AI produces]
- **AI will NOT decide:** [list what requires human judgment]

## Risks & Rabbit Holes
### AI-Specific Risks
- [Expected AI failure modes — hallucinated APIs, incorrect logic, missing edge cases]

### Project Risks
- [Dependencies, unknowns, integration risks, external blockers]

### Rollback Strategy
- [How to undo if this goes wrong — required for code/infrastructure plans; for research-only plans, state "N/A — no artifacts to roll back"]

## Overall Success Metric
[Single sentence: when is this plan DONE?]

---

## Phase N: [Phase Name]
> *[One-line phase summary]*

**Gate:** [standard review gates | human-approval | specific gate needed]  <!-- omit for standard review gates (default) -->
**Dependencies:** [Phase X must be complete | None]

- [ ] **Task N.M:** [Action verb + subject + key constraints]
  - [Guidance — constraints, approach hints, tools to use]
  - Verify: [How to validate — human-checkable or machine-checkable]

### Acceptance Criteria (Phase N)
- [ ] [Testable criterion 1]
- [ ] [Testable criterion 2]

---

## Design Decisions
<!-- OPTIONAL — include when approach matters (tech stack, architecture choices) -->
| Decision | Choice | Why | Alternative Rejected |
|----------|--------|-----|---------------------|

## Validation Strategy
Every AI-generated artifact must have verification steps defined BEFORE generation:
- Deterministic tests
- Compile/build validation
- Schema validation where applicable
- Regression tests for existing functionality
```

## Template Field Reference

| Field | Required? | Purpose |
|-------|-----------|---------|
| **Status** | Always | Plan lifecycle: Draft → Active → Blocked → Complete → Archived |
| **Objective** | Always | What success looks like |
| **Non-goals** | Always | Explicit scope exclusions — prevents scope creep |
| **AI Usage Declaration** | Always | What AI generates vs what humans decide |
| **Risks & Rabbit Holes** | Always | AI-specific risks + general project risks + rollback strategy |
| **Overall Success Metric** | Always | Single sentence: when is this plan DONE? |
| **Phase Gate** | Per-phase | Context-specific guardrails beyond system defaults. Defaults to `standard review gates` if omitted. |
| **Phase Dependencies** | Per-phase | What must complete before this phase starts |
| **Task Verify** | Per-task | How to validate the task output |
| **Acceptance Criteria** | Per-phase | Testable criteria for phase completion |
| **Design Decisions** | Optional | When approach matters — tech stack, architecture choices |
| **Validation Strategy** | Always | How AI artifacts are verified before acceptance |

## Guardrail Stratification

Plans do NOT repeat system-level guardrails (code review, plan review, research review). Those are always-on in `multi-agent-orchestration.instructions.md`.

Plans only declare **context-specific guardrails** beyond system defaults:
- **"standard review gates"** — no additional guardrails needed (most phases)
- **"human-approval"** — this phase requires explicit human sign-off before proceeding
- **[specific gate]** — e.g., "database migration review", "security audit"

## Rules
- Every plan MUST include an AI validation section
- AI generation tasks MUST be followed by verification tasks
- Plans without a validation strategy are INCOMPLETE
- Pattern: Plan → Review → Execute → Verify → Iterate
- NEVER: Plan → Generate → Done
- Plans are reviewed by @cresselia before execution (plan-review gate)
- Plan status must be updated as lifecycle progresses

## GitHub Issue Cross-Referencing
- When a plan task maps to a GitHub Issue, add `(GitHub: #N)` after the task title
- Create a `plan:NNN` label on the repo and tag all related issues
- Issues without a `plan:` label are standalone backlog items
- When completing a plan task, close the corresponding issue
- Not every task needs an issue (e.g., manual/validation tasks) and not every issue needs a plan task (backlog items)

## Plan Numbering
- Auto-increment: scan `Plans/` for highest NNN, use NNN+1
- Format: `Plans/NNN-plan-[slug].md`
- Archive completed plans: move to `Plans/archive/`

## Migration
- v2 template applies to all NEW plans going forward
- Existing plans (004-008) are NOT retroactively updated — they remain valid as-is
- When resuming work on an older plan, no template migration is needed
