# Plan 008: Multi-Agent Orchestration Redesign (v2)

## Objective
Redesign the multi-agent orchestration model to implement the 9 locked design decisions from Joe + Mewtwo's architecture session. All routing logic consolidates into @mewtwo. Agent files become positive capability declarations with output contracts. Workflow state becomes a live-tracked protocol. The system becomes composable — adding a new agent requires zero changes to existing agents.

## Background
- **Research:** T3 deep research by @uxie, validated by @tapu-fini at 19.5/25
- **Key sources:** Anthropic "Building Effective Agents," OpenAI Agents SDK, Google ADK, MAST paper, Microsoft multi-agent reference architecture
- **Architecture:** Hierarchical Router + Evaluator-Optimizer Hybrid
- **Design decisions:** 9 locked constraints (see below)
- **Research departure:** Decision #2 departs from raw research recommendation (which included `handoff_triggers` in agent manifests). The architecture session determined that all routing/handoff logic belongs exclusively in @mewtwo for cleaner composability. This was a deliberate choice, not an oversight.
- **Rollback strategy:** All changes are to tracked `.md` files. Rollback is `git checkout` per-file if needed.

## Design Decisions (LOCKED — constraints, not open questions)
1. Orchestrator owns ALL routing and review loops
2. No handoff triggers in agent files
3. No "What You DON'T Do" sections — define agents positively
4. No tier-based routing (T1/T2/T3) — decomposition drives depth
5. Workflow state protocol — live status block with phase, agent, gates, history
6. Subagent transparency — every invocation visible to user
7. Token cost is NOT a design constraint
8. Human-in-the-loop at strategic points (destructive actions, iteration limit [2 max], agent disagreements, novel situations)
9. Agents compose naturally — adding Agent Z requires zero changes to existing specialist Agents A-F. The orchestrator's capability registry is updated as a configuration change, not a coupling change.

## AI Usage Declaration
- **AI will generate:** Agent file templates, refactored agent content, updated instructions content, workflow state protocol format
- **AI will NOT decide:** Whether design decisions should change (locked), which agents to keep/remove, whether to merge the orchestration detail doc
- **Expected failure modes:**
  - Omitting routing logic that was in agent files without moving it to @mewtwo
  - Breaking the commit gate protocol during refactor
  - Creating implicit coupling between agents that violates composability
  - Workflow state protocol too verbose or too sparse for practical use
  - Losing the review rubric awareness from maker agents during positive-only rewrite

---

## Overall Success Metric
v2 is successful when: all Phase 5 validation tasks pass, brain health is clean, and the next real task (not a dry-run) routes correctly through the v2 architecture.

---

## Phase 1: Agent Definition Format & Template
**Goal:** Define the v2 agent file structure and create a reusable template.

- [ ] **Task 1.1:** Design the v2 agent file schema
  - Fields: `name`, `description` (YAML frontmatter), Pokémon lore block, `Role` (1 sentence), `Capabilities` (what this agent does — bullet list), `Output Contract` (what it produces, format, quality criteria), `Quality Standards` (for makers: what "good output" looks like — the bar the agent holds itself to, independent of who reviews it), `Standards References` (instruction files to load)
  - Explicitly EXCLUDED from agent files: handoff triggers, "What You DON'T Do", routing logic, tier references, workflow patterns
  - Verify: Template can be filled for any agent (maker, reviewer, orchestrator) without referencing other agents by name in routing sections
  - Verify: A hypothetical @rotom (DevOps) agent file can be created from this template without touching any other file

- [ ] **Task 1.2:** Create `AGENT-TEMPLATE.md` in `.github/agents/`
  - Concrete template file with placeholder sections and inline comments explaining each section
  - Verify: Template passes a self-consistency check — no section references another agent's behavior

- [ ] **Task 1.3:** Document composability contract
  - Write a short section (for inclusion in orchestration instructions) that states: "An agent file declares what the agent IS and PRODUCES. It never declares routing, handoffs, or negative scope. The orchestrator is the only consumer of agent capabilities for routing purposes."
  - Verify: This contract, combined with the template, makes it clear how to add a new agent

**Acceptance Criteria (Phase 1):**
- [ ] v2 schema documented with all fields defined
- [ ] Template file created in `.github/agents/`
- [ ] Composability contract written
- [ ] @rotom thought experiment passes: can write the file without modifying any existing file
- [ ] No field in the template references another agent's behavior or routing

**Dependencies:** None — this phase is standalone.

---

