# Roadmap & Backlog

Feature backlog for the Household platform, grouped by scope.

## General (cross-cutting)

### Multi-currency support
All services currently assume EUR. Adding a `currency` column to monetary
tables (contributions, valuations, movements, transactions) would enable
multi-currency tracking with EUR as the aggregation currency.

### Pagination UI
Backend pagination is implemented (via hh-shared/httputil) but the frontend
fetches all records. Add pagination controls when data volume grows.

### Immediate token revocation
If admin force-logout becomes a requirement, add a Redis revocation list.
hh-auth writes revoked token hashes on logout; JWT middleware checks Redis
before accepting a token. Redis failure degrades gracefully to
signature-only verification. No architectural changes needed.

### Cross-service member consistency
Services reference member IDs as plain UUIDs with no cross-database FK
constraints. If a member is deleted from hh-users, other services still
hold stale references. Options: shared identity service, event-driven sync,
or a periodic consistency check.

### Multi-tenancy
Currently single-household per deployment. Supporting multiple households
per instance would require a `household_id` column on most tables and
tenant-scoped queries.

### Database-level row security
Authorization is currently enforced at the service level. PostgreSQL
row-level security policies could provide a second layer of defense.

### Bootstrap package for main.go
Every service duplicates ~90 lines of identical startup logic (config, logger,
DB, migrations, seeds, signal handling, graceful shutdown). A
`hh-shared/bootstrap` package could reduce each `main.go` to ~30 lines of
service-specific wiring. Affects all 4 services.

### Config validation
Services validate `JWKS_URL` with a manual check in main.go. As more
required config fields are added, consider adding a `config.Validate()`
method or required field tags to `config.Config`.

## hh-auth

### Password reset
Email-based password reset flow. Requires an email sending capability
(SMTP or third-party service).

### Account lockout
Rate limiting on failed login attempts to prevent brute-force attacks.

### OAuth / social login
Support Google, GitHub, or other OAuth providers alongside email + password.

## hh-users

### Invitations
Invite a person to join the household by email.

### Profile photos
Support uploaded avatars in addition to the icon-based system.

### Per-member settings
Timezone, currency preference, notification preferences.

### Cross-service cascade on deletion
Propagate member deletion to dependent services (goals, investments,
expenses) — currently, deleting a member leaves orphaned references.

## hh-goals

### Per-member goals
Goals are currently household-wide. A `goal_members` join table would allow
assigning goals to specific members.

### Expense automation
When the expenses service is implemented, an expense category flagged as
"savings" could automatically create a movement in hh-goals.

## hh-investments

### Outflows / withdrawals
Record money taken out of an instrument. Needed for accurate gain/loss
calculation.

### Joint instruments
Allow multiple member_ids per instrument for shared investment accounts.

### Dashboard / summary endpoint
A pre-aggregated portfolio summary endpoint (total invested, portfolio value,
breakdowns by type/entity/member, time series). Was removed during migration
to keep the service CRUD-only; can be re-added when the frontend needs it.

## hh-expenses (planned service)

### Core implementation
Build the expenses service: categories, groups, transactions, summaries.
This is the next service to be extracted from the monorepo stub.

### Recurring transactions
Support for recurring income/expenses (monthly salary, subscriptions) that
auto-generate transaction records.

### Budget tracking
Set monthly budgets per category and track spending against them.

## hh-shared

### Multi-dependency readiness checks
The `health.Readiness` handler currently accepts a single `Pinger`. Evolve
to accept multiple dependencies (Postgres, Redis, external APIs) and report
individual status per dependency.

### Package documentation and examples
Add `example_test.go` files to key packages (httputil, config, health,
apperr, validate, server) so usage examples show up in `go doc` and are
tested automatically.

### Configurable Postgres version in testutil
`StartPostgresContainer` hardcodes `postgres:18-alpine`. Make the image
configurable via parameter or environment variable for testing against
different Postgres versions.

## hh-infra

### Rename hh to hh-infra
The orchestration repo is currently named `hh`. It should be renamed to
`hh-infra` once all services are extracted, to match the naming convention.

### Full-stack integration testing
Compose all `-dev` images into a single docker-compose for automated
end-to-end testing of the full platform.

## hh-web (planned)

### Extract frontend to own repo
The React SPA currently lives inside the `hh` (infra) repo as `fe/`. It
should be extracted to its own `hh-web` repository with independent CI/CD,
matching the polyrepo pattern.

### Authentication UI
Login page, token refresh handling, logout flow. Currently partially
implemented in the monorepo frontend.

### Dashboard views
Per-service dashboards (goals progress, investment portfolio summary,
expense breakdown) that consume the CRUD APIs.

### Responsive design
Mobile-friendly layouts for on-the-go access to household finances.

