# hh-finances

Income and expense tracking with Excel import.

Repository: [github.com/tiagorocha94/hh-finances](https://github.com/tiagorocha94/hh-finances)

For what this service does and how it works, see [Features > Finances](../features/finances.md).

## Technical Details

- Database: `hh_finances`
- Categories and groups are household-wide (no member_id)
- Transactions and accounts are per-member (via file_import → member_id)
- File imports are unique per (member_id, year, month) — re-upload replaces
- Budgets are versioned per member; active budget = latest effective_from
- Monetary values use `numeric(15,2)`
- Transactions require both category_id and account_id (not optional)

### API Surface

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| GET | `/v1/groups` | Bearer | List category groups (with categories) |
| POST | `/v1/groups` | Bearer | Create group |
| PUT | `/v1/groups/{id}` | Bearer | Update group |
| DELETE | `/v1/groups/{id}` | Bearer | Delete group |
| GET | `/v1/categories` | Bearer | List categories |
| POST | `/v1/categories` | Bearer | Create category |
| PUT | `/v1/categories/{id}` | Bearer | Update category |
| DELETE | `/v1/categories/{id}` | Bearer | Delete category |
| PUT | `/v1/categories/{id}/group` | Bearer | Assign/unassign category to group |
| GET | `/v1/accounts` | Bearer | List accounts (?member_id=) |
| POST | `/v1/accounts` | Bearer | Create account |
| DELETE | `/v1/accounts/{id}` | Bearer | Delete account |
| GET | `/v1/transactions` | Bearer | List transactions (filtered) |
| PUT | `/v1/transactions/{id}` | Bearer | Update transaction |
| DELETE | `/v1/transactions/{id}` | Bearer | Delete transaction |
| GET | `/v1/imports` | Bearer | List file imports (?member_id=) |
| POST | `/v1/imports` | Bearer | Upload Excel file |
| DELETE | `/v1/imports/{id}` | Bearer | Delete import (cascades transactions) |
| GET | `/v1/members/{id}/budgets` | Bearer | List budget versions |
| POST | `/v1/members/{id}/budgets` | Bearer | Create budget version |

### Schema

| Table | Key columns |
|-------|-------------|
| `category_groups` | id, name, color, description, target_pct, sort_order |
| `categories` | id, name, type (expense/income), category_group_id, color, icon |
| `accounts` | id, member_id, name |
| `file_imports` | id, member_id, year, month, filename, row_count (UNIQUE member+year+month) |
| `transactions` | id, file_import_id, account_id, category_id, type, amount, date, description |
| `member_budgets` | id, member_id, income, effective_from |
| `member_budget_allocations` | member_budget_id, category_group_id, target_pct, amount |
