# Central node mode

Postgres / app configured as central node (aggregates, leaf-only UI suppressed).

## Sub-features

- `central-flag` — `app.is_central_node` enabled in Postgres command/settings

## How to get to it (user POV)

- Operators deploy this compose; leaf-only UI should not be the primary experience

## Driving it with browser / docker

Preconditions: stack up with a successfully initialized Postgres (`heart360tk_database` exists).

- Action: `docker compose config` / inspect postgres service command includes `app.is_central_node=true`; query `SHOW app.is_central_node;` as image superuser `heart360tk_root` on `heart360tk_database`
- Observe: setting `true`; Grafana loads without requiring leaf upload UX (Overdue tab gated in image dashboards)
- Evidence: snippet of `docker compose config` or query output saved under evidence/

## Gotchas

- Do not use leaf demo compose as proof of central mode
- Doctor default user is **`heart360tk_root`**, not `heart360tk`
- If `./.database` bind-mount init failed, the GUC may be unsettable until Postgres is healthy on a clean volume (use verify override named volume)
