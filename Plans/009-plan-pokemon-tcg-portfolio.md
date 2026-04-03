# Plan 009: Pokemon TCG Portfolio (v2 Features)
Status: Draft

## Objective
Add a buy/sell execution layer to the Pokemon TCG Tracker — transforming it from a signal-only tool into a full portfolio manager. Users can log purchases, view held positions with real-time P&L, log sales (including partial sells), track trade history with win/loss stats, set sell-target alerts on positions, and manage sealed product holdings. The portfolio backend is buildable independently; the frontend layers on top of Plan 005's React app.

## Non-goals
- Payment processing — we link out to marketplaces, not process transactions
- Graded card variants (PSA, BGS) — roadmap item, not v1
- Multi-user or shared portfolios
- Tax optimization logic — we document raw numbers, not financial advice
- Lot-splitting helper — v1 requires manual per-card cost-basis entry
- Real-time sealed product price feeds — sealed prices are manual entry only
- Position editing/correction — v1 requires delete-and-rebuy to fix data entry mistakes

## AI Usage Declaration
- **AI will generate:** SQLAlchemy models, Pydantic schemas, FastAPI endpoints, pytest tests, React components, TypeScript types, TanStack Query hooks, P&L calculation logic
- **AI will NOT decide:** Fee percentage defaults (Joe confirms marketplace norms), portfolio UI layout priority, which sealed product categories to support, whether to use a separate DB file vs extend the existing one

## Risks & Rabbit Holes
### AI-Specific Risks
- Hallucinated SQLAlchemy relationship patterns (e.g. incorrect FK cascade behavior with SQLite)
- P&L calculation errors — floating point rounding on currency, incorrect partial-sell cost-basis math
- Incorrect Pydantic model validation (nullable vs required fields for sealed products vs singles)
- Stale shadcn/ui or TanStack Table API patterns — must verify against installed versions

### Project Risks
- **Plan 005 dependency:** Frontend phases require Plan 005 Phase 6+ complete (alerts panel). If 005 stalls, backend work can proceed but frontend is blocked. **Contingency:** If Plan 005 Phase 6 is not complete when Phase 4 finishes, options are: (a) build a minimal standalone portfolio page without alert-panel integration, or (b) block and prioritize Plan 005 completion.
- **Schema migration:** Adding 3 new tables to an existing SQLite DB without Alembic. Plan 004 decided against Alembic, but 3 new tables pushes that boundary — revisit decision.
- **Scope creep:** PRD is large (~8 endpoints, 3 models, 5 frontend pages). Strict phase gating prevents half-done features.
- **Sealed products diverge:** Different data shape (no card_id, no variant) risks polluting the singles model. `product_type` discriminator must be clean.

### Rollback Strategy
- Each phase is committed as a working checkpoint. If a phase breaks the app, `git revert` to the previous phase's commit.
- New tables are additive — existing `cards` and `price_snapshots` tables are not modified.
- Frontend pages are new routes — existing pages are not touched.

## Overall Success Metric
A user can log a card purchase, see it in a portfolio dashboard with live P&L, log a partial or full sale, view trade history with realized gains, and set sell-target alerts — all through the React UI backed by tested API endpoints.

---

## Phase 1: Schema & Models
> *Define the data layer for positions, sales, and sell targets*

**Dependencies:** None (fully independent of Plan 005)

- [ ] **Task 1.1:** Revisit Alembic decision — **must produce binary decision**
  - Plan 004 Task 5.3 decided "NOT now" for Alembic because the schema was 2 tables and stable. We're adding 3 new tables.
  - **If Alembic adopted:** scaffold initial migration covering all 5 tables.
  - **If Alembic rejected:** document the specific, concrete trigger for adoption (e.g., "first ALTER TABLE needed on any of the 3 new portfolio tables").
  - Decision must be binary (adopt or reject with trigger) — not "evaluate."
  - Verify: Design Decisions table updated with choice and trigger criteria. If adopted, `alembic upgrade head` creates all tables.

- [ ] **Task 1.2:** Create SQLAlchemy models for `positions`, `sales`, `sell_targets`
  - Add to `src/models.py` (or new `src/portfolio_models.py` — see Design Decisions)
  - `positions`: id, product_type (enum: "single" | "sealed"), card_id (nullable for sealed), card_name, set_id, variant (nullable for sealed), condition, qty_purchased, qty_remaining, cost_basis_per_unit, marketplace, buy_date, notes, created_at
  - `sales`: id, position_id (FK → positions), qty_sold, sale_price_per_unit, marketplace_fee_pct, sale_date, marketplace, realized_gain_loss, notes, created_at
  - `sell_targets`: id, position_id (FK → positions), target_price (nullable), target_gain_pct (nullable), review_date (nullable), triggered (bool default False), created_at
  - Verify: `python -c "from src.models import Position, Sale, SellTarget; print('OK')"` imports cleanly. `init_db()` creates all 3 new tables in SQLite.

