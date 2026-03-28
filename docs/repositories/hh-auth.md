# Auth Service

> This service is independently versioned and deployed. It depends on
> [`hh-shared`](https://github.com/tiagorocha94/hh-shared) for common
> utilities and is orchestrated by [`hh`](https://github.com/tiagorocha94/hh).

## What this service does

The Auth service is the authentication and identity provider for the Household
platform. It manages user accounts (email + password), issues signed JWTs, and
serves the public key so other services can verify tokens locally.

Every authenticated request in the platform flows through a JWT issued by this
service. Other services never call the auth service per-request — they verify
tokens independently using the public key from the JWKS endpoint.

---

## Who uses this service

| Consumer | How |
|---|---|
| **Household members** | Log in with email and password to get a JWT |
| **Platform admin** | Create and manage user accounts, link users to members |
| **Other services** | Fetch the JWKS public key once on startup, verify JWTs locally |
| **Frontend** | Calls login/refresh/logout, attaches JWT to all API requests |

---

## Core concepts

### User
An authentication account. A user has:
- An **email** — used as the login identifier
- A **password** — stored as a bcrypt hash, never in plaintext
- A **role** — `member` or `admin`
- An optional **member_id** — links the auth account to a member in `hh-users`

### JWT (Access Token)
A short-lived (1 hour) ES256-signed token containing the user ID, member ID,
and role. Sent as a Bearer token in the `Authorization` header.

### Refresh Token
A long-lived (7 day) opaque token stored as an HTTP-only cookie. Used to
obtain a new access token without re-entering credentials. Stored as a
SHA-256 hash in the database. Rotated on every refresh.

### JWKS
The JSON Web Key Set endpoint (`/v1/jwks`) serves the ES256 public key.
Other services fetch this once on startup and cache it to verify JWTs
without calling the auth service per-request.

---

## Key behaviours

- Login validates credentials and returns an access token (JSON body) and a
  refresh token (HTTP-only cookie).
- Refresh rotates the refresh token — the old one is invalidated.
- Logout clears the refresh cookie and deletes the token from the database.
- The `/v1/me` endpoint returns the current user based on the JWT.
- User management (`/v1/users`) is admin-only.
- The ES256 private key is generated on first startup and stored as a file
  in a Docker volume. It never enters env vars, Git, or images.

---

## What this service does not do

- **Authorisation** — this service authenticates (who are you?) but does not
  enforce what you can do. Each service enforces its own authorisation rules
  using the identity from the JWT.
- **Password reset** — not implemented yet. Planned for a future phase.
- **OAuth / social login** — not in scope. Email + password only.
- **Session management** — no server-side sessions. JWTs are stateless.

---

## Future directions

| Capability | Notes |
|---|---|
| Password reset | Email-based flow |
| Account lockout | Rate limiting on failed login attempts |
| Token revocation | Redis-based revocation list for immediate invalidation |
| OAuth providers | Google, GitHub login |
| Multi-household | A user could belong to multiple households |
