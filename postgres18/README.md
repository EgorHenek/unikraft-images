# PostgreSQL 18 Unikernel Image

PostgreSQL 18.4 compiled from source and packaged as a unikernel for Unikraft Cloud.

## Features

- **PG 18.4** — latest stable PostgreSQL
- **Scale-to-zero** — built-in `pg_ukc_scaletozero` plugin (patched for PG 18 API)
- **Minimal footprint** — Alpine 3.21 rootfs, runtime dependencies only
- **CI-built** — GitHub Actions, native x86_64

## Quick Start

```bash
unikraft run --metro fra \
  --scale-to-zero policy=idle,cooldown-time=1000,stateful=true \
  -p 5432:5432/tls -m 1Gi \
  -e POSTGRES_PASSWORD=unikraft \
  --image egorhenek/postgres:18
```

Connect:

```bash
psql -U postgres -h <instance>.fra.unikraft.app
# password: unikraft
```

## Persistent Storage

Create a volume:

```bash
unikraft volume create --set metro=fra --set name=postgres --set size=1G
```

Run with volume attached:

```bash
unikraft run --metro fra \
  --scale-to-zero policy=idle,cooldown-time=1000,stateful=true \
  -p 5432:5432/tls -m 1Gi \
  -e POSTGRES_PASSWORD=unikraft \
  -e PGDATA=/volume/postgres \
  --volume postgres:/volume \
  --image egorhenek/postgres:18
```

On first boot `initdb` initializes `/volume/postgres`. Data persists across restarts and scale-to-zero events.

## Environment Variables

| Variable | Default | Description |
|---|---|---|
| `POSTGRES_PASSWORD` | *(required)* | Superuser password |
| `POSTGRES_USER` | `postgres` | Superuser name |
| `POSTGRES_DB` | `$POSTGRES_USER` | Default database |
| `PGDATA` | `/var/lib/postgresql/data` | Data directory |

## CI Build

Manual trigger via GitHub Actions:

1. Actions → «Build Unikraft Image» → Run workflow
2. `image`: `postgres18`
3. Image published as `egorhenek/postgres:18`

## Local Build

Prerequisites: Docker (BuildKit) + Unikraft CLI + Unikraft Cloud token.

```bash
unikraft login --token <file>
cd postgres18
unikraft build . --output egorhenek/postgres:18
```

## Known Limitations

- `pg_ukc_scaletozero` is patched locally for PG 18 — may need re-patching after PG minor updates
- Without a volume, data is lost on scale-to-zero idle shutdown
- `shared_preload_libraries` is set in Kraftfile (required for scale-to-zero)

## Instance Management

```bash
# List instances
unikraft instances list

# Remove instance
unikraft instance remove <name>

# Remove volume (destroys data!)
unikraft volume remove postgres
```
