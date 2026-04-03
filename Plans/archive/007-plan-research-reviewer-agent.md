# Plan 007: Research Reviewer Agent

## Objective
Fill the maker/reviewer gap for research by creating a dedicated Research Reviewer agent. This completes the symmetry of the orchestration model where every maker phase has an independent reviewer:

| Domain | Maker | Reviewer |
|--------|-------|----------|
| Code | @porygon | @absol |
| Plans | @metagross | @cresselia |
| Research | @uxie | **@tapu-fini** ← new |

## Design Tension: Plan 006 Said "No" to This Agent
Plan 006's Research Loop design had Mewtwo acting as research evaluator, arguing:
> *"A dedicated research reviewer agent would add complexity without clear benefit — the evaluation criteria are structural, not domain-specific."*

**Why we're overriding that decision:**
1. **Symmetry principle** — the brain's core pattern is maker/reviewer pairs with independent verification at every boundary. Research is the only domain where the orchestrator doubles as the reviewer, breaking the pattern.
2. **Separation of concerns** — Mewtwo context-switches between orchestrating the overall workflow and evaluating research quality. A dedicated reviewer lets Mewtwo focus on routing/coordination.
3. **Review depth** — Mewtwo checks structural completeness ("are there citations?") but a dedicated reviewer can assess source reliability, detect hallucinations, evaluate trade-off coverage, and catch bias — tasks that require focused analytical attention.
4. **Scalability** — as the brain grows, research quality becomes a bottleneck. A dedicated reviewer is investable (can evolve its rubric, learn domain patterns).

**Impact on Plan 006:** If Plan 006 hasn't been executed yet, its Research Loop design (Loop 1: Uxie ↔ Mewtwo) should be updated to Uxie ↔ Tapu Fini, with Mewtwo still driving the loop. If Plan 006 has been executed, this plan retrofits the change.

