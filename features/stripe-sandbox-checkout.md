# Synthetic Stripe sandbox Checkout experiment

**Status:** Developer-only local experiment. No public API, order acceptance, financial settlement, or invoice issuance is implemented.

## Purpose and scope

The backend can create a Stripe-hosted test Checkout Session for a synthetic EUR 40.00, two-hour Home Cleaning example. This checks the Stripe Python SDK request shape and sandbox connectivity without changing Cleanzza's draft-only order model. The [Stripe integration plan](../operations/stripe-integration-plan.md) describes the later application flow and approval gates.

Run `.venv/bin/python -m scripts.stripe_sandbox_checkout` from `back/`. The command reads `STRIPE_API_KEY` from the ignored backend `.env`, creates a random attempt ID, and prints the attempt ID and test Checkout URL. `--attempt-id <UUID>` retries the same logical attempt with a stable Stripe idempotency key. `STRIPE_PUBLISHABLE_KEY` is not used in this hosted Checkout experiment.

## Behavior and safeguards

- The command runs only when `ENVIRONMENT=local` and the key has a Stripe test prefix (`rk_test_` or `sk_test_`). It never accepts a live key.
- The Session is `mode=payment` for EUR 40.00, with a card allowlist, an explicit synthetic product label, a synthetic client reference, and a deterministic integration identifier. It has no `transfer_data`, `application_fee_amount`, `automatic_tax`, or `invoice_creation` setting.
- The SDK response must be a test Session with a hosted URL. The command makes no Cleanzza order, matching offer, refund, worker transfer, or invoice. The local success URL is only a return page and proves no payment state.
- No secret key is sent to the frontend, stored in Git, or printed by the command. Use a least-privilege restricted sandbox key when replacing a test secret.

## Contracts and verification

There is no HTTP route or OpenAPI change. `create_demo_checkout` takes a Stripe client and synthetic UUID and returns a Checkout URL or raises `SandboxCheckoutError`; `require_sandbox_key` rejects non-local environments, missing keys, and live keys. Stripe API errors terminate the developer command without printing credentials.

Canonical checks from `back/`: `.venv/bin/pytest -q tests/test_stripe_sandbox_checkout.py`, `.venv/bin/ruff check .`, `.venv/bin/ruff format --check .`, and `.venv/bin/mypy app scripts`. A controlled provider-boundary check is to run the command against a Cleanzza sandbox and confirm the returned Session is test-mode with EUR 40.00 and no invoice or transfer. Do not complete checkout with real customer details. The production acceptance gate remains in the [external approval record](../operations/phase-1-external-approval-record.md).

On 2026-11-10, the controlled sandbox check created a Session and retrieved it through the Stripe API. It reported `livemode: false`, `amount_total: 4000`, `currency: eur`, `payment_status: unpaid`, and no invoice. Repeating the command with the same attempt ID returned the same Checkout URL. No payment was completed. This verifies Session creation and request idempotency; webhook fulfillment and booking integration are not implemented.
