---
description: "Multi-agent orchestration v2 — architecture, composability, review loops, gates, workflow state, HITL"
applyTo: "**"
---
# Multi-Agent Orchestration — Core Architecture (v2)

## Architecture

This brain uses a **Hierarchical Orchestrator + Evaluator-Optimizer Hybrid**:

- **@mewtwo** (orchestrator) owns all routing, review loops, gate enforcement, and workflow state
- **Specialist agents** declare capabilities and output contracts — they never route or invoke other agents
- **Maker/reviewer pairs** implement the Evaluator-Optimizer pattern for quality assurance
- **Quality gates** enforce checkpoints between workflow phases
- **Human-in-the-loop** at strategic intervention points

> Full reference: `multi-agent-orchestration.detail.instructions.md`

## Design Principles

1. **Orchestrator owns all routing** — routing logic lives in @mewtwo only, never in specialist agents
2. **Define agents positively** — agents declare what they ARE and PRODUCE (capabilities + output contracts). No negative scope ("What You DON'T Do")
3. **Composability over configuration** — adding a new agent requires only: a new agent file + registry entry in @mewtwo. Zero changes to existing agents.
4. **Independent review at every maker boundary** — no agent's output is trusted without peer review
5. **Validate at every boundary** — check agent output before passing to the next phase
6. **Decomposition drives depth** — workflow complexity emerges from task decomposition, not upfront classification

## Agent Composability Contract

An agent file declares what the agent **IS** and what it **PRODUCES**. It never declares routing, handoffs, or negative scope.

- Agent files contain: Role, Capabilities, Output Contract, Quality Standards (makers) or Review Rubric (reviewers), Standards References
- Agent files do NOT contain: routing logic, handoff triggers, "What You DON'T Do", tier references
- The orchestrator is the sole consumer of agent capabilities for routing decisions
- Adding a new agent requires: creating a new agent file + updating @mewtwo's capability registry
- Template: `.github/agents/AGENT-TEMPLATE.md`

## Maker/Reviewer Pairs

Every maker domain has an independent reviewer. The orchestrator manages all review loops.

| Domain | Maker | Reviewer | Review Scope | Max Iterations |
|--------|-------|----------|-------------|----------------|
| Research | @uxie | @tapu-fini | Accuracy, sources, completeness, bias | 2 |
| Code | @porygon | @absol | Bugs, security, standards, edge cases | 2 |
| Plans | @metagross | @cresselia | Completeness, feasibility, risk, validation | 2 |
| Game Design | @rotom | @klefki | Fun, balance, coherence, scope, player agency | 2 |

**Standalone specialists** (no reviewer pair):

| Agent | Domain | Scope |
|-------|--------|-------|
| @gardevoir | Teaching | Concept explanations, learning exercises, skill scaffolding |

**Review loop:** Orchestrator calls maker → calls reviewer → on PASS, proceed. On REVISIONS/FAIL, send findings back to maker (iteration +1). After max iterations without resolution → escalate to human.

**Severity system:** 🔴 Critical / 🟡 Warning / 🔵 Suggestion. All 🔴 and 🟡 must be resolved before PASS.

## Quality Gates

| Gate | Trigger | Agent | Pass Criteria | Fail Action |
|------|---------|-------|--------------|-------------|
| **Plan Review** | After plan creation, before execution | @cresselia | APPROVED or changes applied | Review loop with @metagross |
| **Research Review** | After research, before consumption | @tapu-fini | PASS or revisions applied | Review loop with @uxie |
| **Rubber-Duck** | After research-review or plan-review gate passes | rubber-duck (built-in) | Zero 🔴 findings; 🟡 findings resolved or explicitly accepted | Revision loop with original maker, then re-review |
| **Code Review** | After code changes, before done | @absol | Zero 🔴 and 🟡 findings | Review loop with @porygon |
| **Design Review** | After game design, before implementation | @klefki | Zero 🔴 and 🟡 findings | Review loop with @rotom |
| **Commit** | After code review passes | (verification) | Build + tests + review all pass | Fix and re-run from failure |

**Commit gate is mandatory.** "Build passes + tests pass" alone is NOT sufficient. @absol review must pass.

**Rubber-Duck gate is mandatory for research and plans** — research and plans are high-leverage artifacts that drive downstream work, so they get a second independent critique pass on top of their domain reviewer. Not currently required for code (@absol already provides bug-focused critique) or game design (@klefki provides design critique).

## Human-in-the-Loop

The orchestrator MUST escalate to human when:

| Trigger | Condition |
|---------|-----------|
| Destructive action | File deletion, git force operations, production changes |
| Iteration limit | Review loop hits max (2) without resolution |
| Agent disagreement | Reviewer and maker cannot converge |
| Novel situation | No matching workflow pattern or unknown domain |
| Scope ambiguity | Task intent unclear after decomposition |

## Workflow State

The orchestrator maintains a live status block during multi-step workflows, providing full transparency on phase, active agent, gate status, and history.

> Protocol details: `multi-agent-orchestration.detail.instructions.md`