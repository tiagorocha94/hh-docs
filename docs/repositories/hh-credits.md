# hh-credits

Credit management service — mortgages, car loans, personal loans with payment tracking.

Repository: [github.com/tiagorocha94/hh-credits](https://github.com/tiagorocha94/hh-credits)

For what this service does and how it works, see [Features > Credits](../features/credits.md).

## Technical Details

- Database: `hh_credits`
- Credit types are household-wide (no member_id)
- Credits belong to a member (member_id)
- Payments belong to a credit (cascade delete)
- total_amount on payments is computed server-side (principal + interest)
- Status is computed at read time (not stored)
- Monetary values use `numeric(15,2)`

### Environment Variables

| Variable | Description |
|----------|-------------|
| `DB_DSN` | PostgreSQL connection string |
| `JWKS_URL` | hh-identity JWKS endpoint for JWT verification |
| `ENV` | `development` or `production` |
| `PORT` | HTTP listen port (default 8080) |

### API Surface

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| GET | `/v1/types` | Bearer | List credit types |
| POST | `/v1/types` | Bearer | Create type |
| PUT | `/v1/types/{id}` | Bearer | Update type |
| DELETE | `/v1/types/{id}` | Bearer | Delete type |
| GET | `/v1/credits` | Bearer | List credits (?member_id=) |
| POST | `/v1/credits` | Bearer | Create credit |
| PUT | `/v1/credits/{id}` | Bearer | Update credit |
| DELETE | `/v1/credits/{id}` | Bearer | Delete credit (cascades payments) |
| GET | `/v1/credits/{id}/payments` | Bearer | List payments |
| POST | `/v1/credits/{id}/payments` | Bearer | Add payment |
| DELETE | `/v1/credits/{id}/payments/{paymentId}` | Bearer | Delete payment |

### Schema

| Table | Key columns |
|-------|-------------|
| `credit_types` | id, name (UNIQUE), color, icon |
| `credits` | id, member_id, type_id (FK), name, entity, contract_number, total_amount, start_date, duration_months, description |
| `payments` | id, credit_id (FK CASCADE), year, month, total_amount, principal, interest, description (UNIQUE credit_id+year+month, CHECK total=principal+interest) |

### Computed Fields (returned on credits)

| Field | Computation |
|-------|-------------|
| `remaining_amount` | total_amount - sum(principal) |
| `total_paid` | sum(payment.total_amount) |
| `total_principal_paid` | sum(principal) |
| `total_interest_paid` | sum(interest) |
| `status` | "completed" if sum(principal) >= total_amount, else "active" |
| `end_date` | start_date + duration_months |
| `progress_pct` | sum(principal) / total_amount * 100 |
