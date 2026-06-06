# unifi-backup

Trigger and optionally download backups from a UniFi OS Server via the local API.

Requires [uv](https://docs.astral.sh/uv/) — dependencies are declared inline and managed automatically.

## Backup types

| Type | What it does |
|---|---|
| `network` | Triggers a Network application backup (`.unf`). Optionally downloads it. |
| `system` | Triggers and downloads a full UniFi OS system backup (`.unifi`). |
| `both` | Does both. Always downloads both files. |

Network backups include device config, clients, and optionally event history (controlled by `--days`).
System backups are config-only snapshots of the entire UniFi OS installation.

## Authentication

| Auth method | `network` | `system` / `both` |
|---|---|---|
| API key | ✅ | ❌ |
| User / password | ✅ | ✅ |

API keys are scoped to the Network application and cannot access OS-level endpoints.
System and both types require user/password auth.

Generate an API key under **UniFi Network → Settings → Control Plane → Integrations**.

## Usage

```bash
chmod +x unifi_backup

# Trigger a network backup (no download)
./unifi_backup --host unifi.example.com --api-key YOUR_KEY

# Trigger and download a network backup
./unifi_backup --host unifi.example.com --api-key YOUR_KEY --download --outdir /backups

# Download a system backup
./unifi_backup --host unifi.example.com --user admin --password secret --type system --outdir /backups

# Trigger and download both
./unifi_backup --host unifi.example.com --user admin --password secret --type both --outdir /backups
```

Credentials should be passed via environment variables rather than CLI flags to avoid exposure in process listings and shell history.

## Options

| Flag | Env var | Default | Description |
|---|---|---|---|
| `--host` | `UNIFI_HOST` | | Controller hostname or IP |
| `--port` | `UNIFI_PORT` | `443` | Controller HTTPS port |
| `--api-key` | `UNIFI_API_KEY` | | API key (network type only) |
| `--user` | `UNIFI_USER` | | Admin username |
| `--password` | `UNIFI_PASS` | | Admin password |
| `--site` | `UNIFI_SITE` | `default` | Site name (network type only) |
| `--type` | `UNIFI_TYPE` | `network` | Backup type: `network`, `system`, or `both` |
| `--download` | `UNIFI_DOWNLOAD` | `false` | Download after triggering (network type only) |
| `--outdir` | `UNIFI_OUTDIR` | `.` | Directory to save downloaded backups |
| `--days` | `UNIFI_DAYS` | `-1` | Days of event history to include in network backup (`-1` = all) |
| `--verify-ssl` | `UNIFI_VERIFY_SSL` | `false` | Verify TLS certificate |

## API endpoints

| Operation | Method | Path |
|---|---|---|
| Network backup trigger | `POST` | `/proxy/network/api/s/{site}/cmd/backup` |
| Network backup download | `GET` | `/proxy/network/dl/backup/{filename}` |
| System backup (trigger + download) | `GET` | `/api/backup/download` |

## Notes

- Network backup filenames are prefixed with a datestamp on download (e.g. `20260606_143000_10.4.57.unf`).
- The `--days -1` default includes all event history, which can make `.unf` files significantly larger. Use `--days 0` for a config-only network backup.
- If running on the UniFi OS Server VM itself, the triggered `.unf` is also written to `/usr/lib/unifi/data/backup/` inside the Network application container and accessible on the VM filesystem at `/home/uosserver/.local/share/containers/storage/volumes/uosserver_var_lib_unifi/_data/backup/`.

