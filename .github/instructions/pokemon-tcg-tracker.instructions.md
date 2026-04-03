---
description: "Pokemon TCG Tracker — repo-specific project context, commands, architecture"
applyTo: "**/pokemon-tcg-tracker/**"
---
# Pokemon TCG Tracker — Project Instructions

## Overview
Local-first Pokemon TCG market tracker. Fetches card prices, stores history in SQLite, computes buy/sell/hold signals via a 9-rule engine, and exposes a FastAPI REST API.

## Tech Stack

### Backend
- **Language:** Python 3.9.18
- **API Framework:** FastAPI + Uvicorn
- **ORM/DB:** SQLAlchemy + SQLite (`data/tcg_tracker.db`)
- **Price Sources:** TCGdex (free, no key) → pokemontcg.io (free key, 20k/day)
- **Config:** JSON files in `config/`
- **Deployment:** Railway (via Procfile)

### Frontend
- **Framework:** React 19 + Vite 6 + TypeScript (strict)
- **Styling:** Tailwind CSS 4 + shadcn/ui v4 (uses `@base-ui/react`, NOT Radix)
- **Data Layer:** TanStack Query v5 (caching/mutations) + TanStack Table v8 (sortable/filterable tables)
- **Routing:** React Router v6 (NOT v7 — avoids Remix complexity)
- **Charts:** Recharts
- **Testing:** Vitest + React Testing Library + jsdom
- **Key Gotcha:** shadcn/ui v4 components use `@base-ui/react` — no `asChild` prop on triggers

## Directory Layout
```
config/          — JSON config (watchlist, alerts, signal_rules, signal_overrides) + settings.py
data/            — SQLite database (gitignored)
scripts/         — CLI entry points (run_api, run_fetch, seed_data, add/remove card)
src/             — Core modules (api, fetcher, signals, trends, alerts, models, db)
outputs/         — PRDs and documentation
frontend/        — React + Vite frontend app
  src/
    api/         — Typed fetch functions (client.ts)
    components/  — UI components (layout/, ui/)
    hooks/       — TanStack Query hooks (use-api.ts)
    pages/       — Route page components
    types/       — TypeScript interfaces (api.ts)
    lib/         — Utilities (utils.ts)
    test/        — Test setup
```

## Run Commands
```powershell
# === Backend ===
# Setup
python -m venv .venv; .venv\Scripts\activate
pip install -r requirements.txt

# Seed sample data (7 cards, 30 days fake history)
python scripts/seed_data.py

# Start API server (http://localhost:8000, Swagger at /docs)
python scripts/run_api.py

# Fetch live prices for watchlist cards
python scripts/run_fetch.py          # normal
python scripts/run_fetch.py --debug  # verbose

# Recompute signals without re-fetching prices
python scripts/recompute_signals.py

# Watchlist management (CLI)
python scripts/add_card.py --name "Charizard"
python scripts/add_card.py --id "swsh4-25"
python scripts/remove_card.py --name "Charizard"
python scripts/remove_card.py --id "swsh4-25"

# === Frontend ===
# Setup (from project root)
cd frontend
npm install

# Dev server (http://localhost:5173, proxies /api → :8000)
npm run dev

# Production build (outputs to frontend/dist/)
npm run build

# Run tests
npm run test
```

## Development Workflow
Run two terminals side by side:
1. **Backend:** `python scripts/run_api.py` (port 8000)
2. **Frontend:** `cd frontend && npm run dev` (port 5173, auto-proxies API calls)

## Config Files
| File | Purpose |
|------|---------|
| `config/watchlist.json` | Cards to track (IDs + names, max 200) |
| `config/signal_rules.json` | 9 signal rules with thresholds |
| `config/signal_overrides.example.json` | Per-card rule overrides (copy to `signal_overrides.json`) |
| `config/alerts.json` | User price/trend alerts |
| `config/settings.py` | Paths, API keys, constants |
| `.env` | `POKEMON_TCG_API_KEY=your_key` (optional) |

## API Endpoints (key routes)
| Method | Path | Purpose |
|--------|------|---------|
| GET | `/api/cards` | All tracked cards with latest prices + signals |
| GET | `/api/cards/{id}` | Single card detail |
| GET | `/api/cards/{id}/trends` | Trend data (7d/30d change, label) |
| GET | `/api/prices/{card_id}` | Price history (query params: variant, source, days) |
| GET | `/api/search?q=X` | Search pokemontcg.io by name |
| GET/POST/DELETE | `/api/watchlist` | Manage watchlist |
| GET/PATCH | `/api/signal-rules` | View/update signal thresholds |
| GET/POST/DELETE | `/api/signal-overrides` | Per-card signal overrides |
| GET/POST/DELETE | `/api/alerts` | Manage user alerts |
| GET | `/api/alerts/config` | All configured alerts with triggered status |
| POST | `/api/refresh` | Trigger price fetch + signal recompute |

## Signal Engine
9 rules, priority-ordered. When buy + sell conflict, highest priority wins. Contributing factors captured (max 3).

| Priority | Rule | Signal |
|----------|------|--------|
| 1 | `strong_buy` | buy |
| 2 | `below_direct_low` | buy |
| 3 | `buy_dip` | buy |
| 4 | `dip_vs_avg7` | buy |
| 5 | `rising` | buy |
| 6 | `sell_opportunity` | sell |
| 7 | `declining` | sell |
| 8 | `weak_sell` | sell |
| 9 | `hold_accumulate` | hold |

## Database
Two tables: `cards` (card metadata + latest signal) and `price_snapshots` (daily price history per card/variant/source).

## Known Issues
- CORS is `allow_origins=["*"]`
- `compute_trends()` still queries per-card (3 queries per card) — potential future optimization
