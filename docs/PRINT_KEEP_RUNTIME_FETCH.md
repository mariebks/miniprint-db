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

## Merge model (Print Keep local edits win)

`miniprint-db` is the **bot feed** (community/host scrapes). Print Keep may keep its own **local overrides** (manual corrections, pins, site-only labels).

On each fetch:
1. Load the latest repo JSON as the **base** layer.
2. **Apply Print Keep local overrides on top** (by stable `machine_id` / print `id`).
3. Result shown in UI = `merge(base, local_overrides)` where **local wins** on any field the user/site has explicitly set.

Rules:
- **New machines / new prints** that appear only in the repo → add them (nothing local to protect).
- **New bot fields** on an existing id where Print Keep has **no** override → fill from repo.
- **Fields the user changed in Print Keep** → **never** overwritten by a later repo fetch.
- Repo is **not** rewritten by Print Keep (one-way: bot → site). Site edits stay in Print Keep’s own store.
- Optional: mark overridden fields as `pinned` / `source: local` so the UI can show “manually set.”

Do **not** wholesale replace Print Keep’s database with the raw fetch if that would wipe user edits.

