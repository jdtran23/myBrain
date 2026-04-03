---
name: "orchestration-evolution"
description: "Analyze brain state and recommend when and how to evolve toward multi-agent orchestration"
---
# orchestration-evolution — Multi-Agent Growth Advisor

> Analyzes your brain's current state and recommends when and how to evolve toward multi-agent orchestration. This is your "am I ready?" command.

## Triggers
- "evolve agents" / "agent evolution"
- "orchestration check" / "orchestration review"
- "am I ready for agents?"
- "compose agents"
- "agent architecture review"

## Maturity Model

The brain evolves through 4 phases. This skill identifies where you are and what to do next.

### Phase 1: Single Agent + Knowledge (Foundation)
**You are here when:** You have instructions and skills but no specialist agents (or just one).
**Signals to move on:**
- 3+ distinct domain areas in your instructions (e.g., C#, testing, deployment, security)
- Any instruction detail file exceeds 200 lines
- You find yourself context-switching between very different tasks in one chat
- You wish Copilot had "different modes" for different work

**Action:** Create your first specialist agents. Start with 2-3 that match your most distinct workflows.

### Phase 2: Specialist Agents (Router)
**You are here when:** You have 2+ agents with distinct scopes.
**Signals to move on:**
- You frequently need to chain work across agents (research → code → review)
- You're manually copying context between agent conversations
- Tasks naturally decompose into "ask agent A, then agent B"
- You have 4+ agents with clear boundaries

**Action:** Create workflow prompts (`.prompt.md`) that chain agent handoffs. Consider an orchestrator agent.

### Phase 3: Orchestrator (Supervisor)
**You are here when:** You have an orchestrator agent that delegates to specialists.
**Signals to move on:**
- Some tasks benefit from parallel exploration (multiple agents investigating simultaneously)
- You need agents to coordinate without you as the intermediary
- Your orchestrator is becoming a bottleneck

**Action:** Add parallel fan-out patterns. Create coordination protocols. Implement validation at handoff boundaries.

### Phase 3→4 Evolution Indicator: Maker/Reviewer Completeness
All maker domains should have paired reviewers with independent verification:
- Code: @porygon ↔ @absol
- Plans: @metagross ↔ @cresselia
- Research: @uxie ↔ @tapu-fini

When all pairs are active with severity systems and iteration limits, you're ready for Phase 4.

### Phase 4: Multi-Agent Workflows (Hybrid)
**You are here when:** Agents coordinate autonomously with validation at every boundary.
**This is the target state.** Continue refining coordination patterns, failure handling, and observability.

---

## How to Run an Orchestration Review

When triggered, perform this analysis:

### Step 1: Inventory
Scan the brain and report:
```
INSTRUCTIONS:  [count] stubs + [count] detail files
SKILLS:        [count] skills
AGENTS:        [count] agents
PROMPTS:       [count] prompts
TSG ENTRIES:   [count] lessons learned
DOMAINS:       [list distinct topic areas found across all files]
```

### Step 2: Identify Phase
Match current state against the maturity model above. Report:
```
CURRENT PHASE: [1-4]
PHASE NAME:    [name]
```

### Step 3: Check Evolution Signals
For the current phase, check each "signal to move on" and report:
```
EVOLUTION SIGNALS:
  ✅ [signal that IS present]
  ❌ [signal that is NOT yet present]

READY TO EVOLVE: [yes/no]
```

### Step 4: Recommend Next Actions
If ready to evolve, recommend specific actions:
- Which agents to create (based on domain clusters in instructions)
- Which skills to compose into agent workflows
- Which instructions contain enough depth to become agent expertise
- What coordination patterns to use (based on multi-agent-orchestration.detail.instructions.md)

If NOT ready, recommend what to build next to get there.

### Step 5: Agent Composition Recommendations
When recommending new agents, follow this template:

**Proposed Agent: `[name]`**
- **Scope:** [what this agent does]
- **Knowledge sources:** [which instruction files it draws from]
- **Skills it uses:** [which skills it invokes]
- **Inputs:** [what it receives]
- **Outputs:** [what it produces]
- **Handoff to:** [which other agents it passes work to]

---

## Domain Clustering Rules

To identify when a domain is ready to become an agent:

1. **Knowledge depth** — Has both a stub AND a detail file (100+ lines in detail)
2. **Distinct workflow** — The domain has its own "how to do X" pattern, not just rules
3. **Tool affinity** — The domain uses specific tools others don't (e.g., Playwright for research, dotnet CLI for C#)
4. **Frequency** — You work in this domain regularly (not a one-off)
5. **Boundary clarity** — You can clearly say what this agent does and does NOT do

If 3+ of these are true → recommend creating a specialist agent.

---

## Generic vs Repo-Specific Agent Split

When composing agents, maintain the separation principle:
- **Generic agents** (researcher, reviewer, planner) — reusable across any repo
- **Repo-specific agents** (e.g., "SDL expert", "auth-service specialist") — tied to one codebase
- Generic agents consume generic instructions; repo-specific agents additionally consume repo-specific instructions
- Keep them in separate files, clearly labeled

---

## Anti-Patterns to Flag

- ❌ **God agent** — one agent that does everything (split it)
- ❌ **Orphan knowledge** — deep instruction files no agent is wired to use
- ❌ **Missing contracts** — agents hand off work without structured input/output schemas
- ❌ **No validation** — agent output accepted without verification
- ❌ **Duplicate scope** — two agents covering the same domain (merge or clarify boundary)
- ❌ **Unreviewed maker output** — maker agent produces artifacts that flow into decisions without independent review
