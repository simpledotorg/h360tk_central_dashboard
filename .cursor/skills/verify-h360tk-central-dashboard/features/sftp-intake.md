# SFTP intake

SFTP service for leaf nodes to drop aggregate uploads.

## Sub-features

- `sftp-up` — SFTP container listening (host `2222` per product compose, or `12222` with verify override)
- `sftp-config` — `sftp_config/users.conf` mounted

## How to get to it (user POV)

- Leaf/ops connect with credentials provisioned for their upload user/folder

## Driving it with docker

Preconditions: stack up.

- Action: `docker compose ps sftp`; confirm port publish; confirm `sftp_config/users.conf` present
- Observe: service Up; config file exists
- Optional: SFTP put of a test aggregate if credentials known from local config — do not invent passwords in shared docs
- Evidence: `ps` output; optional transfer transcript

## Gotchas

- Credentials live in local config/env — never copy them into the private knowledge repo
- Importer reaches SFTP on the Docker network (`SFTP_HOST=sftp`, port `22`), not via the host-mapped port
- Host port differs under [helpers/compose.verify-override.yml](../helpers/compose.verify-override.yml) (`12222`)
