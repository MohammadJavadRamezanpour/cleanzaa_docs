# Customer authentication (B03 backend)

## Scope

A verified phone number creates or signs in a customer whose account is
**pending activation** until the later profile/terms slice records their name,
service address, and acceptance of the current terms. The backend stores a
shared user identity, a `CUSTOMER` role, and a linked customer record. This
slice provides code delivery, verification, revocable sessions, rotation, and
role checks. Profile fields, terms acceptance, booking eligibility, account
closure, phone changes, and frontend screens belong to later work. A customer
account created here cannot book until those later requirements are met. The
`CUSTOMER` role is authentication/authorization context, not proof of account
activation or booking eligibility. This slice has no activation field or order
API; the future profile/terms slice must add the server-side activation gate.

The Phase 1 data scope comes from [ADR 0002](../adr/0002-phase-1-personal-data.md).

## API contract

All request and response bodies are JSON unless the response is `204`. Errors
use `{"error":{"code":"...","message":"..."}}` and include
`X-Request-ID`. The endpoints do not accept an `Idempotency-Key`; duplicate
code requests are controlled by a cooldown and rate limits. `Origin`, when
present, must match a configured CORS origin.

| Method and path | Request | Success | Access and errors |
| --- | --- | --- | --- |
| `POST /auth/customer/request-otp` | `{"phone":"+393331234567"}` | `202 {"status":"sent"}` | Public; `403` disallowed Origin, `422` invalid phone, `429` request limit with `Retry-After: 60`, `503` unavailable configuration, database, or SMS provider. Same response for new and existing accounts. |
| `POST /auth/customer/verify-otp` | `{"phone":"+393331234567","code":"123456"}` | `200 {"user_id":"<UUID>","role":"CUSTOMER","csrf_token":"<opaque>"}` and session cookie | Public; `401` wrong, expired, exhausted, or reused code; `403` disallowed Origin; `422` malformed input; `503` missing backend configuration. |
| `GET /me` | No body; `cleanzza_session` cookie | `200 {"user_id":"<UUID>","roles":["CUSTOMER"],"csrf_token":"<opaque>"}` | Any authenticated role; `401` invalid/expired/revoked session; `503` missing backend configuration. |
| `GET /auth/customer/session` | No body; session cookie | `200` with the same fields as verify | `401` invalid session; `403` user lacks `CUSTOMER` role; `503` missing backend configuration. |
| `POST /auth/session/refresh` | Session cookie plus `X-CSRF-Token` from `/me` or prior auth response | `200 {"csrf_token":"<new opaque>"}` and rotated session cookie | `401` invalid/expired session; `403` invalid/missing CSRF token or disallowed Origin; `503` missing backend configuration. |
| `POST /auth/logout` | Session cookie plus `X-CSRF-Token` | `204` and cookie deletion | `401` invalid/expired session; `403` invalid/missing CSRF token or disallowed Origin; `503` missing backend configuration. Revokes the current device session only. |

Phones must arrive in canonical E.164 form (`+` followed by 8–15 digits) and
are uniquely constrained in `users`; other formats are rejected rather than
guessed. Codes are exactly six digits. All auth responses include
`Cache-Control: no-store` where they expose identity or tokens. The code and
session token never appear in request logs or error messages.

## Security and state

- The code is generated with a cryptographic random generator and stored only
  as an HMAC using `AUTH_SECRET`. It expires after five minutes and is consumed
  once under a PostgreSQL row lock. Five wrong guesses exhaust the challenge.
- A new code request is refused within 60 seconds of the prior request. Limits
  are five requests per phone and 20 per observed client IP in one hour.
  PostgreSQL advisory locks serialize limit checks and issuance. The IP is
  stored only as a keyed HMAC. Staging and production require
  `TRUSTED_PROXY_CIDRS`; only peers in those networks may supply a forwarded
  client address. The rightmost untrusted hop is used, so an earlier forged
  address is ignored. Auth issuance returns `503` if this trust configuration
  is missing outside local development.
- An opaque random session token is stored as a SHA-256 digest. The HTTP-only,
  host-only cookie is `Secure` outside local development. It uses `SameSite=None`
  for the separate frontend/API staging origins and `Strict` locally. CORS
  allows credentials only for configured frontend origins. Mutating
  authenticated operations require an `X-CSRF-Token`; `/me` provides a fresh
  usable value after reload without exposing the session cookie to JavaScript.
- Sessions last seven days, can be rotated with `/auth/session/refresh`, and
  cannot extend beyond 30 days from initial sign-in. Refresh invalidates the
  prior token immediately. Logout revokes only the current session. Server-side
  role checks query `user_roles` on each authenticated request.
- A scheduled cleanup deletes OTP challenge rows older than one day and session
  records 30 days after revocation or expiration. The phone itself stays on an
  active user account; account closure/deletion is a later feature.

The synchronous SMS adapter uses Twilio's Ireland region and requires
`TWILIO_ACCOUNT_SID`, `TWILIO_API_KEY_SID`, `TWILIO_API_KEY_SECRET`, and
`TWILIO_FROM_NUMBER`. Missing credentials or a delivery error fails closed with
`503` and rolls back the challenge. The adapter is a proposed provider choice;
provider privacy and contract review, credentials, and trusted-proxy settings
must be completed before production activation. Browser testing is required
for the deployed frontend/API domain pair because third-party-cookie policies
can affect cross-site sessions.
Only the phone number and message needed for delivery go to the SMS provider;
user, challenge, and session records stay in Cleanzza-controlled storage under
[ADR 0002](../adr/0002-phase-1-personal-data.md). The proposed Render/Vercel
staging deployment must use synthetic data until its data-handling review is
complete.

## Database and rollout

Migration `0003_customer_auth` creates `users`, `user_roles`, `otp_challenges`,
and `auth_sessions`, and adds nullable, unique `customers.user_id`. The column
is nullable so pre-existing minimal customer rows remain valid. New verified
customers receive a linked user and role in one transaction. Rollback of the
migration drops authentication data and must not be used after real signups.

The deployment must set the auth and SMS secrets, apply the migration, and run
the hourly auth cleanup cron. No real SMS is sent in tests; the provider
boundary is tested with a captured HTTP request.

## Verification

From `back/`: `.venv/bin/alembic upgrade head`, `.venv/bin/pytest -q`,
`.venv/bin/ruff check .`, `.venv/bin/ruff format --check .`,
`.venv/bin/mypy app scripts`, and
`.venv/bin/python -m scripts.export_openapi --check`.
