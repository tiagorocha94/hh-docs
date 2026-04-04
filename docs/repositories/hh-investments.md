# hh-investments

Personal investment portfolio tracking for the Household platform.

Repository: [github.com/tiagorocha94/hh-investments](https://github.com/tiagorocha94/hh-investments)

## Overview

hh-investments lets household members track their personal investment portfolios. Each member records the financial institutions they use, the investment positions they hold, the money they put in over time, and a monthly snapshot of what each position is currently worth.

The goal is a simple, honest record of how a portfolio evolves — not real-time market data or automated calculations, but a deliberate monthly ritual of "I invested X, it's now worth Y".

## Authentication

All `/v1` endpoints require a valid JWT bearer token issued by [hh-auth](hh-auth.md). The service validates tokens using a JWKS endpoint configured via the `JWKS_URL` environment variable. Requests without a valid token receive a `401 Unauthorized` response.

## Core Concepts

### Investment Type
A category of investment instrument — ETF, Stock, Bond, Deposit, PPR, Certificados de Aforro, and so on. The service ships with a sensible default set, but members can add custom types or delete ones they will never use. Types carry a color and icon for visual identification.

### Entity
A financial institution where the member holds investments. This could be a broker (DEGIRO, IBKR), a bank (CGD, Millennium), or a state program. Entities are per-member, carry a color and icon, and act as an organisational anchor.

### Instrument
A specific investment position — a particular ETF at a particular broker, a PPR at a bank. An instrument belongs to one member, is held at one entity, and has one type. Instruments can be closed without deleting them, preserving the full history.

### Contribution
A record of money added to an instrument on a specific date. Contributions are inflows only. Immutable once created — delete and re-create to correct.

### Valuation
A manual snapshot of what an instrument is worth at the end of a given month. One valuation per instrument per month. Calling the endpoint twice for the same month updates the record (upsert semantics).

## Key Behaviours

- All data is per-member. Querying instruments, entities, contributions, or valuations always requires a `member_id`.
- Deleting an instrument permanently removes its contribution and valuation history via cascade.
- Deleting an entity or type that still has instruments fails with `409 Conflict`.
- Valuations are entered manually. No live price feeds.
- All monetary values are in EUR.

## Limitations

- No real-time prices or automated valuations.
- Contributions are inflows only — no outflows/withdrawals yet.
- Each instrument belongs to one member (no joint instruments).
- All amounts in EUR (no multi-currency).

## Future Directions

| Capability | Notes |
|---|---|
| Outflows / withdrawals | Record money taken out; needed for accurate gain/loss |
| Multi-currency | Add currency column; convert to EUR for aggregation |
| Joint instruments | Multiple member_ids per instrument |
| Auth integration | Filter queries by member from JWT claim |
