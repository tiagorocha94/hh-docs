# hh-identity

Authentication, household members, and member preferences.

Repository: [github.com/tiagorocha94/hh-identity](https://github.com/tiagorocha94/hh-identity)

Replaces the former hh-auth and hh-users services.

For what this service does and how it works, see [Features > Authentication](../features/auth.md) and [Features > Members](../features/members.md).

## Technical Details

- Database: `hh_identity`
- Does not require `JWKS_URL` (it is the JWT issuer)
- ES256 private key generated on first startup, stored in a Docker volume (`KEY_PATH`)
- Other services verify tokens via the `/v1/jwks` endpoint
- `users.member_id` is a real FK to `members` (not a soft reference)

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
| GET | `/v1/members` | Bearer | List members (paginated) |
| POST | `/v1/members` | Admin | Create member |
| GET | `/v1/members/{id}` | Bearer | Get member |
| PUT | `/v1/members/{id}` | Owner/Admin | Update member |
| DELETE | `/v1/members/{id}` | Admin | Delete member |
| GET | `/v1/members/{id}/preferences` | Owner/Admin | Get preferences |
| PUT | `/v1/members/{id}/preferences` | Owner/Admin | Update preferences |

### Service-Specific Config

| Variable | Required | Default | Description |
|---|---|---|---|
| `KEY_PATH` | No | `keys` | Directory for ES256 key pair storage |

### Schema

| Table | Key columns |
|-------|-------------|
| `members` | id, name |
| `users` | id, email, password_hash, role, member_id (FK → members) |
| `refresh_tokens` | id, user_id, token_hash, expires_at |
| `member_preferences` | member_id (FK → members), theme, notifications_enabled |
