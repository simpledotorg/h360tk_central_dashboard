# Central Grafana up

Central Grafana UI is reachable and login works.

## Sub-features

- `central-grafana-login` — Login page/HTTP OK (host `:3000`, or `:13000` with verify override)
- `central-home` — Provisioned HEARTS360 dashboard `heart360-home` loads after login

## How to get to it (user POV)

- Open `http://127.0.0.1:3000` (or `:13000` with override) and sign in as `admin` using `GF_SECURITY_ADMIN_PASSWORD` from compose

## Driving it with browser

Preconditions: compose up; doctor OK (Grafana answering, not crash-looping).

- Action: open Grafana → login → open `/d/heart360-home/` (or API `GET /api/dashboards/uid/heart360-home`)
- Observe: Grafana shell + Home dashboard JSON/UI without fatal auth errors
- Evidence: screenshot and/or API search listing HEARTS360 Dashboards

## Gotchas

- Conflicts with grafana_core on `:3000` and shared container names `grafana`/`postgres` — use [helpers/compose.verify-override.yml](../helpers/compose.verify-override.yml) rather than stopping an unrelated stack
- Bind-mounted `./.database` can fail Postgres init on Docker Desktop (ownership); override uses a named volume so Grafana’s `grafana` DB role is created
- Dashboards are baked into `simpledotorg/heart360tk-grafana` — not mounted from this repo
