---
description: Guides the creation of new API endpoints. Use when adding routes, handlers, or controllers to the project
---

# API Endpoints

Follow these conventions whenever you add or change an API endpoint. If the project already does something differently, match the project and point out the difference. Don't silently mix the two styles.

## Where route files live

Each resource gets its own set of files, split by responsibility:

```
src/
  routes/
    index.ts              # mounts every resource router under /api/v1
    users.routes.ts       # route definitions only: path + method + middleware + handler
  controllers/
    users.controller.ts   # request handlers: parse input, call service, send response
  services/
    users.service.ts      # business logic and data access; no req/res objects
  validators/
    users.validator.ts    # request schemas (body, params, query)
tests/
  users.routes.test.ts    # endpoint tests
```

- Route files only wire things together. They contain no logic.
- Controllers stay thin. Any logic you'd want to reuse or test without HTTP goes in a service.
- Register every new router in `src/routes/index.ts`.

## Naming conventions

- **URLs:** plural, lowercase, kebab-case nouns, e.g. `/api/v1/order-items/:orderItemId`. No verbs in paths; the HTTP method is the verb.
- **Nesting:** at most one level deep, e.g. `/users/:userId/orders`. Anything deeper gets its own top-level resource.
- **Methods:** `GET` reads, `POST` creates, `PATCH` partially updates, `PUT` fully replaces, `DELETE` removes.
- **Files:** `<resource>.routes.ts`, `<resource>.controller.ts`, `<resource>.service.ts`, `<resource>.validator.ts`, with the resource name plural and in kebab-case.
- **Handlers:** verb + noun in camelCase: `listUsers`, `getUser`, `createUser`, `updateUser`, `deleteUser`.
- **JSON fields:** camelCase in both requests and responses.

## Error handling pattern

- Validate the body, params and query at the route boundary with the resource's validator, before the controller runs. Validation failures return `400`.
- Services throw typed errors such as `NotFoundError`, `ValidationError`, `ConflictError` and `UnauthorizedError`. They never build HTTP responses themselves.
- Controllers don't use `try/catch` just to reformat errors. Let errors reach the central error-handling middleware, which maps each error type to a status code and the error response shape below.
- Never send stack traces, SQL or internal messages to the client. Log them on the server with the request ID.
- Status codes: `400` for invalid input, `401` when unauthenticated, `403` when forbidden, `404` when not found, `409` for a conflict, `422` when a business rule is violated, and `500` for anything unexpected.

## Response shape

Every response uses the same envelope.

Success:

```json
{
  "data": { "id": "123", "email": "a@example.com" },
  "meta": { "requestId": "abc-123" }
}
```

List, with pagination:

```json
{
  "data": [ { "id": "123" } ],
  "meta": { "requestId": "abc-123", "page": 1, "pageSize": 20, "total": 57 }
}
```

Error:

```json
{
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "No user exists with id 123.",
    "details": []
  },
  "meta": { "requestId": "abc-123" }
}
```

- `code` is a stable SCREAMING_SNAKE_CASE identifier that clients can depend on. `message` is written for people to read.
- For validation errors, `details` lists one entry per invalid field: `{ "field": "email", "message": "Must be a valid email." }`.
- `201 Created` returns the created resource. `204 No Content` returns no body.

## Before committing

Don't commit a new or changed endpoint until all of these are true:

- [ ] The route is registered in `src/routes/index.ts` and is reachable.
- [ ] The request is validated by a schema in `validators/`.
- [ ] Authentication and authorization middleware are applied, or the route is deliberately marked public with a comment explaining why.
- [ ] Success responses and error responses both use the envelopes above.
- [ ] Tests cover the success case, invalid input (`400`), not found (`404`) where it applies, and an unauthorized request (`401`/`403`).
- [ ] The tests, linter and type checker all pass.
- [ ] No secrets, debug logging or commented-out code are left behind.
- [ ] The API documentation (OpenAPI spec or README) describes the new endpoint.
