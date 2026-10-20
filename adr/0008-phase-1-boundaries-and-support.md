# ADR 0008: Phase 1 boundaries and support operations

**Status:** Product direction approved by the owner on 2026-10-20. Staffing, response commitments, and launch operations remain gated as described below.

## Included in Phase 1

- Use the one card payment flow in [ADR 0006](0006-phase-1-payment-direction.md). A customer sees and accepts one reviewed total before booking; no automatic penalty or off-session charge is added.
- Provide a simple case intake for a customer or worker to report a service problem, complaint, payment concern, or safety issue. A case has an order reference when available, a category, description, creator, timestamps, status, and recorded support actions. Authorized support/admin staff can review it. A general case does **not** require a Before Photo or Peace of Mind enrollment and is not rejected automatically at 24 hours. Any special guarantee or compensation program needs a separate policy, terms, and review before it can be advertised or enforced.
- Assign a human Cleanzza operations duty owner during every active job window. That owner can receive and triage urgent cases, contact the affected parties through an approved channel, document the response, and escalate to an authorized decision maker. Support resolution of absent customer confirmation follows [ADR 0007](0007-phase-1-fulfillment-states.md); it does not automatically release a worker payout.

The case intake may be asynchronous. Phase 1 does not promise 24-hour live chat or a specific response time until a roster, coverage hours, contact channel, escalation path, response target, handoff process, and fallback when the duty owner is unavailable have been approved and tested. A named operating owner and sufficient staffing for active jobs are launch gates, not merely application fields. Cases and any attachments follow the private access, audit, retention, and legal-hold rules in [ADR 0002](0002-phase-1-personal-data.md); no raw service photo is exposed to either party merely because a case exists.

## Deferred from Phase 1

- Cash collection and the EUR 10 cash-commission/debt rule.
- Customer balance, top-up, internally custodied wallet, and payout wallet.
- Customer and worker referrals, rewards, and repeat-worker discounts or surcharges.
- In-app emergency-call, police contact, or location-sharing feature.
- Always-available live support chat and any guaranteed response SLA.
- Automatic fines, cancellation penalties, bonuses, or other off-session debits.

These are explicit later-phase candidates, not silently discarded product ideas. Each needs its own product, legal, provider, privacy, safety, and operating decision before implementation. The app must not suggest that a Cleanzza case or on-call owner replaces local emergency services. A user facing an immediate emergency should use the applicable local emergency channel outside the app.

## Consequences and launch gate

The first release needs one case workflow and an operational queue/owner before active paid jobs begin. The backend will own case authorization, state, audit history, and any linkage to order or payout decisions. A support agent cannot create an automatic refund, penalty, or transfer from a free-text case alone. The exact case statuses and response workflow are implementation details to define in the case feature document, consistent with this ADR. Before launch, record the operating owner, staffing schedule, coverage during all active jobs, escalation and fallback channels, customer/worker contact wording, response targets, privacy retention, and a rehearsal of an urgent case. Until these are approved, do not advertise live or guaranteed support.

This decision resolves D6's Phase 1 feature boundary. It does not clear Italian tax, invoicing, payment-provider, identity-data, or production support-readiness gates recorded elsewhere.
