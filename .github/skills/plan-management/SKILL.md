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
3. Plan includes: tasks, acceptance criteria, AI validation section
4. User reviews and edits the plan
5. User says "work on plan NNN" → execute with verification

### Plan File Template
```markdown
# Plan NNN: [Title]

## Objective
[What this plan accomplishes]

## AI Usage Declaration
- **AI will generate:** [list what AI produces]
- **AI will NOT decide:** [list what requires human judgment]
- **Expected failure modes:** hallucinated APIs, incorrect logic, missing edge cases

## Tasks
- [ ] Task 1: [description]
  - Verify: [how to validate]
- [ ] Task 2: [description]
  - Verify: [how to validate]

## Validation Strategy
Every AI-generated artifact must have verification steps defined BEFORE generation:
- Deterministic tests
- Compile/build validation
- Schema validation where applicable
- Regression tests for existing functionality

## Acceptance Criteria
- [ ] [Criterion 1]
- [ ] [Criterion 2]
```

## Rules
- Every plan MUST include an AI validation section
- AI generation tasks MUST be followed by verification tasks
- Plans without a validation strategy are INCOMPLETE
- Pattern: Plan → Review → Execute → Verify → Iterate
- NEVER: Plan → Generate → Done

## GitHub Issue Cross-Referencing
- When a plan task maps to a GitHub Issue, add `(GitHub: #N)` after the task title
- Create a `plan:NNN` label on the repo and tag all related issues
- Issues without a `plan:` label are standalone backlog items
- When completing a plan task, close the corresponding issue
- Not every task needs an issue (e.g., manual/validation tasks) and not every issue needs a plan task (backlog items)

## Plan Numbering
- Auto-increment: scan `Plans/` for highest NNN, use NNN+1
- Format: `Plans/001-plan-[slug].md`
- Archive completed plans: move to `Plans/archive/`
