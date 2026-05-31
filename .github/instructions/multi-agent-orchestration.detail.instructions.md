---
description: "Multi-agent orchestration v2 deep reference — research findings, workflow state protocol, gate specifications, HITL policy, failure modes, framework landscape"
---
# Multi-Agent Orchestration — Detailed Reference (v2)

## Research Foundation

Architecture decisions grounded in T3 deep research (April 2026), validated by @tapu-fini (19.5/25).

**Key sources:** Anthropic "Building Effective Agents" (Dec 2024), Anthropic multi-agent research system (Jun 2025), OpenAI Agents SDK (2025-2026), Google ADK (2026), MAST paper arXiv:2503.13657 (Mar 2025), Andrew Ng agentic design patterns (2024), Microsoft multi-agent reference architecture (2025).

### Validated Findings

1. **Simple, composable patterns outperform complex frameworks.** Anthropic's #1 finding: "The most successful implementations weren't using complex frameworks. They were building with simple, composable patterns." Note: this is Anthropic's philosophy — AutoGen and LangGraph argue for structured complexity in certain domains.

2. **The Evaluator-Optimizer pattern (maker/reviewer) has strong cross-vendor validation.** Anthropic codifies it as a first-class workflow. Google ADK ships Review/Critique and Iterative Refinement patterns. Andrew Ng identifies Reflection as a core agentic design pattern that dramatically improves output quality.

3. **Define agents positively — capabilities, not exclusions.** Every major framework (OpenAI Agents SDK, Google ADK, Anthropic) defines agents through instructions, tools, description, and handoff_conditions. No framework has a "don't do" field. Tools provide implicit negative scope. Note: implicit constraints can be encoded through tool restrictions and routing logic.

4. **"Agents as Tools" is the dominant composition mechanism.** OpenAI (Agent.as_tool()), Google ADK (AgentTool), and Anthropic (subagent spawning) all converge on treating agents as callable tools. The orchestrator maintains control while letting specialists execute bounded subtasks.

5. **Most multi-agent failures relate to coordination, system design, and task verification** — not technical bugs. The MAST paper (arXiv:2503.13657) identified 14 failure modes across 3 categories: system design issues, inter-agent misalignment, and task verification failures.

6. **Two orchestration modes: LLM-driven (flexible) vs code-driven (predictable).** OpenAI Agents SDK makes this the central architectural decision. Production systems blend both.

### Research Departure

Our architecture departs from raw research in one area: the research recommended including `handoff_triggers` in agent capability manifests. We chose to keep ALL routing logic in the orchestrator for cleaner composability. This was a deliberate architectural decision.

---

## Architecture Patterns Reference

### Orchestrator-Workers (our primary pattern)
- Central orchestrator decomposes tasks, delegates to specialists, aggregates results
- Workers focus on single capabilities, don't communicate with each other
- **Pros:** Easy to debug, predictable, good error tracing, full visibility
- **Cons:** Bottleneck at orchestrator for high-parallelism tasks
- **Our implementation:** @mewtwo as orchestrator, specialist agents as workers

### Evaluator-Optimizer / Maker-Reviewer (our review pattern)
- Generator produces output, critic evaluates against rubric, loop until quality bar met
- Anthropic: first-class workflow. Google ADK: LoopAgent + SequentialAgent
- Andrew Ng: "GPT-3.5 wrapped in an agent loop achieves up to 95.1% on HumanEval, versus 48.1% zero-shot"
- **Our implementation:** maker/reviewer pairs with max 2 iterations, human escalation

### Router (embedded in orchestrator)
- Classify task → route to best-fit agent based on capabilities
- Our approach: capability-based matching from task decomposition, not upfront tier classification
- **Our implementation:** @mewtwo decomposes task, matches subtask needs to agent capabilities

### Other Patterns (reference)
- **Hierarchical** — tree of managers → supervisors → workers. Good for multi-stage pipelines.
- **Swarm/Peer** — peer-to-peer coordination, emergent behavior. Good for parallel exploration.
- **Pipeline** — linear chain A → B → C. Good for sequential dependent steps.
- **Parallel fan-out** — spawn multiple agents concurrently, aggregate results.

---

## Workflow State Protocol Specification

### Purpose
Provide full transparency on multi-step workflow progress. The user always knows: what phase we're in, who's working, what gates are pending, and what's been done.

### Ownership
@mewtwo owns all state updates. Agents report completion through their output; @mewtwo interprets and updates the state block. No agent file needs awareness of the state format.

### Format
```
═══════════════════════════════════════════════════════════════
WORKFLOW: [task name]
PLAN: [Plans/NNN-plan-xxx.md if applicable]
STATUS: [🔄 In Progress | ⏸️ Blocked | ✅ Complete]
═══════════════════════════════════════════════════════════════
Phase N: [name]             [✅ Complete | 🔄 Active | ⬜ Pending]
  └─ @[agent] ([action])    [✅ Done | 🔄 Working... | ⬜]
  └─ @[agent] ([action])    [✅ Verdict (summary) | 🔄 Reviewing... | ⬜]
  └─ 🚦 GATE: [gate name]  [✅ Passed | ❌ Failed | ⏳ Pending]
═══════════════════════════════════════════════════════════════
```