- [ ] **Task 1.3:** Create Pydantic request/response schemas
  - Add to `src/schemas.py` (new file) or inline in api module
  - Request schemas: `BuyRequest`, `SellRequest`, `SellTargetCreate`
  - Response schemas: `PositionResponse` (with calculated fields: unrealized_pnl, unrealized_pnl_pct, days_held, current_market_price, current_signal), `SaleResponse`, `SellTargetResponse`, `PortfolioSummary`, `TradeHistoryEntry`
  - Validate: `product_type="sealed"` makes `card_id` and `variant` optional; `product_type="single"` requires `card_id`
  - Verify: `npm run build` N/A — `python -c "from src.schemas import BuyRequest; print(BuyRequest.model_json_schema())"` outputs valid JSON schema. Test that sealed product validation allows null card_id.

- [ ] **Task 1.4:** Add `init_db()` migration for new tables
  - Extend `src/db.py` `init_db()` to create new tables via `Base.metadata.create_all()` (additive — existing tables unaffected)
  - Verify: Start fresh DB → `init_db()` creates all 5 tables. Start existing DB → `init_db()` adds 3 new tables without data loss. Run existing test suite — all 35+ tests still pass.

### Acceptance Criteria (Phase 1)
- [ ] 3 new SQLAlchemy models defined with correct columns, types, and FK relationships
- [ ] Pydantic schemas validate singles vs sealed product_type correctly
- [ ] `init_db()` creates new tables without breaking existing schema
- [ ] Existing test suite passes unchanged

---

## Phase 2: Portfolio API — Core Endpoints
> *Buy flow, portfolio listing, and sell flow*

**Dependencies:** Phase 1 complete

- [ ] **Task 2.1:** Implement `POST /api/portfolio/buy` endpoint
  - Accepts `BuyRequest`, creates a `Position` row, returns `PositionResponse`
  - If `product_type="single"` and card_id not in watchlist, auto-add to watchlist (so price tracking starts)
  - Validate: qty > 0, cost_basis > 0, buy_date not in future
  - Verify: `curl -X POST /api/portfolio/buy -d '{...}'` → 201 with position data. Check DB row created. Watchlist auto-add verified via `/api/watchlist`.

- [ ] **Task 2.2:** Implement `GET /api/portfolio` endpoint
  - Returns all open positions (qty_remaining > 0) with calculated fields
  - Calculated per position: `current_market_price` (latest from price_snapshots), `unrealized_pnl` (market - cost_basis) * qty_remaining, `unrealized_pnl_pct`, `days_held` (today - buy_date), `current_signal` (from cards table)
  - For sealed products: `current_market_price` = null (no automated price source)
  - Support query params: `?product_type=single|sealed`, `?signal=buy|sell|hold`
  - Verify: Create 3 positions via buy endpoint → GET returns all 3 with correct P&L math. Verify filtering works.

- [ ] **Task 2.3:** Implement `POST /api/portfolio/sell` endpoint
  - Accepts `SellRequest` (position_id, qty_sold, sale_price_per_unit, marketplace_fee_pct, sale_date, marketplace, notes)
  - Validate: qty_sold <= position.qty_remaining, position exists and is open
  - Calculate: `realized_gain_loss = (sale_price * (1 - fee_pct/100) - cost_basis) * qty_sold`
  - Update position: `qty_remaining -= qty_sold`
  - Create `Sale` row with calculated realized_gain_loss
  - Verify: Buy 5 units → sell 2 → position shows qty_remaining=3 → sell remaining 3 → position closed (qty_remaining=0). P&L math manually verified.

- [ ] **Task 2.4:** Implement `GET /api/portfolio/history` endpoint
  - Returns all closed positions (qty_remaining == 0) joined with their sales records
  - Response includes: card info, buy_date, sell dates, cost_basis, total_sale_proceeds, realized_pnl, realized_pnl_pct, days_held
  - For partial sells: a position with multiple sales shows each sale's contribution
  - Verify: Close a position → appears in history. Open positions do NOT appear.

