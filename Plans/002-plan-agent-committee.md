# Plan 002 — stonk-sage Agent Committee v0

**Owner:** Joe
**Repo (code):** `github.com/jdtran23/stonk-sage` (private)
**Repo (this plan):** `myBrain/Plans/`
**Status:** Rev 3 — pending final re-review (@cresselia ✅ 24/25 on rev 2; 🦆 PASS W/ REVISIONS)
**Estimated effort:** 3–5 weekends of hobby time

---

## Changelog

**Rev 3.1** — Added "Post-v0 Future Directions" section to capture the OpenClaw integration intent (chat with the committee from phone via Telegram/Discord/etc.) as explicitly-deferred-but-acknowledged future work. No changes to Phases 0–2. Direct edit, not gated — section is a roadmap note, not an execution plan.

**Rev 3** — 3 duck execution-trap fixes + 2 Cresselia cosmetic nits. No new tasks, no new phases.

- **D1 (`pytest-socket` missing):** added to Task 0.1 dev deps.
- **D2 (no benchmark data in MarketSnapshot):** Task 0.4 contract gains `spy_return_same_window`, `ticker_return_same_window`, optional `sector_return_same_window`; fixture populated; Task 1.6 checklist item 4 now requires citing those specific fields.
- **D3 (cassette identity ambiguity):** Task 0.6 `call_llm` signature gains `agent_id` + `mode`; Task 1.5 keys cassette + `.prompthash` filenames off `agent_id`; verify test covers same-model/different-agent routing.
- **C1 (dependency diagram):** updated to show 1.2 and 1.3 as parallel branches off 1.1, matching the prose.
- **C2 (AI Usage Declaration stale):** mitigations now point at concrete rev-2 tasks (0.6 wrapper, 0.5 disagreement checklist) + new D2 hallucinated-benchmark entry.

**Rev 2** — addresses 16 findings from Cresselia (🟡 APPROVED W/ CHANGES) and 🦆 rubber-duck (PASS W/ REVISIONS). Surgical task-level adds only; no new phases, no ceremony.

- **B1 (look-ahead bias):** Task 0.4 now asserts no snapshot row has date `> as_of`; mirrored in Task 2.1 smoke test as an invariant.
- **B2 (replay not network-free):** New frozen `MarketSnapshot` fixture; replay loads fixture, not yfinance/edgartools. Phase 2 exit gate runs `--disable-socket`.
- **B3 (vcrpy/httpx brittleness):** Task 1.5 cassette discipline spelled out (sync, non-streaming, `temperature=0`, retry bounds, header filtering, one cassette per agent, `record_mode='none'`, 30-min fallback to `pytest-recording`/`respx`). Risks table row added.
- **I1 (parse_or_repair owner):** New **Task 0.6 — Shared LLM call wrapper** owns this; all agents route through it.
- **I2 (provider preflight):** New **Task 0.0 — Provider preflight** before Phase 0; `docs/model-ids.md` records exact LiteLLM strings.
- **I3 (structlog ordering):** Minimal stderr-JSON sink config moved into Task 1.4; Task 1.7 keeps file persistence + enrichment.
- **I4 (Bull/Bear convergence):** Task 0.5 verify replaced with explicit checklist (direction-or-driver disagreement, ≥2 distinct evidence each, ≤1 shared item).
- **I5 (fan-in token budget):** `MarketSnapshot` emits extracted facts not raw 10-K; all contracts get explicit `max_length` bounds; DA/CIO see summaries only.
- **I6 (prompt hash):** Task 1.5 records SHA256 of prompt+model+temperature as sidecar `.prompthash`; replay fails on mismatch.
- **I7 (memo quality bar):** Task 1.6 verify replaced with a 6-item acceptance checklist.
- **S1–S6:** Absorbed as one-line additions to 1.4 (TradingAgents time-box), 2.2 (`--live` vs `--record`), 1.7 (token tracing is best-effort), 0.4 (`EDGAR_IDENTITY`), 2.3 (re-record cost note), 2.x (notebook fate).

---

## Objective

Build the first working 6-agent investment-memo committee for stonk-sage and prove it end-to-end on one ticker (AAPL, one historical date), running off cassettes so iteration costs $0. Ship to the existing private GitHub repo with a tiny CLI.

