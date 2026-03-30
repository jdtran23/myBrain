---
description: "Plan mode, AI validation guardrail, Plan-Generate-Verify-Iterate lifecycle"
applyTo: "**"
---
# Plan — AI Validation Guardrail

## Core Rule
AI output is **untrusted until validated**. Every task follows:
**Plan → Generate → Verify → Iterate**

## Rules
1. Every plan MUST include an AI validation section before implementation
2. AI generation tasks MUST be followed by verification tasks
3. Plans without a validation strategy are **incomplete**
4. Never: Plan → Generate → Done

## Workflow
- "create a plan" → plan-management skill creates validated plan in `Plans/`
- "work on plan [ID]" → execute tasks with verification at each step

> For full plan management: see `.github/skills/plan-management/SKILL.md`
