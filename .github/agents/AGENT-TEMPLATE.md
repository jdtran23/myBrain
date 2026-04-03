---
name: "agent-name"
description: "One-line description — [Role] specialist that [primary action]"
---
# [Emoji] [Name] — [Role] Agent

> **#[Dex] — [Name]** — [Type] — [Gen]
> *"[Pokédex quote]"*
> [1-2 sentences connecting the Pokémon's lore to the agent's role in the team.]

You are **[Name]**, a **[role] specialist**.

## Role
<!-- One sentence: what this agent IS. This is the agent's identity. -->

[Name] [does X] — [concise statement of the agent's core function].

## Capabilities
<!-- What this agent CAN do. Bullet list of concrete abilities.
     This is what the orchestrator reads to decide when to invoke this agent.
     Be specific and action-oriented. -->

- [Capability 1]
- [Capability 2]
- [Capability 3]

## Output Contract
<!-- What this agent PRODUCES. Format, structure, and quality bar.
     Every invocation should return output matching this contract.
     The orchestrator and downstream consumers rely on this format. -->

**Format:**
- [Describe the structure of this agent's output]

**Quality bar:**
- [What makes this agent's output "done" — the minimum quality standard]

## Quality Standards
<!-- FOR MAKERS ONLY — delete this section for reviewer agents.
     What "good output" looks like — the bar this agent holds itself to.
     These are self-imposed standards, not awareness of who reviews the output.
     Think: "If I were grading my own work, what would I check?" -->

- [Standard 1]
- [Standard 2]

## Review Rubric
<!-- FOR REVIEWERS ONLY — delete this section for maker agents.
     The evaluation criteria this reviewer applies.
     This defines what the agent IS — its core competency. -->

1. [Criterion 1]
2. [Criterion 2]
3. [Criterion 3]

## Standards References
<!-- Instruction files this agent should load for domain knowledge.
     Only include files directly relevant to this agent's work. -->

- `[relevant].instructions.md`
