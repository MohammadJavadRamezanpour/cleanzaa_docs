# Phase 1 Product Scope — Demo

**Decision:** Phase 1 is a working, synthetic product demo, as recorded in [ADR 0009](adr/0009-phase-1-demo-target.md). It is not a public service launch. The Next.js frontend and FastAPI backend remain separate repositories.

**Setting:** Example requests use Comune di Milano and the three catalogue services: Home Cleaning, Car Wash, and Pet Care. The EUR hourly amounts in [ADR 0004](adr/0004-phase-1-pricing-direction.md) are demo examples. Tax treatment is unknown. The nominal 20%/80% split in [ADR 0006](adr/0006-phase-1-payment-direction.md) may appear only as simulated accounting.

## Demo success criteria

A reviewer can complete the main story on a working website without a real phone number, worker identity document, card, or service appointment:

1. Start as a synthetic customer, choose a launch service, enter an example Milano address and schedule, and see a versioned example price.
2. Save and submit a demo request. The website records a retry-safe simulated payment outcome and shows the request in customer history. Cancelling a demo request produces a simulated refund outcome.
3. Open a synthetic cleaner view. A preapproved example cleaner sees a privacy-limited offer, accepts it once, and then sees the example job details. Assignment and schedule conflicts are handled consistently in the demo.
4. Move the job through Requested, Accepted, On the Way, In Progress, and Completed. The cleaner can simulate travel, check-in, Before/After evidence, and completion; the customer can see the status and confirm completion. Evidence is clearly fake or placeholder content, never a real home image or identity capture.
5. Show simulated cleaner earnings and payout state, customer rating/feedback, and both parties' relevant history. A simple synthetic support case may be shown without promising live staff or a response time.
6. Let the reviewer repeat the story with fresh synthetic data. The UI clearly states that no worker is dispatched, no service is booked, and no card is charged.

The backend remains authoritative for demo ownership, status, amount, idempotency, and one-cleaner assignment. Ordinary requests or polling are sufficient; real-time transport, optimization, and elaborate ranking are unnecessary. Feature documents and tests should describe the behavior actually implemented. A demo is complete only when the full story above can be run and verified; a screen or Trello card alone is not proof.

## Implementation boundary

- Use a fake provider for capture, refund, transfer, and payout displays. Stripe test mode is optional and cannot block the demo. Live Stripe charges and connected-account onboarding are outside Phase 1.
- Use synthetic customer and cleaner identities, example addresses, and placeholder service evidence. Production collection of identity documents, selfies, and real service photos is outside Phase 1.
- Demo quote snapshots may retain `acceptance_ready: false` and unknown tax fields. Simulated submission must be distinguishable from real accepted quotes and orders. Never present a VAT rate, invoice, legal relationship, or worker net amount as approved.
- Keep demo data separate from real fulfillment and payment records. Enable synthetic actions only in explicit local or controlled demo environments; any hosted preview needs access control or equivalent creation limits and clear demo copy.
- User authorization, ownership, retry safety, concurrency checks, API contracts, and meaningful tests still apply. The [development rules](development_rules.md) describe the practical demo workflow.

The companion local demo code covers customer demo sign-in, draft quote/request, simulated payment, cancellation/refund, and draft history. It does **not yet** cover the synthetic cleaner assignment and completion story above. [The order-history feature contract](features/order-history-foundation.md) describes that implemented slice.

## Later live product

The earlier customer and cleaner feature lists remain product ideas for a future release: verified phone onboarding, profile and terms activation, live cleaner identity/qualification approval, real service photos, staffed support, real booking, Stripe payment and payout, data rights, and operational launch. [ADRs 0002–0008](adr/) preserve those directions and their unresolved decisions. They do not block synthetic Phase 1 implementation.

Before any real customer request, worker dispatch, payment, payout, or identity collection is enabled, the owner must separately resolve the [tax, legal, Stripe, privacy, hosting, and support record](operations/phase-1-external-approval-record.md) and approve a live release plan. The demo does not imply those decisions have been made.
