# API Design Standards

## URL Structure
- Noun-based paths, plural: `/users`, `/sessions`, `/clips`, `/purchases`
- Nested resources for clear ownership: `/sessions/{sessionId}/clips`
- Actions as sub-resources when needed: `/sessions/{sessionId}/clips/{clipId}/purchase`
- Lowercase, kebab-case for multi-word segments: `/camera-setup`, `/field-tech`
- API version prefix only when introducing breaking changes: `/v2/users`

## HTTP Methods
| Method | Use | Body | Idempotent |
|---|---|---|---|
| GET | Read resource | None | Yes |
| POST | Create resource or action | JSON | No |
| PUT | Full replace | JSON | Yes |
| PATCH | Partial update | JSON | No |
| DELETE | Remove resource | None | Yes |

## Request/Response Shape

**Success — single resource:**
```json
{ "sessionId": "abc", "userId": "user123", "status": "active" }
```

**Success — list:**
```json
{ "items": [...], "nextToken": "..." }
```

**Error — always this shape:**
```json
{ "error": "Human-readable message", "code": "VALIDATION_ERROR" }
```

Never return: stack traces, internal IDs, SQL/NoSQL error messages.

## Status Codes
| Code | Meaning | Use when |
|---|---|---|
| 200 | OK | Successful GET, PATCH |
| 201 | Created | Successful POST that creates a resource |
| 204 | No Content | Successful DELETE |
| 400 | Bad Request | Invalid input, missing required fields |
| 401 | Unauthorized | Missing or invalid auth token |
| 403 | Forbidden | Valid auth, but not allowed to access this resource |
| 404 | Not Found | Resource doesn't exist (or you're hiding its existence) |
| 409 | Conflict | Duplicate resource, concurrent modification |
| 422 | Unprocessable | Valid JSON but business rule violation |
| 500 | Internal Error | Unexpected server error |

## Authentication Headers
```
# User / Admin endpoints
Authorization: Bearer <jwt>

# Device endpoints
x-api-key: <device-api-key>

# Never both on the same endpoint
```

## Pagination
- Cursor-based (DynamoDB `LastEvaluatedKey` → opaque `nextToken` string)
- Never offset-based (breaks on concurrent inserts)
- Default page size: 20, max: 100
- Response includes `nextToken` only if more results exist

## Versioning
- URL versioning (`/v2/`) only for breaking changes
- Additive changes (new fields) are not breaking — don't bump version
- Deprecated fields: keep for 2 release cycles, mark in docs

## CORS
- Set explicitly on all browser-facing endpoints
- Origin whitelist: production domain + localhost for dev
- Never `Access-Control-Allow-Origin: *` on authenticated endpoints

## Idempotency
- POST endpoints that create payment/payout records must accept `idempotencyKey`
- Store idempotency keys in DynamoDB with TTL
- Pattern: `<action>-{userId}-{periodStart}`
