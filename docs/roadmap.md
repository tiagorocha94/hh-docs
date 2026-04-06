# Roadmap & Backlog

Feature backlog for the Household platform, grouped by scope.

Priority: 🔴 High — next up, 🟡 Medium — important but not urgent, 🟢 Low — future consideration.

## General (cross-cutting)

### 🟢 Multi-currency support
All services currently assume EUR. Adding a `currency` column to monetary
tables (contributions, valuations, movements, transactions) would enable
multi-currency tracking with EUR as the aggregation currency.

### 🟡 Pagination UI
Backend pagination is implemented (via hh-shared/httputil) but the frontend
fetches all records. Add pagination controls when data volume grows.
Depends on hh-web being implemented first.

### 🟢 Immediate token revocation
If admin force-logout becomes a requirement, add a Redis revocation list.
hh-auth writes revoked token hashes on logout; JWT middleware checks Redis
before accepting a token. Redis failure degrades gracefully to
signature-only verification. No architectural changes needed.

### 🟢 Cross-service member consistency
Services reference member IDs as plain UUIDs with no cross-database FK
constraints. If a member is deleted from hh-users, other services still
hold stale references. Options: shared identity service, event-driven sync,
or a periodic consistency check. Not needed now — members won't be deleted
in the immediate use case.

### 🟢 Multi-tenancy
Currently single-household per deployment. Supporting multiple households
per instance would require a `household_id` column on most tables and
tenant-scoped queries.

### 🟢 Database-level row security
Authorization is currently enforced at the service level. PostgreSQL
Row-Level Security (RLS) policies could provide a second layer of defense
by automatically filtering rows based on the current session context.

For example, a policy on the `instruments` table could enforce
`member_id = current_setting('app.member_id')` so even if application code
has a bug that skips the WHERE clause, the database itself prevents data
leakage across members. This would also be a good foundation for
multi-tenancy (filtering by `household_id`) once that's implemented.

### 🟢 Bootstrap package for main.go
Every service duplicates ~90 lines of identical startup logic (config, logger,
DB, migrations, seeds, signal handling, graceful shutdown). A
`hh-shared/bootstrap` package could reduce each `main.go` to ~30 lines of
service-specific wiring. Waiting for services and shared lib to mature
further before extracting this pattern.

### 🟡 Config validation
Services validate `JWKS_URL` with a manual check in main.go. A better
approach: each service defines its own config struct using composition
with `config.Config` as the base, adds service-specific fields with
validate tags, and a generic `config.LoadAndValidate[T]()` function
handles both loading from env vars and struct validation in one call.

Example:
```go
type ServiceConfig struct {
    config.Config
    JWKSURL string `env:"JWKS_URL" validate:"required,url"`
}

cfg, err := config.LoadAndValidate[ServiceConfig]()
```

### 🟡 Email sending infrastructure
Several features depend on the ability to send emails (password reset,
account lockout notifications, invitations). The approach is still under
investigation — could be SMTP, a third-party API (SendGrid, SES), or a
lightweight internal service. This should be resolved before implementing
any email-dependent features.

### 🟢 Notification system
A background service that periodically checks for conditions and acts on
them — sending push notifications, triggering emails, updating badges, or
running scheduled tasks. Could function as a cron-like service that
listens for changes or runs on a schedule. Future use cases: budget
alerts, goal deadline reminders, household reminders.

## hh-auth

### 🟡 Password reset
Email-based password reset flow. Depends on email sending infrastructure
being resolved first.

### 🟡 Account lockout
Rate limiting on failed login attempts to prevent brute-force attacks.
Should also send an email to the user reporting the lockout. Depends on
email sending infrastructure.

### 🟢 OAuth / social login
Support Google, GitHub, or other OAuth providers alongside email + password.

## hh-users

### 🟡 Invitations
Invite a person to join the household by email. Depends on email sending
infrastructure being resolved first.

### 🟢 Per-member settings
Timezone, currency preference, notification preferences. More relevant
once a notification system exists.

### 🟢 Cross-service cascade on deletion
Propagate member deletion to dependent services (goals, investments,
finances) — currently, deleting a member leaves orphaned references.
Not needed now — members won't be deleted in the immediate use case.

## hh-goals

### 🟡 Per-member goals
Goals are currently household-wide. A `goal_members` join table would allow
assigning goals to specific members.

### 🟢 Expense automation
When the finances service matures, an expense category flagged as
"savings" could automatically create a movement in hh-goals. Cross-service
automation is a future concern.

## hh-investments

### 🟡 Outflows / withdrawals
Record money taken out of an instrument. Needed for accurate gain/loss
calculation.

### 🟡 Joint instruments
Allow multiple member_ids per instrument for shared investment accounts.

## hh-finances

### 🟢 Recurring transaction detection
Flag income/expenses that repeat monthly and aggregate separately.
Automation should be done after core flows are consistent.

### 🟢 Budget alerts
Warn when a group's actual spend exceeds its target percentage. More
relevant once a notification system exists.

## hh-shared

### 🟢 Configurable Postgres version in testutil
`StartPostgresContainer` hardcodes `postgres:18-alpine`. Make the image
configurable via parameter or environment variable for testing against
different Postgres versions.

## hh-dashboard (planned service)

### 🔴 Centralised read-optimised dashboards
A dedicated read service that consumes data from all other services and
serves pre-aggregated views: portfolio summaries, savings progress,
expense breakdowns, per-member overviews, and household-wide KPIs.
One of the most impactful features but brings significant complexity —
changes across all services (event publishing) and new infrastructure
(NATS). See [Dashboard Architecture](dashboard-architecture.md).

## hh-infra

### 🟡 Rename hh to hh-infra
The orchestration repo is currently named `hh`. It should be renamed to
`hh-infra` once all services are extracted. This involves more than just
renaming — the frontend needs to be extracted first, nginx config updated,
and docker-compose references cleaned up.

### 🟡 Full-stack integration testing
Compose all `-dev` images into a single docker-compose for automated
end-to-end testing of the full platform. Best done once services and
frontend are stable.

## hh-web (planned)

### 🔴 Extract frontend to own repo
The React SPA currently lives inside the `hh` (infra) repo as `fe/`. It
should be extracted to its own `hh-web` repository with independent CI/CD,
matching the polyrepo pattern. The repo should include a way to run the
app against mocked APIs so local development and implementation doesn't
depend on running the actual backend services. In the future, a
docker-compose setup could start all services in dev mode for full
integration testing, but that's heavy for now.

### 🔴 Authentication UI
Login page, token refresh handling, logout flow. Critical — should be
implemented immediately after the frontend is extracted to its own repo.

### 🟡 Dashboard views
Per-service dashboards (goals progress, investment portfolio summary,
expense breakdown) that consume the dashboard API. Can only be done once
hh-dashboard is implemented.

### 🟢 Responsive design
Mobile-friendly layouts for on-the-go access to household finances.
