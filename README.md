# Miniprint machine database (handoff)

Canonical machine + inventory data for the Ana Inciardi collector site.

**Producer:** Grok Bot (Facebook group + Messenger scrape, seeded from miniprintdatabase.com)  
**Consumer:** Fiancée’s Codex / website build

## Layout

| Path | Purpose |
|------|---------|
| `data/machines.json` | Current state of every machine (site should load this) |
| `data/prints.json` | Print catalog: `print_id` → name and metadata |
| `data/updates.ndjson` | Append-only **machine** event log |
| `data/print_signals.ndjson` | Append-only **rarity/trade** community signals |
| `data/print_guide_index.json` | Rollup for Ask Print Guide chatbot |
| `data/product_signals.ndjson` | Append-only shop-link recommendations |
| `data/products.json` | Rollup of recommended products (Amazon ASINs etc.) |
| `machines/*.md` | Optional human-readable audit trail |
| `docs/SCHEMA.md` | Field contract for Codex / the website |

## Sync (Print Keep)

**Runtime fetch** (chosen): Print Keep loads JSON from public raw GitHub URLs — no site rebuild on each scrape.  
See `docs/PRINT_KEEP_RUNTIME_FETCH.md` for endpoints.

Grok Bot commits + pushes after each successful scrape when something changed.  
GitHub raw CDN may lag ~5 minutes.

(Optional alternative: Vercel/Netlify deploy hook on `data/**` push — only needed if the site bakes JSON at build time.)

## IDs

Stable machine and print IDs are the miniprintdatabase.com Bubble IDs.

Also see `docs/PRINT_GUIDE_SCHEMA.md` for rarity/fair-trade indexing.

Also see `docs/PRODUCTS_SCHEMA.md` for affiliate-candidate product links.
