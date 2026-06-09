# Savings Goals

Household uses envelope budgeting for savings. Instead of tracking money
passively, every saved euro gets a job — assigned to a named goal with a
budget and a deadline.

## Why envelopes?

Many households save vaguely without a clear destination. By dividing savings
into named envelopes — holidays, a car, an emergency fund — it becomes easier
to see progress, stay motivated, and make trade-offs when budgets are tight.

## Core concepts

### Accounts

Savings accounts that hold real money. Each account has a derived balance
(sum of all movements). Accounts come in two flavours:

- **Household account** (`member_id = NULL`) — shared by everyone. Only one
  can exist. Household goals (holidays, shared purchases) live here.
- **Personal account** (`member_id` set) — owned by one member. Each member
  can have at most one. Personal goals (mortgage overpayment, individual
  savings) live here.

If a member has no personal account, they only interact with the household
account.

### Goals

Named saving envelopes. Each goal belongs to exactly one account. A goal has
a color, icon, budget, target date, and start date.

A goal moves through statuses: **planned** (start date in the future) →
**active** (start date reached) → **completed** or **cancelled**.

When a goal is completed with a positive balance, the surplus can optionally
be returned to an account as a movement. If no account is selected, the
surplus stays as unallocated balance on the goal's account.

### Deposits and allocations

When you record a deposit into an account, it increases the account balance.
You then manually allocate funds to goals via the allocations API. This
gives full flexibility over how money is distributed — you decide which goals
get funded and by how much.

The planned allocations (budget ÷ months) serve as a guide, showing how much
each goal "needs" per month. The UI shows whether each month is met, partial,
or missed based on the actual allocation compared to the plan.

### Allocations

Each goal has a monthly plan (budget ÷ months). As you manually allocate
funds to goals, actual allocations are created. The gap between planned and
actual drives the "met / partial / missed" status shown in the UI.

When an allocation is created or updated, a corresponding movement is
recorded on the goal's account — debiting the allocated amount. This ensures
the account balance always reflects what's truly available (unallocated).
Deleting an allocation reverses the movement, returning money to the account.
This creates a full audit trail of every allocation event.

### Expenses

Spending from a goal's virtual envelope. Recording an expense reduces the
goal balance without touching any real account. This lets you track how much
of a goal's budget has actually been spent.

### Versions

When you change a goal's budget or target date, a new version is created.
Past months keep their original plan. This preserves history and makes it
easy to see how a savings plan evolved over time.

## Account → Goal relationship

The relationship between accounts and goals mirrors the entity → instrument
pattern in the investments module:

```
Household Savings (member_id = NULL)
├── Summer Holiday (shared goal)
├── Car Fund (shared goal)
└── Emergency Fund (shared goal)

Alice's Savings (member_id = Alice)
└── Mortgage Overpayment (personal goal)
```

- Allocations on goals within "Household Savings" debit that account
- Allocations on goals within "Alice's Savings" debit Alice's account
- Surplus from a completed goal stays in the same account

## Example flow

1. Create a household account "Household Savings"
2. Create a personal account "Alice's Savings"
3. Create a shared goal "Summer Holiday" on Household Savings — budget €3,200
4. Create a personal goal "Mortgage Extra" on Alice's Savings — budget €2,000
5. Each month, deposit €800 into Household Savings
6. Manually allocate funds to shared goals (e.g. €267 to Holiday, €400 to Car)
7. Each month, deposit €200 into Alice's Savings
8. Manually allocate funds to Alice's personal goals
9. Book flights → record an expense of €450 against Summer Holiday
10. When the holiday is done, mark the goal as completed — surplus stays in Household Savings
