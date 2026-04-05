# hh-users

Household member management.

Repository: [github.com/tiagorocha94/hh-users](https://github.com/tiagorocha94/hh-users)

For what this service does and how it works, see [Features > Members](../features/members.md).

## Technical Details

- Database: `hh_users`
- Members are referenced by UUID across all other services (no cross-DB FK)
- Deleting a member does not cascade to other services

### API Surface

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| GET | `/v1/members` | Bearer | List members (paginated) |
| POST | `/v1/members` | Admin | Create member |
| GET | `/v1/members/{id}` | Bearer | Get member |
| PUT | `/v1/members/{id}` | Owner/Admin | Update member |
| DELETE | `/v1/members/{id}` | Admin | Delete member |

### Schema

| Table | Key columns |
|-------|-------------|
| `members` | id, name, color, icon, avatar_url |
