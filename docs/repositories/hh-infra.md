# hh-infra

Infrastructure orchestration repository. Pulls all polyrepo services together into a single Docker Compose deployment with nginx reverse proxy, PostgreSQL, backup/restore scripts, and a Makefile.

## Repository Structure

```
hh-infra/
├── docker-compose.dev.yml      # Dev stack (latest-dev / latest images)
├── docker-compose.yml          # Production stack (pinned version tags)
├── nginx/
│   └── nginx.conf              # Reverse proxy routing
├── postgres/
│   └── init.sql                # Database creation on first boot
├── scripts/
│   ├── backup.sh               # Dump all databases to snapshot
│   └── restore.sh              # Restore databases from snapshot
├── backups/                    # Snapshot storage (git-ignored)
├── .env.example                # Documented environment variables
├── .gitignore
├── Makefile                    # Convenience targets
└── README.md
```

## Quick Start

```bash
# Authenticate with GHCR (one-time)
echo $PAT | docker login ghcr.io -u YOUR_USERNAME --password-stdin

# Start the development stack
make up
```

Platform available at `http://localhost`.

## Compose Modes

### Development (`docker-compose.dev.yml`)

Uses `latest-dev` images (or `latest` for hh-web) with `ENV=development` so seed data runs automatically.

### Production (`docker-compose.yml`)

Uses pinned version tags with `ENV=production`. No seed data.

| Service | Version |
|---------|---------|
| hh-identity | v0.2.2 |
| hh-goals | v0.3.1 |
| hh-finances | v0.2.1 |
| hh-investments | v0.1.3 |
| hh-credits | v0.1.3 |
| hh-web | v0.10.3 |

## Routing

Nginx listens on port 80 and routes API requests via the `/api/<service>/` prefix. The prefix is stripped before forwarding. Frontend routes (e.g. `/investments/portfolio`) fall through to the SPA catch-all.

| Path Pattern | Service | Internal Port |
|--------------|---------|---------------|
| `/api/identity/*` | hh-identity | 8080 |
| `/api/goals/*` | hh-goals | 8080 |
| `/api/finances/*` | hh-finances | 8080 |
| `/api/investments/*` | hh-investments | 8080 |
| `/api/credits/*` | hh-credits | 8080 |
| `/` (catch-all) | hh-web | 80 |

### Blocked Routes

Requests matching `/<service>/_system/*` are blocked with 403. The `/_system/` endpoints (health, readiness) are internal-only and should not be exposed through the reverse proxy.

### WebSocket Support

The frontend catch-all includes `Upgrade` and `Connection` headers for Vite HMR in development.

## Port Allocation

| Port | Service | Purpose |
|------|---------|---------|
| 80 | nginx | Single HTTP entry point |
| 5432 | postgres | Direct DB access for local tooling |

All backend services communicate internally on port 8080 with no host port mapping.

## Make Targets

| Target | Description |
|--------|-------------|
| `make up` | Start dev stack |
| `make up-prod` | Start production stack |
| `make down` | Stop services (preserves volumes) |
| `make logs` | Follow all container logs |
| `make ps` | Show container status |
| `make backup` | Snapshot databases (optional `LABEL=`) |
| `make restore` | Restore from snapshot (`SNAPSHOT=` required) |
| `make db-reset` | Destroy volumes and restart fresh |
| `make help` | List targets |

## Data Persistence

PostgreSQL uses a named volume (`pgdata`) that survives `docker compose down`. Use `make db-reset` to destroy it and start fresh.

### Backup / Restore

```bash
make backup LABEL=before-migration
make restore SNAPSHOT=backups/before-migration
```

Snapshots are stored in `backups/` (git-ignored). Each contains one `.sql` file per database using `pg_dump --clean --if-exists`.

## GHCR Authentication

All images are private on GitHub Container Registry. Create a PAT with `read:packages` scope and run:

```bash
echo $PAT | docker login ghcr.io -u YOUR_USERNAME --password-stdin
```

## Notes

- hh-web only publishes `:latest` and `:<version>` tags (no `-dev` variant) since it has no seed data concept
- Each service runs its own goose migrations on startup — `init.sql` only creates empty databases
- Inter-service communication uses compose service names (e.g. `http://hh-identity:8080/v1/jwks`)
