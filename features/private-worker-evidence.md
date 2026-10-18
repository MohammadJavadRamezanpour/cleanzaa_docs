# Private worker evidence (B06 first slice)

## Purpose and scope

This slice provides a private S3-compatible upload path for a pending worker's
live identity images and service certificate evidence. It records ownership and
purpose in PostgreSQL, verifies an uploaded object before marking it ready,
and gives an `ADMIN` reviewer one-use, short-lived access with an audit event
for each successful read. The worker sees only file status, never a durable raw
download. The `/worker` browser screen now offers certificate upload for Car
Wash and Pet Care. Live camera capture and editable service selection remain
B04/B05 work.

This is **not all of B06**. Service photos require an order and case model;
correction documents require the verified-correction flow and its own retention
schedule. Case-scoped support reads, service-photo seven-day deletion and
holds, malware review, worker identity retention, backup expiry, and a
production-ready regional/provider configuration remain open. The application
rejects production `PRIVATE_EVIDENCE_ENABLED=true` until ADR 0002's retention
review is implemented. Use only synthetic evidence in controlled development
or staging. The policy is [ADR 0002](../adr/0002-phase-1-personal-data.md);
the generic storage direction is in the
[technical architecture](../technical_architecture.md).

## API contract

All API responses use the standard error envelope
`{"error":{"code":"...","message":"..."}}` and `X-Request-ID`. They use
`Cache-Control: no-store` where they contain file data, metadata, or a token.
Mutating endpoints require an allowed `Origin`, the HTTP-only session cookie,
and `X-CSRF-Token`. The worker must have a database-backed `CLEANER` role;
review access requires `ADMIN`. A caller never chooses the owner or object key.

| Method and path | Request | Success | Access and errors |
| --- | --- | --- | --- |
| `POST /files/worker-uploads` | Session cookie, CSRF, JSON purpose, `content_type`, `size_bytes`, lowercase hex `sha256` | `201` upload ticket below | Worker; `401`, `403`, `422` invalid purpose/type/size/hash, `503` storage disabled or unavailable. |
| `GET /files/{file_id}` | UUID path, session cookie | `200 {"file_id":"<UUID>","purpose":"ID_FRONT","status":"PENDING"}` | Owner worker only; `401`, `403`, `404` other owner/missing, `503`. No raw URL. |
| `POST /files/{file_id}/finalize` | UUID path, session cookie, CSRF, mandatory `Idempotency-Key` header (1–128 chars), no body | `200` status with `READY` | Owner worker only; `401`, `403`, `404`, `409` missing/expired object or key conflict, `422` invalid stored content or missing header, `503`. Same actor, key, and file retry returns the prior success. |
| `POST /files/{file_id}/review-access` | UUID path, admin session, CSRF, `{"reason":"worker identity review"}` | `200 {"token":"<opaque>","expires_in_seconds":60}` | Admin only; `401`, `403`, `404`, `409` file not ready, `422` reason not 10–200 trimmed characters, `503`. Issuance alone creates no read audit. |
| `GET /files/review-access/content` | Admin session and `X-File-Access-Token` header | `200` original bytes with stored media type | Same admin actor who requested access; `401`, `403`, `404` missing/used/expired/other actor token, `409` object missing or changed, `422` missing token header, `503` storage unavailable. A successful response consumes the token and records the reason/actor/file/time. |

The ticket response contains `file_id`, `upload_url`, `fields`, and an
explicit-offset `expires_at`. The browser submits a `multipart/form-data` POST
directly to the private object store at `upload_url`, including every returned
field and the file. The signed policy binds the random server-generated key,
exact media type, SHA-256 checksum, and maximum form size. The signed form
expires after 10 minutes; the pending database record after 15 minutes. The
backend recomputes SHA-256, checks exact byte count and content signature, and
writes those verified bytes to a separate `ready/` object key before marking
the file `READY`. A signed form can write only to the `uploads/` key and cannot
overwrite reviewer evidence. A failed verification deletes the upload object
and pending metadata. There is no public bucket or permanent worker read URL.

