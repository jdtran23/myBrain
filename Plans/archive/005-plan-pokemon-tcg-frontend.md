# Plan 005: Pokemon TCG Tracker — React Frontend

## Objective
Replace the Lovable-hosted UI with a locally-hosted React + Vite + TypeScript frontend. The new UI will be a dashboard that consumes the existing FastAPI backend, displaying card data, signals, trends, alerts, and enabling watchlist management — all running on Joe's local machine.

## AI Usage Declaration
- **AI will generate:** React components, TypeScript types, Vite config, API client, CSS/Tailwind setup, build integration with FastAPI
- **AI will NOT decide:** Visual design preferences (colors, layout priority), which features to defer, production deployment strategy
- **Expected failure modes:** Hallucinated component APIs (shadcn, Recharts), incorrect Vite proxy config, TypeScript type mismatches with API responses, stale Tailwind class names

## Tech Stack Decision
| Layer | Choice | Rationale |
|-------|--------|-----------|
| Framework | React 19 + Vite | Best ecosystem for dashboards, strongest Copilot support |
| Language | TypeScript (strict) | Joe already knows TS, type safety catches API mismatches |
| Styling | Tailwind CSS 4 | Utility-first, pairs with shadcn/ui components |
| Components | shadcn/ui | Beautiful, ownable components (not a dependency — copied into project) |
| Tables | TanStack Table v8 | Sorting, filtering, pagination for card collection |
| Charts | Recharts | Price trend charts, integrates cleanly with React |
| Data Fetching | TanStack Query v5 | Caching, deduplication, loading/error states, cache invalidation after mutations |
| HTTP Client | fetch (native) | No extra dependency needed for simple REST calls |
| Routing | React Router v6 | Simpler API than v7 (Remix merge); adequate for 5 static routes |
| Testing | Vitest + React Testing Library | Fast Vite-native test runner, component smoke tests |
| Build | Vite 6 | Fast HMR, TypeScript support, proxy config for API |

## Phases & Tasks

---

### Phase 1: Project Scaffolding ✅ DONE
> *Set up the React + Vite project within the existing repo*

- [x] **Task 1.1:** Scaffold Vite + React + TypeScript project
  - Run `npm create vite@latest frontend -- --template react-ts` inside `pokemon-tcg-tracker/`
  - Verify: `cd frontend && npm install && npm run dev` starts on `http://localhost:5173`
  - AI Validation: Dev server loads without errors, shows default Vite React page

- [x] **Task 1.2:** Install and configure Tailwind CSS
  - Install Tailwind CSS 4, configure `tailwind.config.ts`, add base styles
  - Verify: A test `<div className="bg-blue-500 text-white p-4">` renders correctly
  - AI Validation: Tailwind classes apply visually in browser

- [x] **Task 1.3:** Install and initialize shadcn/ui
  - Run `npx shadcn@latest init`, configure for Vite + Tailwind
  - Add first component: `npx shadcn@latest add button card badge`
  - Verify: `<Button>` renders styled correctly
  - AI Validation: Component renders, no console errors

- [x] **Task 1.4:** Configure Vite proxy for API
  - Add proxy in `vite.config.ts`: `/api` → `http://localhost:8000`
  - Verify: `fetch('/api/cards')` from the frontend returns card data when both servers run
  - AI Validation: No CORS errors, API response is valid JSON

- [x] **Task 1.5:** Create TypeScript types from API responses
  - Define types in `frontend/src/types/` matching all API response shapes: `Card`, `PriceSnapshot`, `Signal`, `Trend`, `Alert`, `WatchlistItem`, `SignalRule`
  - Verify: Types compile, no `any` types in the API layer
  - AI Validation: Compare type definitions against actual API responses from `/docs`

