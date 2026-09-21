# SFTP intake

SFTP service for leaf nodes to drop aggregate uploads.

## Sub-features

- `sftp-up` — SFTP container listening (host `2222` per compose; `12222` with verify override)
- `sftp-config` — `sftp_config/users.conf` mounted; upload dir `./data/sftp-upload` → remote `/upload`

## How to get to it (user POV)

- Leaf/ops connect with credentials provisioned for their upload user/folder and put `{SOURCE_KEY}.zip` under `/upload`

## Driving it with docker

Preconditions: stack up.

- Action: `docker compose ps sftp`; confirm published port (`2222` or override `12222`); confirm `sftp_config/users.conf` present; optional SFTP put into `/upload` and confirm file under `data/sftp-upload/`
- Observe: service Up; config file exists; optional put visible on host bind mount
- Evidence: `ps` / `port` output; optional transfer transcript (no passwords in evidence)

## Gotchas

- Credentials live in `sftp_config/users.conf` / `.env` — never copy secrets into the private knowledge repo or commit them into evidence
- Importer reaches SFTP on Docker network `sftp:22`; leaves usually use host `2222` (or `12222` when verifying beside grafana_core)
