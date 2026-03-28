# Seed Data Conventions

Dev seeds run automatically when `ENV=development`. They use `ON CONFLICT DO NOTHING` for idempotency.

## Shared Member Identities

All services share fixed member UUIDs so cross-service references resolve correctly.
Defined in `hh-shared/seeds/identities.go` and mirrored in `hh/fe/src/lib/seeds.ts`.

| Constant | UUID | Name | Role |
|---|---|---|---|
| `MemberAliceID` | `a1a1a1a1-0000-0000-0000-000000000001` | Alice | member |
| `MemberBobID` | `b2b2b2b2-0000-0000-0000-000000000002` | Bob | member |
| `MemberCarlaID` | `c3c3c3c3-0000-0000-0000-000000000003` | Carla | member |

hh-auth also seeds an admin user (not member-linked):
- Email: `admin@household.dev`, Role: `admin`

## Adding a New Member

1. Add UUID constant to `hh-shared/seeds/identities.go`
2. Add user row to `hh-auth/seeds/dev/001_users.sql` (with bcrypt hash)
3. Add member row to `hh-users/seeds/dev/001_members.sql`
4. Update `hh/fe/src/lib/seeds.ts` (frontend mirror)
5. Add seed data to any service that references members (investments entities, etc.)
6. Bump hh-shared version and update downstream services

## Seed File Naming

Use numbered prefixes for execution order: `001_users.sql`, `002_goals.sql`, etc.
The `seeds.Run()` function from hh-shared executes files in alphabetical order.