**Success = one command:** `uv run stonk-sage analyze AAPL --as-of 2024-06-01` produces a Markdown memo by replaying cassettes, with no live API calls required.

---

## Non-Goals (explicit)

The following are **out of scope** for this plan. Do not let them creep in:

- Live trading / brokerage integration
- Screening multiple tickers
- Backtest harness or PIT replay infrastructure beyond a single `as_of` date
- Self-hosting / Ollama / local models
- Cost dashboards, Langfuse, observability platform
- Public repo flip
- Sector-specialist agents (banks/REITs/biotech)
- Multi-turn debate rounds (single-pass only)
- RESUME_STATE.md, decision docs, weekly notes, contributor tables, bus-factor planning
- Branch protection, CI cost gates, release automation
- Anything that doesn't directly serve "6 agents → 1 memo in a notebook"

---

## Locked Decisions (no re-litigation)

- Output: per-ticker memo, **structured YAML** is source of truth, Markdown is a renderer.
- Orchestration: **LangGraph**.
- Provider abstraction: **LiteLLM** (one client, model swap via config).
- Structured I/O: **pydantic** contracts per agent.
- Cassettes: **vcrpy** for LLM call replay.
- Package mgmt: **uv**. Lint/format: **ruff**. Types: **pyright**. Tests: **pytest**.
- Logging: **structlog**.
- Data: **yfinance** + **edgartools** (free tier).
- v0 surface: **Jupyter notebook**, then a thin **typer** CLI.
- Agents and model pairings: locked per the input brief (Bull/Bear cross-provider is non-negotiable).
- Facade rule: LangGraph types stay inside `agents/`; callers see only pydantic `CommitteeInput`/`CommitteeResult`.

---

## AI Usage Declaration

**What AI generates in this plan:**
- Per-agent prompts (system + user templates) drawing on `.github/instructions/` brain files
- LangGraph node functions, state schema, edge wiring
- Pydantic output contracts per agent
- The cassette-recording harness and CLI scaffold
- Markdown renderer template

**What needs human judgment (Joe):**
- Whether each agent's output actually *reads* like a competent analyst (smell test, not unit-test)
- Whether the Bull vs Bear cases differ in *substance* and not just *tone* (cross-provider sanity check)
- Whether the CIO memo honors the brain's hard rule (`source_of_edge: none → NO_ACTION`)
- Whether the cassette set covers all 6 agent calls without re-recording

**Expected AI failure modes to watch for:**
- **Hallucinated LangGraph APIs.** LangGraph's API churns. AI may invent `add_node` signatures, state-merge semantics, or conditional-edge syntax that don't exist in the installed version. Mitigation: pin LangGraph version in `pyproject.toml`; verify each node compiles by running the graph after every wiring change.
- **Hallucinated LiteLLM model strings.** Mitigation: **Task 0.0 preflight** + re-runnable **Task 0.2** smoke test gate everything downstream; exact strings recorded in `docs/model-ids.md`.
- **Bull/Bear convergence.** If the same provider runs both, or prompts are too similar, cases will rhyme. Mitigation: **Task 0.5's 4-point disagreement checklist** (recommendation direction OR dominant driver; ≥2 distinct evidence each; Bear cannot restate Bull's `key_risk`; ≤1 overlap on `key_drivers`/`key_risk`). Hard gate — revise prompts before Phase 1 on failure.
- **Pydantic-vs-LLM drift.** LLMs often return YAML/JSON that almost-but-not-quite matches the schema. Mitigation: **Task 0.6's `call_llm` wrapper** with `parse_or_repair` retry semantics (retries once in live mode, fails fast in replay). All agents route through it.
- **Hallucinated benchmark comparison.** CIO is asked for "vs SPY/sector" and may invent numbers if not given them. Mitigation: **Task 0.4** ships structured benchmark fields (`spy_return_same_window`, `ticker_return_same_window`, optional `sector_return_same_window`) in `MarketSnapshot`; **Task 1.6** checklist requires the memo to cite those specific field values, not vague claims.
- **Cassette staleness.** Once recorded, cassettes lock in any bug in the prompt. Mitigation: **Task 1.5's `.prompthash` sidecar** fails replay loudly when the prompt drifts from its recording; one cassette per `agent_id`; re-record command documented in README.

