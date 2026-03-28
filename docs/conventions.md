# Shared Conventions

Applies to all hh-* Go services. This is the single source of truth for
cross-cutting conventions.

## Project Structure

```
hh-<service>/
├── .github/
│   └── workflows/
│       ├── cd.yml                  # CD pipeline (v* tags) — services only, not hh-shared
│       └── ci.yml                  # CI pipeline (push/PR to main)
├── .githooks/
│   ├── pre-commit                  # fmt, vet, go mod tidy
│   └── pre-push                    # build, unit tests, integration tests
├── cmd/
│   └── main.go                     # Wiring only, no business logic
├── docs/
│   ├── database.md                 # Schema, design decisions
│   └── openapi.yaml                # OpenAPI 3 spec (served at /docs)
├── internal/
│   ├── domain/                     # Entities + repository interfaces (no DB imports)
│   ├── handler/                    # HTTP handlers + router (defines own service interface)
│   ├── repository/                 # pgx implementations of domain interfaces
│   └── service/                    # Business logic (depends on interfaces only)
├── migrations/                     # Goose SQL migrations
├── seeds/
│   └── dev/                        # Dev seed SQL (idempotent, ON CONFLICT DO NOTHING)
├── .env.example                    # Environment variable template
├── .gitattributes
├── .gitignore
├── docker-compose.yml              # Local dev: postgres + auth-svc + app
├── Dockerfile                      # Multi-stage build with HEALTHCHECK
├── go.mod
├── go.sum
├── init.sql                        # DB + user creation (first boot)
├── Makefile                        # init, up, down, logs, ps, migrate-*
└── README.md                       # Version, quick start, link to hh-docs
```

## Local Setup

```bash
git clone <repo>
cd hh-<service>
make init    # sets up git hooks + copies .env.example to .env
make up      # starts postgres + auth-svc + app
```

## Git Hooks

Run `make init` after cloning to set up git hooks:

- `pre-commit`: format check, go vet, go mod tidy validation
- `pre-push`: build, unit tests, integration tests

Hooks live in `.githooks/` and are configured via `git config core.hooksPath`.

## Error Handling

- Domain errors: `apperr.NotFound()`, `apperr.Conflict()`, `apperr.BadRequest()`, etc.
- DB errors: `pgerr.HandleErr()` maps constraint violations to apperr types
- Handlers: `httputil.Error(w, err)` maps apperr to HTTP status codes
- Never return raw errors to clients

## HTTP Handlers

- `httputil.DecodeJSON(r, &input)` for request bodies
- `httputil.OK`, `httputil.Created`, `httputil.NoContent` for responses
- `httputil.PathUUID(r, "id")` for UUID path params
- `httputil.QueryInt(r, "page", 1)` for query params
- `httputil.ParsePage(r)` for pagination

## Input Validation

- `validate.Validator` created once in main.go, injected into services
- Services call `v.Struct(input)` — never validate in handlers
- Tags: `safetext`, `safeemail`, `hexcolor`, `iso_date`, `iso_month`, `positive_decimal`, `decimal`

## Authentication

- All `/v1` routes use `middleware.JWTAuth(ctx, jwksURL)` from hh-shared
- `JWKS_URL` env var is required — service fails to start if empty
- `reqctx.Identity` carries UserID, MemberID, Role through context
- `middleware.AdminOnly` for admin-restricted endpoints
- `reqctx.CanWrite(ctx, resourceMemberID)` for member-scoped writes

## Testing

- External test package (`package handler_test`)
- `t.Parallel()` on every test (except integration tests sharing state)
- `testify/assert` for checks, `testify/require` when failure should stop
- `testutil.JWTHelper` from hh-shared for JWT test infrastructure
- `testutil.BearerRequest` / `testutil.BearerJSONRequest` for authenticated requests
- `testutil.StartPostgresContainer` for integration tests
- `//go:build integration` tag for Docker-dependent tests

## OpenAPI

- Every HTTP status the API can return must be documented
- `securitySchemes` with `bearerAuth` JWT on all protected services
- `401` on all `/v1` endpoints, `403` where authorization applies
- `/_system` endpoints are internal (blocked by reverse proxy) and not documented in the spec
- OpenAPI `version` field must match the current git tag

## Docker

- Multi-stage Dockerfile: alpine builder + alpine runtime
- `HEALTHCHECK` in Dockerfile (not in docker-compose)
- Build args: `VERSION`, `COMMIT`, `BUILD_TIME` for ldflags
- Local docker-compose includes auth-svc for JWT validation


## CI/CD

