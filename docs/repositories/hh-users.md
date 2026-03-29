# Users Service

> This service is independently versioned and deployed. It depends on
> [`hh-shared`](https://github.com/tiagorocha94/hh-shared) for common
> utilities and is orchestrated by [`hh`](https://github.com/tiagorocha94/hh).

## What this service does

The Users service is the identity foundation of the Household platform. It manages the people who share a household — referred to throughout the platform as **members**.

Every other service (goals, expenses, investments) operates in the context of members.
A goal belongs to a member. A transaction is made by a member. An investment is tracked per member. The Users service is the source of truth for who those members are.

---

## Who uses this service

| Consumer | How |
|---|---|
| **Household members** | Create and manage their own profile (name, avatar, email) |
| **Platform admin** | Add or remove members from the household |
| **Other services** | Read member IDs to associate their own data (no live calls — IDs are stored locally) |
| **Frontend** | Displays member lists and resolves names/avatars across all modules |

---

## Core concepts

### Member
A person in the household. A member has:  
- A **name** — how they appear throughout the app  
- An **email** — optional, for future notification or auth use  
- An **avatar** — a color and an icon chosen from a predefined palette, used to visually distinguish members across all parts of the UI

### Avatar
Each member is represented visually by a colored circle with an icon inside. The color is a hex value and the icon is a name from the [Lucide icon set](https://lucide.dev). These are stored as plain strings and rendered by the frontend.

This same color + icon pattern is reused in other services (goals, expense categories, investment instruments) for visual consistency across the platform.

---

## Key behaviours

- Members can be created, edited, and deleted at any time.
- Email is optional. When provided it must be a valid email address and must be unique across all members.
- Deleting a member does **not** cascade to other services. Goals, expenses, and investments that reference that member's ID are unaffected — this is intentional. Data integrity across services is a future concern once auth is introduced.
- There is no concept of roles or permissions at this stage. All members have equal access.

---

## What this service does not do

- **Authentication** — no login, no tokens, no sessions. That is planned as a separate `auth-svc` in a future phase.
- **Permissions** — no distinction between an admin and a regular member yet.
- **Cross-service data** — this service knows nothing about goals, expenses, or investments. It only manages identity.

---

## Future directions

| Capability | Notes |
|---|---|
| Auth integration | Members will gain login credentials; this service will issue JWTs or delegate to an auth provider |
| Roles | A household owner vs regular member distinction for managing shared settings |
| Invitations | Invite a person to join the household by email |
| Profile photos | Support uploaded avatars in addition to the icon-based system |
| Per-member settings | Timezone, currency preference, notification preferences |
