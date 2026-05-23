# Data Model — <PROJECT NAME>

## Tables / Collections

### `users`
| Column | Type | Null? | Default | Purpose |
|---|---|---|---|---|
| id | uuid | no | gen_random_uuid() | PK |
| email | text | no | — | unique, lowercased on insert |
| password_hash | text | no | — | bcrypt cost 12 |
| created_at | timestamptz | no | now() | |
| updated_at | timestamptz | no | now() | trigger auto-updates |
| deleted_at | timestamptz | yes | null | soft delete |

Indexes:
- `users_email_lower_unique` on `lower(email)` — UNIQUE
- `users_created_at_idx` for listing

### `<table>`
…

## Foreign keys

| Child | Column | Parent | On delete |
|---|---|---|---|
| orders | user_id | users.id | RESTRICT |

## Non-obvious decisions

- Why UUIDs not bigint: <reason — distributed inserts, opaque to clients>
- Why timestamptz not timestamp: <reason — preserve timezone>
- Why soft-delete column instead of separate audit table: <reason — query simplicity outweighs storage>
