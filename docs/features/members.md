# Members

Members are the people who share a household. Every other feature in the
platform operates in the context of members — a goal belongs to the household,
a transaction is recorded by a member, an investment is tracked per member.

## What you can do

- Add, edit, and remove household members
- Optionally create a login (user account) at the same time as the member
- Each member has a name — avatars are generated automatically

## Creating members with login

When an admin creates a member, they can optionally provide an email and
password in the same request. This creates both the member and a linked user
account (with role `member`) in a single transaction — no need to create the
user separately.

- If only `name` is provided, a member is created without a login
- If `name`, `email`, and `password` are all provided, both a member and a
  linked user account are created atomically
- `email` and `password` must both be present or both absent — providing one
  without the other is an error

This means new household members can log in immediately after being added,
without the admin needing to manually create a user account.

## How avatars work

Avatars are generated client-side using DiceBear with the member's name as
seed. No avatar data is stored in the backend — the same name always
produces the same avatar, and avatars are cached in the browser.

## Roles

There are two roles:

- **Admin** — can create, edit, and delete members. Can manage all resources.
- **Member** — can view all data and edit their own profile.

Roles are assigned at the authentication level (hh-identity) and enforced by
each service independently.
