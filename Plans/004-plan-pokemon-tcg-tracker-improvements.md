# Plan 004: Pokemon TCG Tracker — Project Improvements

## Objective
Take the pokemon-tcg-tracker project from initial prototype to a well-tested, clean, production-quality codebase — then extract reusable skills and plan v2 features.

## AI Usage Declaration
- **AI will generate:** Test scaffolding, `.gitignore`, code fixes, instruction files, skill files, implementation plans
- **AI will NOT decide:** Which dependencies to keep (Joe verifies runtime), v2 feature prioritization, deployment targets
- **Expected failure modes:** Hallucinated API endpoints, incorrect SQLAlchemy patterns, signal logic misinterpretation, test mocks that don't reflect real behavior

## Phases & Tasks

---

### Phase 1: Quick Win ✅ DONE
> *Create repo instruction file for Copilot context*

- [x] **Task 1.1:** Create `pokemon-tcg-tracker.instructions.md` repo instructions file
  - Verify: File exists in `.github/instructions/`, frontmatter is valid, `applyTo` pattern is correct
  - AI Validation: Copilot recognizes repo conventions when editing project files
  - Dependencies: None

---

### Phase 2: First Demo Run ✅ DONE
> *Validate the project runs locally on Windows before making any changes*

- [x] **Task 2.1:** Set up Python virtual environment
  - Steps: `python -m venv .venv` → activate → `pip install -r requirements.txt`
  - Verify: `pip list` shows all deps installed, no errors during install
  - AI Validation: N/A — manual task
  - Dependencies: None

- [x] **Task 2.2:** Seed initial data
  - Steps: `python scripts/seed_data.py`
  - Verify: `data/` directory contains populated SQLite database, no stack traces
  - AI Validation: N/A — manual task
  - Dependencies: Task 2.1

- [x] **Task 2.3:** Start API server and explore
  - Steps: `python scripts/run_api.py` → open `http://localhost:8000/docs`
  - Verify: Swagger UI loads, `/cards` endpoint returns seeded data, `/signals` returns computed signals
  - AI Validation: N/A — manual task
  - Dependencies: Task 2.2

- [x] **Task 2.4:** Run a fetch cycle
  - Steps: `python scripts/run_fetch.py`
  - Verify: Price data is fetched and stored, no API errors, signals recompute
  - AI Validation: N/A — manual task
  - Dependencies: Task 2.3

**Phase 2 Exit Criteria:** ✅ Joe confirmed project runs end-to-end on Windows (2026-03-30).

---

### Phase 3: Critical Fixes ✅ DONE
> *Clean up the project foundation before building on it*

- [x] **Task 3.1:** Create `.gitignore`
  - Scope: Exclude `.venv/`, `data/*.db`, `.env`, `__pycache__/`, `*.pyc`, `outputs/*.json` (if generated), `.mypy_cache/`, `.pytest_cache/`
  - Verify: `git status` no longer shows ignored files; tracked files are unaffected
  - AI Validation: @porygon generates → @absol reviews for completeness against Python/.venv conventions
  - Dependencies: Phase 2 complete (so we know what gets generated)

