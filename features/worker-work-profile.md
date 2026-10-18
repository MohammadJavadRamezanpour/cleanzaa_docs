# Worker services and work profile (B05)

## Purpose and implemented scope

A pending worker can save one structured work profile: the Milano service area,
a short experience description, equipment and supplies declarations, and
offered Home Cleaning, Car Wash, or Pet Care services. Home Cleaning stores a
beginner or professional declaration. Car Wash and Pet Care may reference a
ready private certificate file uploaded through B06. Matching can query these
columns and rows without parsing free text. The catalogue identifier is
`milano-v1`, aligned with [ADR 0003](../adr/0003-phase-1-launch-services.md).

This captures worker declarations, not approval. B07 must review identity,
Home Cleaning level, and Car Wash/Pet Care evidence before an offering is
eligible. `job_eligible` remains false; no job matching or acceptance endpoint
exists yet. The browser editor and certificate association are part of this B05
delivery; private bytes and reviewer access remain governed by
[private worker evidence](private-worker-evidence.md).

## Mobile browser flow

The `/worker` profile loads the current work profile with the legal-name
profile. In the narrow mobile shell, the worker selects Milano, describes work
experience, explicitly chooses yes/no for equipment and supplies, and selects
one or more services. Home Cleaning exposes a beginner/professional level.
Car Wash and Pet Care show whether a private certificate is attached. The
certificate upload step returns a ready file ID and selects the corresponding
service in the unsaved form; the worker then saves work details to link it.
The form shows loading, validation, save-success, and error states. The file
stays in memory only for the upload attempt and is not stored by the browser.
Editing is disabled for a nonpending review state until B07 defines re-review.

## API contract

All responses use the shared error envelope and `X-Request-ID`. An authenticated
`CLEANER` role and HTTP-only session cookie are required. `PUT` also requires
an allowed `Origin` and `X-CSRF-Token`. These paths take no worker ID, query,
pagination, ordering, or filtering parameters; ownership comes from the
session.

| Method and path | Request | Success | Errors |
| --- | --- | --- | --- |
| `GET /cleaners/me/work-profile` | Cookie only | `200` current work profile with `Cache-Control: no-store` | `401` invalid session, `403` no cleaner role, `404` missing cleaner, `503` database unavailable. Before first save, nullable fields are `null`, `services` is `[]`, and `catalog_version` is `milano-v1`. |
| `PUT /cleaners/me/work-profile` | Cookie, CSRF, full JSON body below | `200` replacement profile with `Cache-Control: no-store` | `401`, `403` role/CSRF/Origin, `404` missing cleaner or certificate owned by another worker, `409` nonpending cleaner or certificate not ready, `422` invalid input or certificate purpose, `503` database unavailable. |

Example `PUT` body:

```json
{
  "service_area_code": "IT-MI-MILANO",
  "experience_description": "I have worked with local households for three years.",
  "can_bring_equipment": true,
  "can_bring_supplies": false,
  "services": [
    {"service_code": "HOME_CLEANING", "home_level": "PROFESSIONAL"},
    {"service_code": "CAR_WASH", "certificate_file_id": "<ready-owned-UUID>"}
  ]
}
```

The response adds `catalog_version: "milano-v1"` and returns every offering
with `service_code`, nullable `home_level`, and nullable
`certificate_file_id`, ordered Home Cleaning, Car Wash, Pet Care. The area code
is exactly `IT-MI-MILANO`. Experience is trimmed and must contain 10–1000
characters. Equipment and supplies fields are mandatory booleans. `services`
contains one to three distinct codes from `HOME_CLEANING`, `CAR_WASH`, and
`PET_CARE`. Home Cleaning requires `BEGINNER` or `PROFESSIONAL` and cannot
reference a certificate. Car Wash and Pet Care cannot use `home_level` and
may reference a certificate with the corresponding
`CERTIFICATE_CAR_WASH`/`CERTIFICATE_PET_CARE` purpose. Unknown fields are
rejected. A missing certificate is allowed while onboarding remains pending.

`PUT` replaces the complete profile and offering list. A repeated identical
request returns the same representation; no `Idempotency-Key` is needed for
this value-setting command. Updating a nonpending cleaner returns `409` until
B07 defines re-review transitions. Concurrent writes lock the cleaner row and
serialize; the last successful full replacement wins. Profile data is private
to the worker and authorized operations, with no public listing.

## Storage, migration, and rollout

Migration `0007_worker_work_profile` adds `cleaner_work_profiles` and
`cleaner_offerings`; it does not rewrite existing cleaner rows. Database
constraints enforce the one-area choice, service codes, Home Cleaning level,
and certificate prohibition for Home Cleaning. The offering certificate UUID
is a foreign key to B06 `stored_files`; the application validates ownership,
`READY` state, and purpose in the same PostgreSQL transaction before linking.
The referenced private bytes stay under B06 access and retention rules. A
missing or pending file cannot be linked. Down migration deletes these worker
declarations and must not be used after real onboarding without a data plan.

The profile does not publish worker details or approve qualifications. B07 owns
reviewer decisions and eligibility; B08 owns the shared catalog API and future
version changes. Until real identity and certificate retention reviews are
complete, use only synthetic evidence in development or controlled staging.

From `back/`, run `.venv/bin/alembic upgrade head`, `.venv/bin/alembic check`,
`.venv/bin/pytest -q`, `.venv/bin/ruff check .`,
`.venv/bin/ruff format --check .`, `.venv/bin/mypy app scripts`,
`.venv/bin/python -m scripts.export_openapi --check`, `.venv/bin/pip-audit`,
and `docker build -t cleanzza-api:local .`. PostgreSQL integration tests cover
profile replacement, duplicate services, Milano validation, role/CSRF checks,
approval edit restrictions, and certificate ownership, readiness, and purpose.
The frontend pins the generated OpenAPI contract and runs lint, format,
typecheck, tests, contract check, and production build in CI.
