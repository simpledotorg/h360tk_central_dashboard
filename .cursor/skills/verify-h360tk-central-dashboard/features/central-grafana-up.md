# Central Grafana up

Central Grafana UI is reachable and login works.

## Sub-features

- `central-grafana-login` — Login page/HTTP OK on `:3000` (or `:13000` with verify override)
- `central-home` — A HEARTS360 dashboard loads after login (image home `/d/heart360_drilldown`)

## How to get to it (user POV)

- Open `http://127.0.0.1:3000` (or `:13000` with override) and sign in as `admin` using `GF_SECURITY_ADMIN_PASSWORD` from `docker-compose.yml`

## Driving it with browser

Preconditions: compose up; doctor OK (Grafana HTTP 200 on the port you launched).

- Action: open Grafana → login → open a provisioned HEARTS360 dashboard (`heart360_drilldown` or folder **HEARTS360 Dashboards**)
- Observe: Grafana shell + dashboard without fatal panel errors
- Evidence: screenshot and/or API login + `/api/search` listing under evidence/

## Gotchas

- Conflicts with grafana_core on `:3000` / container name `grafana` — use `helpers/compose.verify-override.yml` (Grafana `:13000`); do not stop the other stack
- Fresh Postgres must finish image init (creates `grafana` DB role). Override uses a named volume when `./.database` bind-mount init fails on Docker Desktop
