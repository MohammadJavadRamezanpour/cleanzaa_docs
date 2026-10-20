# Worker review decisions and authorized actions (B07, B28)

## Scope

An allowlisted, managed-MFA administrator can list workers by review state, inspect one worker's profile and private evidence references, and record an approval, rejection, dismissal, or reinstatement. The backend is authoritative for transitions and job eligibility. This feature uses synthetic identity evidence until the legal, provider, and retention gates in [ADR 0002](../adr/0002-phase-1-personal-data.md) are cleared. [ADR 0005](../adr/0005-admin-managed-mfa.md) defines administrator authentication.

B28's implemented actions are dismissal and reinstatement through this same review endpoint. Fines and bonuses are deferred by [ADR 0008](../adr/0008-phase-1-boundaries-and-support.md). The decision schema accepts only the four values listed below, so `FINE` and `BONUS` return `422` without writing a decision or changing job eligibility. No financial adjustment endpoint or balance mutation exists. Amounts, appeal handling, legal basis, access, and retention must be approved before either action is designed or enabled.

## API

All paths require `Authorization: Bearer <Auth0 admin API access token>`. The token must pass the issuer, audience, client ID, RS256 signature, expiry, issued-at, MFA claim, exact subject allowlist, and local `ADMIN` role checks. A phone-code cookie is never sufficient. Responses set `Cache-Control: no-store`. The standard `{ "error": { "code", "message" } }` envelope applies.

| Method and path | Input | Success | Errors |
| --- | --- | --- | --- |
| `GET /admin/cleaners` | `status=PENDING|APPROVED|REJECTED|DISMISSED` (default `PENDING`), optional UUID `cursor`, `limit=1..50` (default 20) | `200 {"items":[{"cleaner_id":"UUID","first_name":"...","last_name":"...","review_status":"PENDING"}],"next_cursor":"UUID or null"}` | `401` unauthenticated, `403` unauthorized, `422` invalid query, `503` database unavailable. |
| `GET /admin/cleaners/{cleaner_id}/review` | UUID path | `200` review detail below | `401`, `403`, `404` unknown worker, `422` malformed UUID, `503`. |
| `POST /admin/cleaners/{cleaner_id}/decisions` | UUID path; `Idempotency-Key` header 1–128 chars; JSON decision body below; allowed `Origin` | `200` decision record below | `401`, `403`, `404`, `409` state/evidence/key conflict, `422` invalid body/key/UUID, `503`. |

The queue sorts by worker UUID ascending, returns at most `limit` items, and uses the last returned UUID as `next_cursor` when another page exists. A change in review state can remove an item between pages; clients should refresh the queue after decisions.

Review detail contains `cleaner_id`, legal `first_name` and `last_name`, `review_status`, `service_area_code`, `experience_description`, `can_bring_equipment`, `can_bring_supplies`, `document_type`, `identity_file_ids`, `services`, `current_terms_accepted`, and `last_decision`. The identity IDs are in selfie, primary document, secondary document order when present. Each service has `service_code`, optional `home_level`, and optional `certificate_file_id`. The last decision contains its ID, worker ID, local actor user ID, decision, resulting status, reason, and offset timestamp. Missing incomplete profile fields are `null`; the file ID array may be empty. The detail contains references, not raw bytes.

Decision JSON is `{"decision":"APPROVE","reason":"Identity and qualifications checked","reviewed_file_ids":["UUID"],"approved_service_codes":["CAR_WASH"]}`. `decision` is `APPROVE|REJECT|DISMISS|REINSTATE`; reason is trimmed and 10–500 characters. File IDs are at most six and service codes at most three. Unknown fields are rejected. The response contains `decision_id`, `cleaner_id`, `actor_user_id`, `decision`, `resulting_status`, `reason`, and `created_at` with an explicit offset.

## Transitions and evidence