- [x] **Task 1.6:** Create API client module
  - `frontend/src/api/client.ts` — typed functions for every backend endpoint
  - Functions (with HTTP methods): `GET getCards()`, `GET getCard(id)`, `GET getCardTrends(id)`, `GET getPrices(id, opts?)`, `GET searchCards(q)`, `GET getWatchlist()`, `POST addToWatchlist(body)`, `DELETE removeFromWatchlist(params)`, `GET getAlerts()`, `GET getAlertsConfig()`, `POST addAlert(body)`, `DELETE removeAlert(id)`, `GET getSignalRules()`, `PATCH updateSignalRule(body)`, `GET getSignalOverrides()`, `POST addSignalOverride(body)`, `DELETE removeSignalOverride(params)`, `POST refreshPrices()`, `POST recomputeSignals()`
  - Verify: All functions return typed responses, compile cleanly
  - AI Validation: Call each function from browser console with API running, confirm data matches Swagger

- [x] **Task 1.7:** Install and configure TanStack Query
  - Install `@tanstack/react-query` and `@tanstack/react-query-devtools`
  - Create `QueryClientProvider` wrapper in `App.tsx`
  - Create typed query hooks in `frontend/src/hooks/` wrapping API client functions (e.g., `useCards()`, `useCard(id)`, `usePrices(id)`, `useAlerts()`)
  - Create typed mutation hooks for write operations (e.g., `useAddToWatchlist()`, `useRemoveAlert()`) with cache invalidation
  - Verify: `npm run build` passes, React Query DevTools visible in browser
  - AI Validation: `useCards()` returns loading → data states correctly, refetch after mutation works

- [x] **Task 1.8:** Install Vitest and React Testing Library
  - Install `vitest`, `@testing-library/react`, `@testing-library/jest-dom`, `jsdom`
  - Configure `vitest.config.ts` (or extend `vite.config.ts`)
  - Write one smoke test: `App.test.tsx` — renders without crashing
  - Verify: `npm run test` passes
  - AI Validation: Test runner executes, finds and passes the smoke test

**Phase 1 Exit Criteria:** ✅ Vite dev server runs, proxies to FastAPI, all API endpoints have typed client functions with TanStack Query hooks, shadcn/ui components render, Vitest runs. `npm run build` passes with zero errors. (2026-03-30)

---

### Phase 2: Core Dashboard Layout ✅ DONE
> *Build the app shell and navigation*

- [x] **Task 2.1:** Create app layout with sidebar/header navigation
  - Routes: Dashboard (home), Card Detail, Watchlist, Alerts, Settings
  - Use React Router v6 for client-side routing (simpler API, adequate for 5 static routes)
  - Verify: Navigation between routes works, URL changes reflect in browser
  - AI Validation: Each route renders a placeholder, no console errors

- [x] **Task 2.2:** Build Dashboard page (main view)
  - Display summary stats: total cards tracked, buy/sell/hold counts, latest refresh time
  - Use shadcn/ui `Card` components for stat blocks
  - Verify: Dashboard renders real data from `/api/cards`
  - AI Validation: Numbers match what Swagger returns for the same endpoint

- [x] **Task 2.3:** Build responsive layout
  - Sidebar collapses on small screens, content area fills available space
  - Verify: Resize browser — layout adapts cleanly at common breakpoints
  - AI Validation: No overflow, no hidden content at 768px and 1024px widths

**Phase 2 Exit Criteria:** ✅ App has working navigation, dashboard shows live data, layout is responsive. (2026-03-30)

---

### Phase 3: Card Collection View ✅ DONE
> *The primary data table showing all tracked cards*

- [x] **Task 3.1:** Build cards table with TanStack Table
  - Columns: Image (thumbnail), Name, Set, Rarity, Market Price, 7d Change %, Signal, Actions
  - Features: column sorting, text filtering/search, pagination (20 per page)
  - Verify: Table renders all cards from `/api/cards`, sorting works on each column
  - AI Validation: Sort by price → highest first matches manual check; filter by name → correct subset

- [x] **Task 3.2:** Signal badges
  - Color-coded badges: Buy (green), Sell (red), Hold (yellow/neutral)
  - Show `signal_reason` on hover/tooltip
  - Show `contributing_factors` as sub-badges or in expanded row
  - Verify: Badge colors match signal type, tooltip shows reason text
  - AI Validation: Cross-reference with `/api/cards` response — badge matches `signal_type` for each card

