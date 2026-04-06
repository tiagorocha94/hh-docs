# Members

Members are the people who share a household. Every other feature in the
platform operates in the context of members — a goal belongs to the household,
a transaction is recorded by a member, an investment is tracked per member.

## What you can do

- Add, edit, and remove household members
- Each member gets a name and a visual avatar (color + icon)
- Avatars use the [Lucide icon set](https://lucide.dev) and appear consistently across the entire app

## How avatars work

Each member is represented by a colored circle with an icon inside. You pick
a color (hex value) and an icon name. This same pattern is reused for goals,
expense categories, and investment instruments — so the whole app feels
visually consistent.

## Roles

There are two roles:

- **Admin** — can create, edit, and delete members. Can manage all resources.
- **Member** — can view all data and edit their own profile.

Roles are assigned at the authentication level (hh-auth) and enforced by
each service independently.