- [x] **Task 3.2:** Audit and clean `requirements.txt` (GitHub: #6 ✅ closed)
  - Scope: Remove `beautifulsoup4`, `selenium`, `pandas` — confirm they are not imported anywhere in `src/` or `scripts/`
  - Verify: `grep -r "import beautifulsoup4\|import selenium\|import pandas\|from bs4\|from selenium\|from pandas" src/ scripts/` returns zero matches
  - AI Validation: @porygon removes deps → codebase grep confirms no imports → @absol reviews
  - Dependencies: Task 3.1
  - **Also removed:** `schedule` (unused)

- [x] **Task 3.3:** Verify project still runs after dep cleanup
  - Scope: Fresh `pip install -r requirements.txt` in clean venv, run `seed_data.py`, start API, hit key endpoints
  - Verify: All endpoints from Phase 2 still respond correctly, no `ModuleNotFoundError`
  - AI Validation: Run startup + basic endpoint check in terminal, compare output to Phase 2 baseline
  - Dependencies: Task 3.2

**Phase 3 Exit Criteria:** ✅ Clean `.gitignore` in place, 4 unused deps removed, fresh venv verified, API endpoints confirmed working (2026-03-30).

---

### Phase 4: Test Foundation 🧪
> *Establish automated testing to protect against regressions*

- [ ] **Task 4.1:** Set up pytest infrastructure (GitHub: #5)
  - Scope: Create `tests/` directory, `tests/__init__.py`, `tests/conftest.py` with shared fixtures (test DB, FastAPI TestClient), add `pytest` + `httpx` to `requirements.txt` (dev section or separate `requirements-dev.txt`)
  - Verify: `pytest --collect-only` discovers test directory with no errors
  - AI Validation: @porygon generates → run `pytest --collect-only` → must succeed with 0 errors
  - Dependencies: Phase 3 complete

- [ ] **Task 4.2:** Write API endpoint tests
  - Scope: Test all endpoints in `src/api.py` using FastAPI `TestClient` — `GET /cards`, `GET /cards/{id}`, `GET /signals`, `GET /trends`, `POST` endpoints if any. Test both success and error paths (404, invalid input).
  - Verify: `pytest tests/test_api.py -v` — all tests pass, covers happy path + error cases
  - AI Validation: @porygon writes tests → run pytest → all green → @absol reviews test quality (are edge cases covered? are assertions meaningful?)
  - Dependencies: Task 4.1

- [ ] **Task 4.3:** Write signal engine unit tests
  - Scope: Test `compute_signal()` in `src/signals.py` — rule matching, priority resolution, edge cases (no data, conflicting rules, override signals). Test against `config/signal_rules.json` structure.
  - Verify: `pytest tests/test_signals.py -v` — all pass, covers rule priority, threshold boundaries, override behavior
  - AI Validation: @porygon writes tests → run pytest → all green → @absol reviews for logic coverage (boundary conditions, empty inputs, rule conflicts)
  - Dependencies: Task 4.1

- [ ] **Task 4.4:** Write trends computation unit tests
  - Scope: Test trend calculation in `src/trends.py` — price direction, percentage changes, time window handling, insufficient data scenarios
  - Verify: `pytest tests/test_trends.py -v` — all pass
  - AI Validation: @porygon writes tests → run pytest → all green → @absol reviews for mathematical correctness
  - Dependencies: Task 4.1

- [ ] **Task 4.5:** Full test suite validation
  - Scope: Run complete test suite, ensure no flaky tests, generate coverage report
  - Verify: `pytest --tb=short -q` — all tests pass on first run. Run twice to confirm no flakiness.
  - AI Validation: Two consecutive clean runs, no warnings about unclosed resources or DB locks
  - Dependencies: Tasks 4.2, 4.3, 4.4

**Phase 4 Exit Criteria:** `pytest` runs cleanly with tests for API, signals, and trends. All pass. Test count ≥ 15.

---

### Phase 5: Code Quality Improvements 🔬
> *Fix known issues and improve reliability*

- [ ] **Task 5.1:** Fix N+1 query in `get_cards()` API endpoint (GitHub: #9)
  - Scope: Audit `src/api.py` and `src/db.py` for the cards listing query. Use SQLAlchemy eager loading (`joinedload` / `selectinload`) or a single query with joins to eliminate N+1.
  - Verify: Add a test that checks query count (or log query count before/after). Confirm API response is identical.
  - AI Validation: @porygon implements fix → @absol reviews SQL pattern → run API tests → response unchanged, query count reduced
  - Dependencies: Phase 4 complete (tests protect against regressions)

- [ ] **Task 5.2:** Add file locking / atomic writes for JSON config files (GitHub: #8)
  - Scope: `config/watchlist.json`, `config/alerts.json`, `config/signal_rules.json` — ensure concurrent reads/writes don't corrupt data. Use `tempfile` + `os.replace()` for atomic writes, or `filelock` library.
  - Verify: Write a stress test or concurrent access test. Confirm JSON files are never left in a partial-write state.
  - AI Validation: @porygon implements → @absol reviews for race conditions → test with simulated concurrent writes
  - Dependencies: Phase 4 complete

- [ ] **Task 5.3:** Evaluate Alembic for DB migrations
  - Scope: Research whether Alembic adds value given the current simple schema. Document decision. If yes, scaffold initial migration. If no, document why and what would trigger revisiting.
  - Verify: Decision documented in plan or a brief ADR. If proceeding, `alembic upgrade head` runs without error.
  - AI Validation: @uxie researches Alembic fit → decision documented → if implemented, @porygon scaffolds → verify migration runs clean
  - Dependencies: Phase 4 complete

**Phase 5 Exit Criteria:** N+1 query fixed, JSON writes are atomic/safe, Alembic decision documented. All existing tests still pass.

---

### Phase 6: Extract Skills 📦
> *Capture reusable patterns from this project into the brain*

- [ ] **Task 6.1:** Create `pokemon-tcg-ops` skill
  - Scope: Extract operational patterns learned during Phases 2-5 — common commands, debugging steps, data flow, API usage patterns. Save to `.github/skills/pokemon-tcg-ops/SKILL.md` in myBrain workspace.
  - Verify: Skill file follows SKILL.md template, triggers are relevant, workflow steps are actionable
  - AI Validation: @porygon drafts → @absol reviews for completeness and adherence to skill template → test by invoking a trigger phrase
  - Dependencies: Phases 3-5 complete (need the experience to extract from)

- [ ] **Task 6.2:** Create `fastapi.instructions.md` (if warranted)
  - Scope: Evaluate whether patterns from this project (TestClient setup, async endpoints, SQLAlchemy integration) are generic enough to be reusable. If yes, create `.github/instructions/fastapi.instructions.md` in myBrain.
  - Verify: Instructions are generic (not pokemon-tcg-specific), cover common FastAPI patterns, have proper frontmatter with `applyTo: **/*.py`
  - AI Validation: @porygon drafts → @absol reviews for generality — would this help on a DIFFERENT FastAPI project?
  - Dependencies: Phase 4 complete (need test patterns to evaluate)

- [ ] **Task 6.3:** Create `test-scaffolding` skill (if warranted)
  - Scope: Evaluate whether the pytest setup pattern (conftest.py structure, TestClient fixtures, DB fixtures) is reusable. If yes, create `.github/skills/test-scaffolding/SKILL.md`.
  - Verify: Skill is generic, not pokemon-specific, would accelerate test setup on future Python projects
  - AI Validation: @porygon drafts → @absol reviews for reusability
  - Dependencies: Phase 4 complete

**Phase 6 Exit Criteria:** At minimum, `pokemon-tcg-ops` skill created. Other skills created only if justified by reusability.

---

### Phase 7: v2 Planning 🚀
> *Plan the next evolution of the tracker — portfolio management features*

- [ ] **Task 7.1:** Review PRD
  - Scope: Read and analyze `outputs/PRD-PokéMarket-Buy-Sell-Execution.md` in the pokemon-tcg-tracker workspace. Summarize key features, identify technical dependencies, flag scope risks.
  - Verify: Summary captures all major feature areas, dependencies are identified, risks are explicit
  - AI Validation: @uxie reads and synthesizes → present summary to Joe for confirmation
  - Dependencies: Phase 5 complete (codebase is clean and tested)

- [ ] **Task 7.2:** Create v2 implementation plan
  - Scope: Decompose PRD into implementable tasks with acceptance criteria and AI validation. Create as `Plans/005-plan-pokemon-tcg-v2.md`.
  - Verify: Plan follows plan-management skill template, tasks are sized for single-agent execution, dependencies are explicit
  - AI Validation: @metagross creates plan → @cresselia validates plan completeness, task sizing, and risk coverage
  - Dependencies: Task 7.1

- [ ] **Task 7.3:** Plan review
  - Scope: @cresselia performs formal plan review — checks for missing validation steps, unrealistic task sizing, unaddressed risks, missing rollback strategies
  - Verify: Review feedback is actionable, all critical issues are addressed before execution begins
  - AI Validation: @cresselia reviews → feedback incorporated → @metagross updates plan → @cresselia confirms
  - Dependencies: Task 7.2

**Phase 7 Exit Criteria:** v2 plan created, reviewed by @cresselia, approved and ready for execution.

---

## Validation Strategy

Every AI-generated artifact must have verification steps defined BEFORE generation:

| Artifact Type | Validation Method |
|--------------|-------------------|
| `.gitignore` | `git status` shows no ignored files tracked |
| `requirements.txt` changes | `grep` for removed imports, fresh install + run |
| Test files | `pytest` runs green, `--collect-only` finds all tests |
| Code fixes (N+1, file locking) | Existing tests still pass + new targeted tests |
| Skill / instruction files | Follows template, frontmatter valid, triggers work |
| Plans | Follows plan-management template, @cresselia approved |

**Pattern enforced throughout:** Plan → Generate → Verify → Iterate. Never Plan → Generate → Done.

## Acceptance Criteria
- [x] Phase 2: Project runs locally on Joe's Windows machine
- [x] Phase 3: Clean `.gitignore`, no unused deps, project still runs
- [ ] Phase 4: pytest suite with ≥15 tests, all passing
- [ ] Phase 5: N+1 fixed, JSON writes safe, Alembic decided
- [ ] Phase 6: At least `pokemon-tcg-ops` skill extracted
- [ ] Phase 7: v2 plan created and @cresselia approved

## Risk Register
| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Deps marked "unused" are actually used dynamically | Medium | High | Grep ALL files, not just `src/`; test in clean venv |
| Signal logic is more complex than tests assume | Medium | Medium | Read `signal_rules.json` thoroughly before writing tests |
| N+1 fix changes API response shape | Low | High | API tests from Phase 4 catch regressions |
| Alembic adds complexity without value at current scale | Medium | Low | Evaluate-only task; defer if not justified |
| v2 PRD scope is too large for single plan | High | Medium | @cresselia flags scope risks; split into sub-plans if needed |

## Timeline Dependencies (Critical Path)
```
Phase 1 ✅ → Phase 2 (manual) → Phase 3 → Phase 4 → Phase 5 → Phase 6 → Phase 7
                                                          ↑
                                              (tests must exist before
                                               code quality changes)
```

Tasks within Phases 4 and 5 can be partially parallelized (e.g., Tasks 4.2/4.3/4.4 are independent after 4.1).
