# Recommended products (affiliate pipeline)

Captured from FB group + Messenger when people recommend something to buy (sleeves, binders, coin rollers, display cases, travel kits, etc.).

## Files

| Path | Role |
|------|------|
| `data/product_signals.ndjson` | Append-only: each recommendation mention |
| `data/products.json` | Rollup by normalized product / Amazon ASIN |

## `product_signals.ndjson`

```json
{
  "ts": "2026-09-06T15:00:00-07:00",
  "url": "https://www.amazon.com/dp/B0XXXX",
  "asin": "B0XXXX",
  "merchant": "amazon|other",
  "title_guess": "penny sleeves",
  "use_case": "protect miniprints|storage|display|shipping|other",
  "quote": "short excerpt",
  "source": "facebook_group|messenger",
  "source_url": "https://…",
  "confidence": "high|medium|low"
}
```

Only record links/products people actually recommend. Do not invent URLs. Strip tracking junk when normalizing; keep original URL in the signal.

## `products.json`

Keyed by ASIN or normalized URL host+path:
```json
{
  "B0XXXX": {
    "asin": "B0XXXX",
    "url": "https://www.amazon.com/dp/B0XXXX",
    "title_guess": "…",
    "use_cases": ["protect miniprints"],
    "mention_count": 3,
    "last_seen": "ISO",
    "citations": [{"ts":"…","source":"messenger","url":"…"}]
  }
}
```

Affiliate tags are **not** added here — her site/Codex owns tagging later.

## Exclusions (important)

Do **NOT** index as products:
- Mystery packs, print drops, or anything that is itself a miniprint / Ana Inciardi print for sale
- Official inciardiprints.com print/pack listings
- UFT/ISO trade posts for specific prints

**DO** index accessories and tools people recommend for collecting: sleeves, toploaders, binders, frames, display cases, coin rollers/wrappers, shipping supplies, storage boxes, lighting, etc.

## Privacy

Never store personal names of community members. Use anonymous labels only (`community member`, relative time, thread name, source URL). Do not include `authors` fields with real names.
