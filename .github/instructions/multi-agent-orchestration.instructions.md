---
description: "Multi-agent orchestration, agent patterns, supervisor, router, swarm, hierarchical, pipeline, coordination"
applyTo: "**"
---
# Multi-Agent Orchestration — Core Patterns

## Architecture Patterns
1. **Supervisor** — Central orchestrator delegates to workers, aggregates results. Good for control, bottleneck risk.
2. **Router** — Dispatcher routes tasks to best-fit agent. Low overhead, high throughput.
3. **Hierarchical** — Tree of managers → supervisors → workers. Multi-stage pipelines.
4. **Swarm/Peer** — Agents coordinate peer-to-peer, emergent behavior. Best for parallel exploration.
5. **Pipeline** — Linear chain A → B → C. Sequential dependent steps.
6. **Hybrid** — Production systems blend patterns (e.g., hierarchical + swarm).

## Design Principles
- Start simple, scale up — single agent first, multi-agent only when needed
- Narrow agent scope — one composable responsibility per agent
- Contract-driven interfaces — JSON schemas, explicit role declarations, clear handoffs
- Validate everything — independent verification at every agent boundary

## This Brain's Target Architecture
We are building toward a **hierarchical + router hybrid**:
- Agents as specialized personas (already in `.github/agents/`)
- Skills as composable capabilities (already in `.github/skills/`)
- Instructions as shared knowledge (already in `.github/instructions/`)
- Future: orchestrator agent that delegates to specialist agents

> Full reference: `multi-agent-orchestration.detail.instructions.md`
