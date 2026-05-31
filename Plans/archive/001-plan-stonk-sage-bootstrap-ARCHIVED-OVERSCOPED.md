# Plan 001: stonk-sage Bootstrap — Scaffolding for Multi-Agent Investment Research
Status: Draft

## Changelog
- **Rev 2.1** (current) — final refinements applied directly by Joe-approved fast path (no further reviewer cycles). Changes:
  - **Bus-factor decision LOCKED:** No backup write access; Joe accepts that disappearance >2 weeks pauses the project. Rollback Strategy reworded to drop the "teammate can resume" implication; team-read-access is for transparency/audit only.
  - **Cache key tightened (Cresselia N1 + Duck B1):** `SignalCache` path now includes `universe_hash` + `data_snapshot_id`; `Strategy` API split into `compute_signals_live()` (may call LLM) vs `load_signals_from_cache()` (never calls LLM); `precompute()` calls live + writes.
  - **Branch protection deferred (Duck B3):** Task 0.0.7 now only creates repo + collaborators; branch protection moved to Task 0.8 after first successful CI run.
  - **First-live eval reconciliation (Duck B5):** Task 3.4 amended — first `EvalRun --live` is gated, hard-capped at $5 max, actuals-vs-estimate logged; `evals/baseline.json` reconciled; second validating run mandated on >25% divergence (mirrors Task 2.8 pattern).
  - **Task 2.9 timeout (Cresselia N2):** If no teammate available within 2 weeks of Phase 2 acceptance, Joe runs the dry-run on a fresh Windows Sandbox/VM as a stand-in; flagged for re-validation at Task 4.5.
  - **Phase 0 estimate (Cresselia N3):** "1-2 weekends" → "2-3 weekends" reflecting 12-task scope.
- **Rev 2** — addresses @cresselia 🟡 APPROVED WITH CHANGES (21/25) and 🦆 PASS WITH REVISIONS. Major changes:
  - **B1:** New Task 2.4.5 specifies Game 7 pre-compute pattern + `LLMClient.forbid_calls()` context manager; Task 2.7 updated to enforce actively, not observationally.
  - **B2:** New Constraint Traceability table covering all 12 carry-forward constraints.
  - **B3:** GitHub repo creation moved to Phase 0 (Task 0.0.7); CI Actions bootstrap at Task 0.8; Task 4.1 reduced to private→public flip.
  - **B4:** New Task 0.0.8 (`RESUME_STATE.md`); team gets read access from Phase 0; revised non-goal wording.
  - **B5:** `EvalRun.estimate_cost()` + `evals/baseline.json` + Phase 3 eval cost gate (Task 3.4 amended).
  - **I1-I12:** Decision template moved to Task 0.0.5; Langfuse-in-CI split (local trace test vs CI offline smoke); Phase 2 exit-gate explicit second-run rule; `--approver` arg pre-emptively in Task 1.3; provider-vs-checklist reality check in Task 2.1; Cohen's weighted κ + recovery in Task 3.1; Nano sync timeout (1 week); new Task 1.0 (`docs/architecture/rules.md`); branch protection staged; cost reconciliation updates baseline; `stub_thesis` fallback strategy for Phase 3; teammate dry-run at end of Phase 2 (Task 2.9).
  - **S1-S8:** Pre-commit + gitleaks + pip-audit in Task 0.8; Timeline Expectation section; cassette refresh doc (Task 2.1 + recurring); overrides.log in Task 1.3; Windows-native PowerShell verification commands swept; local-OTel no-op fallback in Task 1.8.
- **Rev 1** — initial draft, structure-only changes from research output.

## Objective
Build sustainable, extensible Python scaffolding for `stonk-sage`, a multi-agent AI investment research platform, that the 4-person team (Joe, Synos, Nano, WILD NOOK, RealNumbers) can experiment on as their thinking evolves. Game 7 is the lead strategy target, but Stonks, Trend Guesstimator, and Fundamentals must plug in via the same `Strategy` interface. Success = (1) a thin vertical slice runs end-to-end before any framework deepening, (2) every architectural rule has a machine-checkable enforcement test, (3) the team can clone the repo and reach a working mocked-LLM committee run in under 10 minutes, and (4) PIT discipline is socially enforced via a tiered `PIT_STRICT`/`PIT_PARTIAL`/`PIT_INVALID` grade printed on every report.

## Timeline Expectation
Phases sum to roughly **10-13 active weekends** of work, distributed over **4-6 calendar months** for a hobby-paced project. This is not a deadline — it is the expected envelope so the team can calibrate. Velocity may compress (paired sessions) or stretch (life happens). `RESUME_STATE.md` (Task 0.0.8) makes multi-week pauses safe.

