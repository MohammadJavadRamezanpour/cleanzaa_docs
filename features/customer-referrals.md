# Customer referrals (storage-only slice)

## Scope

A signed-in customer gets one permanent invite code and shares it as
`<frontend origin>/join/<CODE>`. When a **new** phone number signs up for a
customer account after arriving through that link, the backend stores one
`customer_referrals` row recording who invited whom, which code was used, and
when. That attribution is the whole slice: the record exists so a later rewards
feature can consume it, but nothing is calculated, paid, or promised from it
today.

Explicit non-goals for this slice:

- referral rewards, commission, bonuses, wallet credit, or any money movement
- qualifying events such as a first completed or paid order
- link expiry, rotation, revocation, and per-channel or per-campaign codes
- customer → cleaner referrals, which
  [ADR 0008](../adr/0008-phase-1-boundaries-and-support.md) still defers and
  which must remain a separate table
- duplicate-account, same-household, and other fraud detection beyond the
  checks described below
- notifying either party, exposing the referee's identity to the referrer, or
  any referral dashboard beyond the referrer's own count
- attributing accounts that already exist

Phase context: [ADR 0008](../adr/0008-phase-1-boundaries-and-support.md) deferred
referrals from Phase 1. The owner approved this storage-only slice on
2026-11-14, and the ADR records the amendment; rewards remain deferred. The
domain separation in
[technical architecture §25](../technical_architecture.md) still applies.

## Data model

```text
referral_codes
  code            varchar(8)  primary key
  customer_id     uuid        not null, unique, foreign key -> customers.id (cascade)
  created_at      timestamptz not null, default now()

customer_referrals
  referee_customer_id   uuid  primary key, foreign key -> customers.id (cascade)
  referrer_customer_id  uuid  not null, indexed, foreign key -> customers.id (cascade)
  code                  varchar(8) not null, foreign key -> referral_codes.code (cascade)
  created_at            timestamptz not null, default now()
```

Invariants, enforced in PostgreSQL:

- one permanent code per customer (`referral_codes.customer_id` is unique)
- at most one referrer per referee (`referee_customer_id` is the primary key), so
  a retried signup can never double-attribute
- a customer cannot refer themselves
  (`ck_customer_referrals_distinct`, `referrer_customer_id <> referee_customer_id`)
- deleting either account deletes the edge and the code; this feature stores no
  financial record that a deletion would need to preserve

Codes use the 31-character alphabet `23456789ABCDEFGHJKMNPQRSTUVWXYZ` — no
`0/O`, `1/I/L`, or other look-alikes, so a link survives being read aloud or
retyped. Eight characters give roughly 8.5 × 10¹¹ combinations.

## API contract

Errors use the shared `{"error":{"code":"...","message":"..."}}` envelope and
include `X-Request-ID`. Responses that expose identity or tokens include
`Cache-Control: no-store`.

| Method and path | Request | Success | Access and errors |
| --- | --- | --- | --- |
| `GET /customers/me/referral` | Session cookie | `200 {"code":"ABCD2345","invited_count":0}` | `401 UNAUTHENTICATED` invalid/expired session; `403 FORBIDDEN` non-customer role; `404 NOT_FOUND` missing linked customer; `503 SERVICE_UNAVAILABLE` missing backend configuration. No `X-CSRF-Token` is required because the call exposes no mutation; it may create the caller's own code. |
| `POST /auth/customer/verify-otp` | `{"phone":"+393331234567","code":"123456","referral_code":"ABCD2345"}` | Unchanged: `200 {"user_id":"<UUID>","role":"CUSTOMER","csrf_token":"<opaque>"}` and session cookie | Unchanged; `referral_code` is optional and can never change the success or failure of signup. `422` only when the value exceeds 32 characters. |

`referral_code` handling:

- normalized by trimming and upper-casing
- recognized only when it is exactly eight characters from the code alphabet;
  anything else is ignored
- an unknown but well-formed code is ignored
- only applied when this verify call **creates the user** and the requested
  role is `CUSTOMER`
- absent, empty, or `null` values are ignored

Because the signup response does not change, a stale invite link can never block
or fail a registration, and the caller cannot use the response to test which
codes exist.

