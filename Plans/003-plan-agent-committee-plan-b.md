# Plan 003 — stonk-sage Agent Committee v0 (Plan B: agent.md + Copilot CLI)

**Owner:** Joe
**Repo (code):** `github.com/jdtran23/stonk-sage` (private)
**Repo (this plan):** `myBrain/Plans/`
**Status:** Rev 4 — **PHASE 0+1 EXECUTED** in stonk-sage commit `aad94f5`. Tasks 0.0–0.4, 1.1–1.4, 2.1–2.2 done; reviewed by @absol twice (zero red/yellow at commit). Tasks 0.5, 1.5, 2.3 require Joe to run `/analyze` from a Copilot CLI session in `C:\Users\jdtra\Projects\stonk-sage`.
**Estimated effort:** 3 weekends of hobby time (build came in ~1 day with AI execution)
**Supersedes:** Plan 002 (architecture pivot to Copilot CLI agent files)

---

## Changelog

**Rev 4** — addresses 3 findings from 🦆 rubber-duck on Rev 3 (1 blocking + 2 non-blocking). No new tasks; surgical edits to Task 0.0 Toy 3, `guards.py check_vague_edge`, and §0.5 quota-canary text.

- **D1-extension (Toy 3 must smoke the output-capture-to-file seam — duck blocker):** Task 0.0 Toy 3 upgraded from "dispatch + write a one-line file" to actually exercising **sub-agent output capture and persistence** — the exact seam Rev 3's staging convention depends on. Toy 3 now: creates `data/runs/toy_<uuid>/`, dispatches `hello` (instructed to emit a fenced JSON block), captures hello's returned output, persists to `data/runs/toy_<uuid>/hello.md`, asserts the file is non-empty, AND round-trips the fenced JSON via `json.loads`. If output-capture turns out to be impossible from a prompt file, Toy 3 documents the limitation in `docs/dispatch-surface-findings.md` and Task 0.0 forks the plan: either (a) §1.4 has each agent itself write to staging via a tool call, or (b) re-evaluate whether the staging convention is viable. Dependency note added: 0.0 must validate output-capture-to-file before 0.3 unblocks.
- **N1-rev4 (denylist over-aggressive — duck non-blocking):** `check_vague_edge` rule changed from `denylist OR no-quant-anchor → fail` to `(denylist AND no-quant-anchor-in-same-string) OR (no-quant-anchor-at-all) → fail`. Quant anchor in the same string rescues a denylist phrase — a real edge like *"Pricing power drove FY24 gross margin expansion of 320bps per 10-K"* now passes despite containing `pricing power`. Pytest grows 8 → 10 cases.
- **N2-rev4 (Rev 3's C1 fix didn't fully land in §0.5 — duck non-blocking):** Rev 3's bottom Risks table named `gpt-5.4` as preferred DA downshift, but the in-task §0.5 Quota Canary bullet still said `claude-sonnet-4.6`. Both locations now consistent — `gpt-5.4` is the preferred target, with full reasoning inline (DA-matches-Bull rubberstamps; DA-matches-Bear is less bad).
- **N3-rev4 (Cresselia 🔵 folded in — quant-anchor regex too permissive):** Old anchor regex `r'\d|10-K|10-Q|Q\d|FY\d|\b[A-Z]{2,5}\b'` matched generic uppercase tokens like `CEO`, `SEC`, `EPS`, `ROIC`, `USA`, `API`, `AI`, `CFO` — so `"The CEO has executed well"` slipped through. Tightened to **`r'\d|10-K|10-Q|Q\d|FY\d|\$[A-Z]{2,5}\b'`** (drop loose uppercase clause; ticker recognition requires `$` prefix). Docstring note: *"Anchor regex is intentionally conservative; tune in Phase 1 once we see what slips through real-world Bull/Bear outputs."* Pytest 10 → 11 cases (new case: `"The CEO has executed well over multiple cycles"` + `BUY` → fail).

**Rev 3** — addresses 6 findings from 🦆 rubber-duck (2 blocking + 2 non-blocking) and @cresselia (2 minor) on Rev 2. No new tasks; surgical edits to §1.4 + guards.py spec + §0.5 risk + §1.5.

- **D1 (run-staging convention — duck blocker):** `/analyze` now creates `data/runs/<ticker>_<date>_<uuid>/` *before* dispatch and writes each agent's structured output there (`snapshot.json`, `da_digest.md`, `bull.json`, `bear.json`, `risk.json`, `da.json`, `cio_draft.md`). Guard CLI signature changes from `--bull/--memo` files to `--run-dir`. Only on guard pass is `cio_draft.md` published to `examples/`. Failed runs leave the staging dir intact for debugging. `data/runs/` is gitignored.
- **D2 (vague-edge heuristic — duck blocker):** `guards.py` extended with a vague-edge check (**Option B**): denylist of generic phrases (`brand strength`, `quality compounder`, `AI opportunity`, `secular growth`, `strong management`, `market leader`, `defensive moat`, `network effects`, `pricing power`, `category leader`) AND quant-anchor requirement (must contain `\d` OR a ticker symbol OR `10-K|10-Q|Q\d|FY\d`). Either failure → vague-edge violation → run fails → NO_ACTION enforcement applies. Pytest grows from 4 to 8 cases (~30 lines of guard logic added; module now ~110 lines).
- **N1 (stale filename — duck non-blocking):** All remaining mentions of non-versioned `aapl_2024-06-01.md` swapped for the `_<uuid>.md` + `_latest.md` pair from Rev 2 S3.
- **N2 (atomic _latest update — duck non-blocking):** §1.4 now specifies: write `_<uuid>.md` first; then update `_latest.md` via `os.replace()` (atomic on the same volume on Windows + POSIX). The uuid file is canonical; `_latest.md` is convenience-only.
- **C1 (DA downshift target — Cresselia minor):** §0.5 quota-canary risk now says: if downshifting Devil's Advocate off `claude-opus-4.8`, prefer `gpt-5.4` over `claude-sonnet-4.6` — DA sharing Bull's model rubberstamps the agent it's supposed to attack. If forced into `claude-sonnet-4.6`, document the limitation in the run's chat output.
- **C2 (guard-failed denominator — Cresselia minor):** §1.5 now states: a guard-failed run counts as a §1.5 failure; denominator stays at 3. If guard kills ≥2 of 3 runs, harden CIO prompts (likely vague-edge) before re-evaluating — do not proceed to Phase 2.

**Rev 2** — addresses 13 findings from @cresselia (🟡 APPROVED W/ CHANGES, 22/25) and 🦆 rubber-duck (PASS W/ REVISIONS — 4 blocking, 6 non-blocking). One new task (0.0 dispatch-surface spike), no new phases. Now 14 tasks across 3 phases.

