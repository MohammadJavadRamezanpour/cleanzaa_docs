# Versioned pricing rules and draft quotes (B09)

## Current demo rule

Migration `0011_versioned_pricing_rules` seeded historical `milano-hourly-v1` rows. Migration `0017_ferrara_demo_catalog` preserves them, widens the area constraint, and adds the active `ferrara-demo-v1` Home Cleaning row in `IT-FE-FERRARA`: EUR 25.00 per hour and a simulated `0.1500` platform share. The EUR 25.00 base is illustrative; tier ranges and public hourly rates remain open. The 15% product direction is in [ADR 0010](../adr/0010-ferrara-home-cleaning-pilot.md). Historical rows and quote snapshots are not rewritten.

The calculator uses PostgreSQL versioned rows and `Decimal` arithmetic, not a client-supplied rate. For two hours, the demo base is EUR 50.00, simulated platform share EUR 7.50, and simulated professional amount EUR 42.50 before tax or adjustments. `tax_amount` and `tax_treatment_version` are null, and `acceptance_ready` is false. The existing `gross_total` and `hourly_gross` field names are retained for API compatibility, but for `ferrara-demo-v1` they represent **synthetic example base amounts**, not a final tax-inclusive customer charge. No VAT rate, no-VAT treatment, final customer price, invoice issuer, or worker net pay is inferred. Only a reviewed fiscal matrix and approved effective professional profile can support a later binding quote.

## Public preview API

`POST /quotes/drafts` accepts JSON with `area_code`, `city`, `postcode`, and `services` containing one `{service_code: "HOME_CLEANING", duration_hours: 1..8}` entry. Extra fields are forbidden. Active values are `IT-FE-FERRARA`, `Ferrara` (case-insensitive after trim), and CAP `44121`–`44124`. Other service, area, duration, duplicate, or malformed inputs return `422 VALIDATION_ERROR`. A missing or incoherent server rule returns `503 SERVICE_UNAVAILABLE`. No authentication, CSRF, idempotency key, pagination, write, audit event, or asynchronous effect is involved.

A `200` response includes catalog/pricing versions, `EUR`, area, decimal-string line and example amounts, total hours, explicit-offset `quoted_at` and `expires_at` 15 minutes later, null tax fields, and `acceptance_ready: false`. It excludes internal platform and professional split values from the customer response. It uses `Cache-Control: no-store` and `X-Request-ID`. A changed or expired preview must be recalculated. The frontend shows this as an example and makes no real acceptance or charge available.

## Saved drafts and compatibility

The gated `POST /customers/me/orders/drafts` recalculates with the active Ferrara rule, saves a versioned estimate snapshot, and rejects inactive service/area input. Retry with the same owner key returns the original snapshot. Existing Milano drafts remain readable in owner history; the response carries the original raw request and quote versions, while new demo submission rejects a snapshot from a superseded catalog or price rule. The controlled fake-provider demo alone may simulate a request at its example amount. Live quote acceptance, charge, transfer, and fiscal issuance remain closed by the [external approval record](../operations/phase-1-external-approval-record.md).

PostgreSQL integration tests cover seed rows, money constraints, current calculation and 15% share, old version retention, invalid area/service, draft idempotency and ownership, and API validation. The migration is additive; a downgrade requires clearing Ferrara-specific profiles and snapshots first, so deployment should use a forward fix if such data exists. Verify migrations, tests, OpenAPI, and the frontend pinned client together with the commands in [the catalog feature](launch-service-catalog.md).
