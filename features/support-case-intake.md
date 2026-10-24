# Support case intake

## Purpose and current scope

The first Phase 1 support slice lets a signed-in customer or cleaner record a general service, payment, safety, or other concern without a Before Photo, special guarantee, or 24-hour deadline. It stores the case and an attributed creation event. The creator can list their own cases. This slice has no case attachments, order link, staff queue, response, resolution command, notification, or frontend screen yet. It does not promise live support or an emergency response. Those are separate delivery and operating requirements in [ADR 0008](../adr/0008-phase-1-boundaries-and-support.md).

`SUPPORT_CASES_ENABLED` defaults to `false`. Production configuration rejects `true` while the retention and legal-hold process, staffed queue, escalation path, and customer wording are unresolved. Controlled local or staging tests use synthetic descriptions. The flag does not clear any launch gate in [ADR 0008](../adr/0008-phase-1-boundaries-and-support.md) or [ADR 0002](../adr/0002-phase-1-personal-data.md).

## API contract

`POST /support/cases` requires a valid customer or cleaner session cookie, matching `X-CSRF-Token`, an allowed `Origin` when supplied, and `Idempotency-Key` (1–128 characters). The JSON body has exactly `creator_role` (`CUSTOMER` or `CLEANER`), `category` (`SERVICE`, `PAYMENT`, `SAFETY`, or `OTHER`), and `description` (10–2000 characters after trimming). The requested role must be present in the authenticated user's roles. A user with both roles chooses which role the case represents. No order ID, file, contact detail, payment instruction, or requested remedy is accepted in this slice.

The response is `201` with `id`, `creator_role`, `category`, `description`, `status: "OPEN"`, and an explicit-offset ISO 8601 `created_at`. Retrying the same key and normalized body for the same user returns the original `201` response. Reusing the key with different content returns `409 CONFLICT`. Missing or invalid authentication returns `401 UNAUTHENTICATED`; role mismatch or failed CSRF/origin check returns `403 FORBIDDEN`; malformed or extra fields, invalid category, missing key, or an invalid description return `422 VALIDATION_ERROR`. When the feature is disabled or no database session is configured, the endpoint returns `503 SERVICE_UNAVAILABLE`.

`GET /support/cases` requires a valid customer or cleaner session and returns only that user's cases, newest first, as `{ "items": [...], "next_cursor": UUID | null }`. Optional `cursor` is a case UUID from a previous page and must belong to the same user; another user's or an unknown cursor returns `404 NOT_FOUND`. Optional `limit` defaults to 20 and permits 1–50. The response uses `Cache-Control: no-store`; there is no cross-user case search. Both endpoints use the shared safe error envelope. They expose no payment or order state changes.

## Persistence and safety

Migration `0013_support_case_intake` adds `support_cases` and `support_case_events`. A case references the creating user and records role, category, description, OPEN status, idempotency key, request fingerprint, and server timestamp. PostgreSQL checks constrain the role, category, OPEN-only status, and description length. A unique `(creator_user_id, idempotency_key)` constraint and transaction-scoped advisory lock make duplicate creation safe; the creation event is in the same transaction. The event references the case and actor, permits only `CREATED`, and uses a server timestamp. Future case actions must expand the state and event constraints in a reviewed migration.

Descriptions may contain sensitive personal information. They are not logged by the request logger or returned to other users. The schema uses restricted foreign keys until the approved export, retention, deletion, and legal-hold rules are implemented. Do not drop these tables after real cases exist without a data-preservation decision. No staff read or case outcome is enabled by this slice, so a populated case cannot yet be treated as operationally handled.

## Verification

From `back/`, run `.venv/bin/alembic upgrade head`, `.venv/bin/alembic check`, `.venv/bin/pytest -q tests/test_support_cases.py`, `.venv/bin/pytest -q`, `.venv/bin/ruff check .`, `.venv/bin/ruff format --check .`, `.venv/bin/mypy app scripts`, and `.venv/bin/python -m scripts.export_openapi --check` against PostgreSQL. The tests cover auth, CSRF, role ownership, sequential and concurrent retries, changed-key conflicts, creator-only reads, event creation, normalized description limits, and the default-disabled gate. The generated OpenAPI artifact and frontend pinned contract must be updated together before delivery.
