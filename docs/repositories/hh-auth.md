# hh-auth

The authentication and identity provider for the Household platform.

Repository: [github.com/tiagorocha94/hh-auth](https://github.com/tiagorocha94/hh-auth)

## Overview

hh-auth manages user accounts (email + password), issues signed JWTs, and serves the public key so other services can verify tokens locally. Every authenticated request in the platform flows through a JWT issued by this service. Other services never call hh-auth per-request — they verify tokens independently using the public key from the JWKS endpoint.

| Consumer | How |
|---|---|
| **Household members** | Log in with email and password to get a JWT |
| **Platform admin** | Create and manage user accounts, link users to members |
| **Other services** | Fetch the JWKS public key once on startup, verify JWTs locally |
| **Frontend** | Calls login/refresh/logout, attaches JWT to all API requests |

## Authentication

hh-auth is itself the authentication provider. It issues ES256-signed JWTs containing the user ID, member ID, and role. The `/v1/jwks` endpoint serves the public key for token verification. User management endpoints (`/v1/users`) are restricted to admin role.

## Core Concepts

### User
An authentication account. A user has:
- An **email** — used as the login identifier
- A **password** — stored as a bcrypt hash, never in plaintext
- A **role** — `member` or `admin`
- An optional **member_id** — links the auth account to a member in hh-users

### JWT (Access Token)
A short-lived ES256-signed token containing the user ID, member ID, and role. Sent as a Bearer token in the `Authorization` header.

### Refresh Token
A long-lived opaque token stored as an HTTP-only cookie. Used to obtain a new access token without re-entering credentials. Stored as a SHA-256 hash in the database. Rotated on every refresh.

### JWKS
The JSON Web Key Set endpoint (`/v1/jwks`) serves the ES256 public key. Other services fetch this once on startup and cache it to verify JWTs without calling hh-auth per-request.

## Key Behaviours

- Login validates credentials and returns an access token (JSON body) and a refresh token (HTTP-only cookie).
- Refresh rotates the refresh token — the old one is invalidated.
- Logout clears the refresh cookie and deletes the token from the database.
- The `/v1/me` endpoint returns the current user based on the JWT.
- User management (`/v1/users`) is admin-only.
- The ES256 private key is generated on first startup and stored as a file in a Docker volume. It never enters env vars, Git, or images.

## Limitations

- **Authorisation** — this service authenticates (who are you?) but does not enforce what you can do. Each service enforces its own authorisation rules using the identity from the JWT.
- **Password reset** — not implemented yet. Planned for a future phase.
- **OAuth / social login** — not in scope. Email + password only.
- **Session management** — no server-side sessions. JWTs are stateless.

## Future Directions

| Capability | Notes |
|---|---|
| Password reset | Email-based flow |
| Account lockout | Rate limiting on failed login attempts |
| Token revocation | Redis-based revocation list for immediate invalidation |
| OAuth providers | Google, GitHub login |
