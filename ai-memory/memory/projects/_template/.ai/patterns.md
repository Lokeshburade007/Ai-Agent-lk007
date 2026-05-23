# Patterns — <PROJECT NAME>

Conventions specific to this project that the agent should follow. (Generic conventions for the stack live in `ai-memory/rules/<stack>-rules.md`.)

## Error handling

> Example: All thrown errors extend `AppError` with `statusCode` + `code` (string). Error middleware in `src/middlewares/error.ts` translates to HTTP. Never `res.send(err)` directly from a handler.

## Logging

> Example: `pino` logger imported from `src/lib/logger.ts`. Use `logger.info({...}, 'message')` shape. Never `console.log` in production paths.

## Validation

> Example: Every route uses `zod` schema in `src/schemas/<route>.ts`. Validation middleware in `src/middlewares/validate.ts`. Reject before the controller runs.

## Naming conventions specific to this project

> Example:
> - Repository methods: `findById`, `findByX`, `listX`, `insert`, `update`, `delete`. Verbs, not nouns.
> - Use case classes: `<Verb><Noun>UseCase`. e.g. `CreateOrderUseCase`.
> - DTOs: `<Resource>DTO.ts`, separate from domain entity.

## Tests

> Example: Vitest. Unit tests next to source (`foo.ts` + `foo.test.ts`). Integration tests in `tests/integration/`. Mock the DB in unit, use Testcontainers in integration.

## Migration policy

> Example: Migrations are forward-only. Never edit committed migrations. Add nullable, backfill, then `NOT NULL` for any non-null addition.

## Add new patterns as you discover them

Format:
```
## <pattern name>
What we always do, and why we do it.
What we never do, and why we don't.
Example: `src/path/example.ts`
```
