# hh-auth

Authentication and identity provider.

Repository: [github.com/tiagorocha94/hh-auth](https://github.com/tiagorocha94/hh-auth)

For what this service does and how it works, see [Features > Authentication](../features/auth.md).

## Technical Details

- Database: `hh_auth`
- Does not require `JWKS_URL` (it is the JWT issuer)
- ES256 private key generated on first startup, stored in a Docker volume (`KEY_PATH`)
- Other services verify tokens via the `/v1/jwks` endpoint

### API Surface

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| POST | `/v1/login` | None | Authenticate with email + password |
| POST | `/v1/refresh` | Cookie | Refresh access token |
| POST | `/v1/logout` | Bearer | Invalidate refresh token |
| GET | `/v1/me` | Bearer | Current user info |
| GET | `/v1/jwks` | None | Public key for token verification |
| GET | `/v1/users` | Admin | List users |
| POST | `/v1/users` | Admin | Create user |
| PUT | `/v1/users/{id}` | Admin | Update user |
| DELETE | `/v1/users/{id}` | Admin | Delete user |

### Service-Specific Config

| Variable | Required | Default | Description |
|---|---|---|---|
| `KEY_PATH` | No | `keys` | Directory for ES256 key pair storage |

### Schema

| Table | Key columns |
|-------|-------------|
| `users` | id, email, password_hash, role, member_id |
| `refresh_tokens` | id, user_id, token_hash, expires_at, revoked_at |
