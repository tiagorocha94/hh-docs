# hh-shared

Shared Go library providing cross-cutting utilities for all Household services. Not a deployable service — imported as a Go module dependency.

Repository: [github.com/tiagorocha94/hh-shared](https://github.com/tiagorocha94/hh-shared)

## Packages

| Package | Purpose |
|---|---|
| `apperr` | Typed application errors mapped to HTTP status codes |
| `config` | Standard service configuration loader (env vars) |
| `db` | PostgreSQL connection pool setup (`pgxpool`) |
| `health` | Liveness and readiness HTTP handlers |
| `httputil` | JSON response helpers, request parsing, pagination, Scalar API docs UI |
| `logger` | Structured `slog` logger factory |
| `middleware` | Request ID, CORS, request logging, JWT auth middleware (Chi-compatible) |
| `pgerr` | PostgreSQL error code mapping (unique violation, FK violation, etc.) |
| `reqctx` | Request-scoped identity (user/member/role) propagation via context |
| `seeds` | Dev seed SQL runner + shared member UUIDs for cross-service consistency |
| `server` | Production-ready HTTP server with graceful shutdown |
| `system` | Internal `/_system` routes: health, readiness, build info with uptime |
| `testutil` | Test helpers: Postgres testcontainer, JWT signing, response assertions |
| `validate` | Struct validation with custom tags (`hexcolor`, `safeemail`, `safetext`, `iso_month`, `iso_date`, `positive_decimal`, `decimal`) |

## Dev Seed Identities

All services share fixed member UUIDs so cross-service references resolve correctly. Defined in `seeds/identities.go`.

| Constant | UUID | Name |
|---|---|---|
| `MemberAliceID` | `a1a1a1a1-0000-0000-0000-000000000001` | Alice |
| `MemberBobID` | `b2b2b2b2-0000-0000-0000-000000000002` | Bob |
| `MemberCarlaID` | `c3c3c3c3-0000-0000-0000-000000000003` | Carla |
