# hh-investments

Investment portfolio tracking.

Repository: [github.com/tiagorocha94/hh-investments](https://github.com/tiagorocha94/hh-investments)

For what this service does and how it works, see [Features > Investments](../features/investments.md).

## Technical Details

- Database: `hh_investments`
- All data is per-member (entities, instruments, contributions, valuations)
- ON DELETE CASCADE on contributions and valuations when an instrument is deleted
- Deleting an entity or type with active instruments returns `409 Conflict`
- Default investment types are seeded via migration (ETF, Stock, Bond, PPR, etc.)

### API Surface

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| GET | `/v1/types` | Bearer | List investment types |
| POST | `/v1/types` | Bearer | Create type |
| GET | `/v1/types/{id}` | Bearer | Get type |
| PUT | `/v1/types/{id}` | Bearer | Update type |
| DELETE | `/v1/types/{id}` | Bearer | Delete type |
| GET | `/v1/entities` | Bearer | List entities |
| POST | `/v1/entities` | Bearer | Create entity |
| GET | `/v1/entities/{id}` | Bearer | Get entity |
| PUT | `/v1/entities/{id}` | Bearer | Update entity |
| DELETE | `/v1/entities/{id}` | Bearer | Delete entity |
| GET | `/v1/instruments` | Bearer | List instruments |
| POST | `/v1/instruments` | Bearer | Create instrument |
| GET | `/v1/instruments/{id}` | Bearer | Get instrument |
| PUT | `/v1/instruments/{id}` | Bearer | Update instrument |
| DELETE | `/v1/instruments/{id}` | Bearer | Delete instrument |
| GET | `/v1/instruments/{id}/contributions` | Bearer | List contributions |
| POST | `/v1/instruments/{id}/contributions` | Bearer | Create contribution |
| DELETE | `/v1/instruments/{id}/contributions/{cid}` | Bearer | Delete contribution |
| GET | `/v1/instruments/{id}/valuations` | Bearer | List valuations |
| PUT | `/v1/instruments/{id}/valuations/{month}` | Bearer | Upsert valuation |
| DELETE | `/v1/instruments/{id}/valuations/{vid}` | Bearer | Delete valuation |

### Schema

| Table | Key columns |
|-------|-------------|
| `investment_types` | id, name, color, icon |
| `entities` | id, member_id, name, color, icon |
| `instruments` | id, member_id, entity_id, type_id, name, status |
| `contributions` | id, instrument_id, amount, contributed_on |
| `valuations` | id, instrument_id, month, value (UNIQUE instrument_id+month) |
