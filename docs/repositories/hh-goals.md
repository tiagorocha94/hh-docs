# hh-goals

Savings goals and envelope budgeting.

Repository: [github.com/tiagorocha94/hh-goals](https://github.com/tiagorocha94/hh-goals)

For what this service does and how it works, see [Features > Savings Goals](../features/goals.md).

## Technical Details

- Database: `hh_goals`
- Goals are household-wide (no member_id scoping in v1)
- Account balances are always derived from movements, never stored
- Goal versioning: budget/target_date changes create a new version with regenerated planned allocations

### API Surface

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| GET | `/v1/accounts` | Bearer | List accounts |
| POST | `/v1/accounts` | Bearer | Create account |
| GET | `/v1/accounts/{id}` | Bearer | Get account |
| PUT | `/v1/accounts/{id}` | Bearer | Update account |
| DELETE | `/v1/accounts/{id}` | Bearer | Delete account |
| GET | `/v1/accounts/{id}/movements` | Bearer | List movements |
| POST | `/v1/accounts/{id}/movements` | Bearer | Record movement (triggers auto-distribution) |
| DELETE | `/v1/accounts/{id}/movements/{mid}` | Bearer | Delete movement |
| GET | `/v1/goals` | Bearer | List goals |
| POST | `/v1/goals` | Bearer | Create goal |
| GET | `/v1/goals/{id}` | Bearer | Get goal |
| PUT | `/v1/goals/{id}` | Bearer | Update goal |
| DELETE | `/v1/goals/{id}` | Bearer | Delete goal |
| PATCH | `/v1/goals/{id}/status` | Bearer | Change goal status |
| GET | `/v1/goals/{id}/versions` | Bearer | List goal versions |
| GET | `/v1/goals/{id}/allocations` | Bearer | List allocations |
| PUT | `/v1/goals/{id}/allocations/{month}` | Bearer | Upsert allocation |
| DELETE | `/v1/goals/{id}/allocations/{aid}` | Bearer | Delete allocation |
| GET | `/v1/goals/{id}/expenses` | Bearer | List expenses |
| POST | `/v1/goals/{id}/expenses` | Bearer | Create expense |
| DELETE | `/v1/goals/{id}/expenses/{eid}` | Bearer | Delete expense |

### Schema

| Table | Key columns |
|-------|-------------|
| `accounts` | id, name, description |
| `account_movements` | id, account_id, amount, date |
| `goals` | id, name, color, icon, status, start_date |
| `goal_versions` | id, goal_id, version_number, budget, target_date |
| `goal_planned_allocations` | id, goal_version_id, year, month, amount |
| `goal_allocations` | id, goal_id, year, month, amount |
| `goal_expenses` | id, goal_id, amount, spent_on |
