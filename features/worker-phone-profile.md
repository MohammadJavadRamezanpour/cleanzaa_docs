# Worker phone sign-in and pending profile (B04 first slice)

## Purpose and scope

A worker can register or sign in with a single-use phone code, recover a
revocable session, and create or edit their legal first and last name. The
backend stores a `CLEANER` role and an owned cleaner profile in `PENDING`
review status. A person can hold both customer and cleaner roles on the same
verified phone number. The roles do not grant each other automatically.

Phone verification alone does **not** grant job eligibility. The response
returns `job_eligible: true` only after the separate
[review decision](worker-review-decisions.md) approves the worker and the
current terms version remains accepted. Private certificate storage and a test-file upload
screen exist under B06; B05 adds editable services, Milano area, experience,
equipment/supplies declarations, and certificate association. Live identity
capture now uses the [private evidence status](worker-identity-evidence.md)
flow. Versioned [worker terms acceptance](worker-terms-acceptance.md) is now
available when approved terms are configured. No job-offer or job-acceptance API
exists. Real identity evidence remains subject to
[ADR 0002](../adr/0002-phase-1-personal-data.md); service choices follow
[ADR 0003](../adr/0003-phase-1-launch-services.md).

## API contract

All request and success bodies are JSON. Errors use
`{"error":{"code":"...","message":"..."}}` with `X-Request-ID` and the
standard `UNAUTHENTICATED`, `FORBIDDEN`, `NOT_FOUND`, `CONFLICT`,
`VALIDATION_ERROR`, `RATE_LIMITED`, or `SERVICE_UNAVAILABLE` codes. Identity
and session responses have `Cache-Control: no-store`. A supplied `Origin` must
match configured CORS origins.

| Method and path | Request | Success | Access and errors |
| --- | --- | --- | --- |
| `POST /auth/worker/request-otp` | `{"phone":"+393331234567"}` | `202 {"status":"sent"}` | Public; `403` disallowed Origin, `422` invalid phone, `429` rate limit with `Retry-After: 60`, `503` auth/database/SMS unavailable. New and existing numbers receive the same response. |
| `POST /auth/worker/verify-otp` | `{"phone":"+393331234567","code":"123456"}` | `200 {"user_id":"<UUID>","role":"CLEANER","csrf_token":"<opaque>"}` and session cookie | Public; `401` wrong, expired, reused, exhausted, or customer-purpose code; `403` disallowed Origin; `422` malformed input; `503` unavailable. |
| `GET /auth/worker/session` | Session cookie | `200` same response as verify | `401` missing/revoked/expired session; `403` missing cleaner role; `503` unavailable. |
| `GET /cleaners/me/profile` | Session cookie | `200` profile response below | `401` invalid session; `403` missing cleaner role; `404` missing linked profile; `503` unavailable. |
| `PATCH /cleaners/me/profile` | Session cookie, `X-CSRF-Token`, non-empty JSON subset of `first_name`, `last_name` | `200` updated profile response | Same auth errors; `403` bad CSRF or Origin; `409` profile no longer pending; `422` empty, blank, overlong, null, or unknown fields. |

The shared `GET /me`, `POST /auth/session/refresh`, and `POST /auth/logout`
endpoints use the same session cookie and CSRF rules described in
[customer authentication](customer-authentication.md). The worker path uses no
path, query, or optional pagination parameters. It does not accept
`Idempotency-Key`: OTP requests are rate-limited, verification is single-use,
and profile `PATCH` sets explicit field values.

Example new profile:

```json
{"first_name":null,"last_name":null,"review_status":"PENDING","job_eligible":false,"accepted_terms_version":null,"accepted_terms_at":null,"current_terms_version":null}
```

`PATCH` trims names and permits a non-empty subset. Each name is at most 80
characters. A pending profile can be edited repeatedly. Status and eligibility
are read-only; a caller cannot self-approve by including either field. The
server derives the profile owner from the current session, never from a
caller-supplied worker ID. Cross-user reads and edits are impossible through
these paths.

## Security, state, and data

Phone numbers use canonical E.164 form. Codes are six digits, expire in five
minutes, are HMAC-stored, permit at most five guesses, and are consumed once.
Phone and IP request limits are shared across customer and worker flows. A
new request invalidates the latest unused code for that phone. The challenge
stores its intended role, so a customer-purpose code cannot enroll a worker.
PostgreSQL advisory locks serialize issuance and redemption for a phone.
Worker verification adds the `CLEANER` role and pending profile in the same
transaction; it leaves an existing customer profile and role intact.

Session cookies are HTTP-only, secure outside local development, and
revocable. Authenticated writes require a matching CSRF header and allowed
Origin. The server reads roles from PostgreSQL per request. Phone verification
only proves control of the number; it does not establish identity or approval.
The profile stores legal names, phone association through `users`, and no
identity image bytes. Log handlers do not record request bodies or codes.

## Mobile browser flow

The frontend exposes `/worker` in the same narrow, mobile-first shell as the
customer area. It restores a worker session on load, otherwise asks for a phone
number and then a six-digit code. An authenticated customer without the worker
role can verify their number to add pending worker access. The profile screen
edits legal first and last names, shows the review state, and offers the
[work profile](worker-work-profile.md) and
[private certificate upload](private-worker-evidence.md). The
[identity capture](worker-identity-evidence.md) section saves private evidence
status; approval remains a later step. The worker terms section records explicit
acceptance only after approved terms are published. Sign-out revokes the current
session. The frontend never stores the session cookie or private profile in
local storage; requests use the HTTP-only cookie and CSRF token.
The customer screen similarly offers phone verification to a worker-only
session, so the same user can add either role without first signing out.

Migration `0005_worker_phone_profile` gives existing OTP rows the `CUSTOMER`
purpose and creates `cleaners` with a unique user link, nullable names, and a
constrained review status. It is additive for existing customer accounts.
Downgrading would delete worker profiles and purpose separation and is unsafe
after real worker registration. The review flow computes eligibility server-side;
future job APIs must check it transactionally at assignment time.

## Verification and rollout

From `back/`, apply `.venv/bin/alembic upgrade head` to PostgreSQL, then run
`.venv/bin/alembic check`,
`.venv/bin/pytest -q`, `.venv/bin/ruff check .`,
`.venv/bin/ruff format --check .`, `.venv/bin/mypy app scripts`,
`.venv/bin/python -m scripts.export_openapi --check`, `.venv/bin/pip-audit`,
and `docker build -t cleanzza-api:local .`. Worker integration tests cover
role-purpose separation, multi-role accounts, profile ownership, CSRF, session
recovery, validation, and pending eligibility. The frontend pins the generated
OpenAPI schema and checks its client in CI.

Production SMS, privacy, hosting, and real-identity collection gates in ADR
0002 still apply. Development and staging should use synthetic worker data
until provider, regional, and retention reviews are complete.
