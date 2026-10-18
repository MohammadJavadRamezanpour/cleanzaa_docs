# Customer profile and activation (B03 backend)

## Scope and state

Phone verification creates a customer with a `CUSTOMER` role and a **PENDING**
profile. The customer may add or change an ordinary name and service address.
They become **ACTIVE** and booking eligible only when both are present and the
current customer terms version has been accepted. The status is calculated from
PostgreSQL data and the configured current terms version on every profile read;
it is not an independent flag that can drift. Clearing a field or publishing a
new terms version makes the profile pending again.

This slice contains no order creation endpoint. It provides
`require_booking_eligibility(session, user_id, current_terms_version)` for the
future order service to call **inside the order creation transaction**. That
service must reject `CustomerNotActive` before inserting an order. Until it is
wired into an order endpoint, no customer can place an order through this API.
The [customer frontend](customer-mobile-sign-in.md) is delivered in a separate
code repository. Terms content, phone change, identity correction, data export,
and deletion flows are separate work. The policy comes from
[ADR 0002](../adr/0002-phase-1-personal-data.md).

## API contract

Success responses are JSON. Errors use the shared
`{"error":{"code":"...","message":"..."}}` envelope and include
`X-Request-ID`. Profile responses include `Cache-Control: no-store`.

| Method and path | Request | Success | Access and errors |
| --- | --- | --- | --- |
| `GET /terms/customer/current` | No parameters | `200 {"version":"2026-10-17","url":"https://..."}` | Public. `503 SERVICE_UNAVAILABLE` if no approved version and URL are configured. The URL must identify the immutable document for that version. |
| `GET /customers/me/profile` | Session cookie | `200` profile response below | `401 UNAUTHENTICATED` invalid/expired session; `403 FORBIDDEN` non-customer role; `404 NOT_FOUND` missing linked customer; `503 SERVICE_UNAVAILABLE` missing backend configuration. |
| `PATCH /customers/me/profile` | Session cookie, `X-CSRF-Token`, JSON subset of `name` and `service_address` | `200` updated profile response | Same auth errors; `403 FORBIDDEN` missing/invalid CSRF or disallowed `Origin`; `422 VALIDATION_ERROR` invalid body. |
| `POST /customers/me/terms-acceptances` | Session cookie, `X-CSRF-Token`, `{"terms_version":"2026-10-17","accepted":true}` | `200` updated profile response | Same auth errors; `409 CONFLICT` stale version; `422 VALIDATION_ERROR` invalid body; `503 SERVICE_UNAVAILABLE` terms not configured. |

The profile response shape is:

```json
{
  "name": "Ada Rossi",
  "service_address": "Via Roma 1, Milano",
  "accepted_terms_version": "2026-10-17",
  "accepted_terms_at": "2026-10-17T08:30:10+00:00",
  "current_terms_version": "2026-10-17",
  "activation_status": "ACTIVE",
  "booking_eligible": true
}
```

On a new account, profile fields and acceptance fields are `null`, status is
`PENDING`, and `booking_eligible` is `false`. `current_terms_version` is `null`
when terms are not configured. An older accepted version remains visible after
a version change but no longer satisfies activation. Timestamps use ISO 8601
with an explicit offset.

`PATCH` accepts a non-empty subset of the two fields. Strings are trimmed;
blank strings, names over 160 characters, addresses over 500 characters, empty
objects, and unknown fields fail with `422`. Explicit `null` clears a field and
may deactivate the profile. The phone number and verified identity are not
editable here. `POST` requires the literal `true`, and the submitted version
must match the server's current version. A repeated acceptance of the same
version is idempotent: it returns `200`, retains the original timestamp, and
does not insert another record. Neither mutation uses `Idempotency-Key` because
profile updates set explicit values and terms acceptance has a unique key.

## Authorization and database behavior

All profile paths use the authenticated user's ID from their server-side
session and require the database-backed `CUSTOMER` role. No caller-supplied
customer ID is accepted. Mutations check the allowed `Origin` and the CSRF
token from `/me` or sign-in. The customer can read and change only their own
profile. Request bodies and service addresses are not written to request logs.

Migration `0004_customer_profile` adds nullable `name` and `service_address`
columns to `customers` and `customer_terms_acceptances` with a composite
`(customer_id, terms_version)` key, immutable document URL, and acceptance
timestamp. Existing minimal rows remain valid and pending. PostgreSQL
`ON CONFLICT DO NOTHING` makes repeated acceptance safe. Downgrading would
delete profile and acceptance data and must not be used after real collection.

`CUSTOMER_TERMS_VERSION` and `CUSTOMER_TERMS_URL` are paired settings. The URL
must be HTTPS and refer to the approved, immutable document for the version.
No production terms version or content is approved in this repository; without
both settings, the terms endpoint and acceptance return `503`, and profiles
remain pending. Publishing terms, legal approval, and any changed retention
obligations must be reviewed before enabling activation. A terms-version change
must reach every API instance before order creation is enabled; a mixed-version
rollout can give inconsistent eligibility decisions. The frontend must show the
linked document before asking for acceptance.

## Verification

From `back/`: apply `.venv/bin/alembic upgrade head` to PostgreSQL, then run
`.venv/bin/pytest -q`, `.venv/bin/ruff check .`,
`.venv/bin/ruff format --check .`, `.venv/bin/mypy app scripts`, and
`.venv/bin/python -m scripts.export_openapi --check`. Also run
`.venv/bin/pip-audit` and `docker build -t cleanzza-api:local .`. The profile tests use
PostgreSQL to cover ownership, role checks, CSRF, validation, activation,
terms rotation, repeated acceptance, and the booking eligibility policy.