Example link response:

```json
{
  "code": "ABCD2345",
  "invited_count": 2
}
```

The API returns the code, not a URL. The frontend composes the shareable link
from its own origin, so the backend needs no public-base-URL setting.

## Behavior and state transitions

1. A customer opens their account space. The frontend calls
   `GET /customers/me/referral`.
2. If the caller has no code yet, one is generated and inserted. Concurrent
   requests are safe: the insert uses `ON CONFLICT DO NOTHING`, and a caller
   that loses the race reads the winner's code. A code collision with another
   customer is retried up to five times.
3. The customer shares `<origin>/join/<CODE>`. The `/join/<code>` page stores the
   code in a first-party cookie for 30 days and redirects to the normal sign-in
   screen.
4. The visitor verifies their phone. The browser sends the stored code as
   `referral_code` on `verify-otp` and clears the cookie after a successful
   verification.
5. On a successful verify that creates a new customer, the backend resolves the
   code to the referrer's customer row and inserts the `customer_referrals` row
   **inside the same transaction** as the user, role, customer, and session
   inserts. A failed verification leaves no partial attribution.
6. `invited_count` is a live `COUNT` of rows where the caller is the referrer.
   It is derived, never stored, so it cannot drift.

### Edge cases

| Situation | Result |
| --- | --- |
| Repeated `verify-otp` for the same phone | No second row; the referee primary key and the "new user" condition both prevent it. |
| Code used by an existing account | Ignored; attribution only happens at account creation, never retroactively. |
| Cleaner (`/auth/worker/verify-otp`) sends a customer code | Ignored; worker signup is not part of this feature. |
| Unknown, malformed, or stale code | Signup succeeds exactly as without the parameter; no row, no error. |
| The referrer deletes their account | Their codes and edges cascade away. |
| The referee deletes their account | Their edge cascades away; the referrer's count decreases. |
| Two customers somehow share a code | Not possible: `referral_codes.code` is the primary key. |
| Self-referral using the same phone | Not possible: an existing phone never creates a new customer. Two separate accounts on two phones are indistinguishable from a genuine referral and are out of scope. |
| Concurrent first reads of the same customer | One code is stored; both responses return it. |

## Authorization and ownership

The referral link endpoint requires the `CUSTOMER` role with a valid session and
returns only the caller's own code and count. There is no endpoint that reads
another customer's referrals, and no endpoint lists referees. Support and admin
staff read the tables through existing private-data procedures, not a public
API.

## Security, privacy, and retention

- The code is not a secret and is not used for authentication. Its worst-case
  abuse — someone else's signup being attributed to the code owner — grants no
  benefit in this slice and is bounded by the one-row-per-referee constraint.
- The capture cookie contains only the public code. It is first-party,
  `SameSite=Lax`, 30 days, marked `Secure` on HTTPS, and cleared on successful
  verification. It stores no identifier of the visitor.
- Neither party is notified, and the referrer sees only an aggregate count, not
  which phone numbers signed up.
- Rows contain only two pseudonymous customer references, a public code, and a
  timestamp. They follow the retention, deletion, and legal-hold rules in
  [ADR 0002](../adr/0002-phase-1-personal-data.md).
- Nothing in this slice is financial, so no monetary record or audit trail is
  required yet; the later rewards feature must add its own.

## Verification

Backend (`back/`):

```bash
.venv/bin/alembic upgrade head
.venv/bin/alembic check
.venv/bin/pytest -q
.venv/bin/ruff check .
.venv/bin/ruff format --check .
.venv/bin/mypy app scripts
.venv/bin/python -m scripts.export_openapi --check
```

Frontend (`front/`):

```bash
npm run typecheck
npm run lint
npm run format:check
npm test
npm run contract:check
```

The referral tests are PostgreSQL integration tests and skip when
`DATABASE_URL` is unset.

## Related decisions

- [technical architecture §25](../technical_architecture.md) — the two referral
  concepts and why they stay separate
- [ADR 0008](../adr/0008-phase-1-boundaries-and-support.md) — Phase 1 boundary
  and the 2026-11-14 amendment
- [customer authentication](customer-authentication.md) — the `verify-otp`
  contract this slice extends