- **B1 (dispatch-surface spike — both reviewers):** New **Task 0.0 — Dispatch-surface spike** before 0.1. Smokes 3 toy artifacts against Copilot CLI mechanics (workspace agent file dispatch + `model:` frontmatter + Standards References handling + parallel fan-out + slash-command prompt-file orchestration + argument syntax). Findings recorded in `docs/dispatch-surface-findings.md`; Task 0.3 blocks on it.
- **B2 (pipeline verify >1 run — Cresselia I2 + Duck #1):** §1.4 verify now runs `/analyze` **three times**; gates parallel-execution + completeness before §1.5.
- **B3 (honest cost framing — Cresselia I3 + Duck #6):** Removed "$0/run" framing throughout (Objective, Locked Decisions, AI Usage Declaration). New Risks row for Copilot premium-quota exhaustion with §0.5 canary + Opus→Sonnet downshift plan.
- **B4 (system-enforced CIO hard rule — Duck #4):** New `src/stonk_sage/guards.py` post-CIO validator; `/analyze` calls it; fails the run (no memo written) if Bull `source_of_edge` is empty AND memo recommendation ≠ `NO_ACTION`. Hard rule no longer model-trust-only.
- **I1 (fan-in token budget — Duck #7):** Task 0.2 spells out compact-snapshot rules; per-agent Output Contracts must reference `contracts.py` caps explicitly.
- **I2 (uv ARM64 note — Duck #8):** Moved to Task 0.1 pre-step (retained in README).
- **I3 (Saturday `as_of` — Duck #9):** Task 0.2 specifies `<= as_of` semantics for non-trading days + a pytest covering Sat 2024-06-01 → Fri 2024-05-31 close.
- **I4 (first-run UX — Duck #10):** New "Expected First Run" mini-section in Phase 2 README task.
- **I5 (preflight retirement — Cresselia 🔵):** Explicit bullet in Task 0.1 confirming no Plan 002 provider preflight needed.
- **I6 (model-string drift — Cresselia 🔵):** "Agent → Model assignments" notes cross-family invariants are locked, not specific strings; Task 0.0 includes model-availability check.
- **S1 (per-agent tool restrictions):** Each `.agent.md` frontmatter gets a `tools:` restriction — read/search only for analysts; write only for CIO.
- **S2 (filename casing):** `/analyze` normalizes ticker (lower for filename, upper for agent context).
- **S3 (memo overwrite vs versioning):** `/analyze` writes both `examples/<ticker>_<date>_<short-uuid>.md` and updates `examples/<ticker>_<date>_latest.md`.
- **S4 (two-run memo eval):** §1.5 checklist applied to **both** runs from §1.4's three-run verify (any two passes ship; otherwise harden).

**Rev 1 (new plan)** — **Supersedes Plan 002.** Same product (6-agent investment-memo committee), different runtime. Plan 002 stood up LangGraph + LiteLLM + vcrpy cassettes + a typer CLI with paid API keys; Plan B replaces all of that with `.github/agents/*.agent.md` files dispatched by Copilot CLI's `task` tool and a `/analyze` slash command. Joe asked the legitimate question "why API keys when we already pay for Copilot?" mid-execution of Plan 002. Plan B answer: no LangGraph, no LiteLLM, no cassettes, no Typer, no external provider billing, faster prompt iteration, AI-session-native UX.

What Plan 002 **already built** that survives into Plan 003 (no rework): `pyproject.toml` + `uv sync`, `src/stonk_sage/brain.py` (4 tests passing), `src/stonk_sage/contracts.py` (17 tests passing), the 10 brain files at `.github/instructions/`. What Plan 003 **drops**: `litellm`, `langgraph`, `vcrpy`, `typer`, `jinja2`, cassettes, `.prompthash` sidecars, Task 0.0 provider preflight, the `call_llm` wrapper. What Plan 003 **acknowledges as sacrificed** vs Plan 002: replay determinism (each run produces a different memo — LLMs aren't deterministic), portability beyond Copilot CLI (no server/MCP/OpenClaw — see "Hybrid path" below), granular token accounting.

---

## Objective

Ship a working 6-agent investment-memo committee in `stonk-sage` runnable entirely from inside Copilot CLI. Prove it end-to-end on AAPL/2024-06-01.

**Success = one slash command:** Joe types `/analyze AAPL 2024-06-01` in Copilot CLI from the `stonk-sage` repo and gets a Markdown memo written to `examples/aapl_2024-06-01_<uuid>.md` (with `examples/aapl_2024-06-01_latest.md` pointing at it) plus a chat summary.

---

## Non-Goals (explicit)

The following are **out of scope**:

- Live trading / brokerage integration
- Screening multiple tickers
- Backtest harness or PIT replay infrastructure beyond a single `as_of` date
- Anything that requires running outside Copilot CLI (servers, MCP, OpenClaw, phone access — see "Hybrid path" for the future-affordance note)
- Replay determinism / cassettes / prompt-hash sidecars
- LangGraph, LiteLLM, typer, jinja2, external provider API keys
- Sector-specialist agents, multi-turn debate rounds
- RESUME_STATE.md, decision docs, weekly notes, contributor tables, bus-factor planning
- Branch protection, CI cost gates, release automation
- Cost dashboards / Langfuse / observability platform
- Pre-designing the Plan A↔B hybrid (preserve the affordance, don't pre-build)
- **Plan 004.** Ship v0, learn, then decide what's next.

---

## Locked Decisions (no re-litigation)

Inherited from Plan 002 unless marked **NEW**:

- Per-ticker memo output (not screening)
- **NEW:** Runtime is Copilot CLI's `task` tool dispatching `.github/agents/*.agent.md` files; orchestration is a `.github/prompts/analyze.prompt.md` slash command
- Cross-provider Bull vs Bear is **non-negotiable** — one in `claude-*` family, one in `gpt-*` family (specifics in §"Agent → Model assignments")
- Brain at `.github/instructions/` is the knowledge source; agents reference it via Standards References and the Python helper `load_brain()`
- 6-agent topology: `data-analyst → [bull, bear, risk] (parallel) → devils-advocate → cio`
- CIO hard rule: `source_of_edge: none` → `recommendation: NO_ACTION`
- Memo must cite `spy_return_same_window` and `ticker_return_same_window` from the snapshot (no hallucinated benchmarks)
- No-look-ahead invariant: `data.py` asserts before returning a snapshot that every dated field has `date <= as_of`
- Memo quality bar = 6-item checklist from Plan 002 §1.6 (re-used verbatim; not duplicated here)
- Bull/Bear disagreement check = 4-point checklist from Plan 002 §0.5 (re-used verbatim)
- **NEW:** Output format is Markdown directly from the CIO agent (no separate YAML-then-render step — Plan B drops Jinja2)
- Pydantic `contracts.py` (already built) is **kept** as documentation of expected agent output shape — the Python helper layer parses CIO output against `CIOMemo` as a smoke check after the slash command runs
- **NEW (B4):** CIO hard rule is **system-enforced**, not model-trust-only — `src/stonk_sage/guards.py` post-CIO validator fails the run (no memo written) when Bull `source_of_edge` is empty AND memo recommendation ≠ `NO_ACTION`
- **NEW (B3):** Cost framing: **no external provider billing / no API keys.** Runs consume Copilot CLI premium-request quota — not free, just not separately metered. See Risks table.

---

## AI Usage Declaration

**What AI generates in this plan:**
- The 6 `.agent.md` files (frontmatter, role, capabilities, output contract, standards references)
- The `/analyze` slash command prompt
- `src/stonk_sage/data.py` (yfinance + edgartools fetcher; no LLM)
- Each end-to-end memo, at run time, via Copilot CLI dispatch

**What needs human judgment (Joe):**
- Whether each agent's output reads like a competent analyst (smell test)
- Whether Bull and Bear differ in *substance* and not just *tone* (cross-provider sanity check)
- Whether the CIO memo honors the `source_of_edge → NO_ACTION` hard rule
- Whether the committee theme (Pokémon vs functional vs market-themed) is the right call

**Expected AI failure modes to watch for:**
- **Bull/Bear convergence.** Even cross-provider, prompts that are too symmetric will rhyme. Mitigation: Plan 002 §0.5 4-point disagreement checklist run after first end-to-end (Phase 0). Hard gate — revise prompts before Phase 1 on failure.
- **CIO hedge-and-average.** LLMs love splitting the difference. Mitigation: Phase 1 verify step runs **both** the positive case (real `source_of_edge`) and the **blanked** case (Bull's `source_of_edge` manually cleared) and asserts the second produces `NO_ACTION`.
- **Hallucinated benchmark numbers.** CIO asked for "vs SPY" without the numbers will invent them. Mitigation: `data.py` populates `spy_return_same_window` + `ticker_return_same_window` in the snapshot JSON; CIO agent's Output Contract requires citing those specific fields by name.
- **Look-ahead bias in `data.py`.** AI may write "fetch latest 10-K" when it should be "latest filing where `filing_date < as_of`". Mitigation: Phase 0 `data.py` ends with an explicit assertion loop that raises on any row with `date > as_of`; pytest covers it with a fixture where `as_of` is earlier than the most-recent filing.
- **`.agent.md` frontmatter drift.** Copilot CLI's agent-file format may evolve. Mitigation: copy frontmatter shape from `myBrain/.github/agents/AGENT-TEMPLATE.md` verbatim; if a dispatched agent silently no-ops, that template is the first thing to re-check.
- **Snapshot JSON not actually reused.** The slash command is supposed to call `data.py` once and pass the JSON path to all 6 agents. AI may instead instruct each agent to re-fetch. Mitigation: Phase 1 verify reads the run's stderr/trace for exactly one `data.py` invocation.
- **Dispatch-surface assumptions wrong (B1).** Plan assumes Copilot CLI loads workspace `.agent.md` files, honors `model:` frontmatter, auto-injects Standards References, and fans out parallel `task` dispatches from a prompt file. Any of those can be false. Mitigation: **Task 0.0 dispatch-surface spike** validates each mechanic on toy artifacts before any real agent is authored. Findings recorded in `docs/dispatch-surface-findings.md` — referenced by all downstream tasks.
- **CIO obeys the letter not the spirit of the hard rule (B4).** Model emits "NO_ACTION" in some runs but invents a vague `source_of_edge` in others to justify a real recommendation. Mitigation: `guards.py` parses Bull's `source_of_edge` and the memo's Recommendation line; mismatch fails the run before the memo is written.

---

## Agent → Model assignments (locked)

Picked from Joe's Copilot CLI model access. Cross-provider Bull/Bear preserved.

| Agent              | Model                  | Why                                            |
|--------------------|------------------------|------------------------------------------------|
| Data Analyst       | `claude-haiku-4.5`     | Cheap, structured-output reliable              |
| Bull Researcher    | `claude-sonnet-4.6`    | Strong reasoning, Claude family                |
| Bear Researcher    | `gpt-5.4`              | **Different family from Bull** (non-negotiable) |
| Risk Officer       | `claude-haiku-4.5`     | Rules-based, doesn't need a big model          |
| Devil's Advocate   | `claude-opus-4.8`      | Strongest available; needs to attack both sides |
| CIO / Synthesizer  | `gpt-5.5`              | Strong, **different family from Devil's Advocate** |

Model strings go in each agent file's frontmatter `model:` field and in the `/analyze` slash command's `task` tool invocations.

**Model-string drift (I6):** What's locked is the **cross-family invariant** (Bull family ≠ Bear family; Devil's Advocate family ≠ CIO family), not the literal strings. If `claude-opus-4.8` deprecates in Copilot CLI mid-build, swap to the same-family successor (e.g., `claude-opus-4.9` or `claude-sonnet-4.7`) without re-running the planning gate. Task 0.0 includes a model-availability check; if a string disappears later, the §0.5 smoke is the canary.

**Per-agent tool restrictions (S1):** Each `.agent.md` frontmatter declares a `tools:` allow-list:
- Data Analyst / Bull / Bear / Risk / Devil's Advocate: read + search + shell-to-python only (no file write/edit)
- CIO: file-write allowed (it writes the memo); read + search

Reduces blast radius if an agent goes off the rails. Exact `tools:` syntax to be confirmed in Task 0.0.

---

## Committee theme — recommendation (defer to Joe)

myBrain uses Pokémon (porygon, mewtwo, uxie, klefki, metagross, cresselia, gardevoir, rotom, absol, tapu-fini). For stonk-sage I recommend **(a) market-themed Pokémon** — keeps the myBrain naming culture, adds stonk-sage flavor, no overlap with myBrain's taken names:

| Role             | Suggested Pokémon | Rationale                          |
|------------------|-------------------|------------------------------------|
| Data Analyst     | **Klink**         | Gears, structured data, mechanical |
| Bull Researcher  | **Bouffalant**    | Charges hard; literally a bull     |
| Bear Researcher  | **Ursaring**      | Fierce bear                        |
| Risk Officer     | **Registeel**     | Defensive, calculating, contained  |
| Devil's Advocate | **Spiritomb**     | Made of 108 critical spirits; attacks both sides |
| CIO              | **Alakazam**      | Synthesizer; IQ 5000; signs the memo |

Alternatives: **(b)** plain functional names (`data-analyst.agent.md`, `bull.agent.md`, etc.) — boring but unambiguous. **(c)** market-flavored archetypes (`trader`, `analyst`, `pm`) — fine but loses the myBrain flavor.

**Joe decides in Phase 0 Task 0.1.** Plan tasks below use functional names (`data-analyst`, `bull`, ...) as placeholders — global find-replace at decision time.

---

## Phase 0 — Two agents end-to-end

**Goal:** Prove `data.py` fetches AAPL/2024-06-01 with no look-ahead, prove Bull and Bear `.agent.md` files dispatched via Copilot CLI produce *substantively different* theses.

**Estimated effort:** 1 weekend.

### Tasks

**0.0 — Dispatch-surface spike (B1, prereq zero)**
- ~30–60 min, blocks Task 0.3. Validates Copilot CLI mechanics the rest of the plan assumes. Toy artifacts; throwaway.
- **Toy 1: `.github/agents/hello.agent.md`** with frontmatter `model: claude-haiku-4.5`, a `tools:` allow-list (read + search), and Standards References pointing at `.github/instructions/risk-portfolio.instructions.md`. Dispatch via `task` tool from Copilot CLI. Verify:
  - (a) Routes to `claude-haiku-4.5` (chat trace shows the model)
  - (b) Agent has access to `risk-portfolio` content — ask it to quote a phrase from that file
  - (c) `tools:` restriction is honored (ask it to write a file; should refuse or fail)
  - If Standards References is **advisory only** (agent doesn't auto-load brain), record that and revise Phase 0/1 agent files to explicitly `view` brain files in their first step.
- **Toy 2: parallel dispatch test** — single response dispatches `hello` twice with `Run started: $(date +%s%N)` instructions. Verify outputs show overlapping execution windows (true fan-out) vs strictly serial. If serialized, downgrade `/analyze` orchestration (§1.4) to sequential and update Estimated Run Time in I4.
- **Toy 3: `.github/prompts/foo.prompt.md`** that exercises the full staging seam Rev 3's D1 fix depends on. Body: (1) accept one argument; (2) compute `toy_uuid`; (3) `mkdir -p data/runs/toy_${toy_uuid}/`; (4) dispatch `hello` and instruct it to emit a fenced ` ```json ` block like `{"ok": true, "note": "<arg>"}`; (5) **capture hello's returned output** (via whatever surface Copilot CLI exposes — chat-trace last-agent-message, tool-result, fence-extraction pattern); (6) persist that output to `data/runs/toy_${toy_uuid}/hello.md`; (7) shell out: `python -c "import json,re,pathlib; t=pathlib.Path('data/runs/toy_${toy_uuid}/hello.md').read_text(); assert t.strip(), 'empty'; m=re.search(r'\x60\x60\x60json\s*(\{.*?\})\s*\x60\x60\x60', t, re.S); assert m, 'no json fence'; d=json.loads(m.group(1)); assert d['ok'] is True; print('staging+roundtrip ok')"`. Confirm exact argument syntax (likely `${input:Name}` per `myBrain/.github/prompts/orchestrate.prompt.md`, not `$VAR`).
- **Toy 3 forking outcomes (D1-extension):** if step (5) — capturing a sub-agent's output from a prompt file — is not natively supported by Copilot CLI, **stop and document in `docs/dispatch-surface-findings.md`**, then choose:
  - **(a) Agent-writes-its-own-staging:** revise §1.4 so each agent's `.agent.md` declares a `tools:` allow for writing its own structured output to `data/runs/<run_uuid>/<agent>.json` as its final step (run_uuid passed in dispatch args). Guard CLI signature unchanged. Adds ~3 lines per agent file.
  - **(b) Staging convention not viable:** escalate to Joe — guard becomes prompt-internal (slash command body passes both Bull's output and CIO's draft to guard via stdin heredocs), losing debug-from-disk affordance.
  - This decision blocks Tasks 0.3 / 0.4 / 1.1 / 1.2 / 1.3 / 1.4 — output-capture seam must be proven before any real agent is authored.
- **Model availability check (I6):** confirm each of the 6 locked model strings (`claude-haiku-4.5`, `claude-sonnet-4.6`, `gpt-5.4`, `claude-opus-4.8`, `gpt-5.5`) is currently dispatchable via `task`. Any deprecated string → swap to same-family successor; record the swap.
- **Verify:** All three toys behave as documented (or their delta from assumptions is recorded). Findings written to `docs/dispatch-surface-findings.md` (committed) with: dispatch surface confirmed, parallel-or-serial verdict, prompt-arg syntax, `tools:` syntax, Standards References behavior, model availability list. **Tasks 0.3 / 0.4 / 1.1–1.4 must reference this doc.**

**0.1 — Repo cleanup + theme decision**
- **Pre-step (I2):** On Windows ARM64, the official `uv` installer's aarch64 build fails to launch. Use `pip install --user uv` to get the x86_64 wheel that runs cleanly under ARM emulation. Confirm `uv --version` works before touching deps.
- Trim `pyproject.toml`: remove `litellm`, `langgraph`, `vcrpy`, `typer`, `jinja2` (and any transitive devs no longer needed). Keep `yfinance`, `edgartools`, `pydantic`, `pytest`, `pytest-socket` (kept for `data.py` network-isolation tests), `ruff`, `pyright`.
- Rewrite `.env.example`: remove `ANTHROPIC_API_KEY`/`OPENAI_API_KEY`/`GOOGLE_API_KEY`; keep **only** `EDGAR_IDENTITY` (SEC fair-access policy).
- **Retirement bullet (I5):** Plan 002 Task 0.0 (provider preflight) is **explicitly retired**. No Anthropic/OpenAI/Google account signups required for Plan B; the only external API is SEC EDGAR (free, identity-only).
- Joe picks committee theme (Pokémon-market / functional / archetype). Record choice as a 2-line note in `README.md` and find-replace agent slugs throughout this plan's Phase 0–1 task list as appropriate.
- **Verify:** `uv sync` succeeds; `uv run pytest` passes the existing 21 tests (brain + contracts) with the trimmed deps.

**0.2 — `src/stonk_sage/data.py` (no-LLM data fetcher with no-look-ahead invariant)**
- Function `fetch_market_snapshot(ticker: str, as_of: date) -> MarketSnapshot` (returns the existing pydantic class from `contracts.py`).
- Hard no-look-ahead rules:
  - 10-K: pick the most recent filing with `filing_date < as_of`. Never "latest".
  - Prices: slice yfinance to rows with `date <= as_of`.
  - News: stubbed fixture for v0; each item carries a `date`, filtered `<= as_of`.
- **Non-trading-day semantics (I3):** `as_of` may fall on a weekend or holiday (e.g., 2024-06-01 is a Saturday — yfinance has no row for it). Use `<= as_of` semantics throughout: `price_at_as_of` is the most recent prior trading day's close; same window-anchor for all `_return_same_window` calculations.
- **Compact snapshot rule (I1):** `MarketSnapshot` JSON must be **fan-in-budget-friendly**. No full filing excerpts (per-section summaries capped at 600 chars per `contracts.py`); no full price history (only `price_at_as_of`, `high_52w`, `low_52w`, `spy_return_same_window`, `ticker_return_same_window`, optional `sector_return_same_window`). DA + CIO must remain inside Copilot's per-dispatch context budget even after fan-in.
- Populate benchmark fields: `spy_return_same_window`, `ticker_return_same_window`, optional `sector_return_same_window` (TTM ending at `as_of`).
- Final assertion: loop every dated field in the snapshot; raise on any `date > as_of`.
- Writes JSON to `data/snapshots/<ticker>_<as_of>.json` for reuse by the slash command (so agents read from disk, not re-fetch).
- CLI entry: `python -m stonk_sage.data fetch AAPL --as-of 2024-06-01` → prints the JSON path on stdout.
- **Verify:**
  1. Live run on AAPL/2024-06-01 — eyeball populated 10-K, prices, benchmark fields.
  2. Pytest with frozen fixture: load `data/snapshots/aapl_2024-06-01.json`, assert no date field `> as_of`.
  3. Pytest: call `fetch_market_snapshot("AAPL", date(2020,1,1))` (with network) → asserts the chosen 10-K's `filing_date < 2020-01-01`, **not** the 2023 filing reused in fixture #2.
  4. **Pytest (I3):** call `fetch_market_snapshot("AAPL", date(2024,6,1))` (Saturday) → assert `price_at_as_of` equals AAPL's close on **Friday 2024-05-31**, not null and not a future date.

**0.3 — `.github/agents/data-analyst.agent.md`** *(blocked by 0.0)*
- Format per `myBrain/.github/agents/AGENT-TEMPLATE.md`: YAML frontmatter (`name`, `description`, `model: claude-haiku-4.5`, `tools:` per S1), Role (1 sentence), Capabilities, Output Contract, Quality Standards, Standards References. Apply any deltas captured in `docs/dispatch-surface-findings.md` from Task 0.0.
- Role: receive a `MarketSnapshot` JSON path, read it, produce a structured prose digest (1–3 sentence summaries of business overview, recent financials, recent price action, benchmark context) for downstream agents. Does **not** fetch raw data — that's `data.py`'s job.
- Standards References: `.github/instructions/equity-research.instructions.md`, `finance-fundamentals.instructions.md`. (If Task 0.0 found Standards References to be advisory, agent's first step explicitly `view`s these files.)
- Output Contract: digest must cite the benchmark fields by name, must not invent numbers absent from the snapshot. **Per-list caps mirror `contracts.py`** (per I1): ≤5 items per list, ≤200 chars per item, section summaries ≤600 chars.
- **Verify:** Manually invoke from Copilot CLI in `stonk-sage` per the dispatch syntax confirmed in Task 0.0; pass the snapshot JSON path; agent returns a digest matching the Output Contract.

**0.4 — `.github/agents/bull.agent.md` and `bear.agent.md`** *(blocked by 0.0)*
- Same template + `tools:` allow-list. Bull frontmatter: `model: claude-sonnet-4.6`. Bear frontmatter: `model: gpt-5.4`.
- Each takes the Data Analyst's digest + the snapshot JSON path; produces a Thesis with the same fields as `contracts.Thesis`: `headline`, `key_drivers` (≤5), `source_of_edge`, `key_risk`, `confidence`, `evidence` (≥2 distinct items).
- Output Contract: must emit a fenced code block (` ```yaml ` or ` ```json `) parseable into `contracts.Thesis` — Python helper in 0.5 verifies. Output Contract section explicitly references `contracts.Thesis` field caps (I1).
- Standards References: `equity-research`, `finance-fundamentals`, `behavioral-finance` (for Bear's contrarian framing).
- **Bear gets a distinct prompt** — not just "be bearish on the same inputs". Frame it as "What would a credible short-seller note here? Where is consensus wrong?"
- **Verify:** Each agent invoked individually in Copilot CLI produces a code block that parses into `contracts.Thesis` via `Thesis.model_validate_json(...)`.

**0.5 — Phase 0 end-to-end smoke (manual + quota canary)**
- In Copilot CLI from `stonk-sage`: run `python -m stonk_sage.data fetch AAPL --as-of 2024-06-01`, then manually dispatch `data-analyst`, then dispatch `bull` and `bear` in parallel with the snapshot path + DA digest.
- Save the two Theses as JSON, run **Plan 002 §0.5 4-point disagreement checklist** (direction-or-driver disagreement; ≥2 distinct evidence each; Bear cannot restate Bull's `key_risk`; ≤1 overlap on `key_drivers ∪ {key_risk}`).
- **Hard gate:** if the checklist fails, revise Bull/Bear prompts before Phase 1.
- **Quota canary (B3, corrected Rev 4):** record approximate Copilot premium-request consumption for this 3-agent run (e.g., from `/usage` or chat-trace count). Note in `examples/phase-0-smoke.md`. If this single smoke consumes >25% of monthly quota, **downshift Devil's Advocate from `claude-opus-4.8` to `gpt-5.4`** before Phase 1 — **NOT `claude-sonnet-4.6`**, which is Bull's model (DA matching Bull's model rubberstamps the agent it's supposed to attack; `gpt-5.4` matches Bear, which is less bad because Bear is already adversarial-to-Bull). Cross-family invariant DA≠CIO preserved (`gpt-5.4` ≠ `gpt-5.5` family is fine — different generation/role, not different family in the cross-provider sense; the invariant that matters is that DA and CIO disagree by construction, which holds). If forced into `claude-sonnet-4.6` by `gpt-5.4` availability, log the DA≡Bull-model limitation in the run's chat output.
- **Verify:** Checklist passes; smoke note (with quota observation) committed at `examples/phase-0-smoke.md`.

**Exit gate:** Phase 0 smoke note exists, checklist passes, dispatch-surface findings doc committed, all 21 existing tests + 4 new `data.py` tests are green.

---

## Phase 1 — Full committee + `/analyze` slash command

**Goal:** All 6 agents wired through a `/analyze` slash command; one invocation produces `examples/aapl_2024-06-01_<uuid>.md` + `examples/aapl_2024-06-01_latest.md`.

**Estimated effort:** 1–2 weekends.

### Tasks

**1.1 — `.github/agents/risk-officer.agent.md`**
- Frontmatter `model: claude-haiku-4.5`. Takes the snapshot JSON; returns a `RiskAssessment` (sizing band, concentration flags, veto-on-sizing-only) honoring `risk-portfolio.instructions.md`.
- Standards References: `risk-portfolio`, `finance-fundamentals`.
- Output Contract: fenced code block parseable into `contracts.RiskAssessment`.
- **Verify:** Individual dispatch → parses into `RiskAssessment` cleanly.

**1.2 — `.github/agents/devils-advocate.agent.md`**
- Frontmatter `model: claude-opus-4.8` (strongest available). Takes snapshot digest + Bull Thesis + Bear Thesis + RiskAssessment; returns a `DevilsAdvocateCritique` that attacks **reasoning quality of both** Bull and Bear (per Plan 002 §1.2 — common failure mode is DA picking a side).
- Standards References: `behavioral-finance` (bias checklist), `equity-research`.
- Output Contract: critique must contain at least one specific attack on Bull's reasoning AND one on Bear's. Parseable into `contracts.DevilsAdvocateCritique`.
- **Verify:** Joe-eyeballed: DA's output contains explicit "Bull's flaw:" and "Bear's flaw:" content, not one-sided.

**1.3 — `.github/agents/cio.agent.md`**
- Frontmatter `model: gpt-5.5` (different family from Devil's Advocate). Takes all 5 prior outputs; returns a `CIOMemo` directly as a Markdown memo (Plan B drops the YAML→Jinja2 render step — CIO emits Markdown).
- Hard rule encoded verbatim in the agent's Quality Standards: *"If `source_of_edge` from Bull is None, empty, or vague, your `recommendation` MUST be `NO_ACTION`. There is no override."*
- Output Contract: memo sections in fixed order — Recommendation, Source of Edge, Bull Case, Bear Case, Risk Notes, Devil's Advocate, Specialist Disagreements, Benchmark Comparison. Must cite `spy_return_same_window` and `ticker_return_same_window` from the snapshot by their actual values.
- Standards References: all of `equity-research`, `finance-fundamentals`, `risk-portfolio`, `behavioral-finance`.
- **Verify (positive + negative — non-negotiable):**
  1. Real run on AAPL/2024-06-01 with Bull's `source_of_edge` populated → memo has a non-`NO_ACTION` recommendation grounded in that edge.
  2. Same run with Bull's `source_of_edge` manually blanked in the input → memo **must** return `NO_ACTION`. Document both runs in `examples/cio-hard-rule-test.md` (committed).

**1.4 — `.github/prompts/analyze.prompt.md` (the `/analyze` slash command) + `src/stonk_sage/guards.py`**
- Format follows `myBrain/.github/prompts/orchestrate.prompt.md`. Argument syntax per Task 0.0 finding (likely `${input:Ticker}` / `${input:AsOf}`).
- **Ticker normalization (S2):** `${input:Ticker}` is uppercased for agent context (`AAPL`), lowercased for filename (`aapl`). Date kept as-is.
- **Run-staging convention (D1):** every `/analyze` invocation creates `data/runs/${ticker_lower}_${AS_OF}_${short_uuid}/` **before any agent dispatches**. Every agent's structured output is written to this dir as the orchestration progresses — making the run debuggable and giving `guards.py` real files to read (agent outputs live in prompt context only; the slash command must persist them). `data/runs/` is **gitignored**. Failed runs leave their staging dir intact for `cat`-based debugging.
- Body:
  1. Args: `${input:Ticker}` and `${input:AsOf}`. Compute `short_uuid` (e.g., first 8 chars of `uuid4().hex`).
  2. Create `data/runs/${ticker_lower}_${AS_OF}_${short_uuid}/`.
  3. Shell out: `python -m stonk_sage.data fetch ${TICKER_UPPER} --as-of ${AS_OF}` → capture snapshot JSON path; also copy/symlink into staging dir as `snapshot.json`.
  4. Dispatch `data-analyst`; persist its digest to `<run-dir>/da_digest.md`.
  5. **In parallel (or sequentially per Task 0.0 verdict)**, dispatch `bull`, `bear`, `risk-officer` with snapshot path + DA digest. The three must not see each other's outputs (anti-anchoring). Persist their fenced-block outputs (parsed into pydantic, then re-serialized) to `<run-dir>/bull.json`, `bear.json`, `risk.json`.
  6. Dispatch `devils-advocate` with DA digest + Bull + Bear + Risk; persist to `<run-dir>/da.json`.
  7. Dispatch `cio` with all 5 prior outputs; persist its raw Markdown to `<run-dir>/cio_draft.md` (NOT yet published to `examples/`).
  8. **Post-CIO guard (B4 + D2):** shell out to `python -m stonk_sage.guards check --run-dir data/runs/${ticker_lower}_${AS_OF}_${short_uuid}`. Guard reads `bull.json` + `cio_draft.md`; runs the hard-rule check (empty/None/whitespace) **AND the vague-edge heuristic** (denylist + quant-anchor requirement, per `guards.py` spec below). On any violation: exit non-zero with a clear error citing the offending `source_of_edge` value and the parsed Recommendation. **On failure: DO NOT publish to `examples/`; surface the violation to Joe in chat with both offending values quoted; staging dir stays put for inspection.**
  9. **Versioned memo write (S3 + N2):** if guard passes, copy `<run-dir>/cio_draft.md` → `examples/${ticker_lower}_${AS_OF}_${short_uuid}.md` **first**; then update `examples/${ticker_lower}_${AS_OF}_latest.md` via **`os.replace()` atomic-replace** (atomic on the same volume on both Windows and POSIX — avoids stale/partial `_latest` if `/analyze` is interrupted). The `_<uuid>.md` file is **canonical**; `_latest.md` is convenience-only (overwritten each run).
  10. Reply in chat with: uuid memo path, `_latest` path, recommendation line, source-of-edge line, top Bull + Bear headlines, guard result.
- **`src/stonk_sage/guards.py` (~110 lines):**
  - `check_cio_hard_rule(bull: Thesis, memo_markdown: str) -> None` — empty/None/whitespace `source_of_edge` AND recommendation ≠ `NO_ACTION` → raises `HardRuleViolation`.
  - `check_vague_edge(source_of_edge: str) -> None` (D2 — Option B, refined in Rev 4): raises `VagueEdgeViolation` if either:
    - **No quant anchor anywhere** in the string — regex `r'\d|10-K|10-Q|Q\d|FY\d|\$[A-Z]{2,5}\b'` (digit OR filing form OR `$`-prefixed ticker) must match somewhere. No match → vague. Docstring: *"Anchor regex is intentionally conservative — drops loose `\b[A-Z]{2,5}\b` which matched generics like CEO/SEC/EPS/AI. Tune in Phase 1 once we see what slips through real-world Bull/Bear outputs."*
    - **OR** the string contains a `VAGUE_EDGE_DENYLIST` phrase (case-insensitive substring) **AND** no quant anchor in the same string. A denylist phrase **plus** a quant anchor in the same `source_of_edge` is OK (real edge like *"Pricing power drove FY24 gross margin expansion of 320bps per 10-K"* — contains `pricing power` but also `FY24`/`320`/`10-K` → passes).
    - Vague edge → caller treats same as hard-rule violation (run fails unless recommendation already `NO_ACTION`).
  - `VAGUE_EDGE_DENYLIST = ["brand strength", "quality compounder", "AI opportunity", "secular growth", "strong management", "market leader", "defensive moat", "network effects", "pricing power", "category leader"]` (module-level constant, configurable in one place).
  - `__main__` CLI: `python -m stonk_sage.guards check --run-dir <path>` reads `<path>/bull.json` + `<path>/cio_draft.md`, runs both checks, exits 0/non-zero with a human-readable message.
  - Reuses regex `r'\*\*Recommendation:\*\*\s*(BUY|HOLD|TRIM|AVOID|NO_ACTION)'` to extract recommendation.
  - **Pytest (11 cases):**
    1. empty edge + `NO_ACTION` → pass
    2. empty edge + `BUY` → fail (hard rule)
    3. populated specific edge ("moat compounding from rising ROIC over 5 years per 10-K") + `BUY` → pass (has `5`, `10-K`)
    4. populated specific edge + `NO_ACTION` → pass
    5. denylist hit no anchor ("brand strength") + `BUY` → fail (vague)
    6. denylist hit no anchor + `NO_ACTION` → pass (vague is fine if NO_ACTION already chosen)
    7. no quant anchor ("the company's competitive position is durable") + `BUY` → fail (vague)
    8. quant anchor present ("FY24 gross margin expanded 320bps") + `BUY` → pass
    9. **denylist phrase + quant anchor in same string ("Pricing power drove FY24 gross margin expansion of 320bps per 10-K") + `BUY` → pass** (Rev 4 — anchor rescues denylist)
    10. **denylist phrase, no anchor ("Pricing power.") + `BUY` → fail** (Rev 4 — minimal denylist case)
    11. **uppercase generic only ("The CEO has executed well over multiple cycles") + `BUY` → fail** (Rev 4 N3 — tightened regex: CEO no longer counts as a quant anchor)
- **Verify (B2, multi-run):** Run `/analyze AAPL 2024-06-01` **three times**. Each run must:
  - Execute all 6 agent dispatches visible in chat trace
  - Honor the parallel-or-serial verdict from Task 0.0 (if Task 0.0 confirmed parallel: Bull/Bear/Risk truly fan out; if serial: prompt body documents the sequential ordering)
  - Pass the post-CIO guard
  - Write a `_<uuid>.md` file and update `_latest.md`
  - If any run skips, merges, serializes-when-it-shouldn't, or fails the guard, harden the `/analyze` prompt before §1.5. Record three run IDs in `docs/pipeline-runs.md`.

**1.5 — Memo acceptance check (two-of-three runs from §1.4)**
- Apply the **Plan 002 §1.6 6-item memo acceptance checklist** (clear recommendation; `source_of_edge` populated OR `NO_ACTION`; specialist disagreement preserved not averaged; benchmark cites `spy_return_same_window` + `ticker_return_same_window` by value; DA visibly constrains the conclusion; no generic-filler sections) to **all three memos** from §1.4.
- **Two-of-three rule (S4):** since Plan B intentionally drops replay determinism, one passing memo can be a fluke. **Ship Phase 1 only if at least 2 of 3 memos pass all 6 checklist items.** If only 1 passes, harden CIO prompt and re-run §1.4. If 0 pass, fall back to model swap or prompt rewrite.
- **Guard-failed runs (C2):** a run killed by `guards.py` (hard-rule OR vague-edge violation) counts as a **§1.5 failure** for two-of-three purposes; denominator stays at 3, no memo file exists to checklist against. If guard kills **≥2 of 3 runs**, the symptom is almost always a CIO prompt that doesn't internalize the vague-edge discipline — harden CIO prompt (and possibly Bull's `source_of_edge` framing) before re-evaluating. **Do not proceed to Phase 2 with ≥2 guard-failed runs.**
- Optional Python smoke: `tests/test_memo_smoke.py` loads `examples/aapl_2024-06-01_latest.md` and asserts the Recommendation regex matches one of the 5 allowed values.
- **Verify:** ≥2/3 memos pass the 6-item checklist. Record per-memo checklist results as a footer in each `_<uuid>.md`.

**Exit gate:** ≥2/3 `/analyze AAPL 2024-06-01` runs produce memos passing the 6-item checklist; post-CIO guard active and tested; CIO hard-rule positive+negative test documented in `examples/cio-hard-rule-test.md`.

---

## Execution Log

**2026-05-30 — Phase 0 + Phase 1 build complete** (stonk-sage commit `aad94f5`).

| Task | Status | Notes |
|---|---|---|
| 0.0 — Dispatch-surface spike | ✅ Done | Probed in-session. Findings in `stonk-sage/docs/dispatch-surface-findings.md`. Key deltas: Standards References is advisory only (agents `view` brain explicitly); agent.md frontmatter has only `name`+`description` (model set per dispatch). |
| 0.1 — Repo cleanup | ✅ Done | pyproject trimmed 10→4 runtime deps; `.env.example` reduced to `EDGAR_IDENTITY` + auto-loaded via `python-dotenv`. |
| 0.2 — `data.py` | ✅ Done | No-look-ahead invariant at 3 layers + Saturday-rollback semantics; 17 unit tests + 1 live test gated behind `-m live`. |
| 0.3 — `data-analyst.agent.md` | ✅ Done | |
| 0.4 — `bull.agent.md` + `bear.agent.md` | ✅ Done | Cross-family preserved (Claude vs GPT). |
| **0.5 — Phase 0 smoke + quota canary** | ⏸️ Blocked on Joe | Requires Copilot CLI session in stonk-sage to run `/analyze` once + apply Plan 002 §0.5 disagreement checklist + record quota burn. |
| 1.1 — `risk-officer.agent.md` | ✅ Done | |
| 1.2 — `devils-advocate.agent.md` | ✅ Done | |
| 1.3 — `cio.agent.md` | ✅ Done | |
| 1.4 — `/analyze` slash command + `guards.py` | ✅ Done | All 11 guard cases from §1.4 covered + 8 helper tests. `_QUANT_ANCHOR` regex tightened after @absol review (units required). |
| **1.5 — Memo acceptance (2-of-3)** | ⏸️ Blocked on Joe | Requires Joe to run `/analyze AAPL 2024-06-01` three times + checklist each memo. |
| 2.1 — README + Expected First Run | ✅ Done | |
| 2.2 — Dep trim verification | ✅ Done | Final runtime deps: `pydantic`, `yfinance`, `edgartools`, `python-dotenv`. Dev: `ruff`, `pyright`, `pytest`, `pytest-socket`. |
| **2.3 — Commit, push, tag v0.1.0** | ⏸️ Partial | Commit + push done (`aad94f5`). Tag held until 0.5 + 1.5 acceptance gates pass. |

**@absol code-review:** Two rounds. Round 1 surfaced 2🔴 + 2🟡 (publish flow bug, missing dotenv load, deprecated `utcfromtimestamp`, redundant veto failures) + 1🔵 (loose quant-anchor regex). All addressed in the same commit. Round 2: zero remaining 🔴/🟡 — ship verdict.

**Test status at commit:** `57 passed, 1 deselected` (live network test gated behind `-m live`).

**Next step for Joe:** Open Copilot CLI in `C:\Users\jdtra\Projects\stonk-sage` and run:

```
/analyze Ticker=AAPL AsOfDate=2024-06-01
```

That executes the full 6-agent pipeline + guards + publishes the memo. Repeat 3× for Task 1.5; on ≥2/3 passes, tag `v0.1.0`.

---

## Phase 2 — Polish + ship

**Goal:** Fresh-clone reproducibility (within Copilot CLI). Tag v0.1.0.

**Estimated effort:** 1 weekend.

### Tasks

**2.1 — README rewrite + Expected First Run section**
- Sections: What it is (1 paragraph); Quickstart (clone → `pip install --user uv` on ARM64, else official `uv` installer → `uv sync` → set `EDGAR_IDENTITY` → open in Copilot CLI → `/analyze AAPL 2024-06-01`); The 6 agents (one line each with their model); Known limits (single ticker, single date, **intentional non-determinism — each run differs**, Copilot-CLI-only runtime, no backtest, no live trading, consumes Copilot premium-request quota); Acknowledgements (Plan 002 lineage, brain at `.github/instructions/`).
- **Expected First Run section (I4)** — what Joe sees the first time he types `/analyze AAPL 2024-06-01`:
  1. Workspace trust prompt + shell-command approval for `python -m stonk_sage.data fetch …`
  2. File-write approval for the snapshot JSON (`data/snapshots/aapl_2024-06-01.json`)
  3. Chat shows `Dispatching @data-analyst…` then parallel/serial dispatches per Task 0.0 verdict
  4. ~3–8 minutes total wall time (model latency varies; Opus is slowest)
  5. Post-CIO guard runs (shell-command approval again); guard pass/fail printed
  6. Memo path printed (`examples/aapl_2024-06-01_<uuid>.md` + `_latest.md`)
  7. **If stalled >2 min on one dispatch:** check chat trace; restart `/analyze` if no progress. Re-runs are safe — they write a new `<uuid>` and update `_latest.md`.
- Note Windows ARM64 quirk: `uv` installer's aarch64 build is broken; `pip install --user uv` workaround.
- **Verify:** Joe follows his own README on a clean checkout and gets a memo without consulting the code.

**2.2 — Final dep trim + dead-code sweep**
- Confirm `pyproject.toml` has no Plan A leftover deps. Run `uv sync` clean.
- Sweep `src/` for any references to `litellm`, `langgraph`, `vcrpy`, `typer`, `jinja2`, `call_llm`, `record_cassettes`, `.prompthash` — delete or archive any orphan files.
- **Verify:** `grep -r -E '(litellm|langgraph|vcrpy|typer|jinja2|prompthash)' src/ tests/` returns no matches. `uv run pytest` green (includes `guards.py` tests). `uv run ruff check` green. `uv run pyright` green (or warnings-only).

**2.3 — Commit, push, tag v0.1.0**
- Squash-friendly commit history (`feat: phase 0 plan b`, `feat: phase 1 plan b`, `feat: phase 2 plan b`). Push to `main` of `jdtran23/stonk-sage`. Tag `v0.1.0`.
- **Verify:** Fresh clone → `uv sync` → set `EDGAR_IDENTITY` → open in Copilot CLI → `/analyze AAPL 2024-06-01` → memo appears at `examples/aapl_2024-06-01_<uuid>.md` and `_latest.md`.

**Exit gate:** Tag `v0.1.0` exists on GitHub; the README quickstart, followed on a clean clone, produces a memo.

---

## Dependencies

```
0.0 → 0.1 → 0.2 → 0.3 → 0.4 → 0.5
              ↑           ↑
              └───────────┘  (0.3, 0.4 also blocked by 0.0 — dispatch findings)
                              ↓
                             1.1 ─┐
                             1.2 ─┤  (Phase 0 must pass disagreement gate first)
                             1.3 ─┤
                                  ↓
                                 1.4 → 1.5
                                        ↓
                                      2.1 → 2.2 → 2.3
```

`0.0` (dispatch-surface spike) blocks `0.3` and `0.4` (real agent files). `1.1`, `1.2`, `1.3` are independently authorable once Phase 0 passes (each is a single agent file); `1.4` (the slash command + guards) needs all three landed.

---

## Risks (Plan-B-specific)

| Risk | Likelihood | Mitigation |
|------|-----------|------------|
| Copilot CLI's `.agent.md` format or `task` tool dispatch surface differs from assumptions | Medium | **Task 0.0 dispatch-surface spike** smokes all 3 mechanics before any real agent is built. Findings drive Phase 0/1 task-file shapes. |
| `task` tool can't pass file paths / large strings between agents cleanly | Medium | Phase 0 §0.5 manual smoke catches this *before* writing the slash command. Fallback: agents read snapshot JSON directly from `data/snapshots/` rather than receiving it as an argument. |
| Bull and Bear produce similar output despite cross-provider | Medium | Hard gate at end of Phase 0 (§0.5 disagreement checklist). Won't enter Phase 1 without it. |
| CIO hedges instead of honoring `NO_ACTION` hard rule | Medium | Phase 1 §1.3 explicit positive+negative test + **§1.4 post-CIO guard (`guards.py`) fails the run** when Bull `source_of_edge` is empty AND memo ≠ `NO_ACTION`. System-enforced, not model-trust-only. |
| `/analyze` slash command is hard to debug when something fails mid-pipeline | Medium | Each agent step writes a short "received: X / produced: Y" line to chat (Output Contract requirement); Joe can see where the pipeline broke. Multi-run §1.4 verify catches intermittent failures. |
| Copilot premium-request quota exhaustion mid-build | Medium | Phase 0 §0.5 smoke is the canary — records approximate consumption. If a single 3-agent smoke consumes >25% of monthly quota, **downshift Devil's Advocate from `claude-opus-4.8`** — prefer **`gpt-5.4`** over `claude-sonnet-4.6` (per C1: DA matching Bull's model rubberstamps the agent it's supposed to attack; DA matching Bear is less bad because Bear is already adversarial-to-Bull, the DA's primary target). If forced into `claude-sonnet-4.6` (e.g., gpt-5.4 quota also tight), document the DA≡Bull-model limitation in the run's chat output. Cross-family invariant DA≠CIO preserved in both swap targets. Re-evaluate after Phase 1 §1.4 three-run verify. |
| Look-ahead bias slips into `data.py` | Low | Hard assert in `fetch_market_snapshot`; pytest with earlier `as_of` proves filing selection respects it. Saturday-`as_of` pytest proves non-trading-day semantics. |
| `data.py` JSON snapshot file is silently re-fetched by each agent | Low | Slash command verify step checks `data.py` is called exactly once per `/analyze`. |
| Plan B "lock-in" makes a later Plan A pivot painful | Low | Brain, contracts, `data.py`, `guards.py`, and the agent role definitions all carry over verbatim — only the dispatch glue changes. See "Hybrid path" below. |
| Fan-in context overflow at DA / CIO | Low | Task 0.2 compact-snapshot rule + per-agent Output Contract length caps (I1) keep fan-in bounded. |

---

## Open questions for @cresselia / 🦆 (flag, don't block)

1. **Slash-command-emitted Markdown vs structured intermediate.** Plan B currently has CIO emit Markdown directly (no YAML→Jinja2). Alternative: CIO emits structured YAML, the slash command body formats it. Lighter on the CIO model, heavier on the slash command. Recommend Markdown-direct for v0; `guards.py` regex on the Recommendation line works either way; revisit if memo format drifts run-to-run.
2. **Theme choice (§"Committee theme")** — defer to Joe at start of Phase 0.1.

*(Rev 1 Open Question #1 — dispatch-surface assumption — was promoted to Task 0.0 per reviewer B1.)*

---

## Hybrid path (future affordance, NOT in scope for this plan)

If Plan A's portability becomes desirable later (OpenClaw, phone access, server runtime), the migration is straightforward because Plan B keeps Plan A's data shape:

- Brain files, `contracts.py`, `brain.py`, `data.py` → unchanged
- Replace `.agent.md` dispatch with LangGraph + LiteLLM → ~1 weekend
- Add vcrpy cassettes + `.prompthash` sidecars → ~1 weekend
- Add typer CLI → ~few hours
- Wrap in MCP / OpenClaw → ~1 weekend

**Plan 003 must not pre-design this.** Just don't paint into a corner that breaks it.

---

## References (don't repeat their content here)

- Plan 002 §0.5 — Bull/Bear 4-point disagreement checklist (re-used as Phase 0 §0.5 gate)
- Plan 002 §1.6 — 6-item memo acceptance checklist (re-used as Phase 1 §1.5 gate)
- Plan 002 §1.2 — DA "attacks both sides" failure-mode note (re-used in Phase 1 §1.2)
- `myBrain/.github/agents/AGENT-TEMPLATE.md` — agent file format
- `myBrain/.github/prompts/orchestrate.prompt.md` — slash command pattern reference
- `myBrain/.github/instructions/plan.instructions.md` — Plan-Generate-Verify-Iterate guardrail