| Decision | Required state | New state | Evidence arrays |
| --- | --- | --- | --- |
| `APPROVE` | `PENDING` | `APPROVED` | Exact currently linked identity/certificate file IDs and exact offered service codes. |
| `REJECT` | `PENDING` | `REJECTED` | Both empty. |
| `DISMISS` | `APPROVED` | `DISMISSED` | Both empty. |
| `REINSTATE` | `DISMISSED` | `APPROVED` | Same checks as approval, with fresh reviewer reads after dismissal. |

Approval and reinstatement require legal names, a Milano work profile, at least one offering, the current published worker terms version accepted, a live selfie and both document sides/pages, and a ready certificate for each Car Wash or Pet Care offering. Each linked file must be `READY`, owned by the worker, and have the expected purpose. The acting administrator must have successfully opened every current file via the one-use private-file grant flow; merely requesting a grant does not count. A reinstatement requires new reads after the dismissal decision. Missing or stale evidence returns `409` with no state change. A worker can receive work only while `APPROVED` and still accepting the current worker terms. Rejection and dismissal cannot be undone through worker profile edits; reinstatement is the only implemented route from dismissal. Re-review from `REJECTED` is not implemented.

Each decision locks the worker row, records actor, reason, time, decision, resulting state, and an evidence snapshot in `cleaner_review_decisions`, and commits the state and record together. A per-actor, per-key advisory lock serializes retries. The same actor and key with the same request returns the original decision; reusing a key for a different request returns `409`. Concurrent decisions for the same worker serialize on its row and only the first valid transition succeeds. The history is append-only through the API; there is no delete route. Migration `0010_worker_review_decisions` adds the decision table and a unique actor/key constraint; rollback deletes audit history and is unsuitable after real reviews.

B28's dismissal and reinstatement share these permission, audit, and retry rules. A dismissal immediately changes the server's job-eligibility projection to false; reinstatement restores it only after fresh evidence review. A repeated decision with its original key returns the same record, while a changed payload or a new conflicting transition returns `409` and leaves the audit history unchanged. The existing B07 implementation and migration provide these B28 behaviors; B28 adds explicit PostgreSQL regression coverage for the financial-action gate and dismissal/reinstatement retries.

Reviewer reads and decision records contain highly sensitive metadata. Never cache review responses, log bearer tokens or raw file bytes, or expose file IDs to customer/worker screens. Retention, erasure exceptions, backups, and real-evidence access procedures remain launch gates in ADR 0002. No Auth0 tenant, live reviewer, or production evidence is provisioned by this implementation.

## Mobile reviewer screen

`/admin/reviews` uses the existing narrow phone-width shell on mobile and desktop. When the public Auth0 domain, dedicated SPA client ID, and API audience are absent, it displays an operator configuration message. Otherwise the reviewer signs in through Auth0 Universal Login and MFA. The SPA SDK uses authorization code with PKCE and an in-memory token cache; no client secret, refresh token storage, or Cleanzza phone cookie is used for reviewer API calls. The configured Auth0 callback and logout URLs must include the deployed `/admin/reviews` route.

The screen filters and pages the review queue, opens one worker profile, and shows each private file through a 60-second one-use grant. The original file bytes become a temporary browser object URL that is revoked when the preview closes, changes, or the screen unloads. A successful content read marks the file opened in that screen session. The reviewer supplies a 10–500 character reason and can approve or reinstate only after opening every linked evidence file and seeing current terms accepted. The backend repeats all checks. Rejection and dismissal use the same reason field and no evidence arrays. A retry of the same submission reuses its idempotency key. The screen refreshes worker detail after success. Browser UI state is not an audit source and is never a substitute for the server's file-read and decision records.

## Verification

Backend: `.venv/bin/ruff check .`, `.venv/bin/ruff format --check .`, `.venv/bin/mypy app scripts`, `.venv/bin/alembic upgrade head`, `.venv/bin/alembic check`, `.venv/bin/pytest -q`, and `.venv/bin/python -m scripts.export_openapi --check` against PostgreSQL. The integration test exercises MFA authorization, required file-read audits, transition eligibility, idempotency, and the decision record. The frontend pins the generated OpenAPI document and runs its contract, type, test, lint, format, and build checks in its own repository.
