# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Factory Inventory Management System - Full-stack app with Vue 3 frontend, Python FastAPI backend, and in-memory mock data (no database).

## Critical Tool Usage Rules

### Subagents
- **vue-expert**: **MANDATORY** for creating or significantly modifying any `.vue` file
- **code-reviewer**: Use after writing significant code
- **Explore**: Use for codebase structure questions or multi-file pattern searches
- **general-purpose**: Use for complex multi-step tasks

### Skills
- **backend-api-test**: Use when writing or modifying tests in `tests/backend/`

### MCP Tools
- **ALWAYS use GitHub MCP tools** (`mcp__github__*`) for ALL GitHub operations
  - Exception: Local branches — use `git checkout -b`
- **ALWAYS use Playwright MCP tools** (`mcp__playwright__*`) for browser testing
  - Frontend: `http://localhost:3000`, API: `http://localhost:8001`

## Stack
- **Frontend**: Vue 3 + Composition API + Vite (port 3000)
- **Backend**: Python FastAPI (port 8001)
- **Data**: JSON files in `server/data/` loaded at startup via `server/mock_data.py` — changes don't persist across restarts

## Commands

```bash
# One-command startup (both servers)
./scripts/start.sh

# Stop all servers
./scripts/stop.sh

# Backend (from server/)
cd server && uv run python main.py
# API docs: http://localhost:8001/docs

# Frontend (from client/)
cd client && npm install && npm run dev

# Backend tests (55 tests, run from server/)
cd server && uv run pytest ../tests/backend/ -v

# Run a single test file
cd server && uv run pytest ../tests/backend/test_inventory.py -v

# Run a single test
cd server && uv run pytest ../tests/backend/test_inventory.py::TestInventoryEndpoints::test_get_all_inventory -v

# Regenerate mock data
cd server && uv run python generate_data.py
```

## Architecture

### Filter System
4 global filters (Time Period, Warehouse, Category, Order Status) live in `client/src/composables/useFilters.js` as a **module-level singleton** — refs are declared outside the function so all components share the same state. `getCurrentFilters()` maps UI state to API query params (e.g., `selectedLocation` → `warehouse`, `selectedPeriod` → `month`).

Data flow: `FilterBar.vue` → `useFilters` refs → view components watch filters → `api.js` builds query params → FastAPI `apply_filters()` + `filter_by_month()` → filtered JSON → Pydantic validation → response.

### Frontend Composables
- `useFilters.js` — singleton filter state shared across all views
- `useAuth.js` — mock auth with hardcoded user; language-aware via `useI18n`
- `useI18n.js` — i18n with English/Japanese locale support; locale files in `client/src/locales/`

### Backend Filtering
- `apply_filters()` handles warehouse, category, status
- `filter_by_month()` handles direct month (`2025-01`) and quarter (`Q1-2025`) formats
- Inventory endpoints don't support `month` filter (no time dimension on inventory data)

## API Endpoints

| Endpoint | Filters |
|---|---|
| `GET /api/inventory` | warehouse, category |
| `GET /api/inventory/{id}` | — |
| `GET /api/orders` | warehouse, category, status, month |
| `GET /api/orders/{id}` | — |
| `GET /api/dashboard/summary` | warehouse, category, status, month |
| `GET /api/demand` | — |
| `GET /api/backlog` | — (includes `has_purchase_order` flag) |
| `POST /api/purchase-orders` | — |
| `GET /api/purchase-orders/{backlog_item_id}` | — |
| `GET /api/spending/summary` | — |
| `GET /api/spending/monthly` | — |
| `GET /api/spending/categories` | — |
| `GET /api/spending/transactions` | — |
| `GET /api/reports/quarterly` | — |
| `GET /api/reports/monthly-trends` | — |

## Key Patterns

**Reactivity**: Raw data in `ref()` (e.g., `allOrders`, `inventoryItems`), derived/filtered data in `computed()`. Never store derived data in refs.

**v-for keys**: Always use unique domain keys (`sku`, `id`, `month`) — never array index.

**Date safety**: Validate before calling `.getMonth()` — check `!isNaN(date.getTime())`.

**Pydantic sync**: When changing JSON data structure in `server/data/`, update the corresponding Pydantic model in `server/main.py`.

## File Locations
- Views: `client/src/views/*.vue`
- Components: `client/src/components/*.vue`
- Composables: `client/src/composables/`
- API Client: `client/src/api.js`
- Backend: `server/main.py`, `server/mock_data.py`
- Data: `server/data/*.json`
- Tests: `tests/backend/`
- Global styles: `client/src/App.vue`

## Design System
- Colors: Slate/gray (`#0f172a`, `#64748b`, `#e2e8f0`)
- Status colors: green/blue/yellow/red
- Charts: Custom SVG, CSS Grid for layouts
- No emojis in UI
- Revenue goals: $800K/month (single month), $9.6M YTD (all months)
