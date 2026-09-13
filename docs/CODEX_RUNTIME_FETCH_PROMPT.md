# Prompt for Codex — Print Keep runtime fetch from miniprint-db

Copy everything below the line into Codex (in the Print Keep site repo).

---

## Goal

Wire **Print Keep** to load Ana Inciardi machine / print data at **runtime** from the public GitHub data repo. Do **not** bake the JSON into the build, and do **not** add a deploy webhook / rebuild-on-push for data updates.

Producer: a separate bot pushes updated JSON to `mariebks/miniprint-db` after scrapes.  
Consumer: Print Keep should `fetch` those files so the live site stays current within ~5 minutes (GitHub raw CDN cache).

## Canonical data (public)

Repo: https://github.com/mariebks/miniprint-db (`main`)

Read in-repo docs (via GitHub or raw):
- https://raw.githubusercontent.com/mariebks/miniprint-db/main/docs/PRINT_KEEP_RUNTIME_FETCH.md
- https://raw.githubusercontent.com/mariebks/miniprint-db/main/docs/SCHEMA.md
- https://raw.githubusercontent.com/mariebks/miniprint-db/main/docs/PRINT_GUIDE_SCHEMA.md
- https://raw.githubusercontent.com/mariebks/miniprint-db/main/README.md

### Fetch URLs

```
https://raw.githubusercontent.com/mariebks/miniprint-db/main/data/machines.json
https://raw.githubusercontent.com/mariebks/miniprint-db/main/data/prints.json
https://raw.githubusercontent.com/mariebks/miniprint-db/main/data/updates.ndjson
https://raw.githubusercontent.com/mariebks/miniprint-db/main/data/print_guide_index.json
https://raw.githubusercontent.com/mariebks/miniprint-db/main/data/print_signals.ndjson
https://raw.githubusercontent.com/mariebks/miniprint-db/main/data/products.json
https://raw.githubusercontent.com/mariebks/miniprint-db/main/data/product_signals.ndjson
```

## What to implement

1. **Data client** — a small module that fetches + parses:
   - `machines.json` → array (maps/lists/detail)
   - `prints.json` → object keyed by print id
   - `updates.ndjson` → line-delimited JSON (activity / history)
   - `print_guide_index.json` → Ask Print Guide context
2. **Merge, don’t clobber.** Treat repo JSON as the bot **base** layer. Keep a Print Keep **local_overrides** store keyed by stable `machine_id` / print `id`. UI = merge(base, local_overrides) where **local wins** on any field the user/site explicitly set.
   - New repo-only machines/prints → add them.
   - New bot fields with no local override → take from repo.
   - User-edited / pinned fields in Print Keep → **never** overwritten by a later fetch.
   - Do **not** wholesale replace Print Keep’s DB with the raw fetch if that would wipe edits.
   - Fall back to last good cache only if fetch fails.
3. **Cache lightly** (optional): on page load (no aggressive polling). GitHub raw already caches ~300s.
4. **UI sanity check:** after wiring, confirm machines include **Lincoln Financial Field — Pepsi Plaza / Section 107 / Section 113** when loaded from the live URL — proves runtime sync works.
5. Keep stock (`machines.json`) separate from rarity (`print_guide_*`). Do not invent rarity.
6. Document the overlay rules in Print Keep README (repo base + local wins).

## Hard rules

- Do **not** set up GitHub Actions deploy hooks for this task.
- Do **not** scrape Facebook / Instagram / Messenger.
- Do **not** use miniprint.io as a data source.
- Do **not** change the producer file formats; if schema gaps appear, note them — don’t silently reshape without documenting.
- Privacy: never display community members’ personal names from quotes if present; prefer anonymous labels.

## Done when

- Print Keep loads machines/prints from the raw GitHub URLs at runtime.
- Local Print Keep edits are preserved across fetches (overlay / pinned fields).
- No rebuild required for new Bot pushes (aside from CDN lag + page reload).
- README lists feed URLs + merge rules (local wins).
- Summary: files touched, how merge works, and that the three Eagles Linc machines appear from the live feed without wiping sample local overrides (add a quick test override if useful).
