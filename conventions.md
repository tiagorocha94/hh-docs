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
│   ├── openapi.yaml                # OpenAPI 3 spec (served at /docs)
│   └── service.md                  # Service overview
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

- `ci.yml`: fmt check, vet, go mod tidy, build, unit tests, integration tests on push/PR to main
- `ci.yml` uses `tags-ignore: ["v*"]` to skip tagged commits (avoids duplicate runs with cd.yml)
- `cd.yml`: same test suite + Docker build/push on `v*` tags (services only, not hh-shared)
- Tests run without `-race` (CGO not required by the codebase)
- SSH key for private hh-shared dependency

## Git Hooks

Run `make init` after cloning to set up git hooks:

- `pre-commit`: format check, go vet, go mod tidy validation
- `pre-push`: build, unit tests, integration tests

Hooks live in `.githooks/` and are configured via `git config core.hooksPath`.

## Local Setup

```bash
git clone <repo>
cd hh-<service>
make init    # sets up git hooks + copies .env.example to .env
make up      # starts postgres + auth-svc + app
```