- [ ] **Task 2.5:** Implement `GET /api/portfolio/summary` endpoint
  - Aggregate stats: total_invested (sum cost_basis * qty for open positions), current_portfolio_value (sum market_price * qty for open positions), total_unrealized_pnl, total_unrealized_pnl_pct, open_position_count, total_realized_pnl, total_realized_losses, net_realized_pnl, win_rate, avg_holding_period_days, best_trade, worst_trade
  - Verify: Create known positions and sales → summary numbers match manual calculation.

### Acceptance Criteria (Phase 2)
- [ ] All 5 endpoints return correct data for happy-path scenarios
- [ ] P&L calculations are accurate (verified with manual math on test data)
- [ ] Partial sell correctly reduces qty_remaining and creates sale record
- [ ] Sealed products handled (null market price, no signal)
- [ ] Existing API endpoints unaffected — full test suite passes

---

## Phase 3: Portfolio API — Tests
> *Comprehensive test coverage for all portfolio endpoints*

**Dependencies:** Phase 2 complete

- [ ] **Task 3.1:** Create test fixtures for portfolio
  - In `tests/conftest.py`: fixtures for sample positions (singles + sealed), sample sales, sample sell targets
  - Ensure test DB isolation — portfolio tables created and cleaned per test
  - **Important:** Verify that all module-level Path constants used by portfolio code are monkeypatched in conftest.py. Follow the existing pattern of patching both `src.api` and `src.signals` paths — any new module with its own Path constants needs the same treatment.
  - Verify: Fixtures load without error, don't interfere with existing test fixtures

- [ ] **Task 3.2:** Write buy flow tests
  - Happy path: buy single, buy sealed
  - Validation: missing required fields, qty <= 0, future buy_date, invalid product_type
  - Watchlist auto-add: buy a card not on watchlist → verify it's added
  - Verify: All tests pass, cover both product types

- [ ] **Task 3.3:** Write portfolio listing tests
  - Returns only open positions (qty_remaining > 0)
  - P&L calculation accuracy (known cost basis + known market price = expected P&L)
  - Filtering by product_type and signal
  - Sealed positions show null market price
  - Verify: All assertions use manually calculated expected values

- [ ] **Task 3.4:** Write sell flow tests
  - Partial sell: buy 5, sell 2 → qty_remaining=3
  - Full close: sell all remaining → position closed
  - Over-sell: try to sell more than remaining → 400 error
  - P&L math: fee deduction, gain/loss calculation
  - Verify: Edge cases covered — sell with 0% fee, sell at loss, sell at exact cost basis

- [ ] **Task 3.5:** Write history and summary tests
  - History shows only closed positions
  - Summary stats match manual calculation
  - Empty portfolio → summary returns zeros gracefully
  - Verify: Win rate, avg holding period, best/worst trade all tested

### Acceptance Criteria (Phase 3)
- [ ] 20+ new portfolio tests covering buy, sell, list, history, summary
- [ ] Both product types (single, sealed) tested
- [ ] Edge cases: partial sell, over-sell, empty portfolio, zero-fee sale
- [ ] All new + existing tests pass (target: 55+ total tests)

---

## Phase 4: Sell Targets & Alert Integration
> *Set price/gain targets on positions; integrate with existing alert engine*

**Dependencies:** Phase 2 complete

- [ ] **Task 4.1:** Implement sell target CRUD endpoints
  - `POST /api/portfolio/targets` — create a sell target (target_price and/or target_gain_pct and/or review_date)
  - `GET /api/portfolio/targets` — list active (non-triggered) sell targets with current position data
  - `DELETE /api/portfolio/targets/{id}` — remove a sell target
  - Validate: at least one of target_price, target_gain_pct, review_date must be set
  - Verify: Create target → list shows it → delete → gone. Invalid request (no criteria) returns 422.

- [ ] **Task 4.2:** Implement sell target evaluation logic
  - New function (e.g., `evaluate_sell_targets()`) that checks all active targets against current market data
  - Price target: triggered when current_market_price >= target_price
  - Gain % target: triggered when unrealized_pnl_pct >= target_gain_pct
  - Review date: triggered when today >= review_date
  - Mark triggered targets: `triggered = True`
  - Verify: Create targets with known thresholds → inject price data that crosses threshold → evaluate → targets marked triggered.