## AI Usage Declaration
- **AI will generate:** Agent `.agent.md` file, updates to existing agent files, instruction file updates, skill file updates
- **AI will NOT decide:** Final Pokémon name choice (decided: Tapu Fini), whether this agent is actually needed (Joe's call), whether Plan 006's research loop should be retroactively changed (Joe decides)
- **Expected failure modes:**
  - Agent scope creep — reviewer tries to also do research (must stay pure reviewer)
  - Rubric too rigid — reviewer checklist becomes checkbox theater instead of genuine quality assessment
  - Handoff format mismatch — new agent's expected input doesn't match Uxie's output format
  - Over-engineering — agent file bloats beyond 150 lines, LLM ignores nuance
  - Orphan references — updating some files but missing others, creating dead handoffs

## Pokémon Name Decision

**Chosen: Tapu Fini** — Water/Fairy island guardian that tests visitors with fog of confusion; only the truthful pass through. Thematic fit: validates research by testing its truthfulness before it flows downstream.

Agent file: `tapu-fini.agent.md`

---

## Tasks

### Phase 1: Agent Design

- [x] **Task 1.1:** Design `tapu-fini.agent.md` — the Research Reviewer agent file
  - **Content:**
    - YAML frontmatter: name, description, no special tools needed
    - Pokémon lore header (matching absol/cresselia pattern)
    - Core Behavior: high signal-to-noise, cite evidence, severity levels, constructive feedback
    - Research Review Checklist:
      - Source reliability (freshness, authority, deprecated content)
      - Completeness (alternative approaches, trade-offs, missing perspectives)
      - Accuracy/hallucination (do conclusions follow from cited evidence?)
      - Bias detection (over-reliance on single sources, vendor bias)
      - Actionability (structured for next consumer: brain-manager, @metagross, @porygon)
    - Severity system: 🔴 Critical / 🟡 Warning / 🔵 Suggestion (matching Absol/Cresselia pattern)
    - Output format (matching reviewer agent pattern — see Absol/Cresselia for template)
    - Loop participation protocol: how to handle iterations from Mewtwo (receive findings → review → provide feedback → Uxie iterates → re-review delta)
    - What You DON'T Do section
    - Handoffs: findings need deeper research → @uxie; findings affect plan → @metagross; systemic patterns → brain-manager
    - Scope boundary: operates on T2+ research tasks only. T1 quick lookups bypass review. Mewtwo decides tier classification.
  - **Constraints:** Under 150 lines. Pure reviewer — no research, no code, no planning.
  - **Acceptance criteria:**
    - File follows the pattern established by absol.agent.md and cresselia.agent.md
    - Review checklist covers all 5 areas from the task description
    - Severity system is consistent with other reviewer agents
    - Handoffs are bidirectional (Tapu Fini knows who hands off TO it and who it hands off TO)
    - File is ≤150 lines
  - **Verify:** Read the file, confirm it doesn't overlap with Uxie's scope (research) or Cresselia's scope (plan review). Confirm severity system matches Absol's. Confirm output format is consumable by Mewtwo.

### Phase 2: Integration Updates

- [x] **Task 2.1:** Update `mewtwo.agent.md` — Add Tapu Fini to the team table and delegation rules
  - **Changes:**
    - Add Tapu Fini to "Your Team" table with scope and when-to-use
    - Add research review routing: after Uxie produces findings, route to Tapu Fini for review (replaces Mewtwo-as-evaluator pattern)
    - Update Research Loop definition: Uxie ↔ Tapu Fini (Mewtwo drives the loop but doesn't evaluate)
    - Update effort tier routing: T1 research (quick lookups) goes directly from Uxie to consumer — no Tapu Fini review. T2+ research routes through Tapu Fini before feeding into decisions.
    - Update effort tier T3: full pipeline now includes research review step
    - Update delegation rules task-type table to include "Review research output" → @tapu-fini
  - **Acceptance criteria:**
    - Tapu Fini appears in team table with clear scope boundary
    - Research loop updated from Uxie↔Mewtwo to Uxie↔Tapu Fini
    - At least T2+ research tasks route through Tapu Fini
    - File stays ≤185 lines (current ~160 + Tapu Fini additions)
  - **Verify:** Walk through a T3 scenario (e.g., "Add WebSocket support") — confirm Uxie researches, Tapu Fini reviews, then findings flow to Metagross for planning. No dead ends.
  - **Dependencies:** Task 1.1

- [x] **Task 2.2:** Update `uxie.agent.md` — Add handoff to Tapu Fini for research review
  - **Changes:**
    - Create a Handoffs section in uxie.agent.md (currently missing — align with Absol/Cresselia pattern) and add: Research complete → @tapu-fini for review
    - Add awareness of Research Review loop: Uxie knows its output will be reviewed, knows the quality rubric, knows how to iterate on feedback
    - Update output format if needed to align with what Tapu Fini expects as input
  - **Acceptance criteria:**
    - Uxie's handoff section includes Tapu Fini
    - Uxie knows the review rubric (so it can produce quality-first output)
    - Output format is compatible with Tapu Fini's expected input
  - **Verify:** Confirm Uxie's output format matches what Tapu Fini's review checklist evaluates. No format mismatch.
  - **Dependencies:** Task 1.1

- [x] **Task 2.3:** Update `multi-agent-orchestration.instructions.md` — Document maker/reviewer pattern as formal principle
  - **Changes:**
    - Add a "Maker/Reviewer Pairs" section documenting the pattern as a core principle:
      - Every maker domain has an independent reviewer
      - Reviewers use severity systems (🔴/🟡/🔵)
      - Review loops have max iterations with escalation to human
      - Table of all maker/reviewer pairs
    - Update the "Design Principles" section to include: "Independent review at every maker boundary — no agent's output is trusted without peer review"
    - Add Research Loop definition: Uxie ↔ Tapu Fini (Mewtwo drives the loop).
  - **Acceptance criteria:**
    - Maker/reviewer pattern is explicitly documented as a principle, not just implied
    - All three pairs are listed (Code, Plans, Research)
    - Pattern includes iteration limits, escalation, and severity conventions
  - **Verify:** Read the file, confirm the new section is consistent with existing Commit Gate section. Confirm no contradictions with agent files.
  - **Dependencies:** Task 1.1

### Phase 3: Ecosystem Updates

- [x] **Task 3.1:** Update `orchestration-evolution/SKILL.md` — Reflect new agent in maturity model
  - **Changes:**
    - Update domain clustering examples to include "research review" as a distinct domain
    - If there's an agent inventory section, ensure it accounts for 7 agents (not 6)
    - Add "maker/reviewer completeness" as a maturity signal: all maker domains having paired reviewers is a Phase 3→4 evolution indicator
    - Update anti-patterns: add "Unreviewed maker output" (maker agent produces artifacts that flow into decisions without independent review)
  - **Acceptance criteria:**
    - Skill recognizes the research reviewer as part of the agent inventory
    - Maker/reviewer completeness is a trackable maturity signal
    - New anti-pattern is documented
  - **Verify:** Run the skill's inventory checklist mentally — does it correctly count and classify all 7 agents?

- [x] **Task 3.2:** Update `copilot-instructions.md` — Update directive index if agent count or architecture description is mentioned
  - **Changes:**
    - If the file mentions specific agents or agent counts, update to include Tapu Fini
    - If architecture description references the team, update it
  - **Acceptance criteria:**
    - Top-level brain file doesn't contradict the new team composition
  - **Verify:** Read the file, confirm no stale references to "6 agents" or team composition that excludes Tapu Fini

### Phase 4: Validation

- [x] **Task 4.1:** Cross-reference validation — Verify all handoffs are bidirectional and no orphan references
  - **Checks:**
    - Tapu Fini's "receives from" matches Mewtwo's "sends to Tapu Fini" and Uxie's "hands off to Tapu Fini"
    - Tapu Fini's "hands off to" targets all acknowledge receiving from Tapu Fini
    - No agent file references Tapu Fini without Tapu Fini existing
    - No dead-end handoffs (agent A mentions agent B, but agent B doesn't know about agent A)
    - Research loop participants agree on: max iterations, feedback format, escalation path
  - **Acceptance criteria:** Zero orphan references, zero unidirectional handoffs
  - **Verify:** Build a handoff matrix — for every A→B in any agent file, confirm B→A acknowledgment exists

- [x] **Task 4.2:** Brain health scan — Confirm no structural issues introduced
  - **Checks:**
    - All `.agent.md` files have valid YAML frontmatter (name, description)
    - No duplicate agent scopes (Tapu Fini doesn't overlap Cresselia's plan review or Absol's code review)
    - Instruction files referenced by agents actually exist
    - Skills reference correct agent names
  - **Acceptance criteria:** Clean brain health — no missing files, no scope overlaps, no broken references
  - **Verify:** List all agents, their scopes, and verify no two agents claim the same domain

- [x] **Task 4.3:** Scenario dry-run — Walk through 4 research-involving scenarios
  - **Scenario A1 (T1 bypass):** "What's the default port for Redis?" — Uxie answers directly → consumer. No Tapu Fini. Proves T1 bypass works.
  - **Scenario A2 (T2):** "Research WebSocket best practices for our API" — Uxie researches → Tapu Fini reviews → findings delivered. Proves T2 routing.
  - **Scenario B (T3 full pipeline):** "Add Redis caching layer" — Uxie researches → Tapu Fini reviews → Metagross plans with reviewed findings → Cresselia reviews plan → Porygon implements → Absol reviews code.
  - **Scenario C (brain update):** "Update the brain with new MCP patterns" — research → review → brain-manager update.
  - For each: trace the exact agent sequence, confirm no gaps, confirm Tapu Fini adds value without adding friction
  - **Acceptance criteria:** All 4 scenarios route correctly with clear agent traces and no "what happens next?" gaps. Both T1 bypass and T2+ review paths are validated.
  - **Verify:** Each scenario has a complete agent trace from start to finish

### Phase 5: Review

- [x] **Task 5.1:** @cresselia review — Review this plan before execution
  - Send plan to Cresselia for quality review
  - **Note:** Revision 1 — initial review 2026-04-02 (1 🔴, 4 🟡, 2 🔵 — all fixed). Revision 2 — re-review 2026-04-02 (0 🔴, 1 🟡, 1 🔵 — applied). Final re-review pending.
  - **Acceptance criteria:** Cresselia verdict is ✅ APPROVED or 🟡 APPROVED WITH CHANGES (changes applied)
  - **Verify:** All 🔴 must-fix items from Cresselia are resolved

---

## Validation Strategy

### Before Generation (This Plan)
- Plan reviewed by @cresselia (Task 5.1) before execution begins
- Design tension with Plan 006 explicitly documented and resolved
- Pokémon name decision deferred to Joe (human judgment)

### During Generation (Implementation)
- Each file update verified against acceptance criteria in its task
- Agent file stays ≤150 lines (hard constraint)
- Cross-reference checks at every integration point

### After Generation (Integration)
- Full cross-reference validation (Task 4.1)
- Brain health scan (Task 4.2)
- Scenario dry-runs prove the system works end-to-end (Task 4.3)

### Expected Failure Modes & Mitigations
| Failure Mode | Likelihood | Mitigation |
|-------------|------------|------------|
| Scope overlap with Cresselia (both "review" agents) | Medium | Explicit scope boundary: Cresselia reviews PLANS, Tapu Fini reviews RESEARCH. Neither crosses into the other. |
| Tapu Fini becomes rubber stamp (always approves) | Medium | Severity system with concrete criteria, not vague "is it good?". Checklist is specific and falsifiable. |
| Uxie-Tapu Fini loop adds latency for simple lookups | Low | Only T2+ research routes through Tapu Fini. T1 quick lookups skip review. Mewtwo decides tier. |
| Handoff format mismatch (Uxie output ≠ Tapu Fini input) | Medium | Tasks 2.2 and 4.1 explicitly verify format alignment. |
| Agent file bloat | Low | Hard 150-line cap enforced in Task 1.1 acceptance criteria. |

## Acceptance Criteria (Plan-Level)
- [x] New agent file exists at `.github/agents/tapu-fini.agent.md`
- [x] Mewtwo and Uxie agent files updated. Remaining 4 agents (Absol, Cresselia, Metagross, Porygon) verified to not need changes (confirmed in Task 4.2 brain health scan).
- [x] Maker/reviewer pattern documented as formal orchestration principle
- [x] All handoffs are bidirectional — no orphan references
- [x] Brain health check passes — no structural issues
- [x] 4 scenario dry-runs pass with clear agent traces (including T1 bypass validation)

## Dependencies
- **Plan 006 status:** This plan is compatible whether Plan 006 has been executed or not. If executed, this retrofits the research loop. If not, this can be incorporated into Plan 006 execution.
- **No code changes:** This is entirely brain-directive work (`.github/` files only)
- **No external dependencies:** No packages, APIs, or tooling required

## Definition of Done
Plan is DONE when all plan-level ACs pass, Task 4.3 dry-runs succeed, and no 🔴 findings remain from Cresselia review.
