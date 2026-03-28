# New Service Checklist

Steps for adding a new hh-* service to the platform.

## 1. Repository Setup

- [ ] Create repo `hh-<name>` on GitHub
- [ ] Initialize Go module: `go mod init github.com/tiagorocha94/hh-<name>`
- [ ] Add `hh-shared` dependency: `go get github.com/tiagorocha94/hh-shared@latest`
- [ ] Create project structure (see [conventions.md](conventions.md))
- [ ] Add `.gitignore` (Go conventions: `vendor/`, `bin/`, `.env.*`, `!.env.example`)
- [ ] Add `.gitattributes` with `* text=auto eol=lf`

## 2. Application Code

- [ ] `cmd/main.go` — config, logger, DB, migrations, seeds, wiring, server
- [ ] Validate `JWKS_URL` at startup (fail-fast if empty)
- [ ] `internal/handler/router.go` — apply `middleware.JWTAuth(ctx, jwksURL)` to `/v1`
- [ ] Mount `/_system` routes via `system.Routes(pool, buildInfo)`
- [ ] Serve OpenAPI docs at `/docs` and `/openapi.yaml`

## 3. Database

- [ ] Create `migrations/` directory with goose SQL migrations
- [ ] Create `seeds/dev/` with idempotent dev seed SQL
- [ ] Use shared member UUIDs from `hh-shared/seeds/identities.go`

## 4. Docker

- [ ] `Dockerfile` — multi-stage, SSH forwarding, HEALTHCHECK, ldflags
- [ ] `docker-compose.yml` — postgres + auth-svc + app
- [ ] `init.sql` — create service DB + hh_auth DB on same postgres
- [ ] `.env.example` — document all env vars including `JWKS_URL`

## 5. Documentation

- [ ] `docs/openapi.yaml` — full API spec with `securitySchemes`, 401/403 responses
- [ ] `docs/database.md` — schema and design decisions
- [ ] `README.md` — version, quick start, link to hh-docs for platform conventions
- [ ] Add service page to hh-docs (`repositories/<service>.md`)

## 6. CI/CD

- [ ] `.github/workflows/ci.yml` — build, vet, fmt, unit tests, integration tests
- [ ] `.github/workflows/cd.yml` — full tests + Docker push
- [ ] Add `SSH_PRIVATE_KEY` secret for hh-shared access

## 7. Makefile

Standard targets: `init`, `up`, `down`, `logs`, `ps`, `migrate-up`, `migrate-down`, `migrate-status`

## 8. Integration

- [ ] Add service to `hh/infra/docker-compose.yml` (GHCR image)
- [ ] Add nginx upstream and location block
- [ ] Add to `hh/go.work` if using workspace mode
- [ ] Update hh-docs repository table in README