- [ ] **Task 4.3:** Integrate sell targets with existing alert engine
  - When a sell target triggers, surface it via the existing alerts mechanism (so it appears in the alerts panel alongside price alerts)
  - Approach: create `get_triggered_sell_targets(session)` in `src/portfolio.py` (or similar), returning triggered targets in alert-compatible format. The API aggregates both sources (`get_triggered_alerts()` + `get_triggered_sell_targets()`) when returning alerts.
  - Do NOT extend the JSON-based alerts system with SQLite data — keep data sources separate, aggregate at the API layer.
  - The alert should include: card name, position info, target that was hit, "Log Sale" action context
  - Verify: Trigger a sell target → `GET /api/alerts` includes the sell target alert alongside existing price alerts. Existing price alerts unaffected.

- [ ] **Task 4.4:** Write sell target tests
  - CRUD: create, list, delete
  - Evaluation: price target hit, gain % target hit, review date reached, target not yet hit
  - Alert integration: triggered target appears in alerts response
  - Verify: 10+ tests covering targets lifecycle and evaluation

### Acceptance Criteria (Phase 4)
- [ ] Sell target CRUD works (create, list, delete)
- [ ] Evaluation logic correctly triggers targets based on price, gain %, and date
- [ ] Triggered targets surface through the alerts system
- [ ] All new + existing tests pass

---

## Phase 5: Frontend — Portfolio Dashboard
> *Portfolio view, buy flow UI, portfolio summary*

**Gate:** human-approval (first frontend phase — confirm Plan 005 state before starting)
**Dependencies:** Phase 2 complete (API available), Plan 005 Phase 6 complete (alerts panel exists, frontend foundation stable)

- [ ] **Task 5.1:** Add TypeScript types for portfolio API
  - Add to `frontend/src/types/`: `Position`, `Sale`, `SellTarget`, `PortfolioSummary`, `TradeHistoryEntry`, `BuyRequest`, `SellRequest`, `SellTargetCreate`
  - Verify: `npm run build` passes with new types, no `any` types

- [ ] **Task 5.2:** Add portfolio API client functions and TanStack Query hooks
  - API client: `getPortfolio()`, `buyPosition()`, `sellPosition()`, `getHistory()`, `getSummary()`, `getTargets()`, `createTarget()`, `deleteTarget()`
  - Query hooks: `usePortfolio()`, `usePortfolioSummary()`, `useTradeHistory()`, `useSellTargets()`
  - Mutation hooks: `useBuyPosition()`, `useSellPosition()`, `useCreateTarget()`, `useDeleteTarget()` — with cache invalidation
  - Verify: `npm run build` passes. Hooks callable from DevTools with API running.

- [ ] **Task 5.3:** Build Portfolio page with summary bar
  - New route: `/portfolio`
  - Summary bar at top: total invested, current value, unrealized P&L ($, %), open position count
  - Color coding: green for gain, red for loss
  - Verify: Summary bar shows correct numbers matching `/api/portfolio/summary`. Renders with 0 positions gracefully.

- [ ] **Task 5.4:** Build positions table
  - TanStack Table with columns: Card+Set, Variant, Condition, Qty, Cost Basis, Market Price, Signal, Unrealized P&L ($), Unrealized P&L (%), Days Held
  - Sorting on all columns, default: unrealized P&L % descending
  - Filtering: by product_type (singles/sealed), by signal, by P&L direction
  - "Log Sale" button per row
  - Verify: Table renders positions, sorting/filtering works, sealed products show "—" for market price and signal.

- [ ] **Task 5.5:** Build Buy Position dialog/form
  - Entry points: "Add Position" button on Portfolio page, "Log Purchase" on card detail page (if exists)
  - Fields: product_type toggle (single/sealed), card search (for singles — uses existing search endpoint), card_name + set (for sealed — free text), variant, condition dropdown, qty, price paid, marketplace dropdown, date (default today), notes
  - On submit: calls `POST /api/portfolio/buy`, shows confirmation with cost vs market price
  - Verify: Submit buy → position appears in table. Validation errors shown for invalid input.

- [ ] **Task 5.6:** Build Sell Position dialog/form
  - Triggered from "Log Sale" button on a position row
  - Pre-populated: card name, remaining qty, cost basis
  - Fields: qty to sell (max = remaining), sale price, marketplace fee % (defaults by marketplace), sale date, marketplace, notes
  - Shows P&L preview before confirming
  - On submit: calls `POST /api/portfolio/sell`, shows realized gain/loss result
  - Verify: Partial sell → qty updates in table. Full sell → position disappears from portfolio (moves to history).

