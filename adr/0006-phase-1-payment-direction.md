# ADR 0006: Phase 1 payment and worker earnings direction

**Status:** Product direction approved by the owner on 2026-10-19. Italian legal, tax, Stripe account/capability, and contract review remain launch gates before real quotes or payments.

## Decision

Use Stripe Connect for one EUR card payment at booking. Cleanzza is the intended business of record for the customer charge. The worker is unknown when the booking payment is created, so use a platform charge followed by a separate transfer to the assigned worker's connected account after customer confirmation or a recorded support resolution. The platform does not create an internal wallet, advertise an escrow service, or store raw card or bank details. Stripe-hosted connected-account onboarding collects payout/KYC details; Cleanzza stores opaque provider IDs and payout capability status. Select the exact Stripe account API and configuration after validating the current Italian account capabilities, liability, service agreement, and contract terms with Stripe.

The customer sees a final price and explicitly authorizes **one immediate card charge** for that accepted quote. No saved-card, off-session, automatic debit, cash, top-up, customer balance, or wallet flow launches in Phase 1. A booking can be created in a private `PAYMENT_PENDING` state, but it becomes a `REQUESTED` job and enters matching only after the provider confirms the captured payment. Strong Customer Authentication or an asynchronous payment result keeps it pending; failure or expiry never creates an offer. Retrying the same checkout/order command uses an idempotency key and the same logical payment attempt. A changed or expired quote requires new customer acceptance.

The `milano-hourly-v1` gross prices in ADR 0004 remain customer totals. The platform's first share is **20% of the tax-inclusive customer total**; the worker's nominal share is the remaining **80%**. Calculate the platform share in decimal EUR, round once to cents with half-up rounding, and set the worker amount to `gross − platform share` so the two always reconcile. For a EUR 40.00 Home Cleaning quote, the nominal split is EUR 8.00 / EUR 32.00; for EUR 50.00 Car Wash, EUR 10.00 / EUR 40.00; for EUR 60.00 Pet Care, EUR 12.00 / EUR 48.00. Stripe processing and Connect fees are borne from the platform share, not added to the customer total or deducted silently from the worker amount. Tax treatment and invoicing may require changing this model before launch; never present these nominal amounts as tax advice or a guaranteed net wage.

An accepted quote must freeze its pricing, catalogue and commission rule versions, gross service lines, tax basis/rate/amounts, gross total, platform share, worker amount and currency. The tax fields cannot be guessed or set to zero. Until an Italian tax adviser confirms the invoicing party, tax treatment for each service and platform fee, and allocation/rounding, only **synthetic or explicitly draft** quotes may use this split. The production acceptance and payment gate stays closed. Changes to the rate or split create a new rule version and never rewrite an old accepted snapshot.

## Settlement and earnings

- On customer cancellation before service starts, or platform/worker cancellation or worker no-show, issue a full refund of any captured customer charge. Do not create a worker transfer. After service starts, a partial/full refund is a recorded human decision under ADR 0004 and applicable terms/law. Refunds, chargebacks, transfer reversals, fees and adjustments are separate auditable records, not edits to the accepted quote.
- Worker completion alone does not release funds. After the customer confirms, or support records an authorized outcome when confirmation is absent or disputed, transfer the payable worker amount to the worker's enabled Stripe connected account. A failed, paused, or restricted connected account leaves the amount payable and visible to operations; do not mark it paid. Stripe's later payout to the worker's bank is a separate provider event and may follow its own schedule.
- The worker earnings view distinguishes `estimated` (accepted quote and assignment), `completed` (worker finished, confirmation pending), `payable` (confirmed/support-approved, captured funds reconciled, transfer pending), and `paid` (provider confirms external payout). Show gross, platform share, worker amount, adjustments and currency with their real state. Never show estimated as paid.
- Signed provider webhooks and periodic reconciliation are authoritative for charge, refund, transfer and payout outcomes. Persist provider event IDs for duplicate delivery, tolerate out-of-order events, and expose failed/negative balance/dispute cases to operations. A customer-facing success redirect is not proof of captured funds. Provider API calls use idempotency keys.

## Rationale and gates

Separate charge and transfer supports booking before a worker is assigned and keeps the customer's price distinct from worker settlement. Stripe documents Italy in its supported Connect regions and says that omitting `on_behalf_of` on separate charges makes the platform the business of record for the payment. These provider mechanics do **not** decide the legal merchant, employer, invoice issuer, or tax treatment of Cleanzza's services. Before live use, verify Stripe approval and account configuration, Italian marketplace/payment/employment law, the refund/chargeback allocation, worker contract and tax invoices, customer and worker terms, and the platform's ability to fund refunds if a charge is disputed or already transferred. Do not publish a payout date without provider confirmation.

## References

- [Pricing and booking policy](0004-phase-1-pricing-direction.md)
- [Phase 1 scope](../phase_1_product_scope.md)
- [Stripe separate charges and transfers](https://docs.stripe.com/connect/separate-charges-and-transfers)
- [Stripe Connect accounts and onboarding](https://docs.stripe.com/connect/accounts-v2)
- [Stripe webhook handling](https://docs.stripe.com/webhooks)