- [x] **Task 3.3:** Card image thumbnails
  - Display card art from `image_url` in table rows (small thumbnails ~60px)
  - Lazy load images, show placeholder on error
  - Verify: Images load for cards with valid URLs, placeholders show for missing images
  - AI Validation: Right-click image → check URL matches `image_url` from API

- [x] **Task 3.4:** Trend indicators in table
  - Show 7d price change as colored arrow + percentage (green up, red down, gray flat)
  - Verify: Arrows and colors match the `trends.price_change_7d_pct` values
  - AI Validation: Compare a few cards' displayed trends vs. `/api/cards/{id}/trends` response

**Phase 3 Exit Criteria:** ✅ Full card table with sorting, filtering, pagination, signal badges, images, and trend arrows. All data sourced from live API. (2026-03-30)

---

### Phase 4: Card Detail Page ✅ DONE
> *Detailed view for a single card with price history chart*

- [x] **Task 4.1:** Card detail layout
  - Large card image, card metadata (name, set, rarity, supertype, number)
  - Current price block (market, low, mid, high), signal badge with reason + contributing factors
  - Verify: Page renders all fields for a given card
  - AI Validation: Compare displayed data with `/api/cards/{id}` response

- [x] **Task 4.2:** Price history chart (Recharts)
  - Line chart from `/api/prices/{id}` — X axis: date, Y axis: price
  - Lines for market, low, mid, high prices
  - Toggleable timeframes: 7d, 30d, 90d, All
  - Verify: Chart renders, data points match API response
  - AI Validation: Pick a known price point, verify it appears at the correct position on the chart

- [x] **Task 4.3:** Trend summary section
  - Display 7d and 30d price change percentages, trend label (rising/declining/stable)
  - Visual treatment: green for rising, red for declining, neutral for stable
  - Verify: Values match `/api/cards/{id}/trends` response
  - AI Validation: Compute a expected 7d change manually from price history, compare

**Phase 4 Exit Criteria:** ✅ Card detail page with full metadata, signal info, price chart with timeframes, and trend summary. (2026-03-30)

---

### Phase 5: Watchlist & Card Search
> *Add/remove cards from the watchlist*

- [ ] **Task 5.1:** Watchlist management page
  - Fetch watchlist IDs from `/api/watchlist`, then filter `/api/cards` response to matching IDs (client-side join — no new backend endpoint needed)
  - Remove button per card (calls `DELETE /api/watchlist`, then invalidates both `watchlist` and `cards` TanStack Query caches)
  - Show card count vs. 200 limit
  - Verify: Adding/removing cards updates the list immediately
  - AI Validation: After remove, refresh page — card is gone. Check `config/watchlist.json` matches UI.

- [ ] **Task 5.2:** Card search and add flow
  - Search input that calls `/api/search?q=...` with debounce (300ms)
  - Display results as cards with image, name, set — "Add to Watchlist" button
  - After adding, trigger a price fetch for the new card (`POST /api/refresh`)
  - Verify: Search returns results, adding puts card in watchlist, new card appears in collection
  - AI Validation: Search for "Charizard" → results appear → add one → verify in `/api/watchlist` and `/api/cards`

**Phase 5 Exit Criteria:** Users can search for cards, add them to watchlist, and remove them. Watchlist state persists across page reloads.

---

### Phase 6: Alerts Panel
> *View and manage user alert rules*

- [ ] **Task 6.0 (Backend):** Add alerts config listing endpoint
  - Problem: `GET /api/alerts` only returns **triggered** alerts (calls `get_triggered_alerts()`). The UI needs to list **all configured** alerts with a triggered status per alert.
  - Add `GET /api/alerts/config` endpoint that reads `config/alerts.json` and returns all alerts with a `triggered: boolean` field computed by cross-referencing with triggered alerts.
  - Verify: Endpoint returns all alerts from `alerts.json`, each with `triggered` field. Existing `/api/alerts` endpoint unchanged.
  - AI Validation: Create 3 alerts, trigger 1 → `/api/alerts/config` returns all 3 with correct triggered status. `/api/alerts` still returns only the 1 triggered.
  - Dependencies: This is a BACKEND task — update `src/api.py` in the Python project