### Acceptance Criteria (Phase 5)
- [ ] Portfolio page renders with summary bar and positions table
- [ ] Buy and sell flows work end-to-end through UI
- [ ] Sealed products distinguishable from singles
- [ ] All existing frontend pages unaffected
- [ ] `npm run build` and `npm run test` pass

---

## Phase 6: Frontend — Trade History & Sell Targets
> *History view with realized P&L stats, sell target management UI*

**Dependencies:** Phase 5 complete

- [ ] **Task 6.1:** Build Trade History page
  - New route: `/portfolio/history`
  - Table: Card+Set, Buy Date, Sell Date, Days Held, Cost Basis, Sale Price (net), Realized P&L ($), Realized P&L (%)
  - Summary stats bar: total realized gains, total losses, net P&L, win rate, avg holding period, best trade, worst trade
  - Verify: History shows only closed positions. Stats match `/api/portfolio/summary`. Empty state renders cleanly.

- [ ] **Task 6.2:** Build Sell Targets panel
  - Display active sell targets with: card name, target type (price/gain%/review date), target value, current value, progress toward target
  - "Add Target" form on position detail or portfolio row action
  - Delete button per target
  - Verify: Create target → appears in list. Delete → removed. Triggered targets visually distinct.

- [ ] **Task 6.3:** Add portfolio navigation
  - Add Portfolio and Trade History to app sidebar/nav (alongside Dashboard, Watchlist, Alerts, Settings)
  - Verify: Navigation works, active route highlighted

### Acceptance Criteria (Phase 6)
- [ ] Trade history page shows closed positions with accurate stats
- [ ] Sell targets are manageable (create, view, delete) from the UI
- [ ] Portfolio routes integrated into app navigation
- [ ] `npm run build` and `npm run test` pass

---

## Phase 7: Sealed Products
> *Support sealed product positions with adapted data shape*

**Dependencies:** Phase 2 complete (backend), Phase 5 complete (frontend)

- [ ] **Task 7.1:** Verify sealed product backend flow
  - End-to-end test: POST buy with `product_type="sealed"`, card_id=null, variant=null, condition="Sealed", card_name="SV Base Set Booster Box"
  - GET portfolio → sealed position appears with null market price, no signal
  - Sell sealed position → history shows correct P&L
  - Verify: Existing singles tests unaffected. Sealed-specific edge cases covered.

- [ ] **Task 7.2:** Implement manual price override endpoint for sealed products
  - `PATCH /api/portfolio/{id}/price-override` — accepts `{current_value: float}`, stores on the position
  - Override persists and is used in P&L calculations for this position (replaces null market price)
  - Only allowed for `product_type="sealed"` positions — return 400 for singles (they have automated prices)
  - Add `manual_current_value` nullable Float column to positions model (Phase 1 Task 1.2)
  - Verify: Set override on sealed position → P&L calculates using the override. Clear override → P&L shows null.

- [ ] **Task 7.3:** Add sealed product UI adaptations
  - Buy form: when `product_type="sealed"` selected, hide card search, show free-text product name + set fields, default condition to "Sealed"
  - Portfolio table: sealed positions show "Sealed" badge, "—" for market price and signal columns
  - Manual price override: "Set Current Value" action on sealed positions (calls PATCH endpoint from Task 7.2)
  - Verify: Sealed buy → position with badge in table. Manual price entry updates P&L calculation.

- [ ] **Task 7.4:** Sealed products excluded from signal-based alerts
  - Ensure sell target alerts work for sealed (price targets, date targets) but signal-engine alerts don't fire for sealed positions
  - Verify: Sealed position with sell target gets alert. Sealed position without target gets no signal-based alert.

### Acceptance Criteria (Phase 7)
- [ ] Sealed products buyable, viewable, and sellable through the full flow
- [ ] Sealed positions visually distinct from singles
- [ ] Manual price override works for sealed P&L
- [ ] Sealed products excluded from signal alerts but included in sell-target alerts

---

## Phase 8: Integration Testing & Polish
> *End-to-end validation, edge cases, and UX polish*

**Dependencies:** All previous phases complete

- [ ] **Task 8.1:** End-to-end integration tests
  - Full lifecycle: buy → view portfolio → set sell target → target triggers → log sale → view history
  - Both singles and sealed products
  - Verify: No broken state transitions. All numbers consistent across views.

- [ ] **Task 8.2:** Edge case hardening
  - Buy 1 unit, sell 1 unit (minimum flow)
  - Buy 100 units, sell in 10 batches (many partial sells)
  - 0% marketplace fee vs 15% fee
  - Cost basis of $0.01 (penny card) — P&L percentages don't explode
  - Empty portfolio → all views render gracefully
  - Verify: No crashes, NaN, Infinity, or division-by-zero in P&L calculations

