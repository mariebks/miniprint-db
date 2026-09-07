# Print Guide context index

Data the **Ask Print Guide** chatbot can load as approved community context (rarity, fair trades, catalog vs custom, release notes). Separate from machine stock files.

## Files

| Path | Role |
|------|------|
| `data/prints.json` | Canonical print records (id, name, type, rarity aggregates) |
| `data/print_signals.ndjson` | Append-only raw community signals (one JSON line each) |
| `data/print_guide_index.json` | Chatbot-ready rollup per print + global FAQ nuggets |

Conflict rule: newer signals inform aggregates; keep raw signals forever so the Guide can show provenance.

## `print_signals.ndjson` (one event per line)

```json
{
  "ts": "2026-09-06T15:22:00-07:00",
  "signal_type": "fair_trade|rarity_claim|catalog_vs_custom|want_have|release_note|ooc_note|machine_stock_print|other",
  "print_refs": [{"name": "Foxglove", "id": null, "number": null}],
  "print_refs_secondary": [{"name": "custom NY Liberty", "id": null}],
  "claim": "Custom for catalog is uneven; fairer is custom↔custom or custom↔rare catalog",
  "direction": "custom_asks_catalog|catalog_asks_custom|rare_guidance|unknown",
  "consensus_hint": "agree|disagree|mixed|single_voice",
  "authors": ["Kelly"],
  "source": "facebook_group|messenger",
  "source_url": "https://...",
  "thread_name": "NYC/NJ",
  "quote": "short excerpt",
  "confidence": "high|medium|low"
}
```

`print_refs` = the print(s) being discussed. `print_refs_secondary` = counterparty in a trade suggestion.

## `print_guide_index.json`

```json
{
  "updated_at": "ISO",
  "prints": {
    "<print_id_or_slug>": {
      "id": "...",
      "name": "1997",
      "number": "1997",
      "type": "custom|catalog|unknown",
      "rarity_label": "Rare|Uncommon|Common|Unknown",
      "rarity_basis": "community|print_keep|mixed",
      "fair_trade_notes": ["Custom↔custom preferred over custom↔common catalog"],
      "trade_examples": [{"gave": "...", "wanted": "...", "community_take": "uneven|fair|..."}],
      "release_notes": [],
      "ooc": false,
      "signal_count": 3,
      "last_signal_at": "ISO",
      "citations": [{"ts": "...", "source": "messenger", "url": "..."}]
    }
  },
  "global_tips": [
    {
      "id": "custom-vs-catalog-trade",
      "topic": "fair_trade",
      "text": "Customs are often treated as higher trade value than common catalog pulls; community often suggests custom↔custom or custom↔rare.",
      "confidence": "high",
      "signal_count": 12
    }
  ]
}
```

## What to capture on each sweep

- Someone asks “is X rare?” / rarity labels (Rare, OOC, etc.)
- Fair-trade advice (especially custom vs catalog, rare vs common)
- Want/have posts that imply relative value
- Release location / drop context (“only at Barclays”, pop-up exclusive)
- OOC / out-of-circulation claims
- Do **not** invent rarity — only index what people say, with quotes + links

## Chatbot use

Print Guide should prefer `print_guide_index.json` for answers, and fall back to recent lines in `print_signals.ndjson` for provenance. Machine stock stays in `machines.json` (availability ≠ rarity).
