# Aggregate importer service

Importer container that pulls aggregate files into the central DB.

## Sub-features

- `importer-running` — `importer` service is stably `Up` (not crash/restart loop)
- `importer-env` — Aggregate import enabled via env (`IMPORT_AGGREGATE_DATA`)

## How to get to it (user POV)

- Central admin relies on scheduled import of leaf aggregate files (no manual patient CSV)

## Driving it with docker

Preconditions: stack up.

- Action: `docker compose ps importer`; `docker compose logs importer --tail 100`; confirm compose/`.env` set `IMPORT_AGGREGATE_DATA=true`
- Observe (healthy image): container `Up`; logs show config OK, `IMPORT_AGGREGATE_DATA : True`, and scheduler started / idle wait — not crash loop
- Evidence: `ps` + log excerpt files

## Gotchas

- **`HEART360TK_VERSION=0.5.0` product bug:** image reads misspelled `IMPORT_AGGREAGATE_DATA`, ignores compose `IMPORT_AGGREGATE_DATA=true`, logs `IMPORT_AGGREAGATE_DATA=false`, exits 0 → `Restarting (0)`. Mark `importer-running` **verified-unreachable** on that image; still record env + typo log evidence. Fix belongs in `h360tk_grafana_core` importer image, not this map.
- When import is intentionally disabled, process exits 0 and `restart: unless-stopped` also restart-loops — “Up” alone is not enough; check logs for scheduler vs disabled-exit
- End-to-end aggregate proof needs a leaf-produced aggregate ZIP fixture; without it, only prove service health / config
