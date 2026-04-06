# Household Platform — Documentation

Platform-wide documentation for the Household project. Service-specific docs
live in each service repo; this repo covers cross-cutting conventions,
architecture, and onboarding.

## Contents

| Document | Description |
|---|---|
| [Architecture](docs/architecture.md) | Service map, data flow, tech stack |
| [Conventions](docs/conventions.md) | Shared coding, testing, and API conventions |
| [New Service Checklist](docs/new-service-checklist.md) | Step-by-step guide for adding a service |

## Repositories

| Repo | Purpose | Document | Latest |
|---|---|---|---|
| [hh-shared](https://github.com/tiagorocha94/hh-shared) | Shared Go library | [hh-shared](docs/repositories/hh-shared.md) | v0.10.0 |
| [hh-auth](https://github.com/tiagorocha94/hh-auth) | Authentication service (JWT, JWKS) | [hh-auth](docs/repositories/hh-auth.md) | v0.2.0 |
| [hh-users](https://github.com/tiagorocha94/hh-users) | Member management | [hh-users](docs/repositories/hh-users.md) | v0.4.0 |
| [hh-goals](https://github.com/tiagorocha94/hh-goals) | Savings goals, accounts, allocations | [hh-goals](docs/repositories/hh-goals.md) | v0.2.0 |
| [hh](https://github.com/tiagorocha94/hh) | Monorepo (FE, infra, investments) | — | — |
