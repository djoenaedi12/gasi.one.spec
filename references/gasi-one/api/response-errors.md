# API Response, Query, and Error Contract

Baseline sources: `core-api` and `GlobalExceptionHandler` at `gasi.one.api`
commit `88417135daeeb2b10c2df3fb8c23d345b734849c`.

## Response envelope

`ApiResponse<T>` has this logical shape:

```json
{
  "success": true,
  "message": "...",
  "data": {},
  "errors": [],
  "timestamp": "2026-09-29T00:00:00Z"
}
```

Actual fields:

```text
boolean success
String message
T data
List<ErrorDetail> errors
Instant timestamp
```

`ErrorDetail` contains `code`, optional `field`, and `message`. Consumers should
use `code`/`field` for structured behavior and display the safe `message`; do
not parse messages to make business decisions.

## Pagination

`PageResult<T>`:

```text
content, page, size, totalElements, totalPages
```

`QueryRequest`:

```text
filter, sorts, fields, page, size
```

- Pages are zero-based.
- Default page: 0.
- Default size: 10.
- Maximum size: 100.
- `fields` is the public response projection; a controller may force mandatory
  fields such as the ID to remain present.

## Filters and sorting

Polymorphic filters use the `type` discriminator:

```json
{ "type": "simple", "field": "name", "operator": "LIKE", "value": "admin" }
```

```json
{
  "type": "and",
  "filters": [
    { "type": "simple", "field": "status", "operator": "EQUALS", "value": "ACTIVE" }
  ]
}
```

Baseline operators:

```text
EQUALS, NOT_EQUALS,
GREATER_THAN, GREATER_THAN_OR_EQUALS,
LESS_THAN, LESS_THAN_OR_EQUALS,
LIKE, IN, IS_NULL, IS_NOT_NULL
```

Sorting uses `{field, direction}` with `ASC` or `DESC`. Public fields may be
restricted or translated by the persistence adapter; do not assume arbitrary
paths are safe for filtering or sorting.

## Exception mapping

| Condition | HTTP | Primary code/source |
|---|---:|---|
| Entity not found | 404 | `ENTITY_NOT_FOUND` |
| Endpoint not found | 404 | `NO_HANDLER` |
| Method not supported | 405 | `METHOD_NOT_ALLOWED` |
| `BusinessException` | 422 | Business detail from the exception |
| `BadRequestException` | 400 | Request detail from the exception |
| Database unique/FK constraint | 409 | `DATA_CONFLICT` |
| Optimistic locking | 409 | `OPTIMISTIC_LOCK_CONFLICT` |
| Failed/anonymous authentication | 401 | `UNAUTHORIZED` |
| Authenticated actor without permission | 403 | `ACCESS_DENIED` |
| Bean/method/constraint validation | 400 | Validation code and field |
| Unreadable JSON | 400 | `MALFORMED_BODY` |
| Type mismatch | 400 | `TYPE_MISMATCH` |
| Unhandled error | 500 | `INTERNAL_ERROR` |

Important: standard Bean Validation maps to **400**, not 422. Use 422 for
business rules thrown as `BusinessException`. Feature OpenAPI documents must
distinguish these categories or document a specific adapter.

## Optimistic locking

Generated update DTOs implement `VersionedRequest` when the resource contract
requires a version. The client must send the last-read version and handle 409
with reload/review; it must not silently retry with a new version.

## Error security

- 401/403 responses do not disclose protected resource payloads.
- Database errors must not expose raw SQL or constraint details to clients.
- Validation messages may be actionable but must not contain credentials,
  tokens, secrets, or sensitive before/after values.
- Custom domain codes must be stable and documented in the feature contract.
