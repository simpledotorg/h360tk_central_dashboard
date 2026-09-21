# Aggregate importer service

Importer container that pulls aggregate files into the central DB.

## Sub-features

- `importer-running` — `importer` service stays Up (scheduler idle or importing — not exit/restart loop)
- `importer-env` — Aggregate import enabled for the **process the image actually reads**

## How to get to it (user POV)

- Central admin relies on scheduled import of leaf aggregate files (no manual patient CSV)

## Driving it with docker

Preconditions: stack up.

- Action: `docker compose ps importer`; `docker compose logs importer --tail 100`; inspect container env for both spellings below
- Observe: container Up; logs show scheduler/activity or clean idle wait — not crash loop / immediate exit
- Evidence: `ps` + log excerpt files

## Gotchas

- Compose wires `IMPORT_AGGREGATE_DATA` from `.env`. Image **0.5.0** logs and honors misspelled **`IMPORT_AGGREAGATE_DATA`** (extra `A`), which defaults to `false` inside the image. Result: compose can show `IMPORT_AGGREGATE_DATA=true` while the process disables import and exits → `restart: unless-stopped` restart loop. That is a **product gap** (image/compose env name mismatch), not a verify-map pass.
- End-to-end aggregate proof needs a leaf-produced aggregate fixture **and** a running importer that actually enables aggregate import; without either, only prove wiring / report the gap
- `.env` may define `SFTP_DEST_PATH`; compose does not pass it — folder path is `IMPORT_FOLDER_PATH`