Accepted purposes are `ID_FRONT`, `ID_BACK`, `PASSPORT_IDENTITY`,
`PASSPORT_SECONDARY`, `LIVE_SELFIE`, `CERTIFICATE_CAR_WASH`, and
`CERTIFICATE_PET_CARE`. Identity/selfie purposes accept JPEG or PNG up to
8 MiB. Certificate purposes accept JPEG, PNG, or PDF up to 10 MiB. The claimed
media type must match the file's signature; a certificate PDF is not an ID
capture. The API does not itself prove that an image came from a live camera;
the B04 browser capture flow will not offer gallery substitution, and B07's
human reviewer must still assess identity evidence.

No list endpoint, pagination, ordering, or filtering is provided in this
slice. Reviewer selection by worker and approved service belongs to B07.

## Mobile certificate upload

On `/worker`, a signed-in worker selects Car Wash or Pet Care and a JPEG, PNG,
or PDF certificate of at most 10 MiB. The mobile form hashes the file locally,
requests a private ticket with the worker session and CSRF token, submits the
returned form fields and file directly to storage, and finalizes with a random
`Idempotency-Key`. It shows preparing, percentage upload progress, finalizing,
success, and failure states. A retry after an interrupted storage transfer
reuses the pending ticket; a retry after a lost finalization response uses the
same file ID and idempotency key. An expired or missing upload gets a new
ticket. Changing the service or file starts a new attempt. The browser does
not receive a permanent read URL or store the file in local storage.

The screen instructs workers to use only test files until the real-evidence
policy is approved. Backend production collection remains disabled. This
certificate upload does not yet associate the file with an editable worker
service/qualification record; B05 will add that link. It does not claim live
ID/selfie capture, which belongs to B04.

## Database, idempotency, and cleanup

Migration `0006_private_worker_files` adds `stored_files`,
`file_finalizations`, `file_read_grants`, and `file_read_audit`. PostgreSQL
constraints limit purpose, status, and size. File ownership uses the verified
user ID. Finalization locks the owned file row and stores actor, key, request
fingerprint, result, and one-day idempotency expiry. Reusing a key for another
file returns `409`; a second key cannot finalize the same ready file. The
grant token is stored only as a SHA-256 digest, is actor-bound, expires in one
minute, and is consumed under a row lock. The token travels in a header so
request-path logs do not expose it. Each successful byte read creates an audit
row; issuing a grant or a failed read does not.

The hourly `scripts.cleanup_files` job deletes expired unfinalized objects and
rows, temporary upload objects after successful finalization, expired read
grants, and expired finalization records. It runs only when private evidence is
enabled. The storage bucket must be private, encrypted, deny public access,
and apply a safety lifecycle to the `uploads/` prefix for failures that occur
between object upload and database cleanup. The `ready/` prefix requires a
separate approved retention policy.
Ready identity/certificate objects do not yet have an approved retention and
backup deletion schedule. Production collection therefore remains disabled.
Downgrading migration 0006 deletes metadata and audit and is unsafe after
real evidence exists.

## Configuration and verification

`PRIVATE_EVIDENCE_ENABLED` defaults to false. When enabled outside production,
`S3_ENDPOINT`, `S3_BUCKET`, `S3_ACCESS_KEY`, and `S3_SECRET_KEY` must be set;
`S3_REGION` defaults to `eu-central-1`. Staging storage requires HTTPS. The
endpoint must be reachable both by the backend and by the worker's browser for
the direct upload. No credentials are committed to Git. Provider region,
contract, CORS, bucket encryption, lifecycle, and backup behavior must be
reviewed before using real data.

From `back/`: `.venv/bin/alembic upgrade head`, `.venv/bin/alembic check`,
`.venv/bin/pytest -q`, `.venv/bin/ruff check .`,
`.venv/bin/ruff format --check .`, `.venv/bin/mypy app scripts`,
`.venv/bin/python -m scripts.export_openapi --check`, `.venv/bin/pip-audit`,
and `docker build -t cleanzza-api:local .`. The PostgreSQL integration tests
cover owner isolation, role/CSRF enforcement, content verification, duplicate
finalization, short-lived one-use reviewer access, audit, and pending cleanup.
The S3 adapter policy test checks exact key, type, checksum, and size bounds.
An actual configured S3-compatible provider upload/read/delete exercise is
required before staging activation; tests with synthetic storage do not prove
provider configuration or bucket policy.
