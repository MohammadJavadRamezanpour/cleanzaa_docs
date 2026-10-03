# API conventions

## OpenAPI export

FastAPI routes and Pydantic schemas are the source for the backend API
contract. The generated artifact is `back/openapi/openapi.json`; its current
`info.version` is `0.1.0`. From `back/`, run
`.venv/bin/python -m scripts.export_openapi` after changing a route or schema.
Backend CI runs `.venv/bin/python -m scripts.export_openapi --check` and fails
if the committed artifact is stale.

Document each endpoint's actual request, response, errors, and access rules in
its feature document when that endpoint is built. A breaking API change also
needs a compatibility and deployment plan before the frontend updates its
pinned contract.

## Validation errors

The FastAPI application returns a consistent response when request data fails
FastAPI/Pydantic validation:

```json
{"error":{"code":"VALIDATION_ERROR","message":"Request validation failed."}}
```

The response status is `422`. The message does not repeat submitted values or
internal validation details. The response includes the same `X-Request-ID`
header as other API responses. This handler applies to validation of path,
query, header, and body parameters on backend routes. FastAPI routes and
Pydantic schemas remain the source of truth for each endpoint's accepted
parameters and OpenAPI contract.

The current health endpoints accept no validated path, query, header, or body
parameters, so this change does not alter the exported OpenAPI artifact. Their
existing response contracts are documented in
[application-foundation.md](application-foundation.md). No authentication,
authorization, idempotency, database write, or audit event is involved in
handling a validation failure.

Verification: from `back/`, run `.venv/bin/pytest -q` and
`.venv/bin/python -m scripts.export_openapi --check`.

## Routing errors

An unknown backend path returns `404` with
`{"error":{"code":"NOT_FOUND","message":"Resource not found."}}`.
Calling an existing backend path with an unsupported HTTP method returns `405`
with `{"error":{"code":"METHOD_NOT_ALLOWED","message":"Method not allowed."}}`.
Both responses include `X-Request-ID`; the `405` response retains its `Allow`
header. These messages do not expose a route's internal exception detail.

For the current health routes, these errors require no authentication and have
no request body, idempotency, database write, or audit effect. Explicit `404`
exceptions also use this format; any endpoint-specific access rules remain with
that endpoint. The generated OpenAPI operations are unchanged.

## Other expected HTTP errors

When backend code raises an HTTP error, its status is preserved and its detail
is replaced with a safe, machine-readable response:

| Status | Code | Message |
| --- | --- | --- |
| `400` | `BAD_REQUEST` | `Invalid request.` |
| `401` | `UNAUTHENTICATED` | `Authentication required.` |
| `403` | `FORBIDDEN` | `Access denied.` |
| `409` | `CONFLICT` | `Request conflicts with current state.` |
| `429` | `RATE_LIMITED` | `Too many requests.` |

Explicitly raised HTTP statuses not listed here or under routing errors use
`HTTP_ERROR` and `Request failed.`
The response retains headers supplied with the exception, including
`WWW-Authenticate` on `401` and `Retry-After` on `429`, and receives
`X-Request-ID`. This is response formatting only; authentication and permission
rules are not implemented by this handler. No production route or generated
OpenAPI operation changes here.

## Unexpected server errors

An unhandled backend exception returns `500` with
`{"error":{"code":"INTERNAL_ERROR","message":"Internal server error."}}`.
The response includes `X-Request-ID`, and a server error log records that same
ID, the method, path, and `500` status. Exception details are not returned to
the client or written to that structured request log. This change introduces
no new route or database migration, so the generated OpenAPI operations remain
unchanged.
