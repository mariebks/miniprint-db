# Print Keep — runtime fetch from miniprint-db

**Chosen sync model (2026-09-12):** Print Keep loads data at **runtime** from the public GitHub repo. Do **not** rebuild/redeploy the site on every scrape push.

Repo: https://github.com/mariebks/miniprint-db (public, `main`)

## Endpoints (raw)

Base: `https://raw.githubusercontent.com/mariebks/miniprint-db/main/data/`

| File | URL |
|------|-----|
| Machines (current state) | https://raw.githubusercontent.com/mariebks/miniprint-db/main/data/machines.json |
| Print catalog | https://raw.githubusercontent.com/mariebks/miniprint-db/main/data/prints.json |
| Machine event log | https://raw.githubusercontent.com/mariebks/miniprint-db/main/data/updates.ndjson |
| Print Guide rollup | https://raw.githubusercontent.com/mariebks/miniprint-db/main/data/print_guide_index.json |
| Print Guide signals | https://raw.githubusercontent.com/mariebks/miniprint-db/main/data/print_signals.ndjson |
| Products rollup | https://raw.githubusercontent.com/mariebks/miniprint-db/main/data/products.json |
| Product signals | https://raw.githubusercontent.com/mariebks/miniprint-db/main/data/product_signals.ndjson |

Schemas: `docs/SCHEMA.md`, `docs/PRINT_GUIDE_SCHEMA.md`, `docs/PRODUCTS_SCHEMA.md`

## Implementation notes for Print Keep

1. `fetch` these URLs on page load (or via a short-lived server cache of 1–5 minutes).
2. Prefer `machines.json` for maps/lists; use `updates.ndjson` for activity/timeline.
3. GitHub raw CDN caches ~**5 minutes** (`Cache-Control: max-age=300`). Expect up to ~5 min lag after a Bot push.
4. Optional: add `?t=` or `cache: 'no-store'` only if you need fresher than CDN (don’t hammer).
5. CORS: raw.githubusercontent.com allows browser GETs for public files.
6. Rebuild/redeploy Print Keep only when **site code** changes — not when Bot updates JSON.

## What Bot does

Grok Bot scrapes → writes JSON → pushes `miniprint-db`. No deploy hook required for this model.
