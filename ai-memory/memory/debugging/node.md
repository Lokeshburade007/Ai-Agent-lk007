# Node.js / Express Debugging Playbook

Most common Node/Express bugs you'll hit, in priority order.

## EADDRINUSE — "address already in use"

Another process holds the port. Find and kill:
```bash
lsof -i :3000             # find PID
kill -9 <PID>
# or in one line:
kill -9 $(lsof -t -i:3000)
```

## "Cannot find module 'X'"

1. `npm install` (or `pnpm install` / `yarn`) — fresh clone?
2. Module-resolution mismatch: TypeScript path aliases work in `tsc` but not in `ts-node` / Node — use `tsconfig-paths/register` or build first.
3. Case sensitivity: imported `User.ts` but file is `user.ts`. Works on macOS, fails on Linux.

## Async error not caught — "UnhandledPromiseRejectionWarning"

A `Promise` rejected with no `.catch` / `try` around the `await`. Causes:
- Forgot `await` somewhere (`asyncThing()` returns a promise that throws later).
- `forEach` doesn't await — use `for...of` or `Promise.all(arr.map(...))`.
- Middleware that returns a promise but `next` is not called on error — wrap with `express-async-handler` or `next(err)`.

## CORS errors in the browser

- Server didn't send `Access-Control-Allow-Origin`. Add `cors()` middleware.
- Preflight (OPTIONS) failing: server must respond to OPTIONS, allow the right headers and methods.
- Credentials: `credentials: 'include'` on client requires `Access-Control-Allow-Credentials: true` and a **specific** origin (not `*`).

## Memory leak / slow growth

- Open a long heap snapshot in Chrome DevTools (`--inspect` flag). Take three snapshots; diff them.
- Common culprits: event listeners not removed, timers not cleared, closures holding references, in-memory caches without eviction.
- DB connections: verify pool size; `db.collection(...)` cursors closed.

## "Headers already sent" error

Code path called `res.send()` / `res.json()` twice. Usually:
- Sent a response, then threw and the error middleware sent another.
- Forgot `return` after `res.json(...)` — handler continued.

Fix: always `return res.json(...)` and ensure error middleware checks `res.headersSent`.

## ECONNREFUSED to your own DB / Redis

- The service isn't running.
- Wrong host: `localhost` vs `127.0.0.1` vs container hostname. In Docker Compose, use the service name, not localhost.

## JWT verification fails

- Secret mismatch between sign and verify.
- Token expired — check `exp` claim.
- Algorithm mismatch — server expects `HS256`, token signed with `RS256`.
- Token sent in wrong header format — should be `Authorization: Bearer <token>`.

## Mongoose query returns nothing

- `_id` is `ObjectId`, you're passing a string. Convert: `new mongoose.Types.ObjectId(id)`.
- `lean()` returns plain objects without virtuals — your code expects a Mongoose document.
- `findOne` returns null silently — log the query.

## Slow endpoint — where do I look first?

1. Add `console.time` / `console.timeEnd` (or proper APM) around suspected blocks.
2. Check the DB query: `.explain('executionStats')` in Mongo, `EXPLAIN ANALYZE` in Postgres. Missing index?
3. N+1: loop calling `findById` inside `forEach`. Refactor to single `find({ _id: { $in } })`.
4. Synchronous CPU-bound work: blocks the event loop. Move to worker thread.

## Process crashes silently in production

- Run under a process supervisor (`pm2`, `systemd`, container restart policy).
- Log unhandled exceptions:
```js
process.on('uncaughtException', (err) => { logger.fatal(err); process.exit(1); });
process.on('unhandledRejection', (err) => { logger.fatal(err); process.exit(1); });
```
- Don't swallow them and continue — process state is unknown after.
