# ADR 0004: Phase 1 Milano pricing and booking policy

**Status:** Product direction approved by the owner's 2026-10-18 delegation to choose the simplest pricing recommendation. ADR 0006 sets the nominal merchant/payment/commission direction; tax, legal and provider validation remain launch gates.

## Customer-facing price

Use EUR and a versioned `milano-hourly-v1` price table. Each selected service has its own whole-hour duration from 1 through 8, as in ADR 0003. The quoted service window is the sum of those hours, scheduled contiguously. Prices below are **customer totals per hour**, with any applicable tax included. The customer never sees an extra tax added at checkout.

| Service | EUR per hour | Example |
| --- | ---: | ---: |
| Home Cleaning | 20.00 | 2 hours = EUR 40.00 |
| Car Wash | 25.00 | 2 hours = EUR 50.00 |
| Pet Care | 20.00 | 3 hours = EUR 60.00 |

A combined 2-hour Home Cleaning and 1-hour Car Wash request is EUR 65.00 and reserves 3 hours. The service line is `unit_price × whole_hours`; the quote total is the sum of lines. Use decimal arithmetic and round each final currency amount to EUR cents with half-up rounding. No dynamic pricing, discount, equipment/supply fee, weekend fee, distance fee, or repeat-worker surcharge applies to this first price version. The product description's repeat-worker surcharge remains a later policy, not a hidden Phase 1 fee. Changing a rate or adding a fee creates a new rule version. Historical accepted quotes never change.

## Quote and booking rules

- A displayed quote is valid for 15 minutes, ending at its server-issued `expires_at`. Recalculate on any change to service, hours, date/time, address, service inputs, or price/catalogue version. An expired quote cannot be accepted. The customer must see and accept the replacement total before order submission.
- Travel is not a chargeable service line. Matching reserves a 30-minute buffer before and after the service window for a worker; the customer sees only requested service hours. This is an eligibility/scheduling rule, not an extension of billed work.
- Overtime is never billed automatically. If more time is needed, work pauses at the agreed end and the customer must accept a fresh whole-hour quote and schedule extension before additional paid work. If an extension cannot be accepted, the worker ends at the original time.
- The customer can cancel without a fee before service starts. A worker or platform cancellation, or worker no-show, entitles the customer to a full refund of any captured amount. A customer no-show has no automatic fee in this first version. After service starts, a refund or partial refund requires a recorded human review of the actual service and applicable law/terms; the platform does not silently keep the full price. No automatic cancellation penalty is introduced without a new policy and published terms.
- A refund never changes the accepted price snapshot; it is a separate auditable money movement. Payment authorization, capture, settlement, refund execution and timing depend on D3.

## Tax and commission gate

The EUR prices are gross customer-facing price ceilings for the listed scope. The European Commission lists Italy's **standard VAT rate as 22%**, but that does **not** establish the applicable tax treatment of each Cleanzza service or the party that invoices it. An Italian tax adviser must confirm merchant-of-record, invoice flow, applicable rate/exemption for each service and platform fee, and rounding before real quotes may be accepted. The backend must fail closed for quote acceptance in production until a versioned tax treatment is configured and reviewed. Do not insert a zero or guessed VAT amount into an accepted snapshot.

The [Italy tax and Stripe review checklist](../operations/italy-tax-stripe-review-checklist.md) lists the written answers and worked examples required to close this gate. It records questions for reviewers, not approved tax treatment.

[ADR 0006](0006-phase-1-payment-direction.md) defines the provisional 20% platform share, 80% worker share, Stripe Connect direction and booking charge. B09 can build deterministic draft quote calculation and synthetic acceptance tests, but cannot create real accepted orders until the tax, invoicing, legal and provider gates are cleared. An accepted snapshot eventually needs catalogue and pricing versions, each gross line, tax treatment/amount, discount/surcharge (zero in this version), gross total, platform commission, worker amount, currency and scheduled duration.

## Rationale and revisit

Round prices and no automatic penalties make the first Milano catalogue easy to explain and test. These are product recommendations, not market-price or tax findings. Review conversion, worker economics and operating costs before production launch. Publish customer/worker terms that match cancellation, no-show and overtime policy before enabling bookings. ADR 0003 remains authoritative for location and required service inputs; ADR 0002 remains authoritative for private data.

## References

- [Phase 1 scope](../phase_1_product_scope.md)
- [Architecture](../technical_architecture.md)
- [EU VAT rates for Italy](https://europa.eu/youreurope/business/finance-and-tax/vat/vat-rules-rates/index_it.htm)