- [ ] **Task 8.3:** Update project documentation
  - Update `README.md` with portfolio feature overview
  - Update `pokemon-tcg-tracker.instructions.md` with new models, endpoints, and frontend routes
  - Update `pokemon-tcg-ops` skill if warranted
  - Verify: Documentation accurate, new endpoints listed, data model described

- [ ] **Task 8.4:** Frontend tests for portfolio components
  - Smoke tests: Portfolio page renders, History page renders, Buy form renders
  - Interaction tests: Buy flow, sell flow, target creation
  - Verify: `npm run test` passes with new tests

### Acceptance Criteria (Phase 8)
- [ ] Full lifecycle tested end-to-end for singles and sealed products
- [ ] Edge cases don't produce calculation errors
- [ ] Documentation updated
- [ ] Frontend test coverage for portfolio features
- [ ] Full test suite (backend + frontend) passes green

---

## Design Decisions

| Decision | Choice | Why | Alternative Rejected |
|----------|--------|-----|---------------------|
| Database | Extend existing SQLite DB (same `data/tracker.db`) | Simpler ops — one DB file, one connection, shared card_id lookups. Portfolio queries join against `cards` and `price_snapshots` for market prices. | Separate `portfolio.db` — rejected because cross-DB joins aren't simple in SQLite, and the data is tightly coupled (positions reference card_ids, P&L needs price_snapshots). |
| Partial sell approach | Decrement `qty_remaining` on position; each sale is a separate `sales` row linked by FK | Simple, auditable, supports multiple partial sells per position. Cost basis stays fixed per position (FIFO not needed — each buy is its own position). | FIFO lot matching — rejected as over-engineering for a personal portfolio. User controls cost basis per position explicitly. |
| Fee calculation | Store `marketplace_fee_pct` per sale; apply at realized P&L calculation time: `realized = (sale_price * (1 - fee/100) - cost_basis) * qty` | Transparent — user sees what fee was assumed. Editable if marketplace changes rates. | Pre-deducted net price — rejected because it hides the fee and can't be recalculated. |
| Model file organization | Add new models to existing `src/models.py` | Small project, related domain. One import location. Splitting adds indirection without benefit at this scale. | Separate `src/portfolio_models.py` — acceptable if `models.py` gets unwieldy, but 3 small models don't warrant it. |
| Sealed product_type | `product_type` enum column on `positions` table ("single" \| "sealed") | Single table with discriminator is simpler than separate tables. Queries filter by product_type. Nullable card_id/variant for sealed. | Separate `sealed_positions` table — rejected because buy/sell/P&L logic is identical, only the source data differs. |
| P&L precision | Use Python `round(value, 2)` for all currency calculations. Store as `Float` in SQLite. Tests assert with 2-decimal precision. | Good enough for personal portfolio. Avoids `Decimal` complexity. SQLite doesn't have a native decimal type anyway. | `Decimal` type — over-engineering for personal use. Integer cents — adds conversion overhead. |
| Alert integration | Separate `get_triggered_sell_targets(session)` function, aggregated at API layer alongside `get_triggered_alerts()` | Keeps JSON-based alerts and SQLite-based targets as separate data sources. Clean separation. | Extending `alerts.json` system with SQLite data — mixes data sources. |
| Alembic adoption | TBD — Task 1.1 decision | Adding 3 tables pushes beyond the original "2 simple tables" rationale for skipping Alembic. | — |

## Validation Strategy
Every AI-generated artifact must have verification steps defined BEFORE generation:

| Artifact Type | Validation Method |
|--------------|-------------------|
| SQLAlchemy models | `init_db()` creates tables, import succeeds, existing tests pass |
| Pydantic schemas | `.model_json_schema()` outputs valid JSON, validation rejects bad input |
| API endpoints | pytest tests with known input/output, P&L math manually verified |
| P&L calculations | Hand-calculated expected values in test assertions — never trust AI math |
| React components | `npm run build` passes, visual check, data matches API response |
| TypeScript types | `npm run build` with strict mode, no `any` types |
| Frontend tests | `npm run test` passes, covers happy paths and key interactions |
| Integration | Full lifecycle test: buy → portfolio → target → sell → history |

**Pattern enforced throughout:** Plan → Generate → Verify → Iterate. Never Plan → Generate → Done.
