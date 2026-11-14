# Controlled demo phone login

## Purpose and scope

The protected synthetic preview lets a reviewer enter any valid E.164-format
**test** phone number and sign in as a customer or worker without an SMS
provider. The API creates the normal single-use challenge and returns its
six-digit code to the requesting browser. The frontend displays the code on
the verification screen. No message is sent, and this flow does not prove
control of the phone number or a worker's identity.

The mode is off by default. Backend startup rejects `DEMO_AUTH_ENABLED=true`
unless `ENVIRONMENT=demo`. The hosted preview uses a dedicated synthetic
database and access control. The frontend labels the entire site as a demo and
states that no worker is dispatched, service booked, or card charged. The
frontend build sets `NEXT_PUBLIC_DEMO_AUTH_ENABLED=true` to show that copy;
the backend setting is the authority for issuing a demo code. This mode must
never be enabled with real customer or worker records.

## API contract

All bodies are JSON. `Origin`, when supplied, must match `CORS_ORIGINS`.
Phones use canonical E.164 format (`+` followed by 8–15 digits). The existing
standard error envelope and `X-Request-ID` apply. No query or path parameters
or `Idempotency-Key` are accepted.

| Method and path | Request | Demo success | Errors |
| --- | --- | --- | --- |
| `POST /auth/customer/request-otp` | `{"phone":"+390000000001"}` | `202 {"status":"demo","demo_code":"123456"}`; `Cache-Control: no-store` | `403` disallowed Origin, `422` invalid phone, `429` rate limit with `Retry-After: 60`, `503` unavailable database, secret, or proxy configuration |
| `POST /auth/worker/request-otp` | Same shape | Same shape | Same errors |
| `POST /auth/customer/verify-otp` | `{"phone":"+390000000001","code":"123456"}` | `200` customer session and secure, HTTP-only cookie | `401` wrong, expired, exhausted, or reused code; `403` disallowed Origin; `422` invalid input; `503` unavailable configuration |
| `POST /auth/worker/verify-otp` | Same shape | `200` worker session and secure, HTTP-only cookie | Same errors |

When demo mode is off, the request endpoints keep their existing
`202 {"status":"sent"}` response and send the code through the configured
SMS provider. They never return a code in that mode. Session, profile, role,
CSRF, and logout contracts remain those in [customer authentication](customer-authentication.md)
and [worker phone profile](worker-phone-profile.md).

## State and safeguards

- A code is random, HMAC-stored with `AUTH_SECRET`, valid for five minutes,
  consumed once, and limited to five guesses. Customer and worker challenges
  remain purpose-scoped. Existing phone and observed-IP request limits apply.
- A demo response reveals the code only to the caller that requested it. The
  response is not cached, and request logs do not record request or response
  bodies. The mode never calls the SMS adapter.
- Session cookies remain `Secure` and HTTP-only. Browser requests use the
  existing CSRF and role checks. Outside local development,
  `TRUSTED_PROXY_CIDRS` must identify the actual reverse-proxy peer so the
  server can enforce rate limits using the client address.
- Anyone with preview access can request a code for any test number. Use only
  synthetic data, and do not reuse a live database. A demo sign-in does not
  approve a worker, activate terms, or enable real orders or payments.
- The hourly auth cleanup removes expired challenges and stale sessions as
  described in the existing authentication contract.

No database migration is required. The backend OpenAPI export and frontend
pinned contract must be updated together. Verify with the backend PostgreSQL
integration suite, OpenAPI check, frontend tests, contract check, and a browser
sign-in through the protected HTTPS preview.
