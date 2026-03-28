# hh (Infra)

> **WIP** — This repo is being updated to reflect the polyrepo split.

The `hh` monorepo is the orchestration layer for the Household platform. It wires all independently deployed services together into a running stack.

## What it contains

- **Frontend** — React + Vite + Tailwind SPA (`fe/`)
- **Infrastructure** — Docker Compose, nginx reverse proxy, PostgreSQL init scripts (`infra/`)
- **Services being migrated** — investments service and expenses stub still live here (`services/`)

## How it works

- `docker-compose.yml` pulls service images from GHCR and runs them alongside postgres and nginx
- nginx routes `/auth/*`, `/users/*`, `/goals/*`, etc. to the correct upstream service
- `init.sql` creates all service databases on a single postgres instance for local dev
- The frontend talks to services through the nginx proxy

## Future

As services are extracted into their own repos (like hh-auth, hh-users, hh-goals), the monorepo shrinks. Eventually it will contain only the frontend and infrastructure configuration.
