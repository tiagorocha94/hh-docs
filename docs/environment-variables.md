# Environment Variables

All services use `config.Load()` from hh-shared which reads these common variables:

| Variable | Required | Default | Description |
|---|---|---|---|
| `DB_DSN` | Yes | — | PostgreSQL connection string |
| `PORT` | No | `8080` | HTTP server port |
| `ENV` | No | `development` | Runtime environment (`development` or `production`) |
| `LOG_LEVEL` | No | `info` | Minimum log level (`debug`, `info`, `warn`, `error`) |
| `JWKS_URL` | Yes* | — | JWKS endpoint URL for JWT validation |

*`JWKS_URL` is required by all services except hh-auth (which is the JWT issuer).

## Service-Specific Variables

### hh-auth

| Variable | Required | Default | Description |
|---|---|---|---|
| `KEY_PATH` | No | `keys` | Directory for ES256 key pair storage |

## Local Development Values

For `docker-compose.yml` (inter-container networking):

```
DB_DSN=postgres://<svc>_user:<svc>_pass@postgres:5432/<svc>?sslmode=disable
JWKS_URL=http://auth-svc:8080/v1/jwks
```

For `.env` (host-to-container):

```
DB_DSN=postgres://<svc>_user:<svc>_pass@localhost:5432/<svc>?sslmode=disable
JWKS_URL=http://localhost:8081/v1/jwks
```
