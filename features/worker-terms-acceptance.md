# Versioned worker terms acceptance (B04)

## Purpose and implemented scope

A worker with a verified phone and `CLEANER` role can view the currently
published worker terms and explicitly accept that version. The backend stores
one durable acceptance per worker and version, with the immutable document URL
and acceptance time. This gives B07 a server-side fact to check before manual
approval and future job eligibility. It does not approve identity, service
qualifications, or job access. `job_eligible` stays false in this B04 slice.

No terms are published by code or these docs. A policy owner must approve an
immutable worker terms document and its public HTTPS URL before configuring
`WORKER_TERMS_VERSION` and `WORKER_TERMS_URL` together. The API returns `503`
for current terms while the pair is absent. A new published version needs a
new version identifier and URL; old acceptance records remain unchanged.

## API contract

Responses use the standard error envelope and `X-Request-ID`. There are no
path or query parameters, pagination, sorting, or filtering.

| Method and path | Request | Success | Errors |
| --- | --- | --- | --- |
| `GET /terms/worker/current` | None; public | `200 {"version":"worker-2026-10","url":"https://..."}` with `Cache-Control: no-store` | `503` terms not configured. |
| `GET /cleaners/me/profile` | Worker session cookie | `200` current private profile including `accepted_terms_version`, offset timestamp `accepted_terms_at`, and `current_terms_version` | `401` missing session, `403` missing worker role, `404` missing worker, `503` database. |
| `POST /cleaners/me/terms-acceptances` | Worker session cookie, allowed Origin, `X-CSRF-Token`, JSON body below | `200` updated private profile, `Cache-Control: no-store` | `401`, `403` role/CSRF/Origin, `404` missing worker, `409` stale version, `422` invalid body, `503` terms or database unavailable. |

Example request:

```json
{"terms_version":"worker-2026-10","accepted":true}
```

`accepted` must be literal `true`; no extra fields are allowed. A caller
cannot submit a worker ID. The authenticated session chooses the owner. The
version must exactly match the configured current version. A repeat request
for the same worker/version returns the original acceptance timestamp and
does not create another record, so no `Idempotency-Key` is needed. A stale
version returns `409`, prompting the browser to fetch and display the new
published terms. Profile responses show the latest recorded acceptance, even
when it differs from `current_terms_version`; a later approval policy must
require equality with the current version.

## Mobile browser flow

The `/worker` profile shows worker terms as the final onboarding section. If
terms are not yet published, it explains that the worker can save the profile
and return later. When terms are available, the worker opens their HTTPS
document, checks an explicit acceptance box, and submits. The UI shows the
accepted version afterward. If the version changes before submission, it
clears the checkbox and loads the current version. This screen remains in the
same narrow mobile layout on desktop.

## Storage, security, and rollout

Migration `0009_worker_terms_acceptance` adds
`cleaner_terms_acceptances` with a `(cleaner_id, terms_version)` primary key
and worker foreign key. It is additive; rollback deletes acceptance history
and must not be used after real acceptance without a data plan. The stored
document URL is a version-specific immutable public document, not a private
identity image. The browser receives neither a reusable auth token nor
another worker's acceptance. Terms are not personally sensitive, but the
acceptance record belongs to the worker and should follow the approved
account-data retention and export policy in B26.

Backend verification: `.venv/bin/alembic upgrade head`,
`.venv/bin/alembic check`, `.venv/bin/pytest -q`, `.venv/bin/ruff check .`,
`.venv/bin/ruff format --check .`, `.venv/bin/mypy app scripts`,
`.venv/bin/python -m scripts.export_openapi --check`, `.venv/bin/pip-audit`,
and `docker build -t cleanzza-api:local .`. PostgreSQL tests cover owner
isolation, current-version validation, CSRF, durable idempotency, and pending
job eligibility. Frontend verification runs lint, formatting, typecheck,
tests, contract check, and production build. Backend OpenAPI and the frontend
pinned TypeScript contract are synchronized in the same delivery.
