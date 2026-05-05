# Credits

Household tracks loans and credits for each member — mortgages, car loans,
personal loans, and other credit products.

## Why manual payment tracking?

Credit payments vary over time due to variable interest rates, extra down
payments, and renegotiations. Rather than trying to predict future payments,
members record each payment as it happens with the actual principal and
interest breakdown. This builds an accurate history of what was paid and
how much of the loan remains.

## Core concepts

### Credit Types
Categories for credits — House, Car, Personal, Other. Each type has a
color and icon for visual identification. Types are household-wide and
come pre-seeded.

### Credits
A specific loan held by a member. Each credit tracks:
- **Name** — what the credit is for (e.g. "Apartment Lisbon")
- **Entity** — the bank or institution
- **Contract number** — reference identifier
- **Total amount** — the original loan value
- **Start date** — when the credit began (YYYY-MM)
- **Duration** — number of months

The system computes end date, remaining amount, progress percentage, and
status (active/completed) from the payment history.

### Payments
Monthly payment records split into two components:
- **Principal** — the amount that reduces the loan balance
- **Interest** — the cost of the credit (doesn't reduce the balance)
- **Total** — computed as principal + interest (server-side)

An optional description allows noting special payments (e.g. "extra down
payment", "renegotiation").

### Auto-completion
A credit is automatically marked as "completed" when the sum of all
principal payments equals or exceeds the total loan amount.

## Example flow

1. Alice creates a credit: "Apartment" at CGD, €150,000, 360 months, starting 2024-01
2. Each month she records the payment: principal €350 + interest €300 = total €650
3. The system shows remaining balance decreasing and progress increasing
4. After a bonus, she makes an extra down payment: principal €5,000 + interest €0
5. The remaining balance drops significantly, progress jumps
6. After 360 months (or earlier with extra payments), the credit auto-completes