### Update Rules
- Update when any agent starts or finishes work
- Include review verdicts and finding counts in history
- Show gate status clearly — blocked progress should be visible at a glance
- For simple tasks (1-2 steps): use lightweight inline status, not full block

### Examples

**Simple task — "Fix the typo in README"**
```
@porygon (fix) ✅ → @absol (review) ✅ PASS → 🚦 commit ✅
```

**Standard task — "Research GraphQL and update brain"**
```
═══════════════════════════════════════════════════
WORKFLOW: Research GraphQL
STATUS: 🔄 In Progress
═══════════════════════════════════════════════════
Phase 1: Research              🔄 Active
  └─ @uxie (research)          ✅ Done
  └─ @tapu-fini (review)       🔄 Reviewing...
  └─ 🚦 GATE: research-review  ⏳ Pending

Phase 2: Brain Update          ⬜ Pending
  └─ brain-manager skill       ⬜
═══════════════════════════════════════════════════
```

**Full pipeline — "Add Redis caching"**
```
═══════════════════════════════════════════════════════════════
WORKFLOW: Add Redis Caching
PLAN: Plans/010-plan-redis-caching.md
STATUS: 🔄 In Progress
═══════════════════════════════════════════════════════════════
Phase 1: Planning              ✅ Complete
  └─ @metagross (plan)          ✅ Done (Plans/010-plan-redis-caching.md)
  └─ @cresselia (review)        ✅ APPROVED (0🔴, 2🟡 fixed)
  └─ 🚦 GATE: plan-review       ✅ Passed
  └─ 🦆 rubber-duck (plan)      ✅ PASS (0🔴, 1🟡 accepted)
  └─ 🚦 GATE: rubber-duck       ✅ Passed

Phase 2: Research              ✅ Complete
  └─ @uxie (Redis patterns)     ✅ Done
  └─ @tapu-fini (review)        ✅ PASS (22/25)
  └─ 🚦 GATE: research-review   ✅ Passed
  └─ 🦆 rubber-duck (research)  ✅ PASS (no blind spots)
  └─ 🚦 GATE: rubber-duck       ✅ Passed

Phase 3: Implementation        🔄 Active
  └─ @porygon (implement)       🔄 Task 3/5
  └─ @absol (review)            ⬜
  └─ 🚦 GATE: code-review       ⬜
  └─ 🚦 GATE: commit            ⬜
═══════════════════════════════════════════════════════════════
```

---

## Gate Specifications

### Plan Review Gate
- **Trigger:** @metagross creates a plan, before any execution begins
- **Agent:** @cresselia
- **Pass criteria:** Verdict is APPROVED or APPROVED WITH CHANGES (after changes applied)
- **Fail action:** Enter review loop — send findings to @metagross for revision, re-review
- **Max iterations:** 2 before human escalation

### Research Review Gate
- **Trigger:** @uxie produces research findings, before findings feed into planning or implementation
- **Agent:** @tapu-fini
- **Pass criteria:** Verdict is PASS or PASS WITH REVISIONS (after revisions applied)
- **Fail action:** Enter review loop — send findings to @uxie for revision, re-review
- **Max iterations:** 2 before human escalation

### Rubber-Duck Gate
- **Trigger:** Immediately after the Research Review Gate or Plan Review Gate passes — adds a second independent critique layer on the two highest-leverage artifacts (research, plans) before they drive downstream work
- **Agent:** rubber-duck (built-in critique agent, distinct from domain reviewers)
- **Pass criteria:** Zero 🔴 Critical findings remaining; 🟡 Warning findings either resolved by the maker or explicitly accepted with rationale documented
- **Fail action:** Send findings back to the original maker (@uxie for research, @metagross for plans). Maker revises. Domain reviewer (@tapu-fini / @cresselia) re-reviews. Rubber-duck re-runs.
- **Max iterations:** 1 rubber-duck round after the first pass. If blind spots persist after revision, escalate to human.
- **Rationale:** The domain reviewer is a specialist focused on that artifact's quality criteria. Rubber-duck is a generalist focused on blind spots, hidden assumptions, missing edge cases, and design flaws. Specialist + generalist = stronger validation.
- **Scope:** Currently mandatory for research and plans only. Not currently applied to code (@absol already provides bug-focused critique) or game design (@klefki provides design critique).

### Code Review Gate
- **Trigger:** @porygon writes or modifies code, before code is considered done
- **Agent:** @absol
- **Pass criteria:** Zero 🔴 Critical and zero 🟡 Warning findings remaining
- **Fail action:** Enter review loop — send findings to @porygon for fix, re-review
- **Max iterations:** 2 before human escalation