## Phase 2: Orchestrator Redesign (@mewtwo)
**Goal:** Rewrite `mewtwo.agent.md` as the single source of truth for routing, review loops, gate enforcement, and workflow state.

- [ ] **Task 2.1:** Define the Maker/Reviewer Registry
  - Central table in @mewtwo that maps domains to maker/reviewer pairs
  - Format: `| Domain | Maker | Reviewer | Review Scope | Max Iterations |`
  - This is the ONLY place that knows which reviewer gates which maker
  - Verify: Every current maker/reviewer pair is represented

- [ ] **Task 2.2:** Define the Review Loop Protocol
  - @mewtwo calls maker → receives output → calls reviewer → reads verdict → decides: accept, iterate (max 2), or escalate to human
  - Encoded as a clear protocol in @mewtwo's agent file, not as instructions scattered across agents
  - Verify: Protocol handles all three outcomes (pass, iterate, escalate)

- [ ] **Task 2.3:** Define Gate Definitions
  - Gates are checkpoints where @mewtwo validates quality before proceeding
  - Gate types: `plan-review` (after @metagross, before execution), `research-review` (after @uxie, before consumption), `code-review` (after @porygon, before commit), `commit` (build + test + review all pass)
  - Each gate definition includes: trigger condition, required agent, pass criteria, fail action
  - Verify: Commit gate matches existing mandatory pipeline (code → build → test → @absol → fix → commit)

- [ ] **Task 2.4:** Define the Workflow State Protocol
  - Live status block format that @mewtwo maintains during multi-step workflows
  - Must show: workflow name, current phase, active agent, gate status (pending/pass/fail), step history with timestamps
  - Format:
    ```
    WORKFLOW: [name]
    PHASE: [current phase]  |  AGENT: [active]  |  STATUS: [in progress/blocked/complete]
    GATES: plan-review ✅ | research-review ⬜ | code-review ⬜ | commit ⬜

    HISTORY:
    [1] ✅ @metagross — Plan created (Plans/008-plan-xxx.md)
    [2] ✅ @cresselia — APPROVED (0 critical, 1 warning fixed)
    [3] 🔄 @porygon — Implementing task 2/5
    ```
  - @mewtwo owns all workflow state updates — agents report completion through their output, @mewtwo interprets and updates state. No state protocol coupling in agent files.
  - Verify: Protocol is parseable and unambiguous at a glance
  - Verify: No agent file needs awareness of the state format

