# Draft order and history storage foundation (B11)

## Purpose and scope

This first B11 slice adds internal persistence for a draft order and its attributed history. It exposes no HTTP route and cannot submit a service request, accept a quote, start payment, issue an offer, or show customer order history. Real booking remains gated by the reviewed tax, invoicing, legal, and Stripe decisions in [ADR 0004](../adr/0004-phase-1-pricing-direction.md) and [ADR 0006](../adr/0006-phase-1-payment-direction.md). The [review checklist](../operations/italy-tax-stripe-review-checklist.md) lists the written answers still required.

## Database contract

Migration `0012_order_history_foundation` adds `orders` and `order_events` without changing existing tables. `orders` stores an application-generated UUID, a required customer ID referencing `customers`, `fulfillment_state`, and a server timestamp. A database check permits only `DRAFT`. There is no accepted quote, schedule, service address, order item, or payment reference in this slice; later B09/B11 migrations must add those and replace the draft-only check before enabling real booking.

`order_events` stores an application-generated UUID, required order ID, `actor_kind`, optional `actor_user_id`, `event_type`, and server timestamp. Its order and user references are foreign keys with restricted deletion. The actor kind is one of `CUSTOMER`, `CLEANER`, `ADMIN`, `SUPPORT`, `SYSTEM`, or `PROVIDER`; a user kind requires a user ID, while system/provider kinds require no user ID. Only `DRAFT_CREATED` is permitted by the current database check. Later command slices must expand the allowed event types together with their state rules. Events are returned by occurrence time and UUID for stable ordering.

The repository provides `create_draft_order`, `get_order`, `add_order_event`, and `list_order_events`. It flushes writes but never commits; an application service will later own the transaction that creates an order and its first event. There is no public path calling these helpers yet. No idempotency behavior, quote validation, customer activation check, schedule check, or API contract is claimed by this storage-only slice. B11's create-order and history endpoints must add and test those rules against PostgreSQL before customers can use them. The backend remains authoritative for the resulting order state and actor attribution under [ADR 0007](../adr/0007-phase-1-fulfillment-states.md).

## Privacy, rollout, and verification

This slice stores only a customer reference and audit metadata; it does not copy a customer address or payment details. It uses synthetic records in tests. The restricted foreign keys protect audit integrity while customer export/deletion and legal-hold handling remain B26 work subject to ADR 0002's retention review. No migration rollback should drop these tables after real history exists; use a forward fix or reviewed data-preservation plan.

From `back/`, apply `.venv/bin/alembic upgrade head`, verify `.venv/bin/alembic check`, and run `.venv/bin/pytest -q tests/test_order_repository.py` with PostgreSQL `DATABASE_URL`. The integration tests check the migrated schema, caller-controlled commit/rollback, stable reads, draft-only state and event constraints, and actor attribution. Before delivery, run the full test suite, Ruff lint and format, mypy, and the OpenAPI export check. OpenAPI is unchanged because no endpoint is added.
