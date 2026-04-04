# hh-users

The identity foundation of the Household platform, managing the people who share a household.

Repository: [github.com/tiagorocha94/hh-users](https://github.com/tiagorocha94/hh-users)

## Overview

hh-users manages the people who share a household — referred to throughout the platform as **members**. Every other service (goals, expenses, investments) operates in the context of members. A goal belongs to a member. A transaction is made by a member. An investment is tracked per member. This service is the source of truth for who those members are.

## Authentication

All `/v1` endpoints require a valid JWT bearer token issued by [hh-auth](hh-auth.md). The service validates tokens using a JWKS endpoint configured via the `JWKS_URL` environment variable. Requests without a valid token receive a `401 Unauthorized` response.

## Core Concepts

### Member
A person in the household. A member has:
- A **name** — how they appear throughout the app
- An **email** — optional, for future notification use
- An **avatar** — a color and an icon chosen from a predefined palette, used to visually distinguish members across all parts of the UI

### Avatar
Each member is represented visually by a colored circle with an icon inside. The color is a hex value and the icon is a name from the [Lucide icon set](https://lucide.dev). These are stored as plain strings and rendered by the frontend.

This same color + icon pattern is reused in other services (goals, expense categories, investment instruments) for visual consistency across the platform.

## Key Behaviours

- Members can be created, edited, and deleted at any time.
- Email is optional. When provided it must be a valid email address and must be unique across all members.
- Deleting a member does **not** cascade to other services. Goals, expenses, and investments that reference that member's ID are unaffected — this is intentional. Cross-service cascade is a future concern.
- There is no concept of permissions at this stage beyond the admin/member role distinction enforced by hh-auth.

## Limitations

- **Cross-service data** — this service knows nothing about goals, expenses, or investments. It only manages identity.
- **Cross-service cascade** — deleting a member does not clean up references in other services.

## Future Directions

| Capability | Notes |
|---|---|
| Invitations | Invite a person to join the household by email |
| Profile photos | Support uploaded avatars in addition to the icon-based system |
| Per-member settings | Timezone, currency preference, notification preferences |
| Cross-service cascade | Propagate member deletion to dependent services |