# Household Platform — Overview

The Household project is a personal household management platform. It helps
families track and organise their shared finances — members, expenses, savings
goals, and investments — through a set of independently deployed microservices.

## Architecture

The platform follows a polyrepo architecture: each service lives in its own
repository, owns its own database, and is deployed independently. A shared Go
library (`hh-shared`) provides common utilities across all services.

See [Architecture](architecture.md) for the tech stack, auth flow, and data
ownership model.