---

## Phase 0 — Notebook prototype (smallest viable thing)

**Goal:** Prove the three providers work, prove we can fetch real data *without look-ahead*, prove Bull and Bear say *substantively different* things about AAPL. No LangGraph yet.

**Estimated effort:** 1 weekend (preflight may bleed into a second evening if any provider account isn't ready).

### Tasks

**0.0 — Provider preflight (prereq zero)**
- Create/verify provider accounts and billing for **Anthropic**, **OpenAI**, **Google AI**. Confirm model access for: Claude Sonnet 4.6, Claude Haiku 4.5, Claude Opus 4.7, GPT-5.4, GPT-5.5, Gemini 3.5 Flash.
- Hit each one with a single `"Reply OK"` call via LiteLLM and record the **exact** LiteLLM model string (e.g. `anthropic/claude-sonnet-4-6-20250514`) in `docs/model-ids.md` — one page, table form, not ceremony.
- Note observed rate limits / quota tier per provider.
- **Verify:** All six model IDs return non-empty responses. Any quota/billing/access failure resolves *here*, before Task 0.1 — do not let bad strings or missing access propagate into agent code.

**0.1 — Repo + tooling bootstrap**
- `uv init`, create `pyproject.toml` with deps: `litellm`, `langgraph`, `pydantic`, `structlog`, `yfinance`, `edgartools`, `vcrpy`, `typer`, `jinja2`, `jupyter`. Dev: `ruff`, `pyright`, `pytest`, `pytest-socket`.
- Add `.python-version` (3.12+), `.env.example` with the three API key names **plus `EDGAR_IDENTITY`** (contact email — required by SEC fair-access policy), `.gitignore` for `.env` and `__pycache__`.
- **Verify:** `uv sync` succeeds; `uv run python -c "import litellm, langgraph, pydantic"` exits 0.

**0.2 — Provider smoke test (re-runnable)**
- Promote the preflight one-shot from 0.0 into a re-runnable `scripts/smoke_providers.py` that reads `docs/model-ids.md`'s strings and pings each model.
- **Verify:** all three providers return non-empty responses. This script is the canary for any future API/model-string change.

**0.3 — Brain loader**
- `stonk_sage/brain.py`: function `load_brain(*names) -> dict[str, str]` that reads from `../stonk-sage/.github/instructions/<name>.instructions.md` (or local copy — Joe to decide path) and returns a dict the agents can splice into prompts.
- **Verify:** `load_brain("equity-research", "risk-portfolio")` returns two non-empty strings; assert via a 3-line pytest.

**0.4 — Data Analyst as a plain function (with no-look-ahead invariant)**
- `agents/data_analyst.py`: function takes `(ticker, as_of)`, returns a pydantic `MarketSnapshot`.
- **No-look-ahead rules (hard):**
  - 10-K: select the most recent filing where `filing_date < as_of`. Never "latest."
  - Prices: slice yfinance to rows with `date <= as_of`.
  - News: stubbed fixture for now; each item carries a `date` and is filtered `<= as_of`.
  - Assert at the end of the function: no row/field in the snapshot has a date `> as_of`. Raise on violation.
- **Fan-in budget (per I5):** `MarketSnapshot` carries **extracted facts** (key numbers, ratios, 1–3 sentence section summaries), **not** raw 10-K chunks. Explicit `max_length` on every string list field (see Task 1.1).
- **Benchmark fields (per D3 — new in rev 3):** `MarketSnapshot` includes:
  - `spy_return_same_window: float` — trailing-12-month return of SPY ending at `as_of`
  - `ticker_return_same_window: float` — same window, for the analyzed ticker
  - `sector_return_same_window: float | None` — optional, if easily fetched
  These give CIO real numbers to cite in the benchmark-comparison section instead of hallucinating.
- **Frozen fixture (per B2):** After the function works live once, dump its output to `tests/fixtures/market_snapshot_aapl_2024-06-01.json` — **including populated benchmark fields with real values**. A `load_snapshot_fixture(ticker, as_of)` helper returns this in replay mode. Live fetch happens **only** during `--record` or explicit fixture regeneration.
- `EDGAR_IDENTITY` env var must be set; document in README and `.env.example`.
- **Verify:**
  1. Run live on AAPL/2024-06-01; eyeball that 10-K, prices, news fields are populated.
  2. Pytest: load the saved fixture, assert no date field `> as_of`.
  3. Pytest: monkey-patch `as_of` earlier than the saved 10-K's `filing_date` → assert the function picks an older filing, not the saved one.

**0.5 — Bull and Bear as plain functions**
- `agents/bull.py` and `agents/bear.py`: each takes the `MarketSnapshot`, calls its model **via the shared wrapper from Task 0.6**, returns a pydantic `Thesis` (fields: `headline`, `key_drivers: list[str]`, `source_of_edge`, `key_risk`, `confidence`, `evidence: list[str]`).
- Prompts splice in `equity-research` + `finance-fundamentals` brain.
- **Verify — disagreement checklist (replaces vague smell test, per I4):**
  - Bull and Bear must disagree on **recommendation direction** OR **dominant driver** — not just framing.
  - Each must cite **≥2 distinct evidence points** absent from the other (compute set difference on `evidence`).
  - Bear's `headline`/`key_drivers` must not be a paraphrase of Bull's `key_risk` (and vice versa).
  - Overlap check on `key_drivers ∪ {key_risk}`: at most **1** shared item across the two outputs.
  - If the checklist fails → **revise prompts and re-run before Phase 1**. Hard gate.

**0.6 — Shared LLM call wrapper (`parse_or_repair`)**
- `stonk_sage/llm.py`: single entry point —
  ```python
  def call_llm(
      agent_id: Literal["data_analyst", "bull", "bear", "risk", "devils_advocate", "cio"],
      model: str,
      system: str,
      user: str,
      schema: type[BaseModel],
      mode: Literal["live", "replay", "record"],
      *,
      temperature: float = 0.0,
  ) -> BaseModel
  ```
- `agent_id` (per D3) is **the** routing key for cassettes and prompt-hash sidecars — not the model id, not call order. Bull and Bear could share a model and still route to distinct cassettes.
- Behavior:
  - Asks the model to reply with JSON (or YAML) in a fenced block; extracts and parses.
  - On `pydantic.ValidationError`: **in `live` mode**, retries once, appending the parser error + offending payload to the prompt. **In `replay` mode**, fails fast with no retry — drift must surface loudly. **In `record` mode**, same as live but writes the cassette + `.prompthash` afterward.
  - Logs the offending payload via structlog; redacts `Authorization`/API-key headers and env values.
  - Wraps LiteLLM with: sync only, non-streaming, `temperature=0` default, retries bounded to 1.
- **All agents in 0.5, 1.2, 1.3 call through this — no agent constructs its own LiteLLM call.**
- **Verify:**
  1. Stub LLM returns malformed JSON once then valid JSON → wrapper in `live` mode retries once → succeeds.
  2. Stub returns malformed twice → `live` raises `ValidationError`; `replay` raises **immediately** on first malformed response (no retry).
  3. **Cassette routing test (per D3):** invoke `call_llm` twice with the same `model` but different `agent_id` ("bull" vs "bear") → two distinct cassette files written, two distinct `.prompthash` sidecars, no clobbering.

**Exit gate:** Notebook cell `prototype.ipynb` runs end-to-end, prints a `MarketSnapshot` (fixture-loaded) plus a Bull `Thesis` and a Bear `Thesis` that pass the disagreement checklist on AAPL.

---

## Phase 1 — Full committee in LangGraph

**Goal:** All 6 agents wired through LangGraph, cassettes recorded, CIO emits structured YAML, Markdown renderer turns it into a memo.

**Estimated effort:** 2 weekends.

### Tasks

**1.1 — Pydantic contracts for all 6 agents (with bounds)**
- `stonk_sage/contracts.py`: `MarketSnapshot`, `Thesis` (Bull/Bear share schema), `RiskAssessment`, `DevilsAdvocateCritique`, `CIOMemo`. `CIOMemo` includes `source_of_edge`, `benchmark_comparison`, `specialist_disagreements`, `recommendation` (one of `BUY | HOLD | TRIM | AVOID | NO_ACTION`).
- **Explicit length bounds on every string-list field** (per I5) — e.g. `key_drivers: list[str] = Field(max_length=5)`, each str `max_length=200`; `MarketSnapshot` section summaries `max_length=600` chars each; `evidence: list[str]` capped at 5×200. DA/CIO inputs therefore have a hard ceiling.
- Top-level facade types `CommitteeInput`, `CommitteeResult` exposed from `stonk_sage/__init__.py`.
- **Verify:** pytest round-trips each contract through `model_dump_json` → `model_validate_json`. Second pytest: oversize input (6 drivers, 300-char strings) raises `ValidationError`.

**1.2 — Risk Officer and Devil's Advocate agents**
- `agents/risk_officer.py`: takes `MarketSnapshot`, returns `RiskAssessment` honoring `risk-portfolio` brain (sizing band, concentration flags). Veto power on sizing only. Routes via the Task 0.6 wrapper.
- `agents/devils_advocate.py`: takes `(MarketSnapshot, bull: Thesis, bear: Thesis, risk: RiskAssessment)`, returns `DevilsAdvocateCritique` attacking *reasoning quality* of both sides, using `behavioral-finance` brain bias checklist. Receives the bounded-summary `MarketSnapshot` per I5 — never raw filing text.
- **Verify:** Notebook cell prints both outputs for AAPL; Joe confirms DA actually critiques *both* theses rather than picking a side (common failure mode). If DA sides with one team, revise prompt before 1.3.

**1.3 — CIO synthesizer agent**
- `agents/cio.py`: takes all 5 prior outputs, returns `CIOMemo`. Prompt encodes the hard rule: `if source_of_edge is None or empty → recommendation = NO_ACTION`. Routes via Task 0.6 wrapper. Inputs are the bounded summaries — no raw 10-K injection.
- **Verify (positive + negative):**
  1. Real run on AAPL with a Bull that has a populated `source_of_edge` → memo has a non-`NO_ACTION` recommendation grounded in that edge.
  2. Same run with Bull's `source_of_edge` manually blanked → memo **must** return `NO_ACTION`. This is a correctness check, not a smell test.

**1.4 — LangGraph wiring**
- **Time-box** (per S1): before wiring, spend at most **60–90 min** studying `github.com/TauricResearch/TradingAgents` for state-schema patterns, node signatures, and prompt-output boundaries. **Pattern study only — no copying.**
- `agents/graph.py`: state schema with all intermediate outputs; nodes for each agent; edges: `data_analyst → [bull, bear, risk] (parallel) → devils_advocate → cio`. The three parallel nodes must not see each other's outputs (anti-anchoring).
- Public facade `run_committee(input: CommitteeInput) -> CommitteeResult` — LangGraph types do not leak past `agents/`.
- **Inline structlog config (per I3):** wire a minimal stderr JSON sink here (~5 lines) so this task's verify works. File persistence + enrichment is owned by Task 1.7.
- **Verify:** `run_committee` executes on AAPL/2024-06-01 (using fixture snapshot + live LLM, one time). Inspect structlog stderr to confirm parallel nodes received only the snapshot, not each other's outputs.

**1.5 — Cassettes via vcrpy (with discipline)**
- Configure vcrpy to record LiteLLM HTTP calls into `tests/cassettes/aapl_2024_06_01/`. **One cassette per `agent_id`** (six total): `data_analyst.yaml`, `bull.yaml`, `bear.yaml`, `risk.yaml`, `devils_advocate.yaml`, `cio.yaml`. Routing key is the `agent_id` arg to `call_llm` (Task 0.6) — **not** model id, **not** call order.
- **`.prompthash` sidecars share the agent-id name:** `bull.prompthash`, `bear.prompthash`, etc., living beside their `.yaml` cassette. Replay reads the current hash for that `agent_id` and **fails loudly on mismatch**.
- **Cassette discipline (per B3):**
  - LiteLLM calls are **sync, non-streaming**.
  - `temperature=0`; retries bounded to 1 during recording; 0 in replay.
  - Filter `Authorization`, `x-api-key`, and any provider auth headers from cassettes; never commit a key.
  - Replay uses `record_mode='none'`. Any unexpected outbound request fails the test.
- **Prompt-hash sidecar (per I6):** for each `agent_id` write `<agent_id>.prompthash` containing `sha256(prompt_template + model_id + temperature)`. Replay reads current hash and **fails loudly on mismatch** — prevents silent regressions where the prompt changes but cassette doesn't.
- **30-minute fallback (per B3):** if vcrpy fails to cleanly intercept httpx (Anthropic/Gemini SDKs route through httpx and vcrpy coverage is uneven), switch to `pytest-recording` or `respx`. Document the swap in a 3-line note at the top of the cassette dir. Do not yak-shave.
- `scripts/record_cassettes.py` wipes the cassette dir, runs the committee live, writes fresh cassettes + `.prompthash` files.
- **Verify:** Delete cassette dir → record script → run committee twice in replay mode → both produce byte-identical `CIOMemo` JSON. Mutate one prompt template → replay fails with a hash-mismatch error pointing at the changed agent.

**1.6 — YAML + Markdown renderer**
- `stonk_sage/render.py`: dumps `CIOMemo` to YAML, plus a Jinja2 template that produces a Markdown memo with sections: Recommendation, Source of Edge, Bull Case, Bear Case, Risk Notes, Devil's Advocate, Specialist Disagreements, Benchmark Comparison.
- Save AAPL output to `examples/aapl_2024_06_01.md`.
- **Verify — memo acceptance checklist (per I7, replaces "eyeballs readability"):**
  1. Clear recommendation (`BUY|HOLD|TRIM|AVOID|NO_ACTION`) stated upfront.
  2. Explicit `source_of_edge` populated OR `NO_ACTION` chosen — never both empty.
  3. Bull/Bear/Risk disagreement summarized in `specialist_disagreements`, not averaged away.
  4. At least one benchmark comparison — **must cite `spy_return_same_window` and `ticker_return_same_window` from `MarketSnapshot` (and `sector_return_same_window` if populated)**, not vague claims like "vs the broader market."
  5. DA critique visibly **changes or constrains** the final conclusion — not just appended as a footer.
  6. No section is generic filler — each cites a specific number or evidence point from the snapshot/theses.
  - Any failure → fix the CIO prompt (not the renderer) and re-run.

**1.7 — Structlog tracing (persisted)**
- Extend the 1.4 stderr sink with a JSONL file sink at `runs/<timestamp>/trace.jsonl`. One event per node entry/exit including model/provider used and outcome.
- **Token counts are best-effort (per S3):** log them when LiteLLM surfaces them; do not crash a run when a provider omits them.
- **Verify:** Trace file exists after a run and contains at least 12 events (6 agents × enter/exit), each tagged with node name + model.

**Exit gate:** `python -c "from stonk_sage import run_committee, CommitteeInput; print(run_committee(CommitteeInput(ticker='AAPL', as_of='2024-06-01')).memo_markdown[:500])"` prints the start of a memo, running entirely from the snapshot fixture + cassettes with no outbound network.

---

## Phase 2 — Polish + ship

**Goal:** Anyone (incl. future-Joe on a fresh laptop) can clone, sync, and run a memo. One smoke test guards against regressions.

**Estimated effort:** 1 weekend.

### Tasks

**2.1 — Smoke test (network-disabled, no-look-ahead invariant)**
- `tests/test_smoke.py`: runs `run_committee` on AAPL/2024-06-01 against the snapshot fixture + cassettes; asserts:
  - Memo has non-empty `source_of_edge` OR `recommendation == "NO_ACTION"`.
  - Bull `headline` ≠ Bear `headline`.
  - **No-look-ahead invariant (per B1):** every dated field in the loaded `MarketSnapshot` has `date <= as_of`.
- Run with `pytest-socket` `--disable-socket` (no network at all). Cassette + fixture must satisfy the entire run.
- **Verify:** `uv run pytest --disable-socket` is green.

**2.2 — Typer CLI**
- `stonk_sage/cli.py`: `stonk-sage analyze TICKER --as-of DATE [--replay | --live | --record]`. Default `--replay` (fixture + cassettes, no network). `--live` analyzes a fresh ticker/date **without overwriting canonical cassettes** (writes to `runs/<ts>/` only). `--record` is the explicit destructive flag that overwrites cassettes + fixture for the given ticker/date.
- Per S2: `--live` and `--record` are different verbs. Only `--record` (or `scripts/record_cassettes.py`) can mutate canonical fixtures.
- Register entry point in `pyproject.toml`.
- **Verify:** `uv run stonk-sage analyze AAPL --as-of 2024-06-01` writes `runs/<ts>/memo.md` with sockets disabled. `--live AAPL --as-of 2025-01-15` produces a memo *without* touching `tests/fixtures/` or `tests/cassettes/`.

**2.3 — README**
- Sections: What it is (1 paragraph), Quickstart (clone → uv sync → set API keys + `EDGAR_IDENTITY` → run replay), How to re-record cassettes (and **re-record cost note per S5**: "~$1–3 per full live record — one Opus + one GPT-5.5 + ~4 mid-tier calls. Iterate at replay scale; re-record sparingly."), The 6 agents (one-line each), Known limits (single ticker, single date, no backtest, no live trading).
- **Verify:** Joe follows his own README on a clean checkout and gets a memo without consulting the code.

**2.4 — Commit, push, and notebook fate**
- **Notebook decision (per S6):** move `prototype.ipynb` → `examples/v0.1-prototype.ipynb` as a **frozen artifact** (committed as-is, not maintained against future refactors). README notes its frozen status. Do not leave an unmaintained notebook in the repo root.
- Squash-friendly commit history (`feat: phase 0`, `feat: phase 1`, `feat: phase 2`), push to `main` of `jdtran23/stonk-sage`. Tag `v0.1.0`.
- **Verify:** Fresh clone → `uv sync` → set env → `uv run stonk-sage analyze AAPL --as-of 2024-06-01` → memo appears, all with sockets disabled.

**Exit gate:** `git clone … && cd stonk-sage && uv sync && uv run pytest --disable-socket && uv run stonk-sage analyze AAPL --as-of 2024-06-01` all succeed with no outbound network calls.

---

## Dependencies

```
0.0 → 0.1 → 0.2 → 0.3 → 0.4 → 0.6 → 0.5
                                ↓
                               1.1 ─┬─→ 1.2 ─┐
                                    └─→ 1.3 ─┤
                                             ↓
                                            1.4 → 1.5 → 1.6 → 1.7
                                                              ↓
                                                  2.1 → 2.2 → 2.3 → 2.4
```

- 0.0 is hard prereq — no agent code starts until model IDs are confirmed.
- 0.6 (LLM wrapper) precedes 0.5 because Bull/Bear route through it.
- 1.2 and 1.3 fan out in parallel from 1.1; both must land before 1.4 wires the graph.

---

## Risks (short and concrete)

| Risk | Likelihood | Mitigation |
|---|---|---|
| **Look-ahead bias** corrupts historical analyses | High | Task 0.4 asserts no row has date `> as_of`; 2.1 smoke test re-asserts the invariant. |
| **Replay not actually network-free** (yfinance/edgartools live) | High | Frozen `MarketSnapshot` fixture (Task 0.4 + B2); Phase 2 exit gate runs `--disable-socket`. |
| **vcrpy/httpx interception gap** for Anthropic/Gemini SDKs | Medium-High | Sync, non-streaming, `temperature=0`, header filtering, one cassette per agent; 30-min fallback to `pytest-recording`/`respx` documented in Task 1.5. |
| **LangGraph API churn** invalidates AI-generated wiring | High | Pin exact version in `pyproject.toml`; verify graph compiles after every edit; consult installed package's docstrings, not model memory. |
| **Hallucinated model identifiers** in LiteLLM | High | Task 0.0 preflight + 0.2 re-runnable smoke test gate all later work. |
| **Bull/Bear converge** despite different providers | Medium | Task 0.5 disagreement checklist is a hard gate (per I4). |
| **Cassettes hide prompt regressions** | Medium | `.prompthash` sidecar in Task 1.5; replay fails on mismatch. |
| **Pydantic validation fails on LLM YAML** | Medium | Task 0.6 wrapper retries once in live mode, fails fast in replay. |
| **CIO ignores `source_of_edge` hard rule** | Medium | Task 1.3 has positive + negative test cases, not eyeball check. |
| **DA/CIO context blow-up** from raw 10-K | Medium | `max_length` bounds on all contracts; `MarketSnapshot` carries extracted facts, not chunks. |
| **Scope creep** back toward archived 12-phase plan | High | Non-Goals section is load-bearing; refer back when tempted. |

---

## Verification Strategy (summary)

Every task above ends with a **Verify** step. The pattern is **Plan → Generate → Verify → Iterate**:

- **Structural verification** (does it import / parse / typecheck): pyright + pytest round-trips, contract length-bound tests.
- **Behavioral verification** (does it do the right thing): smoke test, cassette determinism, prompt-hash mismatch, no-look-ahead invariant, replay runs with sockets disabled.
- **Judgment verification** (does it read like an analyst): Bull/Bear disagreement checklist (0.5) and memo acceptance checklist (1.6) replace prior vague smell tests. Joe still owns the final read on AAPL's memo.
- **Hard-rule verification** (does the `NO_ACTION` rule fire): explicit positive + negative test in Task 1.3.

A phase is **not** complete until its exit-gate command runs clean from a fresh shell with sockets disabled.

---

## Post-v0 Future Directions (deferred, NOT part of this plan)

The following is named here so it doesn't get lost as scope drift, but **explicitly out of scope** for v0. When v0.1.0 ships, write a separate plan for whichever of these (if any) is next.

### OpenClaw integration — chat with the committee from phone (likely v0.1)

[OpenClaw](https://docs.openclaw.ai/) is a self-hosted gateway that bridges messaging apps (Discord, Telegram, WhatsApp, Slack, Signal, iMessage, Matrix, etc.) to AI agents. A small OpenClaw tool plugin would let Joe chat with the stonk-sage committee from his phone via any of those channels.

**Why this composes well with the v0 plan as-is:**
- The CLI Phase 2 produces (`stonk-sage analyze TICKER --as-of DATE`) is directly callable from an OpenClaw tool plugin via subprocess
- Output is already structured (YAML) — easy to return from a plugin's `execute()`
- Cassettes make integration testing free
- No changes needed to Phases 0–2 to keep this path open

**Rough shape of the future v0.1 plan (a sketch, not a commitment):**
- ~100 lines of TypeScript using OpenClaw's `defineToolPlugin` SDK
- Exposes one tool: `stonk_sage_analyze(ticker, as_of)`
- Bridges to stonk-sage via subprocess (simplest) or an HTTP wrapper (better for long-running calls)
- Setup: Node.js toolchain, `openclaw plugins init`, configure a channel (Telegram is the docs' "fastest" option)
- Estimated effort: one weekend after v0 ships

**Why deferred (not in v0):**
- v0's goal is "build the engine." OpenClaw integration is a delivery surface for the engine. Ship the engine first.
- Adds a TypeScript toolchain on top of Python — manageable but real overhead
- Easier to wire after the CLI is stable and the YAML schema is settled (i.e., after Phase 2 lands)

### Other directions worth tracking (less specific, less committed)

- **MCP server wrapper** for Claude Desktop / Cursor / Cline (~50 lines Python). Skip if OpenClaw is the only "ask from anywhere" surface needed.
- **Multi-turn conversation** — current committee produces a complete memo per call. Conversational follow-ups ("what about TSLA?", "explore the risk angle?") would need a different shape.
- **Multiple tickers / screening** — current committee analyzes one ticker per invocation.
- **Sector-specialist agents** (banks/REITs/biotech) — the brain has the knowledge; agents using that knowledge are future work.
- **Live data fetching** beyond a single frozen fixture — v0 explicitly avoids PIT replay infrastructure.

When v0 ships, **write a new plan based on what was learned**. Do not pre-design v0.1 here.

---

## Out-of-band notes

- The `.github/instructions/` brain lives in the stonk-sage repo (already shipped). Phase 0.3 decides whether to read it from there or copy locally — both are fine; just be deliberate.
- If LangGraph wiring eats a whole weekend on its own, that's a signal to step back and re-read its quickstart, not a signal to switch frameworks. The locked decision stands.
- This plan does *not* plan the next plan. When v0.1.0 ships, write a new plan based on what was learned. Do not pre-design v0.2 here.
