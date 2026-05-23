# Architecture — <PROJECT NAME>

## What this project is

One paragraph. What problem it solves. Who uses it. What it deliberately is *not*.

## Stack

- Frontend: <e.g. React + Vite + Tailwind + TanStack Query>
- Backend: <e.g. Node 20 + Express + Mongoose>
- Database: <e.g. Postgres 16 with pgx, or Mongo 7>
- Auth: <e.g. JWT short-lived + refresh tokens hashed in DB>
- Infra: <e.g. Fly.io for backend, Vercel for frontend>

## Boundaries

- **HTTP layer** (`src/routes/`, `src/controllers/`) — thin, no business logic
- **Use cases** (`src/services/`) — orchestration, framework-agnostic
- **Repositories** (`src/repositories/`) — only DB access
- **Domain** (`src/models/`, `src/types/`) — pure types, no I/O

Dependency direction: HTTP → services → repositories → domain. No skipping layers.

## Data flow for the core feature

3–5 lines. Example:
> Client POSTs `/orders` with `{items}` → `controllers/orders.create` calls `services/order.create` which calls `repositories/inventory.reserve` then `repositories/order.insert`, returns `Order`. Failure rollback via DB transaction.

## Things you should NOT do here

- <e.g. "Don't add GraphQL — we're committed to REST">
- <e.g. "Don't import lodash — use ramda">
- <e.g. "Don't store secrets in env vars at runtime — use Vault">

## Open architectural questions

Things still undecided that affect future code:

- [ ] <e.g. "When the orders table hits 100M rows, do we shard by tenant_id or archive cold rows?">
- [ ] <e.g. "Do we keep email sending in-process or move to a queue?">