- [ ] **Task 6.1:** Alerts display
  - Show all configured alerts from `GET /api/alerts/config` (new endpoint from Task 6.0)
  - Triggered alerts highlighted (badge/icon)
  - Display: card name, condition, threshold value, triggered status
  - Verify: Alerts render, triggered alerts are visually distinct
  - AI Validation: Compare displayed alerts with `/api/alerts/config` response

- [ ] **Task 6.2:** Create/delete alerts
  - Form to add alerts: select card (from tracked cards), condition (price_below, price_above, change_7d_above_pct), value
  - Delete button per alert
  - Verify: New alert appears after creation, deleting removes it
  - AI Validation: Create alert via UI → check `config/alerts.json` → delete → confirm gone

**Phase 6 Exit Criteria:** Full alert CRUD via the UI, triggered alerts visually highlighted.

---

### Phase 7: Settings & Signal Tuning
> *View and adjust signal rules*

- [ ] **Task 7.1:** Signal rules viewer
  - Display all 9 signal rules from `/api/signal-rules`
  - Show: rule type, signal direction, priority, current thresholds
  - Verify: All rules rendered, values match config
  - AI Validation: Compare with `config/signal_rules.json`

- [ ] **Task 7.2:** Threshold editor
  - Inline editing of threshold values per rule
  - Save calls `PATCH /api/signal-rules`
  - After save, offer "Recompute Signals" button (`POST /api/signals/recompute`)
  - Verify: Edit threshold → save → recompute → card signals update
  - AI Validation: Change a threshold, recompute, check a specific card's signal changed as expected

- [ ] **Task 7.3:** Signal overrides viewer/editor
  - Display per-card overrides from `/api/signal-overrides`
  - Add/remove per-card overrides
  - Verify: Override changes reflected in card signals after recompute
  - AI Validation: Add override → recompute → verify affected card signal

**Phase 7 Exit Criteria:** Signal rules are viewable and editable, overrides manageable, recompute triggers from UI.

---

### Phase 8: Polish & Production Build
> *Final polish and integration into a single-process local deployment*

- [ ] **Task 8.1:** Add loading states and error handling
  - Skeletons/spinners for data loading states
  - Error boundaries for component crashes
  - Toast notifications for actions (card added, alert created, refresh complete)
  - Verify: Loading states appear on slow network (throttle in DevTools), errors show user-friendly messages
  - AI Validation: Kill API, load frontend — error states render gracefully, no white screens

- [ ] **Task 8.2:** FastAPI static file serving
  - After `npm run build`, configure FastAPI to serve `frontend/dist/` as static files
  - Mount at root: `app.mount("/", StaticFiles(directory="frontend/dist", html=True))`
  - **IMPORTANT:** API routes (`/api/*`) and other routes must be registered BEFORE the static file catch-all mount, or they'll be shadowed. This is a known FastAPI gotcha — `app.mount()` at `"/"` must be the LAST mount.
  - SPA fallback: `html=True` handles client-side routing by serving `index.html` for unknown paths
  - Verify: `python scripts/run_api.py` serves both API and frontend on port 8000
  - AI Validation: Build frontend → start only the Python server → open `http://localhost:8000` → UI loads and fetches data. Also test: navigate to `/watchlist` directly → SPA fallback serves index.html → React Router handles the route. Also test: `GET /api/cards` still returns JSON, not HTML.

- [ ] **Task 8.3:** Update project documentation
  - Update `README.md` with frontend setup instructions
  - Update `pokemon-tcg-tracker.instructions.md` with new directory layout and commands
  - Add `frontend/` to `.gitignore` entries (node_modules, dist if desired)
  - Verify: Fresh clone → follow README → both backend and frontend work
  - AI Validation: README instructions are complete and accurate

