# Schema

## `data/machines.json`

Array of machine objects. Website loads this for maps/lists.

Required fields:
- `id` (string) — stable machine_id
- `name`, `city`, `state`, `address`
- `stock_status` — e.g. `in_stock` | `low_stock` | `out_of_stock` | `unknown`
- `prints_available` — array of `{ id, name }` when known (name may be null if unmapped)
- `last_updated` — ISO-8601
- `last_source` — `miniprintdatabase.com seed` | `messenger` | `facebook_group` | …

Optional: `latitude`, `longitude`, `parking`, `nearby`, `parking_cost`, `payment_type`, `pull_limit`, `website`, `instagram`, `waiting_for_restock`, `available_print_count`, `source_url`

Conflict rule: newer `last_updated` / newer `updates.ndjson` entry wins.

## `data/prints.json`

Object keyed by print_id:
```json
{ "1787…": { "id": "1787…", "name": "Foxglove", ... } }
```

## `data/updates.ndjson`

One JSON object per line, append-only, newest typically at end:
```json
{"ts":"2026-09-06T18:20:00-07:00","machine_id":"…","source":"messenger","fields":{"stock_status":"out_of_stock"},"note":"…","thread":"…"}
```

## Print Guide (rarity / trades)

See `PRINT_GUIDE_SCHEMA.md`. Additional handoff files:
- `data/print_signals.ndjson`
- `data/print_guide_index.json`
