# Central node mode

Postgres / app configured as central node (aggregates; leaf-only UI suppressed).

## Sub-features

- `central-flag` — `app.is_central_node` enabled in Postgres command/settings

## How to get to it (user POV)

- Operators deploy this compose (not leaf demo). Expect national dashboards; leaf-only flows suppressed (Overdue Patient List tab / overdue access denied; Admin Refresh button hidden; hourly matview refresh no-op)

## Driving it with browser / docker

Preconditions: stack up; Postgres finished init (`heart360tk_database` exists).

- Action: `docker compose config` shows `app.is_central_node=true`; query `SHOW app.is_central_node;` as image superuser `heart360tk_root` (or equivalent); optionally confirm provisioned dashboards reference `is_central_node` / `IsCentralNode`
- Observe: GUC is `true` / `on`; Grafana loads without requiring leaf overdue / refresh UX
- Evidence: snippet of `docker compose config` and query output under evidence/

## Gotchas

- Do not use leaf demo compose as proof of central mode
- Doctor `psql` user is `heart360tk_root` (image default), not necessarily `.env` `POSTGRES_USER`
- Compose healthcheck still targets `db_prod` while the DB name is `heart360tk_database` — health may be misleading (product drift in compose)
