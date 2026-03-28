# Household Platform — Documentation

Platform-wide documentation for the Household project. Service-specific docs
live in each service repo; this repo covers cross-cutting conventions,
architecture, and onboarding.

## Contents

| Document | Description |
|---|---|
| [Architecture](architecture.md) | Service map, data flow, tech stack |
| [Conventions](conventions.md) | Shared coding, testing, and API conventions |
| [New Service Checklist](new-service-checklist.md) | Step-by-step guide for adding a service |
| [Environment Variables](environment-variables.md) | All env vars across all services |
| [Seed Data](seed-data.md) | Shared dev seed identities and conventions |
| [CI/CD](ci-cd.md) | Pipeline patterns and delivery workflow |

## Repositories

| Repo | Purpose | Latest |
|---|---|---|
| [hh-shared](https://github.com/tiagorocha94/hh-shared) | Shared Go library | v0.10.0 |
| [hh-auth](https://github.com/tiagorocha94/hh-auth) | Authentication service (JWT, JWKS) | v0.2.0 |
| [hh-users](https://github.com/tiagorocha94/hh-users) | Member management | v0.4.0 |
| [hh-goals](https://github.com/tiagorocha94/hh-goals) | Savings goals, accounts, allocations | v0.2.0 |
| [hh](https://github.com/tiagorocha94/hh) | Monorepo (FE, infra, investments) | — |
