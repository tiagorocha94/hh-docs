# CI/CD Patterns

All services follow a two-workflow pattern. Git hooks provide the same checks
locally — run `make init` after cloning to enable them.

## ci.yml — Continuous Integration

Triggers on push/PR to `main`. Skips tagged commits (`tags-ignore: ['v*']`)
to avoid running tests twice when pushing a commit and tag together.

Steps:

1. Checkout + Go setup
2. SSH key configuration (for private hh-shared dependency)
3. Format check (`gofmt -l .`)
4. `go vet ./...`
5. `go mod tidy` check (ensures go.mod/go.sum are clean)
6. `go build ./...`
7. Unit tests (`go test ./... -count=1`)
8. Integration tests (`go test ./... -tags=integration -count=1`)

Tests run without `-race` (CGO is not required by the codebase).

## cd.yml — Continuous Delivery

Triggers on `v*` tags. Two jobs in sequence:

### 1. test

Runs the full CI suite (same steps as ci.yml) to ensure the tagged commit is clean.

### 2. build-and-push

- Logs into GHCR
- Builds Docker image with version/commit/build-time ldflags
- Pushes `ghcr.io/tiagorocha94/hh-<name>:<tag>` and `:latest`

> **Note:** `cd.yml` only applies to service repos (hh-auth, hh-users, hh-goals, etc.). hh-shared is a library and does not produce Docker images — it only has `ci.yml`.

## Secrets

| Secret | Purpose |
|---|---|
| `SSH_PRIVATE_KEY` | Deploy key for accessing private hh-shared module |
| `GITHUB_TOKEN` | Auto-provided, used for GHCR push |

## Release Process

1. Commit changes
2. Tag: `git tag vX.Y.Z`
3. Push: `git push origin main --tags`
4. CD pipeline runs automatically
5. Update dependent services' `go.mod` if hh-shared was bumped
