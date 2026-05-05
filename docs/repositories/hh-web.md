# hh-web

Frontend SPA for the Household platform.

Repository: [github.com/tiagorocha94/hh-web](https://github.com/tiagorocha94/hh-web)

## Technical Details

- React 18 + TypeScript + Vite 6
- Tailwind CSS for styling
- TanStack Query for server state
- react-router-dom for routing
- MSW (Mock Service Worker) for local development without backend
- Docker (nginx) for production deployment

### Tech Stack

| Layer | Technology |
|-------|-----------|
| Framework | React 18 |
| Language | TypeScript 5.9 |
| Build | Vite 6 |
| Styling | Tailwind CSS |
| State | TanStack Query v5 |
| Routing | react-router-dom v6 |
| Icons | Lucide React |
| Toasts | react-hot-toast |
| Avatars | DiceBear (initials style, client-side) |
| Testing | Vitest + Testing Library |
| Mocking | MSW v2 |

### Modules

| Module | Path | Description |
|--------|------|-------------|
| Auth | `/login` | Login page, AuthContext, token management |
| Members | `/members` | Household member list with DiceBear avatars |
| Goals | `/goals/goals`, `/goals/accounts` | Savings accounts, goals, allocations, expenses |
| Investments | `/investments/portfolio`, `/investments/types` | Types, entities, instruments, contributions, valuations |
| Finances | `/finances/transactions`, `/finances/accounts`, `/finances/imports`, `/finances/categories` | Categories, accounts, transactions, imports, budgets |
| Profile | `/profile` | User profile and preferences |

### API Integration

All API calls go through a shared client (`lib/api.ts`) that handles:
- Bearer token injection for authenticated requests
- 401 intercept with single-refresh queue
- Error classification (validation → inline, others → toast)

| Service | Base path | Endpoints |
|---------|-----------|-----------|
| hh-identity | `/api/identity/v1` | login, refresh, logout, me, members CRUD |
| hh-goals | `/api/goals/v1` | accounts, goals, allocations, expenses |
| hh-investments | `/api/investments/v1` | types, entities, instruments, contributions, valuations |
| hh-finances | `/api/finances/v1` | groups, categories, transactions, imports, budgets |

### Mock Server (MSW)

Local development runs against MSW handlers that simulate all backend APIs.
Activated via `VITE_MOCK_API=true` in `.env.development`. Mock data includes
two users (admin + member) with realistic fixture data across all modules.

### Docker

Multi-stage build: Node 20 builder → nginx alpine serving static files.
SPA fallback configured (`try_files $uri /index.html`).

### CI/CD

- **CI** (push/PR to main): tsc, eslint, vitest, vite build
- **CD** (v* tags): CI suite + Docker build + push to GHCR

### Releases

| Version | Scope |
|---------|-------|
| v0.1.0 | Scaffold, auth, members, MSW, Docker, CI/CD |
| v0.2.0 | Goals module, investments module |
| v0.3.0 | Finances module |
| v0.3.1 | API path refactor (hh-identity consolidation) |
| v0.4.0 | Inline login creation in member form |
| v0.4.1 | Goal status change UI (complete/cancel with surplus handling) |
| v0.5.0 | Profile page, theme toggle |
| v0.6.0 | Finances module (categories, accounts, transactions, imports, budgets) |
| v0.6.1 | Bug fixes |
| v0.6.2 | Bug fixes |
| v0.7.0 | Investment dashboard, portfolio improvements |
| v0.8.0 | URL-based tab routing (tabs map to sub-routes for deep-linking and refresh) |
| v0.9.0 | API paths migrated to /api/<service>/v1/ prefix, accounts removed from finances |
