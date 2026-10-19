# ADR 0007: Phase 1 fulfillment and confirmation

**Status:** Approved product direction by the owner on 2026-10-19. Implement with B11–B19; no order flow exists yet.

## Customer labels and backend facts

| Customer label | Authoritative fulfillment fact |
| --- | --- |
| Requested | Captured booking payment confirmed and the order is open for worker offers; no worker assigned. |
| Accepted | One eligible worker accepted an offer in a transaction and is assigned. |
| On the Way | The assigned worker explicitly tapped **Start travel** after acceptance. |
| In Progress | The assigned worker checked in, submitted the required private Before Photo, and explicitly tapped **Start service**. |
| Completed | The assigned worker submitted the private After Photo and explicitly tapped **Finish work**. Show `Confirmation pending` until the customer confirms or support records an outcome. |

The five labels are projections of server-recorded events. Arrival/check-in does not itself start paid work. Capturing or finalizing the Before Photo does not itself start paid work. The backend records the start only on a valid Start service command after both prerequisites. A camera/browser timestamp is never authoritative. A worker can upload the After Photo only after service has started, and Finish work requires it. The customer cannot retrieve the raw service photos.

The customer confirms only their own worker-completed order. Confirmation is a separate event and can happen only once; it does not rewrite the worker's completion time. If the customer does not confirm, the order stays Completed with confirmation pending and the worker transfer stays on hold. Support receives a review case and may record a reasoned, authorized resolution; there is no automatic confirmation or payout. A disagreement is routed to that review case, with photos accessible only to authorized case-scoped staff under ADR 0002. Any payout follows the separate D3 payment rule in ADR 0006. A rating is offered only after customer confirmation or a support-approved resolution, and remains optional.

## Allowed transitions

```text
DRAFT -- captured provider payment --> REQUESTED
REQUESTED -- one offer accepted --> ACCEPTED
ACCEPTED -- assigned worker starts travel --> ON_THE_WAY
ON_THE_WAY -- assigned worker checks in --> ON_THE_WAY (arrival recorded)
ON_THE_WAY -- Before Photo ready and worker starts service --> IN_PROGRESS
IN_PROGRESS -- After Photo ready and worker finishes --> WORKER_COMPLETED
WORKER_COMPLETED -- customer confirms --> CONFIRMED
WORKER_COMPLETED -- support approves outcome --> SUPPORT_APPROVED
WORKER_COMPLETED -- support flags disagreement --> REVIEW_REQUIRED
```

`WORKER_COMPLETED`, `CONFIRMED`, `SUPPORT_APPROVED`, and `REVIEW_REQUIRED` project to the customer label **Completed** with distinct confirmation/substatus text. The implementation may use separate fulfillment and confirmation aggregates rather than these exact stored enum names; it must preserve the event rules. A support outcome that denies completion must keep payment/payout on hold and route to a refund or correction decision. Support cannot silently fabricate customer confirmation.
Payment pending, capture, refund and payout are separate financial states under ADR 0006; a draft order does not enter matching until payment capture is confirmed.

Before Start service, the customer can cancel without a fee, including after worker assignment/travel; the worker/platform can also cancel and the customer receives a full refund under ADR 0004. A no-show or inability to fulfill becomes a terminal failure with an explicit reason and full refund. After Start service, ordinary self-service cancellation is unavailable: support records an outcome and any partial/full refund after review. `CANCELLED` and `FAILED` are terminal customer-visible states outside the five active/completed labels. The customer sees the reason and refund progress; a failed payment never reaches Requested. Offer expiry does not by itself cancel a still-valid order; if no worker can fulfill it by the operating cutoff, support or a defined expiry command fails it and refunds the customer. The exact cutoff and support staffing belong to D6 before live booking.

Each command validates actor, assignment/order ownership, current state and required evidence in one transaction, uses a server timestamp, records a history event, and handles duplicate retries idempotently. Out-of-order actions return a stable conflict. The assigned worker alone can start travel, check in, start service and finish; the owning customer alone can confirm; designated support/admin alone can resolve a missing confirmation. The worker must not receive another overlapping job while this one is active. B14 owns atomic assignment and schedule exclusion. B18 owns private photo linking, retention and read audit.

## References

- [Phase 1 product scope](../phase_1_product_scope.md)
- [Pricing, overtime, cancellation and no-show](0004-phase-1-pricing-direction.md)
- [Payment and payout direction](0006-phase-1-payment-direction.md)
- [Personal data and photo access](0002-phase-1-personal-data.md)