- [ ] **Task 8.4:** Remove Lovable prompt files
  - Delete all `LOVABLE-*.md` files from repo root (7 files)
  - Verify: Files removed, git status shows deletions, no remaining Lovable references
  - AI Validation: `grep -r "lovable" .` returns zero matches (excluding git history)

**Phase 8 Exit Criteria:** Single `python scripts/run_api.py` serves the complete app. Loading/error states work. Docs updated. Lovable files removed.

---

## Validation Strategy

Every AI-generated artifact must have verification steps defined BEFORE generation:

| Artifact Type | Validation Method |
|--------------|-------------------|
| React components | `npm run dev` — renders without console errors, visual check |
| TypeScript types | `npm run build` — compiles with zero errors |
| API client functions | Browser console test against running API |
| Vite config (proxy) | `fetch('/api/cards')` from frontend works |
| Tailwind/CSS | Visual inspection at multiple viewport sizes |
| Production build | Single-process `run_api.py` serves both API and UI |
| Data accuracy | Cross-reference displayed data with Swagger/API responses |
| Phase exit gate | `npm run build` must pass with zero errors at each phase exit |
| Frontend tests | `npm run test` must pass at each phase exit |

**Pattern enforced throughout:** Plan → Generate → Verify → Iterate. Never Plan → Generate → Done.

**Rollback strategy:** Each phase is committed as a working checkpoint before starting the next phase. If a phase breaks the app, `git revert` to the previous phase's commit.

## Acceptance Criteria
- [ ] Phase 1: Vite project scaffolded with TS, Tailwind, shadcn/ui, API client
- [ ] Phase 2: App shell with navigation and dashboard summary
- [ ] Phase 3: Card collection table with signals, images, trends, sorting/filtering
- [ ] Phase 4: Card detail page with price history chart
- [ ] Phase 5: Watchlist management with search-and-add
- [ ] Phase 6: Alerts CRUD panel
- [ ] Phase 7: Signal rules and overrides editor
- [ ] Phase 8: Production build served from FastAPI, docs updated, Lovable files removed

## Risk Register
| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| shadcn/ui init fails with Vite (config mismatch) | Medium | Medium | Follow official Vite setup guide; fall back to manual component copy |
| Recharts/TanStack API changes (v9 breaking changes) | Low | Medium | Pin exact versions in package.json |
| API response shapes change during Phase 5 of Plan 004 | Medium | High | Types are defined from current API; update if backend changes |
| Card images from TCGdex fail to load (CORS/404) | Medium | Low | Placeholder fallback already in design; proxy through backend if needed |
| Tailwind 4 + shadcn/ui compatibility | Medium | Medium | Decide upfront in Task 1.2: check shadcn/ui Tailwind 4 support before scaffolding. If uncertain, use Tailwind 3 (upgrading later is trivial; downgrading mid-project is painful) |
| Vite proxy doesn't forward WebSocket (if needed later) | Low | Low | Not needed now; configure if added later |

## Timeline Dependencies
```
Phase 1 (scaffolding) → Phase 2 (layout) → Phase 3 (cards table)
                                          → Phase 4 (card detail)  [parallel with 3]
                         Phase 2 complete → Phase 5 (watchlist)
                                          → Phase 6 (alerts)       [parallel with 5]
                                          → Phase 7 (settings)     [parallel with 5, 6]
                         All phases done  → Phase 8 (polish + build)
```

Phases 3-7 depend on Phase 2 (layout) but are largely independent of each other after initial routing is in place.

## Relationship to Plan 004
This plan (005) can execute **in parallel** with Plan 004 Phases 5-7, with a coordination rule: **If Plan 004 Phase 5 changes any API response shape, update `frontend/src/types/` BEFORE continuing frontend development against that endpoint. Run `npm run build` to catch breakage.** Ideally, complete Plan 004 Phase 5 before starting Plan 005 Phase 3+.
