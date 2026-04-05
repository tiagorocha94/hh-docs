# hh-goals

Envelope budgeting for household savings — every saved euro gets a job.

Repository: [github.com/tiagorocha94/hh-goals](https://github.com/tiagorocha94/hh-goals)

## Overview

hh-goals applies envelope budgeting to household savings. Instead of tracking money passively, it gives every saved euro a job: each deposit is automatically distributed across active saving goals, prioritised by their deadline.

Many households save vaguely without a clear destination. By dividing savings into named, targeted envelopes — holidays, a car, an emergency fund — it becomes much easier to see progress, stay motivated, and make trade-offs when budgets are tight.

## Authentication

All `/v1` endpoints require a valid JWT bearer token issued by [hh-auth](hh-auth.md). The service validates tokens using a JWKS endpoint configured via the `JWKS_URL` environment variable. Requests without a valid token receive a `401 Unauthorized` response.

## Core Concepts

### Accounts
Real bank or savings accounts. Balances are never stored; they are always derived by summing all movements. This ensures the balance is always accurate and consistent.

### Movements
Every real money event: a monthly transfer in (positive) or a withdrawal (negative). Deposits trigger the auto-distribution algorithm. Movements are immutable — delete to correct.

### Goals
Named saving envelopes. Each goal has a color, icon, budget, and target date. Goals are household-wide (no member-level isolation in v1). Status transitions: `planned` (start date in the future) → `active` (start date reached) → `completed` or `cancelled`.

### Versions
Every time a goal's budget or target date changes, a new version is created and a new set of planned allocations is generated. Past months are frozen under the old version. This preserves history and makes it easy to see how a plan evolved over time.

### Planned Allocations
Auto-generated monthly instalments for a version (budget ÷ months, last month absorbs rounding). Never edited directly.

### Actual Allocations
One row per (goal, year, month). Primarily created by auto-distribution when a deposit is recorded. Can be manually adjusted via the API. The gap between planned and actual drives the "met / partial / missed" status shown in the UI.

### Expenses
Spending from a goal's virtual envelope. Reduces the goal balance without touching any real account. Immutable once created.

## Key Behaviours

### Auto-distribution algorithm
When a deposit is recorded:

1. Compute household unallocated balance = SUM(all movements) − SUM(all allocations)
2. Sort active goals by target_date ASC (urgency), then name ASC to break ties
3. For each goal: needed = planned_this_month − already_allocated_this_month; allocate MIN(needed, available)
4. Return the movement, the list of distributions, and the remaining unallocated amount

### Goal completion
Marking a goal as `completed` is a manual action (PATCH status). Optionally, an account can be specified to receive the surplus (goal balance − expenses). If provided, a withdrawal movement is created on that account returning the surplus.

## Limitations

- **Per-member goals** — goals are household-wide in v1; no member-level isolation.
- **Expense automation** — no automatic link to an expenses service yet.

## Future Directions

| Capability | Notes |
|---|---|
| Per-member goals | A `goal_members` join table will allow assigning goals to specific members |
| Expense automation | When the expenses service is implemented, an expense category flagged as "savings" can automatically create a movement here |