All services follow a two-workflow pattern. Git hooks provide the same checks
locally — run `make init` after cloning to enable them.

### ci.yml — Continuous Integration

Triggers on push/PR to `main`. Skips tagged commits (`tags-ignore: ['v*']`)
to avoid running tests twice when pushing a commit and tag together.

Steps:

1. Checkout + Go setup
2. SSH key configuration (for private hh-shared dependency)
3. Format check (`gofmt -l .`)
4. `go vet ./...`
5. `go mod tidy` check (ensures go.mod/go.sum are clean)
6. `go build ./...`
7. Unit tests (`go test ./... -count=1`)
8. Integration tests (`go test ./... -tags=integration -count=1`)

Tests run without `-race` (CGO is not required by the codebase).

### cd.yml — Continuous Delivery

Triggers on `v*` tags. Two jobs in sequence:

**1. test** — Runs the full CI suite (same steps as ci.yml) to ensure the tagged commit is clean.

**2. build-and-push:**

- Logs into GHCR
- Builds Docker image with version/commit/build-time ldflags
- Pushes `ghcr.io/tiagorocha94/hh-<name>:<tag>` and `:latest`

> **Note:** `cd.yml` only applies to service repos (hh-auth, hh-users, hh-goals, etc.). hh-shared is a library and does not produce Docker images — it only has `ci.yml`.

### Secrets

| Secret | Purpose |
|---|---|
| `SSH_PRIVATE_KEY` | Deploy key for accessing private hh-shared module |
| `GITHUB_TOKEN` | Auto-provided, used for GHCR push |

### Release Process

1. Commit changes
2. Tag: `git tag vX.Y.Z`
3. Push: `git push origin main --tags`
4. CD pipeline runs automatically
5. Update dependent services' `go.mod` if hh-shared was bumped

## Environment Variables

All services use `config.Load()` from hh-shared which reads these common variables:

| Variable | Required | Default | Description |
|---|---|---|---|
| `DB_DSN` | Yes | — | PostgreSQL connection string |
| `PORT` | No | `8080` | HTTP server port |
| `ENV` | No | `development` | Runtime environment (`development` or `production`) |
| `LOG_LEVEL` | No | `info` | Minimum log level (`debug`, `info`, `warn`, `error`) |
| `JWKS_URL` | Yes* | — | JWKS endpoint URL for JWT validation |

*`JWKS_URL` is required by all services except hh-auth (which is the JWT issuer).

### Service-Specific Variables

#### hh-auth

| Variable | Required | Default | Description |
|---|---|---|---|
| `KEY_PATH` | No | `keys` | Directory for ES256 key pair storage |

### Local Development Values

For `docker-compose.yml` (inter-container networking):

```
DB_DSN=postgres://<svc>_user:<svc>_pass@postgres:5432/<svc>?sslmode=disable
JWKS_URL=http://auth-svc:8080/v1/jwks
```

For `.env` (host-to-container):

```
DB_DSN=postgres://<svc>_user:<svc>_pass@localhost:5432/<svc>?sslmode=disable
JWKS_URL=http://localhost:8081/v1/jwks
```

## Seed Data

Dev seeds run automatically when `ENV=development`. They use `ON CONFLICT DO NOTHING` for idempotency.

### Shared Member Identities

All services share fixed member UUIDs so cross-service references resolve correctly.
Defined in `hh-shared/seeds/identities.go` and mirrored in `hh/fe/src/lib/seeds.ts`.

| Constant | UUID | Name | Role |
|---|---|---|---|
| `MemberAliceID` | `a1a1a1a1-0000-0000-0000-000000000001` | Alice | member |
| `MemberBobID` | `b2b2b2b2-0000-0000-0000-000000000002` | Bob | member |
| `MemberCarlaID` | `c3c3c3c3-0000-0000-0000-000000000003` | Carla | member |

hh-auth also seeds an admin user (not member-linked):
- Email: `admin@household.dev`, Role: `admin`

### Adding a New Member

1. Add UUID constant to `hh-shared/seeds/identities.go`
2. Add user row to `hh-auth/seeds/dev/001_users.sql` (with bcrypt hash)
3. Add member row to `hh-users/seeds/dev/001_members.sql`
4. Update `hh/fe/src/lib/seeds.ts` (frontend mirror)
5. Add seed data to any service that references members (investments entities, etc.)
6. Bump hh-shared version and update downstream services

### Seed File Naming

Use numbered prefixes for execution order: `001_users.sql`, `002_goals.sql`, etc.
The `seeds.Run()` function from hh-shared executes files in alphabetical order.
