# Customer document verification in the protected preview

Customers can sign in with a test phone code or create a test account from the home page. **Verification** in the customer space uses the [worker sample checklist](worker-document-verification.md): an Italian passport or the front and back of a residence permit, a Codice Fiscale, a facial selfie, proof of residence, and a Partita IVA document when the required Yes/No answer is Yes.

The preview accepts the supplied sample images and stores only their fixture identifiers. Documents and the accompanying municipality are illustrative. A customer's service address must be in the existing Ferrara booking area regardless of the sample residence municipality. No real identity, tax, or facial evidence is collected. Real customer activation continues to follow its [separate approved policy](customer-profile.md).

Customers can browse, receive estimates, and save a draft while verification is underway. A new simulated booking requires **Approved**. Existing requests remain visible and cancellable, and exact retries of a previously submitted request retain their result.

A completed attachment set enters **Pending Review**. Reviewers use the **Customers** queue in `/admin/verification` and decide each pending document. The customer sees **Rejected – Resubmission Required** with a reason for each rejected item on revisit, refresh, or the next visible-page poll. A replacement preserves approvals for unchanged attachments. Changing identity type, VAT answer, or sample municipality requires a full set. A pending or approved submission is read-only. Notices are delivered in the customer account.

The shared verification form presents one step at a time and shows progress and a review screen. Customers can move back without losing their selections on the page. On a replacement, the form opens directly on the rejected attachments and offers a separate action to edit document details.

## API

- `GET /customers/me/verification` returns the current owner snapshot. The customer session must have the `CUSTOMER` role. With no submission the response has `revision: 0`, `status: NOT_SUBMITTED`, and `can_book: false`.
- `PUT /customers/me/verification` takes `revision`, `document_type`, `has_vat_number`, `municipality`, and the exact required `samples` map for a new or replacement submission. It requires the customer cookie, allowed origin when present, and `X-CSRF-Token`. Its fixture codes and validation are the same as the worker submission. It returns the snapshot with `customer_id`, per-document status and reason, registration completion, `can_book`, and submission and review timestamps.
- `GET /admin/customers/verification` returns pending customer snapshots sorted by customer UUID. It accepts an optional UUID `cursor` and integer `limit` of 1–50, default 20, and returns `items` and `next_cursor`.
- `PUT /admin/customers/verification/{customer_id}` records the `revision` and a complete map of pending document decisions. Rejections require a supported reason and unreadable or expired fixtures cannot be approved. Both reviewer routes require `X-Preview-Reviewer-Key` and an allowed origin when present.
- `POST /customers/me/orders/{id}/demo-submit` returns `403 FORBIDDEN` for a new submission without customer approval when the preview gate is enabled.

Successful reads and writes return 200 for verification with `Cache-Control: no-store`. Missing or invalid credentials return `401 UNAUTHENTICATED`. Wrong role, origin, or CSRF returns `403 FORBIDDEN`. Missing accounts or submissions return `404 NOT_FOUND`, stale revisions or incompatible decisions return `409 CONFLICT`, invalid amounts of evidence or invalid decisions return `422 VALIDATION_ERROR`, and unavailable or disabled verification returns `503 SERVICE_UNAVAILABLE`.

Submissions and reviews lock the customer row. An exact retry of the current submission or decision shares the worker retry behavior. Migration `0021_customer_verification` adds customer-only revisions and leaves existing worker packets in their existing table. Booking records and customer activation remain independent from synthetic verification revisions.

## Verification and rollout

Enable `CUSTOMER_VERIFICATION_PREVIEW_ENABLED` only with the protected verification preview and existing reviewer key. The reverse proxy sends `/admin/customers/verification` and its detail paths to the API under the site access gate. Deploy the API and migration first, then the frontend. Verify on PostgreSQL that unapproved customers can save drafts but cannot submit, that rejection permits replacement, and that approved customers can submit. Run the backend schema, OpenAPI, and tests, and the frontend generated-contract checks and preview build.