## Non-goals
- **Shipping a trading strategy or live trading.** This is scaffolding. No broker integration, no paper-trading loop, no signal publishing.
- **A profitable Game 7 implementation.** Game 7 in Phase 2 is a *single-maker / single-reviewer / single-judge* skeleton wired through the abstractions — not a tuned strategy.
- **Committee-of-N agents in v0.** Constraint #7: prove value with the simplest topology before expanding.
- **TS/React dashboard.** CLI + Langfuse UI only for v0.
- **Self-hosted LLMs or self-hosted Langfuse.** Both are abstracted so they can be swapped later; v0 uses managed providers via LiteLLM + Langfuse Cloud free tier.
- **Political-data-driven strategies enabled by default.** Quarantine module exists, but is opt-in via env flag and CI-refused by default.
- **Retroactive PIT correction.** v0 reports label PIT confidence; they do not attempt to *fix* upstream data gaps.
- **Active solicitation of team contributions before Phase 4.** Team has *read access* to the private GitHub repo from Phase 0 (so any teammate can resume Joe's work using `RESUME_STATE.md` + the latest phase tag) — but Joe owns all commits and does not assign work to others until Phase 4.
- **Public GitHub repo before Phase 4.** Private until then; flipped public at Task 4.1.

## AI Usage Declaration
- **AI will generate:** All Python scaffolding code (pyproject.toml, src/ layout, DataProvider Protocol implementations, LLMClient wrapper around LiteLLM, Committee facade hiding LangGraph, Strategy base class, baseline_etf strategy, Game 7 maker/reviewer/judge prompts and chains, vcrpy cassette generators, structured logging setup, Langfuse instrumentation, CLI commands, evals harness, hand-labeled judge benchmark templates, CI workflow YAML, architectural enforcement tests, README + onboarding doc, /decide template, contributor README stubs, `stub_thesis` fallback strategy, `LLMClient.forbid_calls()` context manager, pre-commit config).
- **AI will explain:** How LangGraph is contained behind the Committee facade (rule #8); how the LLMClient single call site enforces budget tagging; how `as_of` propagates through the data layer; how the PIT grade is computed and watermarked; how the backtest pre-compute pattern keeps LLMs out of the loop.
- **AI will NOT decide:** Strategy semantics for Game 7 (Synos owns ideation), what counts as "shareable" within the team's Discord norms, the 20-50 hand-labeled judge ground-truth scores (human judgment only), final cost budget thresholds for the team's wallet, whether to flip `--accept-weak-pit` on any specific report, the final pick between `ruff flake8-tidy-imports` vs `import-linter` for rule #8 enforcement (Task 1.6 verification spike picks based on actual tool behavior), the Cohen's-κ threshold tuning if v1 labels disagree (human-iterated rubric).

## Risks & Rabbit Holes
### AI-Specific Risks
- **LangGraph API churn.** LangGraph 1.x is recent; AI may generate calls against 0.x patterns. **Mitigation:** Pin exact version in `pyproject.toml`; Committee facade has a unit test that imports and exercises every LangGraph primitive used; CI fails on any LangGraph import outside `src/stonk_sage/agents/committee/`.
- **Hallucinated yfinance / edgartools / Finnhub method signatures.** All three have evolving APIs. **Mitigation:** Every provider is wrapped behind `DataProvider` Protocol; all tests use vcrpy cassettes — no live calls in CI; PIT Validity Checklist is a per-provider markdown doc + a pytest parametrized smoke test; checklist reconciled against actual installed library version at Task 2.1.
- **Hallucinated ruff config keys** (constraint #12). `flake8-tidy-imports.banned-module-level-imports` may not exist as named. **Mitigation:** Task 1.6 is an explicit verification spike with both branches pre-specified (ruff + import-linter fallback). Decision recorded in `docs/decisions/`.
- **AI silently calls LLMs inside backtest loops.** Catastrophic cost risk. **Mitigation:** Three layers — (a) backtest pre-compute pattern (Task 2.4.5) computes all signals before runner starts; (b) `LLMClient.forbid_calls()` context manager wraps runner iteration and raises `LLMCallInBacktestLoopError` on any attempted call; (c) architectural test forbids `llm` imports from `backtest/`.
- **AI generates code that imports LangGraph/LangChain types into strategy public interfaces** (rule #8 violation). **Mitigation:** Architectural test (constraint #10): `tests/architecture/test_strategy_purity.py` introspects every `Strategy` subclass's method signatures via `inspect.signature` + `typing.get_type_hints` and asserts no name in any annotation resolves to a module under `langgraph` or `langchain`.
- **AI fabricates Langfuse Cloud event-tier limits or pricing.** **Mitigation:** Documented in `docs/decisions/0003-observability.md` with "verified on YYYY-MM-DD" stamp; local-OTel no-op fallback (Task 1.8) means Langfuse Cloud is not a hard dependency.
- **AI generates Python that pyright passes but pytest fails** due to fixture/IO assumptions. **Mitigation:** Every code-generating task has both `pyright` and `pytest` in its Verify step.
- **AI generates Unix-only verification commands.** **Mitigation:** All Verify lines in this plan use PowerShell-native commands (e.g. `Select-String -Path`, `Get-ChildItem`); pytest commands work cross-platform.

### Project Risks
- **Two parallel scaffolding efforts.** Nano is also scaffolding. **Mitigation:** Task 0.0 includes a 1-week timeout — if no Nano response, proceed greenfield and record unilaterally.
- **Hobby-project velocity drift.** Weeks may pass between sessions. **Mitigation:** `RESUME_STATE.md` (Task 0.0.8) updated at every commit; each phase has a single exit-gate command that proves you can resume.
- **Bus-factor on Joe.** Joe is the only one doing core scaffolding. **Mitigation:** Team has private-repo read access from Phase 0; `RESUME_STATE.md` is single-page-recoverable; Phase 4 distributes ownership.
- **Free-tier provider limits.** yfinance/Finnhub rate limits will bite. **Mitigation:** `data/cache/` is a first-class layer, not an afterthought; cassettes for tests; live fetches go through a single throttled call site.
- **PIT shareability norm not adopted.** **Mitigation:** Watermark on every report + `docs/overrides.log` audit trail; Phase 4 handoff post pins the norm.
- **Cost overrun before guardrails exist.** **Mitigation:** Phase 2 ships with `--dry-run` default; first `--live` run is gated. Phase 3 eval gate (B5) extends cost discipline to eval runs.
- **Game 7 quality is unusable but eval harness still needs a thesis-producing strategy.** **Mitigation:** `stub_thesis` deterministic strategy specified in Phase 3 as fallback for exercising eval plumbing without Game 7 quality.
- **Cassette staleness.** vcrpy recordings drift as vendor APIs change. **Mitigation:** `docs/cassette-refresh.md` (Task 2.1) documents how to re-record; cadence = at every phase entry and on suspicious test failures; CI never re-records.

### Rollback Strategy
- Git repo on GitHub from Task 0.0.7; commits at every task boundary; tag at each phase exit (`phase-0`, `phase-1`, ...).
- Each phase exit-gate command must pass before tagging. If a later phase breaks, revert to the prior phase tag.
- `RESUME_STATE.md` is Joe's recovery document — when Joe returns from a pause, it tells him where he was. Teammates have read access for transparency and audit, but **Joe's disappearance pauses the project**; no backup maintainer has write access (Joe-accepted bus-factor risk, Rev 2.1). If Joe is unreachable for an extended period, the team's recovery path is to wait for Joe or to request admin handoff from Joe's GitHub account, NOT for a teammate to take over commits.
- LangGraph, LiteLLM, Langfuse, vectorbt are all behind facades — any one is swappable without changing strategy code.
- LLM hosting deferred by design (LiteLLM Router); no rollback needed for "wrong provider" — change config.
- If Phase 2's Game 7 maker proves unworkable, Phase 3 uses `stub_thesis` to exercise the eval harness; Game 7 quality is bumped to Phase 5+ without blocking eval infrastructure value.

## Constraint Traceability

All 12 carry-forward planning constraints from the validated research phase:

| # | Constraint | Task ID(s) | Verification |
|---|------------|------------|--------------|
| 1 | Tiered PIT grade (`STRICT`/`PARTIAL`/`INVALID`) + `NOT SHAREABLE` watermark + `--accept-weak-pit` override | 1.3 | Snapshot test of report renderer covers all 3 tiers + override path; override stamped into report. |
| 2 | Thin vertical slice milestone (1 ticker, 1 signal, 1 cached path, 1 backtest, 1 report — no framework deepening before) | 0.4, 0.5, 0.6 + Phase 0 exit gate | `uv run stonk-sage backtest --strategy baseline_etf` exits 0; no LLM deps in `uv.lock`; Phase 0 acceptance criteria gate Phase 1 entry. |
| 3 | Discord-first workflow (`docs/decisions/` + `/decide` template + weekly "accepted experiments" note) | 0.0.5, 4.2, 4.3 | Template exists pre-0.0; ≥5 decisions backfilled by Phase 4; weekly file convention in README. |
| 4 | Contributor bus-factor (named owner + README + smoke test + sample command per component; no contributor blocks main Game 7 path) | 4.4 | Every non-Game-7 strategy folder has `README.md` with `Owner:` line + smoke command <30s; Game 7 owned by Joe (cannot be blocked). |
| 5 | First-60-minutes onboarding <10 min, CI-verified fixture snapshots | 0.7 (Joe self-time), 2.9 (mid-project teammate time), 4.5 (Phase 4 teammate time) | Three timings recorded in `docs/onboarding-timing.md`; each <10 min (Task 2.9 allows <15 min as soft signal, hard <10 min by Phase 4). |
| 6 | `baseline_etf` mandatory control hard-coded in `evals/pit_accuracy.py` and `backtest/runner.py` | 0.4 (strategy exists), 3.3 (eval refusal logic), 3.5 (report refusal) | Report renderer raises if baseline row absent; covered by `tests/evals/test_baseline_required.py`. |
| 7 | Single-strong-maker + cheap-reviewer + judge first; expand to committee-of-N only if evals justify | Phase 2 (all of it); Phase 5+ defers expansion | Phase 2 scope explicitly forbids N>1 of any role; Phase 5+ section names committee-of-N as deferred. |
| 8 | Hand-labeled judge benchmark (20-50 examples) before R3-1 thesis-quality evals are trusted | 3.1, 3.2 | `theses_v1.jsonl` ≥20 rows; judge correlation vs human labels reported (Task 3.2) before judge used for ranking. |
| 9 | Tiered PIT grade not binary (v0 sources have known gaps; per-source confidence) | 1.2, 1.3 | Per-provider checklist (Task 1.2) yields per-source grade; report shows per-source + aggregate (Task 1.3 snapshot). |
| 10 | Architectural test for rule #8 (importable test, not just ruff) | 1.7 | `tests/architecture/test_strategy_purity.py` introspects signatures via `typing.get_type_hints`; deliberate-violation fixture fails the test. |
| 11 | CI cost-baseline workflow tiered (warn 110%, fail 150%); failure includes assumption diff; baseline update via commit trailer or PR label | 0.8 (CI exists), 2.6 (backtest cost gate), 3.4 (eval cost gate), 2.8 (baseline reconciliation update) | Two baseline files (`costs/baseline.json`, `evals/baseline.json`); deliberate prompt bloat triggers warn → fail; trailer or label clears. |
| 12 | Verify ruff `banned-module-level-imports` syntax; fall back to `import-linter` `forbidden` contracts if needed | 1.6 | Spike with deliberately-violating file; whichever tool fires, wire it; decision recorded in `docs/decisions/0002-import-enforcement.md`. |

## Overall Success Metric
The team can `git clone stonk-sage && uv sync && pytest && stonk-sage backtest --strategy game7 --ticker AAPL --as-of 2023-06-30 --dry-run` in under 10 minutes on a fresh Windows 11 machine, see a `PIT_STRICT`-graded report with baseline_etf comparison, a Langfuse trace link (or local-OTel file path), and a cost estimate — without ever touching LangGraph, LangChain, or a vendor SDK directly.

---

## Phase 0: Skeleton + Thin Vertical Slice
> *Scaffold the repo, push to GitHub, prove the end-to-end path with the simplest possible strategy (`baseline_etf`) before any abstraction work.*

**Gate:** standard review gates
**Dependencies:** None
**Estimated effort:** 2-3 weekends (12 tasks; repo + CI + RESUME_STATE.md + pre-commit + audit infra is real work)
**Owner:** Joe (primary); WILD NOOK invited to pair on logging/observability setup if available.

- [ ] **Task 0.0.5:** Author `/decide` template and `docs/decisions/` index
  - Create `docs/decisions/_template.md` with sections: Context, Options Considered, Decision, Consequences, Date, Decider(s). Add `docs/decisions/README.md` as the index.
  - Owner: Joe
  - Verify: Template file parses as a markdown doc with all 5 sections present; index references it; `pytest tests/docs/test_decision_template.py` asserts shape (use `python -m pytest` on a simple regex check).

- [ ] **Task 0.0:** Sync with Nano on parallel scaffolding work (1-week timeout)
  - Reach out via Discord. Decision matrix recorded in `docs/decisions/0001-scaffolding-fork-or-converge.md`:
    - If Nano's scaffold passes Phase 0 acceptance criteria within 1 review session → adopt as base.
    - Else cherry-pick listed components, keep this plan's structure.
    - If no Nano response within **1 calendar week**, proceed greenfield with note "awaiting Nano input — will reconcile at Phase 4 handoff."
  - Owner: Joe + Nano
  - Verify: Decision doc exists, dated, with chosen branch of the matrix; this plan's status updated if direction changes.

- [ ] **Task 0.0.7:** Create private GitHub repo `jdtran23/stonk-sage`
  - Create private repo. Add `CODEOWNERS` granting Joe ownership of all paths. Invite Synos, Nano, WILD NOOK, RealNumbers as **read-only** collaborators (per Rev 2.1 bus-factor decision — no backup write access). **Branch protection configured later in Task 0.8** (after first successful CI run; can't require status checks before the workflow exists — Duck B3 fix).
  - Owner: Joe
  - Verify: Repo URL accessible; collaborators listed as read-only; CODEOWNERS file committed.

- [ ] **Task 0.0.8:** Create `docs/RESUME_STATE.md`
  - Single-page status doc with sections: **Current Phase**, **Current Task**, **What Works Now** (last passing exit-gate command), **What's Broken** (if anything), **What's Next** (next 1-3 tasks). Add pre-commit hook (added in Task 0.8) that warns if commit message includes "Task X.Y" but `RESUME_STATE.md` is unchanged in the diff.
  - Owner: Joe
  - Verify: File exists with all 5 sections; first commit updates it; teammate (any of Synos/Nano/WILD NOOK/RealNumbers) reads it cold and can name the next command to run.

- [ ] **Task 0.1:** Initialize repo with `uv init` and src/ layout
  - `uv init stonk-sage --package`; create `src/stonk_sage/` and `tests/`; first commit + push.
  - Owner: Joe
  - Verify: `Get-ChildItem src/stonk_sage` shows package; `git log` shows initial commit; GitHub shows the push.

- [ ] **Task 0.2:** Configure pyproject.toml with locked toolchain
  - Add: pydantic, pydantic-settings, structlog, typer (CLI), pytest, pytest-cov, ruff, pyright, vcrpy. Pin Python 3.12.
  - Configure `[tool.ruff]`, `[tool.pyright]`, `[tool.pytest.ini_options]` with strict settings.
  - Owner: Joe
  - Verify: `uv sync` succeeds; `uv run ruff check .` returns 0; `uv run pyright` returns 0; `uv run pytest` finds 0 tests cleanly.

- [ ] **Task 0.3:** Create directory skeleton matching research output
  - Create empty `__init__.py` for: `config, logging, observability, budgets, llm/{client,registry}, data/{base, snapshot, yfinance_provider, edgar_provider, finnhub_provider, cache, political}, agents/{base, prompts, committee}, strategies/{base, game7, stonks, trend_guesstimator, fundamentals, baseline_etf}, backtest, evals/{datasets, judges, pit_accuracy}, cli`.
  - Add `.gitignore` entries for `data/political/output/`, `.venv/`, `.langfuse-cache/`, `__pycache__/`, `.local-otel/`.
  - Owner: Joe
  - Verify: `Get-ChildItem -Recurse src/stonk_sage -Directory` matches the spec; `pyright` still clean.

- [ ] **Task 0.4:** Implement `Strategy` base class (minimal) and `baseline_etf`
  - `Strategy` is an ABC with `name: str`, `estimate_cost(as_of) -> CostEstimate`, `generate_signals(as_of) -> list[Signal]`. NO LLM, NO LangGraph imports.
  - `baseline_etf` returns a fixed "buy SPY equal-weight, hold" signal — zero LLM, zero external data beyond a hard-coded SPY closing series fixture.
  - Owner: Joe
  - Verify: `pytest tests/strategies/test_baseline_etf.py` passes; `Select-String -Path src/stonk_sage/strategies/baseline_etf/*.py -Pattern 'langgraph|langchain|openai'` returns no matches.

- [ ] **Task 0.5:** Implement minimal `backtest/runner.py` for `baseline_etf`
  - Iterates over a date range, calls `strategy.generate_signals(as_of=date)`, accumulates returns from a fixture price series. NO vectorbt yet — pure pandas. NO LLM. Asserts no `LLMClient` is registered at entry.
  - Owner: Joe
  - Verify: `pytest tests/backtest/test_runner_baseline.py` passes.

- [ ] **Task 0.6:** Implement minimal `cli` with `stonk-sage backtest --strategy baseline_etf`
  - Typer-based CLI; prints a report with: strategy name, date range, return, PIT grade (`PIT_STRICT` for baseline_etf), `NOT SHAREABLE` watermark control (omitted on PIT_STRICT, present otherwise), baseline_etf comparison (self, trivially).
  - Owner: Joe
  - Verify: `uv run stonk-sage backtest --strategy baseline_etf` exits 0 and prints a report; snapshot test compares stdout against `tests/cli/snapshots/baseline_report.txt`.

- [ ] **Task 0.7:** Write `README.md` first-60-minutes onboarding (constraint #5)
  - Steps: install uv → `uv sync` → `pytest` → `stonk-sage backtest --strategy baseline_etf`. Target: <10 min on a clean Windows 11 box.
  - Owner: Joe
  - Verify: Joe wipes `.venv/` and `__pycache__/` (`Remove-Item -Recurse -Force .venv, __pycache__`), follows README verbatim, times it. Records timing in `docs/onboarding-timing.md`. If >10 min, fix README and re-time before Phase 0 exit.

- [ ] **Task 0.8:** CI Actions bootstrap + pre-commit + secrets scan + dep audit + branch protection
  - Add `.github/workflows/ci.yml`: `uv sync && uv run ruff check . && uv run pyright && uv run pytest`. Triggered on push + PR.
  - Add `.pre-commit-config.yaml`: ruff, ruff-format, end-of-file-fixer, `RESUME_STATE.md` updated-on-task-commit hook (custom local hook).
  - Add `gitleaks` (or `trufflehog`) action to CI workflow — fails on detected secrets. (Absorbs S1, S2.)
  - Add weekly scheduled workflow `.github/workflows/audit.yml`: `uv run pip-audit` against `uv.lock`. Failures open a GitHub issue. (Absorbs S3.)
  - Document `pre-commit install` in README onboarding steps (doesn't count toward 10-min budget — it's a one-time per-clone step).
  - **After first CI run is green:** enable main branch protection — status checks required (CI green); **PR review NOT required yet** (Joe is sole committer; staged per I9 + Rev 2.1 bus-factor decision). Configure via `gh api -X PUT repos/jdtran23/stonk-sage/branches/main/protection`.
  - Owner: Joe
  - Verify: First push to GitHub shows green CI; introduce a fake secret in a throwaway branch → gitleaks fails; restore → green. `uv run pre-commit run --all-files` clean. `gh api repos/jdtran23/stonk-sage/branches/main/protection` shows required status checks.

### Acceptance Criteria (Phase 0)
- [ ] `uv run stonk-sage backtest --strategy baseline_etf` produces a `PIT_STRICT`-graded report with baseline comparison.
- [ ] `uv run pytest` is green; `uv run ruff check .` clean; `uv run pyright` clean.
- [ ] First-60-min onboarding times under 10 minutes on a fresh machine.
- [ ] Zero LLM dependencies installed yet (no openai, no langgraph, no langchain in `uv.lock`).
- [ ] GitHub repo private + team has read access + CI green + secrets-scan green + pip-audit configured.
- [ ] `RESUME_STATE.md` exists, current, and teammate-readable.
- [ ] `git tag phase-0 && git push --tags` done.

---

## Phase 1: Core Abstractions
> *Build the facades and enforcement tests that every later phase will rely on. No real LLM calls yet; no external data calls yet — just the abstractions and the rules.*

**Gate:** standard review gates + architectural enforcement test green
**Dependencies:** Phase 0 complete
**Estimated effort:** 2-3 weekends
**Owner:** Joe; WILD NOOK invited to own `observability/` module.

- [ ] **Task 1.0:** Author `docs/architecture/rules.md` (constraint #10 prerequisite)
  - List all 8 architectural rules from the research output, one paragraph each, with an **Enforcement** column. Column starts mostly empty; fills across Phase 1 as each enforcement test/linter lands. Cross-link each rule to the test/linter file path once it exists.
  - Owner: Joe
  - Verify: All 8 rules present with name + description; enforcement column has at least placeholder rows; subsequent Phase 1 task Verify lines cite specific rule numbers from this doc.

- [ ] **Task 1.1:** Implement `DataProvider` Protocol with mandatory `as_of`
  - Protocol methods all take `as_of: datetime` (rule #6). Implement `SnapshotProvider` wrapper that records every call to a JSON manifest for PIT audit.
  - Owner: Joe
  - Verify: `pytest tests/data/test_protocol.py` enforces all methods accept `as_of`; pyright clean; architectural test asserts no method in any `DataProvider` subclass lacks `as_of`.

- [ ] **Task 1.2:** Author PIT Validity Checklist template + per-provider stubs
  - `docs/pit-checklists/{yfinance,edgar,finnhub}.md` covering: adjustments, corporate actions, symbol history, delistings, universe semantics, restatements. Each item → tiered grade (`STRICT`/`PARTIAL`/`INVALID`). Stubs reflect research-phase claims; will be reconciled against installed library reality at Task 2.1.
  - Owner: Joe (drafts); RealNumbers invited to review for data-source realism.
  - Verify: `tests/data/test_pit_checklists.py` asserts each checklist file parses to a `PITChecklist` pydantic model with all 6 dimensions present.

- [ ] **Task 1.3:** Implement tiered `PITGrade` + `--accept-weak-pit --approver <name>` + overrides log
  - `pit_accuracy.py` computes `PITGrade.STRICT | PARTIAL | INVALID` from per-provider checklists. Report renderer prints aggregate grade + per-source grade + `NOT SHAREABLE` watermark for non-STRICT unless `--accept-weak-pit --approver <name>` passed (repeatable; minimum 1, recommended 2 per team norm — see Open Q5). Override stamps `OVERRIDE: approvers=[a,b]` into the report AND appends a structured row to `docs/overrides.log` (JSON-lines: `{timestamp, command, approvers, pit_grade, justification?}`).
  - Owner: Joe
  - Verify: `pytest tests/evals/test_pit_grade.py` covers all 3 tiers + override flag behavior + repeated `--approver`; snapshot test of watermarked report stdout; overrides.log appended after override run; missing `--approver` on weak PIT raises usable error.

- [ ] **Task 1.4:** Implement `LLMClient` as single call site + `forbid_calls()` context manager
  - Wraps LiteLLM Router. Budget tags ($200/mo team, $20/day user, $5/backtest) via `pydantic-settings`. Every `.call()` records cost to structlog + Langfuse span. `__init__` accepts a `purpose: Literal["maker","reviewer","judge","estimate"]` tag.
  - **New: `LLMClient.forbid_calls()` context manager** — within the context, any `.call()` raises `LLMCallInBacktestLoopError`. Used by backtest runner (Task 2.7).
  - Owner: Joe
  - Verify: `pytest tests/llm/test_client.py` with a stub provider asserts every call carries tag + budget; test `test_forbid_calls_raises.py` enters context, calls, asserts raise; architectural test asserts NO other file in `src/stonk_sage/` imports `litellm` directly (only `llm/client.py`).

- [ ] **Task 1.5:** Implement `Committee` facade hiding LangGraph (rule #8)
  - Public surface: `class Committee: def run(self, input: CommitteeInput) -> CommitteeResult`. Inside: LangGraph. `CommitteeInput`/`CommitteeResult` are pure pydantic.
  - Owner: Joe
  - Verify: `pytest tests/agents/test_committee_facade.py` covers a no-op committee end-to-end with a stub LLMClient; architectural test (Task 1.7) enforces no LangGraph leakage.

- [ ] **Task 1.6:** Pick + verify rule-#8 import enforcement (constraint #12)
  - Spike: write `tests/architecture/_violation_fixtures/leaks_langgraph.py` that imports LangGraph at module level inside `strategies/`. Run `ruff check` with `flake8-tidy-imports.banned-module-level-imports = ["langgraph", "langchain"]`. If ruff catches it → use ruff. If ruff config key doesn't exist or doesn't fire → switch to `import-linter` with `[importlinter:contract] type = forbidden` against `langgraph`, `langchain` from `stonk_sage.strategies.*` and `stonk_sage.data.*`.
  - Record decision in `docs/decisions/0002-import-enforcement.md`.
  - Owner: Joe
  - Verify: Whichever tool wins, deliberately add a violating import to a strategy file → CI fails. Remove it → CI green. Both transitions recorded as commits.

- [ ] **Task 1.7:** Implement architectural enforcement test (constraint #10)
  - `tests/architecture/test_strategy_purity.py`: walks every `Strategy` subclass, inspects public method signatures via `typing.get_type_hints`, asserts no annotation type's `__module__` starts with `langgraph` or `langchain`. Also: every `data/*_provider.py` exports nothing whose `__module__` starts with `yfinance` / `edgar` / `finnhub` (vendor types stay inside).
  - Owner: Joe
  - Verify: Test currently green (baseline_etf passes); add a fixture that violates → test fails; remove → green. Cite rule numbers from `docs/architecture/rules.md`.

- [ ] **Task 1.8:** Wire structlog + Langfuse Cloud + traceloop-sdk + local-OTel fallback
  - `observability/` module exports `get_logger()` + `instrument()`. Backend selected by `STONK_SAGE_OBSERVABILITY` env var: `langfuse-cloud` (default if `LANGFUSE_*` keys present) or `local-otel` (writes traces to `.local-otel/*.jsonl` — first-class, not stub).
  - Owner: WILD NOOK preferred; Joe as fallback.
  - Verify: **Local pre-tag check:** `stonk-sage backtest --strategy baseline_etf` with `LANGFUSE_*` env set produces a Langfuse trace visible in the dummy project (Joe runs locally before tagging Phase 1). **CI smoke:** `tests/observability/test_smoke.py` runs with no API keys, sets `STONK_SAGE_OBSERVABILITY=local-otel`, asserts (a) `instrument()` doesn't raise, (b) trace context manager enters/exits, (c) `.local-otel/*.jsonl` has at least one span.

- [ ] **Task 1.9:** Implement `Strategy.estimate_cost()` formula + `--dry-run` mode + initial `costs/baseline.json`
  - Each strategy returns `CostEstimate(llm_calls=N, est_tokens=T, est_usd=X)`. `--dry-run` prints estimate and exits without LLM calls. Initial `costs/baseline.json` populated with baseline_etf's zero cost so the gate (Task 2.6) has a starting point.
  - Owner: Joe
  - Verify: `pytest tests/strategies/test_estimate_cost.py` parameterized over baseline_etf (zero) + a stub strategy (non-zero); `costs/baseline.json` parses as valid pydantic.

- [ ] **Task 1.10:** Political-data quarantine wiring
  - `data/political/__init__.py` raises `PoliticalDataDisabled` unless `STONK_SAGE_ENABLE_POLITICAL=1`. `data/political/output/` gitignored. CI explicitly unsets the env var and asserts import raises.
  - Owner: Joe
  - Verify: `pytest tests/data/test_political_quarantine.py` covers both states.

### Acceptance Criteria (Phase 1)
- [ ] All 8 architectural rules listed in `docs/architecture/rules.md` with at least one enforcement (test or linter) each.
- [ ] Rule-#8 enforcement tool decision recorded in `docs/decisions/0002-import-enforcement.md` and CI wired.
- [ ] `LLMClient.forbid_calls()` context exists and tested.
- [ ] `stonk-sage backtest --strategy baseline_etf` still works AND emits a trace (Langfuse Cloud locally / local-OTel in CI).
- [ ] `costs/baseline.json` exists and is valid.
- [ ] `git tag phase-1 && git push --tags` done.

---

## Phase 2: First Real LLM Strategy (Game 7, Single-Maker-Reviewer-Judge)
> *Wire Game 7 through the abstractions as the simplest agent topology (constraint #7). Real LLM calls, real data providers, cassette-backed tests, dry-run by default, pre-computed signals to keep LLMs out of the backtest loop.*

**Gate:** human-approval (first `--live` run requires Joe's explicit approval and is logged)
**Dependencies:** Phase 1 complete
**Estimated effort:** 3-4 weekends
**Owner:** Joe (engineering); Synos (Game 7 prompt/criteria authorship).

- [ ] **Task 2.1:** Implement `yfinance_provider`, `edgar_provider`, `finnhub_provider` + reconcile checklists + author `docs/cassette-refresh.md`
  - Each implements `DataProvider`, wrapped by `SnapshotProvider`, with `as_of` enforced. **For each provider, walk the Task 1.2 checklist row-by-row against the actual installed library version; amend checklist where reality disagrees; commit the diff as part of this task.** Author `docs/cassette-refresh.md`: how to re-record cassettes when vendor APIs change; cadence = phase entry + on suspicious test failures; never in CI.
  - Owner: Joe; RealNumbers invited to review checklist accuracy per provider.
  - Verify: `pytest tests/data/test_providers/` with vcrpy cassettes; no live calls in CI; checklist diff committed; `docs/cassette-refresh.md` includes a worked example.

- [ ] **Task 2.2:** Implement `cache/` layer (data)
  - Disk-backed cache keyed by `(provider, method, args, as_of)`. TTL = infinite for `as_of < today`; configurable for today.
  - Owner: Joe
  - Verify: `pytest tests/data/test_cache.py` covers hit/miss; second call to provider in test never goes through HTTP layer.

- [ ] **Task 2.3:** Author Game 7 maker prompt + signal schema
  - `agents/prompts/game7_maker.md` + pydantic `Game7Signal`. Synos authors criteria; Joe wires it.
  - Owner: Synos (content); Joe (schema/wiring).
  - Verify: Prompt file exists; `pytest tests/agents/test_game7_signal_schema.py` validates with stub LLMClient returning fixture response.

- [ ] **Task 2.4:** Author reviewer + judge prompts
  - `game7_reviewer.md` (cheap model, structured critique), `game7_judge.md` (accept/reject + reason). Single instance of each.
  - Owner: Synos (content); Joe (wiring).
  - Verify: Cassette-backed test exercises full maker→reviewer→judge sequence; output is a `CommitteeResult`.

- [ ] **Task 2.4.5:** Specify and implement Game 7 backtest pre-compute pattern (B1 fix; Rev 2.1 API split)
  - **API split (Rev 2.1):** `Strategy` exposes three methods with non-overlapping responsibilities:
    - `compute_signals_live(as_of, universe) -> list[Signal]` — **MAY call LLMs**. Research mode + precompute use this.
    - `load_signals_from_cache(strategy_version, prompt_version, universe_hash, as_of) -> list[Signal]` — **NEVER calls LLMs**. Raises `SignalCacheMiss` if missing. Backtest uses this exclusively.
    - `precompute(start, end, universe) -> SignalCache` — default impl iterates dates, calls `compute_signals_live`, writes cache. Run once before backtest.
  - **Cache key (Rev 2.1):** `cache/signals/{strategy}/{strategy_version}/{prompt_version}/{universe_hash}/{data_snapshot_id}/{as_of}.json`. `universe_hash` = `sha256(sorted(universe))[:12]`; `data_snapshot_id` = the provider snapshot stamp from `DataProvider.snapshot_id()` (e.g., `yfinance-v0.2.50-2024-06-15`). Different universe OR different data snapshot = different cache entry; cache cannot silently serve stale or wrong-shape signals.
  - **Backtest contract:** `backtest/runner.py` calls `strategy.load_signals_from_cache(...)` only, inside `LLMClient.forbid_calls()` context. Pre-compute is a separate explicit step (CLI: `stonk-sage precompute --strategy game7 ...`).
  - Implement `SignalCache` (pydantic on-disk format) + `Strategy.precompute()` default impl.
  - Owner: Joe (infra); Synos (sanity-check that Game 7 fits this shape — e.g. doesn't depend on prior-day signals; if it does, document feedback shape in cache).
  - Verify: `pytest tests/strategies/test_precompute.py` covers cache write + cache read + version invalidation + universe-hash variation + data-snapshot variation; `pytest tests/backtest/test_no_llm_in_loop.py` runs a full backtest under `forbid_calls()` after pre-compute — passes; without pre-compute → raises `SignalCacheMiss`; calling `compute_signals_live` inside backtest → raises `LLMCallInBacktestLoopError`.

- [ ] **Task 2.5:** Implement `strategies/game7/` using the Committee facade
  - `Game7Strategy(Strategy)` calls `Committee.run(...)` only; no direct LangGraph imports; no direct LLMClient (goes through Committee). Implements both `generate_signals` and uses default `precompute` from Task 2.4.5.
  - Owner: Joe
  - Verify: Architectural test from Task 1.7 still green; `pytest tests/strategies/test_game7.py` exercises full path with cassettes.

- [ ] **Task 2.6:** CI cost dry-run gate for backtests (constraint #11)
  - GitHub Actions workflow `cost-baseline.yml` runs `stonk-sage estimate-cost --strategy game7 --ticker AAPL --as-of 2023-06-30` and compares against `costs/baseline.json`. Warn at 110%, fail at 150%. **Failure output includes a diff of assumptions** (which inputs changed: prompt size? token count? model choice?). Baseline updates via commit trailer `Cost-Baseline-Update: yes` or PR label `cost-baseline-bump`.
  - Owner: Joe
  - Verify: Locally bump prompt size, run workflow → warn fires; bump again → fail fires with assumption diff visible; add trailer → green.

- [ ] **Task 2.7:** Backtest invariant — actively enforced via `forbid_calls()`
  - `backtest/runner.py` enters `LLMClient.forbid_calls()` context before iteration begins. Any LLM call inside raises `LLMCallInBacktestLoopError` (from Task 1.4). Pre-compute (Task 2.4.5) is the only sanctioned way to get LLM-derived signals into a backtest.
  - Owner: Joe
  - Verify: `pytest tests/backtest/test_forbid_calls_active.py` — deliberately-violating fake strategy that calls LLM inside `generate_signals` after pre-compute disabled → raises. Real Game 7 with pre-compute → green.

- [ ] **Task 2.8:** First `--live` Game 7 run (gated) + baseline reconciliation
  - Joe runs `stonk-sage backtest --strategy game7 --ticker AAPL --as-of 2023-06-30 --live` once. Captures Langfuse trace + cost actual + diff vs estimate.
  - **Exit-gate logic (I3 explicit):** If actuals are **within 25%** of estimate → update `costs/baseline.json` with observed value + commit trailer `Cost-Baseline-Update: yes`; tag phase-2. If actuals diverge **>25%** → tune `estimate_cost()` formula → run a **second** gated `--live` run to validate → only after validating run within 25% does `git tag phase-2` happen.
  - Owner: Joe
  - Verify: Trace URL + cost actuals committed to `docs/live-runs/0001-game7-aapl.md`; baseline updated with commit trailer; if second run needed, `0002-game7-aapl-validation.md` also committed.

- [ ] **Task 2.9:** Mid-project teammate fresh-clone dry-run (I12; Rev 2.1 timeout)
  - One teammate (Synos preferred; Nano or WILD NOOK acceptable) clones repo fresh, follows README, reports time + friction in `docs/onboarding-timing.md`. Soft threshold: **<15 min**. Hard threshold: any non-obvious failure blocks Phase 3.
  - **Timeout (Rev 2.1, Cresselia N2):** If no teammate is available within **2 weeks** of reaching Phase 2 acceptance, Joe runs the dry-run on a **fresh Windows Sandbox** (or a fresh VM) as a stand-in proxy for "teammate's machine." Records in `docs/onboarding-timing.md` with `STAND-IN` marker; flag for re-validation at Task 4.5 with a real teammate.
  - Owner: Joe coordinates; teammate executes (or Joe-as-stand-in after 2-week timeout).
  - Verify: Second row in `docs/onboarding-timing.md` with teammate initials (or `JT-STANDIN`) + time + friction notes; if >15 min or blocking failure, file fix issues and resolve before Phase 3 entry.

### Acceptance Criteria (Phase 2)
- [ ] Game 7 single-maker/reviewer/judge runs end-to-end via Committee facade on cassettes (no live calls in CI).
- [ ] `precompute` pattern + `forbid_calls()` context together prove no LLM in backtest loop.
- [ ] CI cost-baseline gate live with assumption-diff output and documented bump procedure.
- [ ] First `--live` run logged; estimate-vs-actual within 25% (or second validating run also logged); `costs/baseline.json` reconciled.
- [ ] Teammate fresh-clone dry-run completed and within thresholds (or fixed and re-run).
- [ ] `git tag phase-2 && git push --tags` done.

---

## Phase 3: Eval Harness
> *Make Game 7's (and any future strategy's) outputs measurable. Hand-labeled ground truth before any thesis-quality judge is trusted (constraint #8). Eval cost gated separately from backtest cost.*

**Gate:** human-approval (judge ground-truth labels require human authorship)
**Dependencies:** Phase 2 complete
**Estimated effort:** 3-4 weekends
**Owner:** Joe (harness); Synos + Joe (labeling); WILD NOOK (Langfuse dataset wiring).

- [ ] **Task 3.0:** `stub_thesis` fallback strategy (I11)
  - Implements `Strategy`. `generate_signals(as_of)` returns a deterministic canned thesis per ticker (hash-derived recommendation + fixed reasoning template). Zero LLM. Purpose: exercise eval plumbing without depending on Game 7 quality.
  - Owner: Joe
  - Verify: `pytest tests/strategies/test_stub_thesis.py`; deterministic output across runs; `stonk-sage backtest --strategy stub_thesis` produces a report.

- [ ] **Task 3.1:** Build 20-50 hand-labeled thesis benchmark
  - 20-50 historical (`as_of` in past) tickers + dates. Synos and Joe each independently write expected accept/reject + reasoning. Reconcile disagreements. Store in `evals/datasets/theses_v1.jsonl`.
  - **Agreement metric (I6):** Cohen's weighted κ on ordinal (reject / weak-reject / weak-accept / accept). Target ≥0.7.
  - **Recovery path:** If <0.7 → iterate rubric in `docs/evals/v1-rubric.md`, re-label disagreements, document each iteration. If still <0.7 after 2 iterations → downgrade `theses_v1` to "exploratory only" (cannot gate judge correlation); defer to v2 in Phase 5+.
  - Owner: Synos + Joe
  - Verify: File exists with ≥20 rows; per-iteration κ recorded in `docs/evals/v1-agreement.md`; final status (gating vs exploratory) explicit.

- [ ] **Task 3.2:** Implement thesis-quality judge (6 dimensions)
  - Dimensions: factual-grounding, PIT-discipline, reasoning-coherence, risk-awareness, base-rate-awareness, novelty. LLM judge prompt in `evals/judges/thesis_quality.md`. Runs against `theses_v1.jsonl`.
  - Owner: Joe (wiring); Synos (prompt content).
  - Verify: Judge run on labeled set; correlation between judge and human labels reported; if <0.5 correlation, **judge is not used for ranking** until prompt iterated (or v1 marked exploratory per Task 3.1 recovery).

- [ ] **Task 3.3:** PIT-strict realized-return eval
  - For each labeled `(ticker, as_of)`, compute Game 7's signal (or `stub_thesis` if Game 7 unusable) + realized forward return at `as_of + N days`. Compare to baseline_etf (constraint #6 — mandatory control).
  - Owner: Joe
  - Verify: `pytest evals/test_pit_realized_return.py`; report refuses to render without baseline_etf row present (covered by `tests/evals/test_baseline_required.py`).

- [ ] **Task 3.4:** Langfuse datasets + prompt regression + **eval cost gate** (B5)
  - Push `theses_v1.jsonl` as a Langfuse dataset. Configure prompt regression run on every Phase-2 prompt change.
  - **Eval cost gate (estimate):** new CLI `stonk-sage estimate-cost --eval theses_v1` returns `EvalRunCostEstimate`. New `evals/baseline.json`. CI workflow `eval-cost-baseline.yml` mirrors `cost-baseline.yml` — warn at 110%, fail at 150%. Prompt-regression CI step requires dry-run estimate to pass before live execution. Baseline bump via commit trailer `Eval-Baseline-Update: yes` or PR label `eval-baseline-bump`.
  - Owner: WILD NOOK (Langfuse wiring) + Joe (eval cost gate).
  - Verify: Dataset visible in Langfuse Cloud project; trigger a prompt change → regression run shows in UI; deliberately bloat judge prompt → eval gate warns then fails; trailer clears.

- [ ] **Task 3.4.5:** First live eval run + actuals reconciliation (Rev 2.1, Duck B5 second-half)
  - **Estimate-only gating from Task 3.4 catches drift in the formula but does NOT catch billing surprises if the formula underestimates.** This task adds the actuals side, mirroring Task 2.8's pattern for backtests.
  - Joe runs `stonk-sage evals run --dataset theses_v1 --live` once, with a hard `--max-usd 5` cap on the EvalRun (raises `EvalBudgetExceeded` before completing if exceeded). Captures Langfuse run URL + cost actual + actual-vs-estimate diff.
  - **Exit-gate logic (mirrors I3):** If actuals within **25%** of estimate → update `evals/baseline.json` with observed value + commit trailer `Eval-Baseline-Update: yes`. If actuals diverge **>25%** → tune `EvalRun.estimate_cost()` formula → run a **second** gated `--live` run to validate → only after validating run within 25% does Phase 3 progress.
  - Owner: Joe
  - Verify: Run URL + actuals committed to `docs/live-runs/eval-0001-theses-v1.md`; baseline updated with trailer; if second run needed, `eval-0002-theses-v1-validation.md` also committed; `--max-usd` cap proven to raise via a test where estimate is artificially low.

- [ ] **Task 3.5:** Eval dashboard surfacing
  - `stonk-sage evals report` prints: pass rate by dimension, baseline_etf comparison, PIT grade distribution, cost-per-thesis. Same `NOT SHAREABLE` watermark logic as backtest reports.
  - Owner: Joe
  - Verify: Snapshot test in `tests/evals/test_report_snapshot.py`.

### Acceptance Criteria (Phase 3)
- [ ] `theses_v1.jsonl` exists with ≥20 hand-labeled rows + κ recorded + status (gating or exploratory) declared.
- [ ] Thesis-quality judge correlation vs human labels reported (not necessarily passing — but measured).
- [ ] `stonk-sage evals report` runs and refuses without baseline_etf control.
- [ ] Eval cost gate live (estimate side) with documented bump procedure; `evals/baseline.json` reconciled.
- [ ] First-live eval run completed with actuals within 25% of estimate (or second validating run also completed within 25%); `--max-usd` cap behaviorally proven.
- [ ] `git tag phase-3 && git push --tags` done.

---

## Phase 4: Team Handoff
> *Flip repo public, distribute ownership, normalize the Discord-first workflow (constraint #3), and verify bus-factor (constraint #4).*

**Gate:** human-approval (team-visibility decision)
**Dependencies:** Phase 3 complete
**Estimated effort:** 1 weekend
**Owner:** Joe (logistics); whole team (ownership acceptance).

- [ ] **Task 4.1:** Flip GitHub repo private → public + tighten branch protection
  - Repo was created in Task 0.0.7 (private, team read-only). Now: flip visibility to public. Enable PR-review requirement on `main` **once the first non-Joe committer has push access** (I9 staged enforcement). Update `CODEOWNERS` to reflect Task 4.4 owners.
  - Owner: Joe
  - Verify: Repo public on GitHub; branch protection page shows status-checks + (when applicable) PR-review required; Discord announcement made.

- [ ] **Task 4.2:** Backfill `docs/decisions/` (constraint #3)
  - Using template from Task 0.0.5. At minimum: scaffolding-vs-Nano (0001), import enforcement (0002), observability (0003), cost-baseline procedure, PIT shareability norm, `--approver` count norm (Open Q5).
  - Owner: Joe
  - Verify: ≥5 decisions documented; README links to decisions index.

- [ ] **Task 4.3:** Weekly "accepted experiments" note convention
  - `docs/experiments/YYYY-WW.md` weekly file; README documents cadence; first week's file pre-populated.
  - Owner: Joe (convention); WILD NOOK (weekly upkeep ownership invited).
  - Verify: First file exists; README mentions cadence.

- [ ] **Task 4.4:** Assign component owners (constraint #4)
  - Each strategy/plugin: named owner + README + smoke test + sample command. Game 7 main path remains owned by Joe (cannot be blocked). Proposed: Stonks → Synos; Trend Guesstimator → TBD; Fundamentals → TBD; Observability → WILD NOOK; Infra/self-host → RealNumbers; Political quarantine → Joe.
  - Owner: Joe proposes in PR; team accepts in Discord; ownership table in `README.md` + `CODEOWNERS`.
  - Verify: Each non-Game-7 strategy folder has `README.md` with `Owner:` line + `make smoke-<name>` (or `uv run stonk-sage smoke <name>`) target running in <30s.

- [ ] **Task 4.5:** Final teammate onboarding re-time
  - Pair with a *different* teammate than Task 2.9 (rotate to broaden bus-factor signal). Fresh clone, follow README, record time.
  - Owner: Joe + (rotating teammate)
  - Verify: Third row in `docs/onboarding-timing.md`; if >10 min, fix README before tagging Phase 4.

- [ ] **Task 4.6:** Announce shareability norm
  - Discord post: "Only `stonk-sage report` output is shareable. Anything else (Jupyter scratch, screenshots of pandas, ChatGPT chats) is NOT a stonk-sage result and should not be quoted as one." Pin.
  - Owner: Joe
  - Verify: Post pinned; norm restated in README.

### Acceptance Criteria (Phase 4)
- [ ] Repo public, CI green, branch protection appropriately tightened.
- [ ] Decisions log ≥5 entries.
- [ ] Every component has named owner + smoke command.
- [ ] Teammate-verified onboarding ≤10 min on rotated teammate.
- [ ] Shareability norm pinned in Discord.
- [ ] `git tag phase-4 && git push --tags` done.

---

## Phase 5+: Future (deferred, not planned in detail)
> *Tracked here so the team knows where the seams are, but explicitly out of scope for this plan.*

- Committee-of-N expansion (only if Phase 3 evals show single-maker/reviewer/judge is the bottleneck).
- Additional strategies: Stonks, Trend Guesstimator, Fundamentals (each a new mini-plan; uses Phase 1 abstractions unchanged).
- Self-hosted Langfuse (trigger: sensitive content OR cost OR contributor growth past N).
- TS/React dashboard (only if CLI + Langfuse UI prove insufficient).
- Live trading integration (NOT a goal of stonk-sage; separate project).
- Political-data strategy enablement (requires its own plan + team sign-off).
- `theses_v2` labeled benchmark (if v1 downgraded to exploratory per Task 3.1 recovery).
- Game 7 quality iteration (if Phase 3 evals show Phase 2 wiring is sound but signals weak).

---

## Design Decisions
| Decision | Choice | Why | Alternative Rejected |
|----------|--------|-----|---------------------|
| Agent framework | LangGraph 1.x behind Committee facade | Mature multi-agent graph primitives, kept behind rule #8 facade so swappable | Pydantic AI (less mature multi-agent), raw LangChain (no graph), DIY (reinventing) |
| LLM abstraction | LiteLLM Router | Single call site for budgets + provider abstraction; LLM hosting deferral is a config change | Direct OpenAI SDK (lock-in), LangChain's LLM layer (couples to LangChain) |
| Data v0 sources | yfinance + edgartools + Finnhub free tier | Zero cost, sufficient for scaffolding, behind DataProvider Protocol | Polygon ($), Tiingo ($) — defer until value proven |
| Backtest engine | pandas (Phase 0) → vectorbt OSS (Phase 2+) | Start trivial, upgrade when needed; nautilus_trader is named escape hatch | nautilus_trader v0 (too heavy), QuantConnect (cloud lock-in) |
| LLM-in-backtest discipline | Pre-compute pattern + `forbid_calls()` context | Active enforcement, not observational; reproducible (cache keyed by strategy+prompt version) | Observational counter (race-prone), banning LLMClient construction (too restrictive) |
| Observability | Langfuse Cloud + traceloop-sdk + local-OTel fallback | Cloud for rich UX; local-OTel so observability isn't a hard dependency | Self-host Langfuse (defer), LangSmith (vendor lock to LangChain) |
| Cost guardrails | LiteLLM budgets + estimate_cost + dry-run + 2 CI gates (backtest + eval) | Defense in depth across both spending surfaces | Single gate (insufficient given eval is bigger repeated spend) |
| Tooling | uv + ruff + pyright + pytest + pre-commit + gitleaks + pip-audit + pydantic-settings + structlog | Modern Python defaults; secrets + deps scanned | poetry (slower), mypy (slower than pyright), no scanners (table-stakes risk) |
| Topology v0 | Single maker + single reviewer + single judge | Constraint #7 — prove value before scaling agents | Committee-of-N upfront (no eval signal yet to justify) |
| PIT grade | Tiered STRICT/PARTIAL/INVALID + per-source + watermark + overrides.log | Constraints #1, #9 — binary is dishonest; audit trail enables team retros | Binary valid/invalid (lies), no grade (no social enforcement) |
| Rule #8 enforcement | Decided in Task 1.6 via verification spike | Constraint #12 — Tapu Fini flagged ruff syntax may not exist | Picking blindly without verifying tool behavior |
| Repo creation timing | Phase 0 private + team read access | Avoids CI/tag sequencing impossibility; supports bus-factor | Phase 4 creation (B3, B4 problems) |
| `--accept-weak-pit` shape | `--approver <name>` repeatable arg | Cheap insurance for Open Q5; supports 1 or 2-approver norm without rework | Single boolean flag (no audit), require resolution before Task 1.3 (blocks impl) |

## Validation Strategy

Every AI-generated artifact in this plan has explicit verification:

1. **Code artifacts:** `uv run ruff check . && uv run pyright && uv run pytest` must all pass after generation. CI enforces on every push (from Task 0.8).
2. **Architectural rules:** Machine-checkable tests in `tests/architecture/` for rules #1, #2, #4, #6, #7, #8 (rule #3 agent-purity and rule #5 strategy-composition covered by the strategy-purity test). Tests fail when given deliberate violation fixtures. All 8 cross-referenced in `docs/architecture/rules.md`.
3. **Cost behavior:** Two CI gates — `cost-baseline.yml` (backtest, Task 2.6) and `eval-cost-baseline.yml` (evals, Task 3.4). First `--live` run (Task 2.8) validates estimator accuracy and reconciles baseline.
4. **PIT behavior:** `pit_accuracy.py` unit-tested for all three tiers (Task 1.3); report renderer snapshot-tested with all watermark states; `docs/overrides.log` appended on every override.
5. **Onboarding:** Timed three times — Joe (Task 0.7), mid-project teammate (Task 2.9), Phase 4 rotated teammate (Task 4.5). All ≤10 min by Phase 4.
6. **Eval ground truth:** Hand-labeled (Task 3.1) with Cohen's weighted κ; recovery path defined if <0.7. Judge correlation measured before judge trusted for ranking.
7. **CI secret handling:** CI has no *data-provider* API keys (yfinance/edgar/finnhub all cassette-backed). Langfuse Cloud key optional — if present in CI secrets, CI runs against dummy project; if absent, observability tests fall back to local-OTel.
8. **LLM-in-backtest invariant:** Active enforcement via `LLMClient.forbid_calls()` context (Task 1.4 + Task 2.7), not observational counters.
9. **Rollback:** `git tag phase-N` at every phase exit; any phase revertable to prior tag.

Pattern is **Plan → Generate → Verify → Iterate**. No task here ends at "generated."

---

## Open Questions for Team

1. **Nano convergence (Task 0.0):** Does Nano's scaffolding overlap enough that this plan becomes a code-review/PR plan instead of greenfield? 1-week timeout defaults to greenfield.
2. **Component ownership (Task 4.4):** Synos for Stonks is the natural fit; who owns Trend Guesstimator and Fundamentals?
3. **Cost budget reality-check:** $200/mo team / $20/day user / $5/backtest are research defaults. Comfortable, or tighter?
4. **Langfuse Cloud account ownership:** Whose account hosts the project? Affects billing past 50K events/mo. RealNumbers may have opinions given self-hosting interest.
5. **PIT override approver count norm:** `--approver` is repeatable (implemented pre-emptively in Task 1.3). Team norm: 1 approver enough, or require 2 for any non-PIT_STRICT report quoted as a "result"? Resolve before first override usage; decision goes in `docs/decisions/` per Task 4.2.
6. **GitHub repo public flip (Task 4.1):** Currently planned for Phase 4. Any reason to delay or accelerate? Affects whether prompts, decisions log, and live-run cost numbers should be sanitized.
7. **Judge model choice (Task 3.2):** Which model runs the thesis-quality judge? Cheap-and-cheerful keeps eval costs low but may correlate poorly with humans.
