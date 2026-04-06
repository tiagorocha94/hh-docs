# Members

Members are the people who share a household. Every other feature in the
platform operates in the context of members — a goal belongs to the household,
a transaction is recorded by a member, an investment is tracked per member.

## What you can do

- Add, edit, and remove household members
- Each member has a name — avatars are generated automatically

## How avatars work

Avatars are generated client-side using DiceBear with the member's name as
seed. No avatar data is stored in the backend — the same name always
produces the same avatar, and avatars are cached in the browser.

## Roles

There are two roles:

- **Admin** — can create, edit, and delete members. Can manage all resources.
- **Member** — can view all data and edit their own profile.

Roles are assigned at the authentication level (hh-auth) and enforced by
each service independently.
