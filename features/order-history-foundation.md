# Draft order and history storage foundation (B11)

## Purpose and scope

B11 currently provides internal persistence for draft orders and their attributed history, read-only customer history endpoints, and a draft-history screen. It cannot submit a service request, accept a quote, start payment, or issue an offer. Real booking remains gated by the reviewed tax, invoicing, legal, and Stripe decisions in [ADR 0004](../adr/0004-phase-1-pricing-direction.md) and [ADR 0006](../adr/0006-phase-1-payment-direction.md). The [review checklist](../operations/italy-tax-stripe-review-checklist.md) lists the written answers still required.

## Database contract

Migration `0012_order_history_foundation` adds `orders` and `order_events` without changing existing tables. `orders` stores an application-generated UUID, a required customer ID referencing `customers`, `fulfillment_state`, and a server timestamp. A database check permits only `DRAFT`. There is no accepted quote, schedule, service address, order item, or payment reference in this slice; later B09/B11 migrations must add those and replace the draft-only check before enabling real booking.

`order_events` stores an application-generated UUID, required order ID, `actor_kind`, optional `actor_user_id`, `event_type`, and server timestamp. Its order and user references are foreign keys with restricted deletion. The actor kind is one of `CUSTOMER`, `CLEANER`, `ADMIN`, `SUPPORT`, `SYSTEM`, or `PROVIDER`; a user kind requires a user ID, while system/provider kinds require no user ID. Only `DRAFT_CREATED` is permitted by the current database check. Later command slices must expand the allowed event types together with their state rules. Events are returned by occurrence time and UUID for stable ordering.

The repository provides `create_draft_order`, `get_order`, `add_order_event`, and `list_order_events`. It flushes writes but never commits; an application service will later own the transaction that creates an order and its first event. The storage slice did not expose a write path, and the read-only API below still cannot call these write helpers. No idempotency behavior, quote validation, customer activation check, or schedule check is claimed yet. B11's future create-order command must add and test those rules against PostgreSQL before customers can submit a request. The backend remains authoritative for the resulting order state and actor attribution under [ADR 0007](../adr/0007-phase-1-fulfillment-states.md).

## Read-only customer draft history API

The next B11 slice exposes two read-only endpoints for the existing `DRAFT` records. It does not create an order, accept a quote, take payment, or enable booking. Draft records currently arise only from internal/test code; an ordinary customer sees an empty list until a reviewed creation flow exists. The synthetic-development assumptions are recorded [here](../operations/phase-1-working-assumptions.md).

`GET /customers/me/orders` requires a valid customer session cookie. It takes optional `cursor` (an order UUID from this customer's preceding page) and `limit` (integer 1–50, default 20). Results are ordered by server `created_at` descending, then UUID descending. The response is `{ "items": [{ "id": "<uuid>", "fulfillment_state": "DRAFT", "created_at": "<offset timestamp>" }], "next_cursor": "<uuid or null>" }`. The cursor is the last item ID when another page exists, otherwise null. An unknown or another customer's cursor returns `404`, not a cross-account page.

`GET /customers/me/orders/{order_id}` takes an order UUID and returns the same summary plus `events`, ordered by server occurrence time and UUID. Each event exposes `actor_kind`, `event_type` (`DRAFT_CREATED` at present), and `occurred_at`; it does not expose actor user IDs. An unknown or another customer's order returns `404`. No customer ID, address, payment data, or private worker data appears in either response. Both endpoints use `Cache-Control: no-store` and the shared request ID.

Both endpoints return `401 UNAUTHENTICATED` without a session, `403 FORBIDDEN` without the customer role, `404 NOT_FOUND` for a missing customer/order or inaccessible cursor, `422 VALIDATION_ERROR` for malformed UUIDs or limits, and `503 SERVICE_UNAVAILABLE` when the database session is unconfigured. They make no writes, so CSRF and idempotency keys are not required. There is no pagination filter or asynchronous effect. Ownership is checked in the service before reading events. The frontend pins the exported OpenAPI types.

PostgreSQL integration coverage checks customer role and ownership, stable pagination, private event projection, and validation. From `back/`, run `.venv/bin/pytest -q tests/test_order_history_api.py`, the full test suite, Ruff lint/format, mypy, and `.venv/bin/python -m scripts.export_openapi --check`. From `front/`, run `npm run contract:check`, `npm run typecheck`, and the canonical frontend checks. No migration is added by this API slice.

## Customer draft-history screen

The customer profile links to `/orders`. The mobile-width screen calls only the two read-only endpoints above with the session cookie and `no-store` caching. It handles loading, no session or customer role, temporary failure with retry, an empty list, cursor-based **Load more drafts**, and an expandable activity panel for each record. The activity panel omits actor user IDs. A missing/stale cursor refreshes the list; an inaccessible detail shows a retryable error. Creation timestamps are displayed in `Europe/Rome` and are explicitly labeled as draft creation times, not appointment times.

Every card says `Draft` and states that no booking or payment has been made. The screen provides no submit, accept, pay, or cancel action and does not save private order data to browser storage. API-client tests verify the owner routes, query parameters, cookie use, `no-store` caching, and error propagation. `npm test`, `npm run lint`, `npm run typecheck`, `npm run contract:check`, `npm run format:check`, and `npm run build` are the frontend checks. This screen may normally be empty until the separately gated order-creation flow exists.

## Privacy, rollout, and verification

This slice stores only a customer reference and audit metadata; it does not copy a customer address or payment details. It uses synthetic records in tests. The restricted foreign keys protect audit integrity while customer export/deletion and legal-hold handling remain B26 work subject to ADR 0002's retention review. No migration rollback should drop these tables after real history exists; use a forward fix or reviewed data-preservation plan.

From `back/`, apply `.venv/bin/alembic upgrade head`, verify `.venv/bin/alembic check`, and run `.venv/bin/pytest -q tests/test_order_repository.py` with PostgreSQL `DATABASE_URL`. The integration tests check the migrated schema, caller-controlled commit/rollback, stable reads, draft-only state and event constraints, and actor attribution. Before delivery, run the full test suite, Ruff lint and format, mypy, and the OpenAPI export check. The read-only API's OpenAPI artifact is already exported and pinned by the frontend; this screen adds no API change.
