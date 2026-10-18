# Worker live identity capture and evidence status (B04)

## Scope

A pending worker can use the mobile camera flow to capture a live selfie and
two document images: ID card front/back or passport identity/required second
page. Each image uses B06 private upload and is linked to the worker after it
is `READY`. The worker sees capture status afterward, never a raw download.
A complete set means three file references exist, not that identity is
approved. B07 requires manual review and an audited decision before jobs.

This slice does not prove legal identity, cryptographic liveness, work
authorization, worker terms acceptance, or job eligibility. The API cannot
prove camera origin; a caller can upload a synthetic image directly. A reviewer
must assess evidence. ADR 0002 still requires a decision on the exact passport
second page, reviewer procedure, retention, provider, region, and legal terms.
Use synthetic evidence only until real collection is approved.

## Mobile flow

The browser opens the user-facing camera for selfie and prefers the rear
camera for documents. It captures a JPEG frame in memory and uses B06's
signed upload and idempotent finalization. The identity flow has no gallery
file input. It shows recorded/missing states, upload progress, errors, retry,
and a save control. Reloading reads saved references. Changing document type
clears the old document references on the next save but keeps the selfie.
Previously uploaded private files remain subject to B06 retention rules.

## API contract

Both endpoints need an HTTP-only worker session and database-backed `CLEANER`
role, derive ownership from the session, send `Cache-Control: no-store`, and
use the standard error envelope and `X-Request-ID`. There is no worker ID,
listing, pagination, or filtering.

| Method and path | Request | Success | Errors |
| --- | --- | --- | --- |
| `GET /cleaners/me/identity-evidence` | Session cookie | `200` current metadata; initially all fields are null and `complete=false` | `401` session, `403` role, `404` missing cleaner, `503` database. |
| `PUT /cleaners/me/identity-evidence` | Session cookie, allowed Origin, `X-CSRF-Token`, full JSON body | `200` replaced metadata and `complete` | `401`, `403` role/CSRF/Origin, `404` missing cleaner/foreign file, `409` nonpending cleaner/file not ready, `422` invalid or mismatched input, `503` database. |

Example body:

```json
{
  "document_type": "ID_CARD",
  "selfie_file_id": "<ready-LIVE_SELFIE-UUID>",
  "primary_file_id": "<ready-ID_FRONT-UUID>",
  "secondary_file_id": "<ready-ID_BACK-UUID>"
}
```

`document_type` is `ID_CARD` or `PASSPORT`. File IDs are nullable to save
partial capture progress. An ID card requires `ID_FRONT` and `ID_BACK`;
a passport requires `PASSPORT_IDENTITY` and `PASSPORT_SECONDARY`. Both use
`LIVE_SELFIE`. References must be distinct, owned by the worker, `READY`,
and purpose-matched in one PostgreSQL transaction. The response repeats the
fields and adds `complete`, true when all three IDs exist. Unknown fields
are rejected. `PUT` replaces the full record under a cleaner row lock, so
repeating a body is safe and the last serialized write wins. Nonpending
workers cannot edit until B07 defines re-review. No `Idempotency-Key` is
needed for this value-setting command.

## Storage and rollout

Migration `0008_worker_identity_evidence` adds one row per cleaner with
foreign keys to B06 private file records, not image bytes. Workers see status
only. Designated admins need a purpose-bound short-lived grant for raw access,
and every successful read is audited. Down migration removes evidence links
and is unsafe after real collection.

The frontend pins the generated backend OpenAPI. Backend checks are Alembic
upgrade/check, pytest, Ruff, mypy, OpenAPI export check, pip-audit, and Docker
build; frontend checks are lint, format, typecheck, tests, contract check, and
production build. PostgreSQL tests cover ownership, READY/purpose checks,
partial/complete status, CSRF, and pending-only editing. A real device camera
and configured private store still need staging verification before release.