### Commit Gate
- **Trigger:** After code review gate passes, before reporting implementation complete
- **Required sequence:** Build passes → Tests pass → Code review gate passed
- **Pass criteria:** All three green
- **Fail action:** Fix the failing step, re-run from that step
- **Hard rule:** Never skip code review. "Build + tests pass" alone is NOT sufficient.

---

## HITL Policy

### Trigger Definitions

| # | Trigger | Condition | Escalation Action |
|---|---------|-----------|-------------------|
| 1 | **Destructive action** | File deletion, git force ops, production changes, external comms | Pause and request explicit approval before proceeding |
| 2 | **Iteration limit** | Review loop hits max (2) without resolution | Present the disagreement, both perspectives, and ask for judgment |
| 3 | **Agent disagreement** | Reviewer and maker cannot converge on approach | Present both approaches with trade-offs, ask human to decide |
| 4 | **Novel situation** | Task type not previously encountered, no matching pattern | Describe situation, propose approach, ask for approval |
| 5 | **Scope ambiguity** | Task intent unclear after decomposition attempt | Ask clarifying questions before proceeding |

### Escalation Format
```
⚠️ ESCALATION NEEDED
Trigger: [which trigger — e.g., "Iteration limit reached"]
Context: [what happened — brief summary]
Options: [A, B, or C — with trade-offs for each]
```

---

## Failure Modes & Recovery

### Common Failure Modes (MAST paper + cross-vendor)

| Failure | Description | Mitigation |
|---------|-------------|------------|
| Coordination breakdown | Ambiguous roles, unclear task definitions | Capability manifests, output contracts, explicit routing |
| Cascading errors | Empty/bad output silently propagates | Output validation at every agent boundary |
| Infinite loops | Agents retry or loop between each other forever | Max iterations (2), human escalation |
| State staleness | Agents act on outdated information | Orchestrator owns state, updates at every step |
| Over-spawning | Too many agents for simple tasks | Decomposition-driven depth, not classification |
| Scope leakage | Agent attempts work outside its capability | Tools provide implicit scope; orchestrator validates output |

### Recovery Patterns
- **Layered validation** — validate at every agent boundary, not just endpoints
- **Circuit breakers** — suspend workflow after repeated failures in same phase
- **Escalation** — human-in-the-loop for irrecoverable failures
- **Retry with feedback** — send specific error context back to the failing agent
- **Agent substitution** — if one agent can't solve it, try a different agent

---

## Framework Landscape (2026 reference)

| Framework | Primary Pattern | Strength |
|-----------|----------------|----------|
| **Anthropic** | Orchestrator-workers | Best documented production case study; simple composable workflows |
| **OpenAI Agents SDK** | Handoffs + Agent-as-tool | Production-ready; built-in tracing and guardrails |
| **Google ADK** | Hierarchical workflows | Most comprehensive taxonomy; model-agnostic; A2A support |
| **Microsoft AutoGen** | Conversational groups | Enterprise integration; Semantic Kernel |
| **LangGraph** | Stateful graphs | Fine-grained control; complex branching |
| **CrewAI** | Role-based teams | Fast prototyping; intuitive role metaphor |

## Interoperability Protocols

- **MCP (Model Context Protocol)** — Tool and context sharing across agent boundaries. Essential for scale.
- **A2A (Agent-to-Agent)** — Google's peer-to-peer agent communication standard.

---

## Sources

- Anthropic: "Building Effective Agents" (Dec 2024) — https://www.anthropic.com/engineering/building-effective-agents
- Anthropic: Multi-agent research system (Jun 2025) — https://www.anthropic.com/engineering/multi-agent-research-system
- OpenAI Agents SDK (2025-2026) — https://openai.github.io/openai-agents-python/
- Google ADK: Multi-Agent Systems (2026) — https://adk.dev/agents/multi-agents/
- MAST paper: "Why Do Multi-Agent LLM Systems Fail?" (Mar 2025) — https://arxiv.org/abs/2503.13657
- Andrew Ng: Agentic Design Patterns (2024) — https://www.deeplearning.ai/the-batch/how-agents-can-improve-llm-performance/
- Microsoft Multi-Agent Reference Architecture (2025) — https://github.com/microsoft/multi-agent-reference-architecture
- Azure AI Agent Design Patterns (2025) — https://learn.microsoft.com/en-us/azure/architecture/ai-ml/guide/ai-agent-design-patterns
- Google: Choose a design pattern for agentic AI (2025) — https://docs.cloud.google.com/architecture/choose-design-pattern-agentic-ai-system
- Multi-Agent Collaboration Mechanisms Survey (Jan 2025) — https://arxiv.org/abs/2501.06322
- IEEE SMC: MAS × LLM Architectural Synergies (Jun 2025) — https://www.ieeesmc.org/wp-content/uploads/2025/06/eNewsletter_FeatureArticle_June2025.pdf