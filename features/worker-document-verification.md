# Worker document verification in the protected preview

Workers attach practice samples after phone verification and pricing selection. Registration documents are complete after submission. Booking eligibility additionally requires an approved verification in the pilot service area.

- Select an Italian passport or a residence permit. The passport requires one attachment, and the permit requires both sides.
- Attach the Codice Fiscale, facial selfie, and Certificato di Residenza.
- Answer the Partita IVA question Yes or No. Yes adds a required VAT registration attachment.
- Select the sample municipality on the residence certificate. Ferrara maps to `IT-FE-FERRARA`. Bologna can be approved, but is outside the current service area.

The upload controls accept only the downloadable Cleanzza practice images, checked locally by their content fingerprints. Workers may also attach those same fixtures directly from the form. Only fixture identifiers reach the API. No real file, file name, image bytes, identity number, face data, or residential address is collected. Reviewers can inspect the fixtures and record individual decisions. Unreadable or expired fixtures cannot be approved. This models manual verification and does not establish legal eligibility, real document validity, or biometric identity.

## Status and resubmission

A complete set enters **Pending Review**. An authorized reviewer records a decision on every pending attachment. All attachments must be approved before the packet becomes **Approved**. A rejection displays **Rejected – Resubmission Required** and the reason beside each rejected document. The worker sees the notification on revisit, refresh, or the next 20-second update while the pending status is visible. The preview sends no SMS or email.

A worker can replace rejected attachments while keeping approved ones. A change to the identity-document type, VAT answer, or municipality requires a complete new attachment set. Pending and approved submissions are read-only. Previous revisions keep their synthetic decision records and timestamps. No existing worker is automatically approved.

The priced worker directory, targeted estimates and drafts, open request inbox, and acceptance command enforce approved Ferrara verification when enabled. Previously accepted requests remain visible to their owner. Worker role, ownership, and the existing review restrictions still apply. These gates do not grant access to real orders or jobs.

## API

- `GET /cleaners/me/verification` uses the worker session and returns the current owner snapshot. A worker with no submission has revision `0`, status `NOT_SUBMITTED`, and no documents.
- `PUT /cleaners/me/verification` requires the worker session, allowed origin when present, and `X-CSRF-Token`. Send the current `revision`, `document_type` (`ITALIAN_PASSPORT` or `RESIDENCE_PERMIT`), strict boolean `has_vat_number`, `municipality` (`FERRARA` or `BOLOGNA`), and an exact `samples` map keyed by required or replacement document kind. The values are `CLEAR`, `UNREADABLE`, or `EXPIRED`. A selfie supports clear and unreadable fixtures. Invalid, extra, or missing values return `422 VALIDATION_ERROR`.
- `GET /admin/cleaners/verification` returns pending snapshots ordered by cleaner ID with optional UUID `cursor`, `limit` 1–50 (default 20), and `next_cursor`.
- `PUT /admin/cleaners/verification/{cleaner_id}` accepts the submission `revision` and a `decisions` map with exactly the pending document kinds. Each result is `APPROVED` or `REJECTED`. Rejections require `UNREADABLE`, `EXPIRED`, or `INCORRECT_DOCUMENT`. Approval requires a clear fixture. The response is the updated snapshot.

The admin routes require the dedicated `X-Preview-Reviewer-Key` and an allowed origin when present. The key only grants access to synthetic verification records. No worker sign-in is needed for a reviewer. Existing live evidence routes retain managed Auth0 access. Successful writes return 200. Missing or wrong credentials return `401 UNAUTHENTICATED`. A role, origin, CSRF, or acceptance eligibility denial returns `403 FORBIDDEN`. Missing resources return `404 NOT_FOUND`, stale revisions or conflicting decisions return `409 CONFLICT`, and a disabled preview or unavailable database returns `503 SERVICE_UNAVAILABLE`. Snapshots are served with `Cache-Control: no-store`.

Submission and review lock the worker row. An exact retry for the current revision returns the same result. Approval is terminal for a packet and eligible admission is enforced inside the request acceptance transaction. Migration `0020_worker_verification` is additive and avoids rewriting existing workers. Use a forward fix after submissions exist.

## Deployment and verification

Enable `WORKER_VERIFICATION_PREVIEW_ENABLED` with a dedicated randomly generated `WORKER_VERIFICATION_REVIEWER_KEY` only on a protected synthetic host with test sign-in, booking simulation, and private evidence storage disabled. Restrict key access to the preview operators. Open `/admin/verification` and enter the key from the staging secret store. Rotate the key through the staging deployment configuration. Never send the key to workers.

Run PostgreSQL integration tests, Alembic upgrade/check, Ruff, mypy, and OpenAPI export. Pin the updated frontend contract and run the frontend tests, lint, format check, and preview build. Verify conditional evidence, rejection and replacement, service location, and booking eligibility on the protected host using synthetic identities and samples. Live identity collection remains subject to the [approved data-handling boundary](../adr/0002-phase-1-personal-data.md) and the [external approval record](../operations/phase-1-external-approval-record.md).
