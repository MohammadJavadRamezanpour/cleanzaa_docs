# ADR 0009: Phase 1 is a synthetic product demo

**Status:** Approved by the owner on 2026-11-10.

## Decision

The Phase 1 deliverable is a reviewable, end-to-end **demo**, not a public marketplace launch. It should let someone explore the main customer and cleaner journey on a working website using synthetic accounts, example addresses, simulated service activity, and fake payment outcomes. The [Phase 1 demo scope](../phase_1_product_scope.md) defines what the demo must show.

The demo may use deterministic examples and explicitly labeled assumptions for price, the nominal 20%/80% split, eligibility, order transitions, cancellation, refunds, earnings, ratings, and support. No Italian tax conclusion, legal supplier determination, Stripe account approval, real worker verification, or staffed service operation is required to implement or review these simulated paths. The assistant may make routine demo implementation choices and record them in feature documentation without waiting for owner decisions.

This decision changes the **delivery target and sequencing** of earlier Phase 1 documents. ADRs 0002–0008 remain useful product directions and constraints for eventual real use, but their external approvals are not gates for synthetic demo implementation. Where those documents say a Phase 1 launch requires an approval, read that as a requirement for a later live launch. Do not change a `Pending` external review entry to approved because the demo works.

## Demo boundary

- Use synthetic or consented test data only. Never collect a real passport/ID, selfie, service photo, payment method, or worker bank detail for the demo. Clearly mark any placeholder evidence as simulated.
- Use a fake payment provider and simulated capture, refund, transfer, and payout states. No live Stripe keys, real card charge, transfer, or payout. A separate Stripe sandbox experiment may be used if configured, but it is not a dependency of the demo.
- Keep simulation endpoints and data isolated from live order/payment/identity paths. Default the demo off outside explicitly configured development or controlled demo environments. A public preview needs access control or an equivalent limit on test-data creation and must disclose that no service will be performed.
- Do not invent VAT rates, invoice issuers, legal approvals, or worker net pay. A displayed gross example is a demo price; tax remains unknown and payout figures are labeled simulated.
- Preserve practical engineering checks for the demo: owner access boundaries, retry-safe commands, server-owned state, meaningful tests, API contract sync, and reviewable delivery. These prove that the website behaves consistently; they do not certify live readiness.

## Consequences

Development can proceed through the entire simulated customer and cleaner journey without pausing for tax, legal, Stripe, identity-provider, or support-staffing decisions. Existing live-use holds still apply to real acceptance, orders, charges, transfers, identity collection, and services. If the owner later chooses to launch with real users, create a separate release plan and close the [external approval record](../operations/phase-1-external-approval-record.md) before enabling those paths.
