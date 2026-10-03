# API conventions

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
handling a validation failure. Other error cases will be covered as B02
continues.

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
