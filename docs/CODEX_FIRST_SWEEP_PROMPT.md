# Prompt for Codex — first manual read of miniprint-db

Copy everything below the line into Codex.

---

You are helping build features on our Ana Inciardi miniprint collector site:
1) a **print archive / machine DB viewer**, and
2) better context for the existing **Ask Print Guide** chatbot (rarity, fair trades, catalog vs custom).

## Data source (canonical)

Private GitHub repo (manual read for this first sweep — no webhook required yet):

**https://github.com/mariebks/miniprint-db**

Producer is a separate Grok Bot that scrapes one Facebook group + a Messenger folder (read-only) and seeds from the retiring miniprintdatabase.com site. **You consume JSON; do not scrape Facebook yourself.**

Read the docs in-repo first:
- `docs/SCHEMA.md`
- `docs/PRINT_GUIDE_SCHEMA.md`
- `docs/PRODUCTS_SCHEMA.md`
- `README.md`

## What to load (priority order)

### For the archive / machine viewer
1. **`data/machines.json`** — array of ~226 machines (current state). Primary UI source for maps/lists.
   - Stable id: `id` (Bubble/miniprintdatabase machine id)
   - Location: `name`/`venue`, `city`, `state`, `address`, `latitude`, `longitude`
   - Stock: `stock_status` (`in_stock` | `low_stock` | `out_of_stock` | `unknown`), `waiting_for_restock`
   - Inventory: `prints_available` = `[{ id, name, printType }]` — `name` may be null if not yet mapped
   - Extras: `parking`, `nearby`, `payment_type`, `pull_limit`, `website`, `instagram`, `last_updated`, `last_source`
2. **`data/prints.json`** — object keyed by print id → `{ id, name, printType, collection, customMachine, image, releaseDate, ... }` (~480 prints so far; catalog still growing). Use to resolve names/types/images.
3. **`data/updates.ndjson`** — append-only machine change events (optional history UI). Newer lines win on conflicts with `machines.json`.

Ignore `machines/*.md` for the product UI unless you want an admin audit view — they are human logs.

### For Ask Print Guide
4. **`data/print_guide_index.json`** — chatbot-ready rollup (`prints` + `global_tips`). Prefer this for answers when populated.
5. **`data/print_signals.ndjson`** — raw community claims (rarity, fair_trade, catalog_vs_custom, OOC, release notes) with quotes + citations. Use for provenance / when index is thin.
6. Join Guide answers to **`data/prints.json`** for official name / `printType` (`catalog` vs `custom`) when ids match.

**Important:** availability (which machine is stocked) ≠ rarity. Stock lives in `machines.json`; rarity/trade lives in print guide files. Do **not** invent rarity — only use indexed community signals (+ any existing Print Keep labels you already have on-site).

### Optional later (affiliate)
7. **`data/products.json`** + **`data/product_signals.ndjson`** — community-recommended Amazon/shop links (no affiliate tags yet). Not required for the first archive viewer / Guide pass unless you want a “collector essentials” section.

## First-sweep goals

1. **Ingest** `machines.json` + `prints.json` into whatever DB/cache the site uses (or read JSON at runtime for a prototype).
2. Ship or stub a **viewer**: browse/filter machines by state/city/stock; show machine detail with named prints when `name` is present; fall back to print id → lookup in `prints.json`.
3. Wire **Ask Print Guide** to pull context from `print_guide_index.json` (and `print_signals.ndjson` if needed) when answering rarity / fair-trade / catalog-vs-custom questions — keep confidence + cite community source when shown.
4. Note gaps honestly in UI if needed: some `prints_available[].name` are still null; print guide / product files may still be filling from a 30-day FB backfill.

## Conflict / freshness rules

- Machine current state: `machines.json` is the snapshot; if both exist, prefer newer `updates.ndjson` fields for that `machine_id`.
- Print Guide: keep raw signals; aggregates in `print_guide_index.json` are derived — don’t delete signals.
- IDs are stable strings like `1787165942218x300799986880750300` — treat as opaque primary keys.

## Out of scope for this first prompt

- Do not set up GitHub webhooks/Actions yet (manual read is enough).
- Do not add Amazon affiliate tags yet.
- Do not scrape Facebook / Messenger.
- Do not rewrite the producer’s file formats without updating `docs/SCHEMA.md`.

When done, summarize what you ingested (counts), what UI/Guide hooks you added, and any schema mismatches you hit.
