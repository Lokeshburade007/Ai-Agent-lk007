# Node.js / Express Rules

Load this when working on any Node backend (MERN, standalone Express, API services).

## Always

- Use **async/await** everywhere. No `.then().catch()` chains. No callback style.
- Use a **central error middleware** as the last `app.use()`. Throw or `next(err)` from routes — never send errors inline.
- Use **express-validator** or **zod** for request validation. Validate *before* the handler runs. Never trust `req.body`.
- Use **dotenv** + a single `config.ts/js` module that reads env at startup and validates with zod. Crash on missing required env. Don't read `process.env` scattered through the code.
- Use **Mongoose schemas** in `/models`, **repository functions** in `/repositories`. Routes call repositories, never `Model.find()` directly.
- Use **Pino** or **Winston** for structured logs. Never `console.log` in production code.
- Use **HTTP status codes** with correct semantics: 201 create, 204 empty, 400 client, 401 auth, 403 forbidden, 404 not found, 409 conflict, 422 validation, 429 rate limit, 500 server.
- Use **Idempotency-Key** header for POST endpoints that create resources.
- Use **CORS allowlist** — never `app.use(cors())` open.

## Never

- ❌ Mutate `req` to attach business data — use `res.locals` or a typed context.
- ❌ Use `body-parser` directly — `express.json()` is built in since Express 4.16.
- ❌ Trust client-supplied IDs without ownership check.
- ❌ Use `JSON.parse` / `JSON.stringify` without try/catch on untrusted input.
- ❌ Pass `req.body` straight into Mongoose queries (NoSQL injection vector).
- ❌ Block the event loop — no sync I/O, no heavy CPU in handlers. Use a worker thread or queue.
- ❌ `process.exit()` from inside a handler. Throw or call `next(err)`.

## File layout

```
src/
  config/         # env loader + validators
  routes/         # express routers, one per resource
  controllers/    # handler functions, thin
  services/       # business logic, framework-agnostic
  repositories/   # DB access
  models/         # Mongoose schemas (or pure types if SQL)
  middlewares/    # auth, error, logging, rate-limit
  utils/          # small pure helpers
  index.ts        # app + listen
```

## Testing

- Unit tests: **Vitest** or **Jest** for services/repositories — mock the DB.
- Integration tests: **Supertest** + a real DB (Testcontainers / mongodb-memory-server). Cover the full request path.
- Don't write tests just for coverage. Each test asserts behavior that a user or another service relies on.

## Error handling

- Define a base `AppError` class with `statusCode` + `code` (string).
- Throw `AppError` from anywhere; error middleware translates to HTTP response.
- Never expose `err.stack` or raw error messages in production responses.

## Performance defaults

- Connection pooling: enabled by default in Mongoose/PG drivers, but verify limits.
- Pagination: cursor-based for any list endpoint. Default `limit=20`, max `100`.
- Caching: only after measuring the read pattern. Default to no cache.
- Compression: `compression()` middleware for responses >1KB.
