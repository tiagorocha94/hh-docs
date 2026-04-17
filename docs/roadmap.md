# Roadmap & Backlog

Feature backlog for the Household platform, grouped by scope.

Priority: 🔴 High — next up, 🟡 Medium — important but not urgent, 🟢 Low — future consideration.

## General (cross-cutting)

### 🟢 Multi-currency support
All services currently assume EUR. Adding a `currency` column to monetary
tables would enable multi-currency tracking with EUR as the aggregation
currency.

### 🟢 Immediate token revocation
If admin force-logout becomes a requirement, add a Redis revocation list.
hh-identity writes revoked token hashes on logout; JWT middleware checks
Redis before accepting a token. Redis failure degrades gracefully to
signature-only verification.

### 🟢 Multi-tenancy
Currently single-household per deployment. Supporting multiple households
per instance would require a `household_id` column on most tables and
tenant-scoped queries.

### 🟢 Database-level row security
PostgreSQL Row-Level Security (RLS) policies could provide a second layer
of defense by automatically filtering rows based on session context. Good
foundation for multi-tenancy once implemented.

### 🟢 Bootstrap package for main.go
Every service duplicates ~90 lines of identical startup logic. A
`hh-shared/bootstrap` package could reduce each `main.go` to ~30 lines.
Waiting for services to mature further before extracting.

### 🟡 Email sending infrastructure
Several features depend on the ability to send emails (password reset,
invitations, notifications). Approach still under investigation — SMTP,
third-party API, or lightweight internal service.

### 🟢 Notification system
A background service that periodically checks for conditions and acts on
them — sending push notifications, triggering emails, updating badges.
Future use cases: budget alerts, goal deadline reminders, household
reminders.

## hh-identity

### 🟡 Password reset
Email-based password reset flow. Depends on email sending infrastructure.

### 🟡 Account lockout
Rate limiting on failed login attempts. Should also notify the user.
Depends on email sending infrastructure.

### 🟢 OAuth / social login
Support Google, GitHub, or other OAuth providers alongside email + password.

### 🟡 Invitations
Invite a person to join the household by email. Creates both the member
and user account in a single transaction. Depends on email infrastructure.

## hh-goals

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

### 🟢 Budget alerts
Warn when a group's actual spend exceeds its target percentage. More
relevant once a notification system exists.

## hh-shared

### 🟢 Configurable Postgres version in testutil
`StartPostgresContainer` hardcodes `postgres:18-alpine`. Make the image
configurable for testing against different Postgres versions.

## hh-dashboard (planned service)

### 🔴 Centralised read-optimised dashboards
A dedicated read service that consumes data from all other services and
serves pre-aggregated views: portfolio summaries, savings progress,
expense breakdowns, per-member overviews, and household-wide KPIs.
See [Dashboard Architecture](dashboard-architecture.md).

## hh-infra (planned)

### 🟡 Full-stack orchestration
Docker Compose setup that runs all services from published images with
nginx as the reverse proxy. Production-like deployment for personal use.
See the hh-infra plan in the workspace.

### 🟡 Full-stack integration testing
Compose all `-dev` images into a single docker-compose for automated
end-to-end testing of the full platform.

## hh-web

### 🟡 Dashboard views
Per-service dashboards (goals progress, investment portfolio summary,
expense breakdown) that consume the dashboard API. Depends on hh-dashboard.

### 🟢 Responsive design
Mobile-friendly layouts for on-the-go access to household finances.
