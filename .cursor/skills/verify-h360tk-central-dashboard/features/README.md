# h360tk_central_dashboard verification map

National / central node. Aggregates only — no patient-level import proofs.

## Baseline preconditions

- Working tree is **h360tk_central_dashboard** (not stale `h360tk_central_core`)
- `docker compose up -d` **or** conflict-safe override in `helpers/compose.verify-override.yml`; doctor OK
- Evidence: `.cursor/skills/verify-h360tk-central-dashboard/evidence/<run-id>/`

## Features

- [Central Grafana up](./central-grafana-up.md)
- [Central node mode](./central-node-mode.md)
- [Aggregate importer service](./aggregate-importer.md)
- [SFTP intake](./sftp-intake.md)
