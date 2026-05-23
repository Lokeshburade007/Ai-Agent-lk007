# API Inventory — <PROJECT NAME>

One line per endpoint / public function. Update when you ship.

## Authentication

| Method | Path | Auth | Handler | Notes |
|---|---|---|---|---|
| POST | `/auth/signup` | none | `auth.signup` | bcrypt + rate-limited |
| POST | `/auth/login` | none | `auth.login` | returns access+refresh |
| POST | `/auth/refresh` | refresh-token cookie | `auth.refresh` | rotates refresh token |
| POST | `/auth/logout` | access-token | `auth.logout` | invalidates refresh |

## Users

| Method | Path | Auth | Handler | Notes |
|---|---|---|---|---|
| GET | `/me` | access-token | `users.me` | profile + permissions |
| PATCH | `/me` | access-token | `users.updateMe` | name, email |

## <Resource>

| Method | Path | Auth | Handler | Notes |
|---|---|---|---|---|
| | | | | |

## Internal (not on the public router)

| Function | Location | Used by |
|---|---|---|
| `sendNotificationEmail(userId, template, data)` | `src/services/notifications.ts` | order events, password reset |
