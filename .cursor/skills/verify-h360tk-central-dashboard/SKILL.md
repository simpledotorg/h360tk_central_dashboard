---
name: verify-h360tk-central-dashboard
description: Verify the HEARTS360 national/central dashboard stack (Grafana + aggregate importer + SFTP). Use when proving central-node behavior in h360tk_central_dashboard.
---

# Verify h360tk_central_dashboard

Agent-facing control skill for the **canonical central / national** repository: https://github.com/simpledotorg/h360tk_central_dashboard

Prefer this clone over any older local `h360tk_central_core` folder.

## Launch

From the repo root (requires `.env` with `HEART360TK_VERSION` and importer/SFTP settings):

```bash
docker compose up -d
```

Typical host ports from compose:

| Service | Host port |
|---------|-----------|
| Grafana | `3000` |
| Postgres | `5432` |
| SFTP | `2222` → container 22 |

Postgres is started with `app.is_central_node=true`.

### Port / name conflicts (grafana_core)

If host `:3000` / `:5432` or container names `grafana` / `postgres` are taken (commonly `h360tk_grafana_core`), **do not stop the other stack**. Use the verify-skill override (renames containers, remaps ports, uses a named Postgres volume so Docker Desktop bind-mount init failures on `./.database` are avoided):

```bash
docker compose -p h360tk_central_verify \
  -f docker-compose.yml \
  -f .cursor/skills/verify-h360tk-central-dashboard/helpers/compose.verify-override.yml \
  up -d
```

| Service | Override host port |
|---------|-------------------|
| Grafana | `13000` |
| Postgres | `15432` |
| SFTP | `12222` |

Ready when Grafana login responds (use `13000` with the override):

```bash
curl -sf -o /dev/null -w "%{http_code}" http://127.0.0.1:3000/login
# or: http://127.0.0.1:13000/login
docker compose ps   # add -p / -f flags when using the override
```

Auth: Grafana user `admin`; password is `GF_SECURITY_ADMIN_PASSWORD` in `docker-compose.yml` (this repo has no README). Do not invent credentials.

Teardown (match the project/flags you used to launch):

```bash
docker compose down
# or:
docker compose -p h360tk_central_verify \
  -f docker-compose.yml \
  -f .cursor/skills/verify-h360tk-central-dashboard/helpers/compose.verify-override.yml \
  down
```

## Doctor

Default ports:

```bash
docker compose ps
curl -sf -o /dev/null -w "grafana:%{http_code}\n" http://127.0.0.1:3000/login
docker compose exec -T postgres psql -U heart360tk_root -d "${POSTGRES_DB:-heart360tk_database}" -c "SHOW app.is_central_node;"
```

With override, use project `-p h360tk_central_verify`, both compose files, and `http://127.0.0.1:13000/login`.

Require: `grafana`, `postgres`, `importer`, `sftp` present (names per compose / override). Image default superuser is `heart360tk_root` (baked into `heart360tk-postgresql`); `.env` `POSTGRES_USER` is for the importer role, not necessarily the `psql` login for Doctor.

**Importer on `HEART360TK_VERSION=0.5.0`:** published image reads misspelled `IMPORT_AGGREAGATE_DATA`, so with compose `IMPORT_AGGREGATE_DATA=true` the importer still exits and `restart: unless-stopped` yields `Restarting (0)`. Treat stable importer `Up` as blocked on that image; prove env wiring from compose/`.env` + log line showing the typo, and file a product fix in `h360tk_grafana_core` — do not paper over it in this map.

## Drive

Browser: Grafana on `http://127.0.0.1:3000` (or `:13000` with override) — central UI suppresses leaf-only controls (Overdue tab / overdue access, admin Refresh button) when `app.is_central_node=true`.

Importer path: confirm compose passes `IMPORT_AGGREGATE_DATA`; on a fixed image, confirm container `Up` and scheduler logs. Full aggregate round-trip needs a leaf export ZIP fixture — mark blocked if fixture missing rather than faking success.

SFTP: `sftp_config/users.conf` mounted; leaves drop files under remote `/upload` (host `./data/sftp-upload`). Override listens on host `12222`.

## Evidence

`.cursor/skills/verify-h360tk-central-dashboard/evidence/<run-id>/`

## Cleanup

```bash
docker compose down
# or the override teardown above
```

Preserve evidence. Prefer the override’s named volume for verify runs; do not wipe a shared `./.database` unless the recipe requires a clean central DB and you own that residue.

## Feature map

See [features/README.md](features/README.md).