- [ ] **Task 2.5:** Define HITL (Human-in-the-Loop) Insertion Points
  - Codify when @mewtwo MUST pause and ask the human:
    1. Destructive actions (file deletion, git force operations, production changes)
    2. Review loop iteration limit reached (2 iterations without resolution)
    3. Agent disagreement (reviewer and maker can't converge)
    4. Novel situations (no matching workflow pattern, unknown domain)
    5. Scope ambiguity (task intent unclear after decomposition)
  - Verify: Each HITL point has a clear trigger condition and escalation message format

- [ ] **Task 2.6:** Define Agent Selection Logic
  - Replace tier-based routing with capability-based selection
  - @mewtwo decomposes task → for each subtask, matches required capability to agent registry
  - No upfront T1/T2/T3 classification — decomposition depth naturally determines how many agents are involved
  - Simple tasks decompose into 1-2 subtasks → 1 agent + reviewer
  - Complex tasks decompose into many subtasks → multiple agents, multiple gates
  - Verify: "What's the default port for Redis?" naturally routes to 1 agent without tier classification
  - Verify: "Research WebSocket patterns and implement caching" naturally spawns research + code + reviews without tier classification

- [ ] **Task 2.6b:** Pre-integration consistency check
  - Review all 6 sub-component definitions (Tasks 2.1-2.6) for consistency before integration
  - Verify: no conflicting terminology, no overlapping responsibilities, formats are compatible
  - Verify: registry format works with selection logic, gate definitions don't overlap with review loop protocol

- [ ] **Task 2.7:** Rewrite `mewtwo.agent.md`
  - Apply v2 template format for the orchestrator-specific sections
  - Include: Maker/Reviewer Registry, Review Loop Protocol, Gate Definitions, Workflow State Protocol, HITL Points, Agent Selection Logic
  - Remove: tier routing, "What You DON'T Do", old workflow patterns with hardcoded agent sequences
  - Keep: Pokémon lore, commit gate (restructured as a gate definition), core orchestrator identity
  - Verify: No routing logic references remain outside this file (checked in Phase 3 + Phase 4)

**Acceptance Criteria (Phase 2):**
- [ ] Maker/Reviewer Registry is complete (3 pairs: code, plans, research)
- [ ] Review Loop Protocol handles pass / iterate / escalate
- [ ] 4 gate definitions written with trigger/agent/criteria/fail-action
- [ ] Workflow State Protocol format defined with example
- [ ] 5 HITL insertion points codified
- [ ] Agent selection is capability-based, not tier-based
- [ ] `mewtwo.agent.md` rewritten with all above sections
- [ ] No "What You DON'T Do" section in the file
- [ ] No tier references (T1/T2/T3) in the file

**Dependencies:** Phase 1 (template format needed before writing @mewtwo).

---

## Phase 3: Specialist Agent Refactor
**Goal:** Rewrite all 6 specialist agents to the v2 format — positive definitions only, no routing logic.

**Execution order:** Refactor one maker+reviewer pair first (@porygon + @absol, Tasks 3.3 + 3.4) to validate the template works for both roles before batch-applying to all 6. The first pair is a learning step.

- [ ] **Task 3.1:** Refactor `uxie.agent.md` (researcher)
  - Apply v2 template: Role, Capabilities, Output Contract, Standards References
  - Remove: "What You DON'T Do", Handoffs section, "Research Review Awareness" section (reviewer awareness stays but reframed as output quality standards), tier references
  - Capabilities: web research, documentation reading, source synthesis, structured output generation
  - Output Contract: Key Takeaways + Detailed Findings + Sources + Brain Recommendation
  - Verify: No references to other agents by name in routing context (may reference review rubric standards)

- [ ] **Task 3.2:** Refactor `tapu-fini.agent.md` (research reviewer)
  - Apply v2 template: Role, Capabilities, Output Contract
  - Remove: "What You DON'T Do", Handoffs section, Loop Participation Protocol (loop logic moves to @mewtwo), tier scope boundary, references to @mewtwo deciding tier
  - Capabilities: research validation, source verification, completeness assessment, bias detection
  - Output Contract: Review format with severity levels + verdict (PASS / PASS WITH REVISIONS / FAIL)
  - Keep: Review Checklist (this is the agent's core competency), Review Rubric (this is what it IS)
  - Verify: No routing logic remains

- [ ] **Task 3.3:** Refactor `porygon.agent.md` (coder)
  - Apply v2 template: Role, Capabilities, Output Contract, Standards References
  - Remove: "What You DON'T Do", Handoffs section
  - Capabilities: code writing, refactoring, debugging, test writing, build verification
  - Output Contract: working code that compiles/runs, follows brain coding standards, includes brief explanation of approach
  - Standards References: csharp.instructions.md, python.instructions.md, typescript.instructions.md, powershell.instructions.md
  - Verify: No routing logic remains

- [ ] **Task 3.4:** Refactor `absol.agent.md` (code reviewer)
  - Apply v2 template: Role, Capabilities, Output Contract
  - Remove: "What You DON'T Do", Handoffs section
  - Capabilities: code review, security analysis, standards compliance checking, edge case identification
  - Output Contract: Review format with severity levels (🔴/🟡/🔵) + positive observations
  - Keep: Review Checklist (core competency), Output Format, Severity Guide
  - Verify: No routing logic remains

- [ ] **Task 3.5:** Refactor `metagross.agent.md` (planner)
  - Apply v2 template: Role, Capabilities, Output Contract, Standards References
  - Remove: "What You DON'T Do", Handoffs section
  - Capabilities: task decomposition, plan creation, AI validation design, acceptance criteria writing, dependency mapping
  - Output Contract: plan file in `Plans/` following plan-management skill format
  - Standards References: plan-management skill
  - Verify: No routing logic remains

- [ ] **Task 3.6:** Refactor `cresselia.agent.md` (plan reviewer)
  - Apply v2 template: Role, Capabilities, Output Contract
  - Remove: "What You DON'T Do", Handoffs section
  - Capabilities: plan evaluation, gap analysis, risk assessment, validation coverage checking, feasibility analysis
  - Output Contract: Review format with verdict (APPROVED / APPROVED WITH CHANGES / NEEDS REVISION) + severity findings
  - Keep: Review Checklist (core competency), Output Format, Severity Guide
  - Verify: No routing logic remains

- [ ] **Task 3.7:** Cross-agent consistency check
  - Read all 7 refactored agent files side-by-side
  - Verify: consistent structure across all files
  - Verify: no agent references another agent in routing context
- [ ] All maker agents have Quality Standards section (what good output looks like — not who reviews them)
  - Verify: all reviewer agents define their rubric without referencing who they review

**Acceptance Criteria (Phase 3):**
- [ ] All 6 specialist agents rewritten to v2 format
- [ ] Zero "What You DON'T Do" sections across all agent files
- [ ] Zero handoff/routing sections in specialist agents
- [ ] Zero tier references (T1/T2/T3) in specialist agents
- [ ] All agents have: Role, Capabilities, Output Contract
- [ ] Reviewer agents retain their review checklists and rubrics
- [ ] Maker agents have Quality Standards (not reviewer identities or rubric awareness)
- [ ] Cross-agent consistency check passes

**Dependencies:** Phase 1 (template), Phase 2 (@mewtwo rewrite — so we know what routing moved where).

---

## Phase 4: Instructions Update
**Goal:** Update orchestration instruction files to reflect v2 architecture.

- [ ] **Task 4.1:** Rewrite `multi-agent-orchestration.instructions.md`
  - **Keep:** Maker/Reviewer Pairs table, Design Principles, Commit Gate
  - **Remove:** Tier routing (T1/T2/T3 references), "This Brain's Target Architecture" (no longer aspirational — it's implemented), Research Review Loop (absorbed into @mewtwo's review loop protocol)
  - **Add:** v2 Architecture Overview (hierarchical router + evaluator-optimizer hybrid), Composability Principle, Workflow State Protocol reference, Gate Definitions reference, HITL Policy reference
  - **Update:** Design Principles to include: "Define agents positively," "Orchestrator owns all routing," "Composability > configuration"
  - **Update:** Maker/Reviewer Pairs section to remove "T1 quick lookups bypass review" and "@mewtwo decides tier"
  - Verify: No tier references remain. Architecture section matches what's actually implemented.

- [ ] **Task 4.2:** Update `multi-agent-orchestration.detail.instructions.md`
  - Incorporate validated research findings (7 key takeaways from @uxie, validated by @tapu-fini)
  - Document: Evaluator-Optimizer pattern, "Agents as Tools" composition, guardrail taxonomy (input/output/tool-level), failure modes from MAST paper
  - Reference: Anthropic, OpenAI, Google ADK, MAST paper, Andrew Ng, Microsoft architecture
  - Remove: any outdated patterns that conflict with v2 decisions
  - Verify: Detail doc cross-references the stub doc correctly

- [ ] **Task 4.3:** Document the Workflow State Protocol (in detail doc)
  - Full specification: format, fields, update rules, who updates what
  - Examples: simple workflow (1 agent), standard workflow (maker + reviewer), full pipeline (plan → research → code → reviews)
  - Verify: Three example workflows are complete and internally consistent

- [ ] **Task 4.4:** Document Gate Definitions and HITL Insertion Points (in detail doc)
  - Full gate specifications with examples
  - HITL trigger conditions, escalation message templates, resolution paths
  - Verify: Every gate definition has a concrete example scenario

- [ ] **Task 4.5:** Update any cross-references in other instruction files
  - Scan instruction files for references to tier routing, old patterns, or outdated agent behavior
  - Fix any stale references
  - Scan scope includes: `.github/instructions/`, `.github/copilot-instructions.md`, `.github/instructions/personality.instructions.md`
  - Verify: `grep` for "T1", "T2", "T3", "tier", "handoff" across `.github/instructions/` AND `.github/copilot-instructions.md` — only expected references remain

**Acceptance Criteria (Phase 4):**
- [ ] `multi-agent-orchestration.instructions.md` reflects v2 — no tier routing, updated design principles
- [ ] `multi-agent-orchestration.detail.instructions.md` incorporates research findings and v2 protocols
- [ ] Workflow State Protocol fully documented with 3 examples
- [ ] Gate definitions and HITL points documented with examples
- [ ] No stale cross-references to old patterns in instruction files
- [ ] Zero tier references (T1/T2/T3) in instruction files

**Dependencies:** Phase 2 (protocol definitions), Phase 3 (agent format finalized).

---

## Phase 5: Validation
**Goal:** Verify the v2 architecture works end-to-end through dry-runs and structural checks.

- [ ] **Task 5.1:** Dry-run — "Add Redis caching to the API"
  - Trace through v2 model: @mewtwo decomposes → selects @metagross (plan) → @cresselia (review plan) → @uxie (research Redis patterns if needed) → @tapu-fini (review research) → @porygon (implement) → build + test → @absol (review) → commit
  - Verify: All routing decisions made by @mewtwo. No agent self-routes. Gates fire correctly. Workflow state block is maintained throughout.

- [ ] **Task 5.2:** Dry-run — "Research GraphQL and update brain"
  - Trace: @mewtwo decomposes → selects @uxie (research) → @tapu-fini (review) → brain-manager skill (update)
  - Verify: Research review loop works without tier classification. Simpler task = fewer agents naturally.

- [ ] **Task 5.3:** Dry-run — "Fix the typo in README.md"
  - Trace: @mewtwo decomposes → 1 subtask → @porygon (fix) → @absol (review) → commit
  - Verify: Simple task uses minimal agents without forced classification step.

- [ ] **Task 5.4:** Composability test — Adding @rotom (DevOps agent)
  - Write a hypothetical `rotom.agent.md` using the v2 template
  - Verify: ZERO existing files need modification
  - Verify: @mewtwo's capability-based selection can route to @rotom for infra tasks by reading @rotom's capabilities
  - The only file that needs awareness of @rotom is @mewtwo's maker/reviewer registry (if @rotom has a reviewer) — and this is acceptable as a registry update, not a coupling change

- [ ] **Task 5.5:** Structural validation
  - `grep` all `.github/agents/*.agent.md` for forbidden patterns: "What You DON'T Do", "Handoffs", "hand off to @", "T1", "T2", "T3" (in routing context)
  - `grep` all `.github/instructions/*orchestration*` for "T1", "T2", "T3" tier routing
  - Verify: only expected references remain (e.g., T1/T2/T3 in historical context or research citations, not in active routing)

- [ ] **Task 5.6:** Brain health check
  - Run brain-manager health scan
  - Verify: no broken cross-references, no orphaned files, no conflicting instructions
  - Verify: all agent files follow consistent structure
  - Verify: instruction files are internally consistent

**Acceptance Criteria (Phase 5):**
- [ ] 3 dry-run scenarios traced successfully with no routing gaps
- [ ] Composability test passes — @rotom addable with zero existing-file changes (except registry)
- [ ] Structural grep finds zero forbidden patterns
- [ ] Brain health check passes clean

**Phase 5 Failure Protocol:** Validation failures route back to the owning phase for fix. After fix, re-run the failed validation task only. If Phase 5 loops more than twice on the same issue, escalate to human.

**Dependencies:** Phases 1-4 all complete.

---

## Validation Strategy

Every AI-generated artifact must be verified before acceptance:

| Artifact | Verification Method |
|----------|-------------------|
| Agent template (Phase 1) | @rotom thought experiment — can a new agent be created without changing existing files? |
| @mewtwo rewrite (Phase 2) | Walk through 3 scenarios: does routing work for simple, medium, and complex tasks? |
| Specialist agent files (Phase 3) | Cross-agent grep for forbidden patterns; structural consistency check |
| Instructions updates (Phase 4) | Cross-reference scan; grep for stale tier/handoff references |
| End-to-end architecture (Phase 5) | 3 dry-run scenarios + composability test + brain health check |

**Expected failure modes:**
- **Routing gaps:** Moving routing from agents to @mewtwo may miss edge cases → dry-runs catch this
- **Lost context:** Removing "review awareness" from makers might degrade output quality → reframe as quality standards, not reviewer identity
- **Scope leakage:** Agents without negative scope might attempt tasks outside their capability → tools list provides implicit scope; verify in dry-runs
- **Stale references:** Old patterns referenced elsewhere in brain → comprehensive grep catches this
- **Workflow state overhead:** Protocol too verbose for simple tasks → dry-run simple task to verify lightweight tracking

---

## Phase Dependency Graph

```
Phase 1 (Template)
    ↓
Phase 2 (Orchestrator)
    ↓
Phase 3 (Specialists)  ← also depends on Phase 1
    ↓
Phase 4 (Instructions)  ← depends on Phase 2 + Phase 3
    ↓
Phase 5 (Validation)   ← depends on all above
```

## Execution Notes
- Phases 1-3 are writing-heavy: all changes are to `.md` files in `.github/`
- Phase 4 requires careful cross-reference management
- Phase 5 is verification-only — no new artifacts
- Total files affected: 7 agent files + 2 instruction files + 1 new template = ~10 files
- No code changes — this is pure brain architecture work
- @absol code review gate does NOT apply (no code changes) — but @cresselia plan review DOES apply to this plan
