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

Typical host ports from product compose:

| Service | Host port |
|---------|-----------|
| Grafana | `3000` |
| Postgres | `5432` |
| SFTP | `2222` → container 22 |

Postgres is started with `app.is_central_node=true`.

**Port / name conflicts:** product compose hardcodes host `3000`/`5432` and container names `grafana`/`postgres`. If another stack (commonly `h360tk_grafana_core`) already holds those, do **not** stop it unless you started it in this run. Instead launch with the verify-skill override (unique names + alternate ports + named Postgres volume):

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

The override also replaces the bind-mounted `./.database` with a named Docker volume. On Docker Desktop (esp. linux/amd64 images on arm64), the bind mount often fails Postgres init (`wrong ownership` → skips DB/role creation → Grafana crash-loops on user `grafana`). Prefer the override for local verify on macOS.

Ready when Grafana login responds (use `13000` when on the override):

```bash
curl -sf -o /dev/null -w "%{http_code}" http://127.0.0.1:3000/login
# or: http://127.0.0.1:13000/login
docker compose ps   # add -p / -f flags when using the override
```

Auth: Grafana admin user `admin`; password from compose env `GF_SECURITY_ADMIN_PASSWORD` (this repo has no README). Do not invent or paste the password into shared docs.

Teardown (default project):

```bash
docker compose down
```

Teardown (override project; keeps evidence; removes verify named volume when intentional):

```bash
docker compose -p h360tk_central_verify \
  -f docker-compose.yml \
  -f .cursor/skills/verify-h360tk-central-dashboard/helpers/compose.verify-override.yml \
  down
# optional clean volume: add -v
```

## Doctor

Default ports:

```bash
docker compose ps
curl -sf -o /dev/null -w "grafana:%{http_code}\n" http://127.0.0.1:3000/login
docker compose exec -T postgres psql -U heart360tk_root -d "${POSTGRES_DB:-heart360tk_database}" -c "SHOW app.is_central_node;"
```

Override ports: same checks against `13000` / project `h360tk_central_verify` and the override `-f` pair above.

Require: `grafana`, `postgres`, `importer`, `sftp` Up (or override container names `h360tk-central-verify-*`). Note: image superuser is `heart360tk_root` (not `heart360tk`). Compose healthcheck still references `db_prod` — ignore that name; doctor should use `heart360tk_database`.

## Drive

Browser: Grafana on `http://127.0.0.1:3000` (or `:13000` with override) — central UI may hide leaf-only controls. After login, open provisioned UID `heart360-home` (folder HEARTS360 Dashboards). Dashboards are baked into the Grafana image (not mounted in this repo).

Importer path: confirm `importer` container behavior and env from `.env` (`IMPORT_AGGREGATE_DATA=true` in compose). **Image 0.5.0 reads misspelled `IMPORT_AGGREAGATE_DATA`** (defaults false) and exits when false, which with `restart: unless-stopped` becomes a restart loop — treat as product gap, not map success. Full aggregate round-trip also needs a leaf export fixture — mark blocked if fixture missing rather than faking success.

## Evidence

`.cursor/skills/verify-h360tk-central-dashboard/evidence/<run-id>/`

## Cleanup

```bash
docker compose down
# or override teardown above
```

Preserve evidence. Do not wipe product `./.database` unless the recipe requires a clean central DB. The verify override uses a separate named volume — safe to `down -v` for that project only.

## Feature map

See [features/README.md](features/README.md).

## Helpers

- [helpers/compose.verify-override.yml](helpers/compose.verify-override.yml) — alternate host ports, unique container names, named Postgres volume for conflict-safe / Docker-Desktop-safe verify.
