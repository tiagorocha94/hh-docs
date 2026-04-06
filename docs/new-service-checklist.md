# New Service Checklist

Steps for adding a new hh-* service to the platform.

## 1. GitHub Repository

- [ ] Create repo `hh-<name>` on GitHub (private)
- [ ] Initialize with README (will be replaced)
- [ ] Clone locally

### SSH Key for hh-shared Access

Each service repo needs an SSH key pair so CI/CD can fetch the private
`hh-shared` module.

```bash
# Generate key pair (no passphrase)
ssh-keygen -t ed25519 -C "hh-<name>-ci" -f hh-<name>-ci -N ""
```

This creates two files: `hh-<name>-ci` (private) and `hh-<name>-ci.pub` (public).

**Add public key to hh-shared:**

1. Go to [hh-shared](https://github.com/tiagorocha94/hh-shared) on GitHub
2. Settings → Deploy keys → Add deploy key
3. Title: `hh-<name>-ci`
4. Paste contents of `hh-<name>-ci.pub`
5. Check "Allow write access" (read-only is also fine)
6. Save

**Add private key to hh-\<name\>:**

1. Go to `hh-<name>` repo on GitHub
2. Settings → Secrets and variables → Actions
3. New repository secret
4. Name: `SSH_PRIVATE_KEY`
5. Value: paste contents of `hh-<name>-ci` (the private key file)
6. Save

**Clean up local key files:**

```bash
rm hh-<name>-ci hh-<name>-ci.pub
```

GHCR permissions are handled automatically by `GITHUB_TOKEN` — no extra
configuration needed.

## 2. Project Scaffold

- [ ] Initialize Go module: `go mod init github.com/tiagorocha94/hh-<name>`
- [ ] Add `hh-shared` dependency: `go get github.com/tiagorocha94/hh-shared@latest`
- [ ] Create project structure (see [conventions](conventions.md#project-structure))
- [ ] Add `.gitignore`, `.gitattributes` (`* text=auto eol=lf`), `.dockerignore`

## 3. Application Code

- [ ] `cmd/main.go` — config, logger, DB, migrations, seeds, wiring, server
- [ ] Validate config at startup: check `JWKS_URL` is not empty (fail-fast)
- [ ] `internal/handler/router.go` — apply `middleware.JWTAuth(ctx, jwksURL)` to `/v1`
- [ ] Mount `/_system` routes via `system.Routes(pool, buildInfo)`
- [ ] Serve OpenAPI docs at `/docs` and `/openapi.yaml`

## 4. Database

- [ ] Create `migrations/` directory with goose SQL migrations
- [ ] Create `dev/seeds/` with idempotent dev seed SQL
- [ ] Use shared member UUIDs from `hh-shared/seeds/identities.go`

## 5. Docker

- [ ] `Dockerfile` — multi-stage, SSH forwarding, HEALTHCHECK, ldflags
- [ ] `dev/docker-compose.yml` — postgres + auth-svc + app
- [ ] `dev/init.sql` — create service DB + hh_auth DB on same postgres
- [ ] `dev/Dockerfile` — dev image (production image + seeds)
- [ ] `.env.example` — document all env vars including `JWKS_URL`

## 6. Documentation

- [ ] `docs/openapi.yaml` — full API spec with `securitySchemes`, 401/403 responses
- [ ] `docs/schema.md` — schema and design decisions
- [ ] `README.md` — version, quick start, link to hh-docs
- [ ] Add feature page to hh-docs (`features/<name>.md`)
- [ ] Add repository page to hh-docs (`repositories/hh-<name>.md`)

## 7. CI/CD

- [ ] `.github/workflows/ci.yml` — vet, mod tidy, build, unit tests, integration tests
- [ ] `.github/workflows/cd.yml` — full tests + Docker push (production + dev images)
- [ ] `.githooks/pre-commit` — vet, go mod tidy check
- [ ] `.githooks/pre-push` — build, unit tests, integration tests

## 8. Makefile

Standard targets: `help`, `init`, `up`, `down`, `logs`, `ps`, `migrate-up`, `migrate-down`, `migrate-status`

## 9. Integration

- [ ] Add service to `hh-infra/docker-compose.yml` (GHCR image)
- [ ] Add nginx upstream and location block
- [ ] Add database to `hh-infra/postgres/init.sql`
- [ ] Update hh-docs architecture page (repo table, dependency diagram)
- [ ] Update hh-docs nav (mkdocs.yml)
