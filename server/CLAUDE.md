# CLAUDE.md - Server

This file provides guidance to Claude Code (claude.ai/code) when working with the FastAPI backend.

## Running the Server

```bash
uv run python main.py       # http://localhost:8001
                             # API docs: http://localhost:8001/docs
uv run python generate_data.py  # Regenerate mock data
```

## Architecture

Single-file API (`main.py`) with all endpoints, Pydantic models, and filtering logic. Data loaded from JSON files in `data/` at startup via `mock_data.py` — all in-memory, changes don't persist across restarts.

## Filtering

Two filter functions handle all filtering:
- `apply_filters(data, warehouse, category, status)` — checks each param for `None` or `'all'`, filters sequentially using case-insensitive comparison
- `filter_by_month(data, month)` — handles both direct month format (`2025-01`) and quarter format (`Q1-2025`)

Filter params are optional query strings. Inventory endpoints don't support `month` (no time dimension on inventory data).

## Pydantic Sync

When changing JSON data structure in `data/`, always update the corresponding Pydantic model in `main.py`. Models: `InventoryItem`, `Order`, `DemandForecast`, `BacklogItem`, `PurchaseOrder`, `CreatePurchaseOrderRequest`.

## Adding Endpoints

1. Define/update Pydantic model
2. Add route with `@app.get`/`@app.post` and `response_model`
3. Use `apply_filters()` / `filter_by_month()` if filterable
4. Return 404 via `HTTPException` for missing resources
5. Write tests in `tests/backend/`

## Data Files

- `inventory.json` — 28 items across 3 warehouses (SF, London, Tokyo) and 7 categories
- `orders.json` — 500+ orders spanning 12 months (2025-01 through 2025-12)
- `demand_forecasts.json`, `backlog_items.json`, `purchase_orders.json`
- `spending.json` — summary + monthly + category breakdowns
- `transactions.json` — recent transaction history
