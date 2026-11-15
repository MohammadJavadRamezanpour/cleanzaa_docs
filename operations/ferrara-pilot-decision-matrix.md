# Ferrara pilot decision matrix

**Source:** Founder V3 brief dated 2026-11-14, recorded as product direction in [ADR 0010](../adr/0010-ferrara-home-cleaning-pilot.md). This is a handoff for synthetic engineering and external review. It does not certify tax treatment, professional eligibility, Stripe capability, or live readiness. The [external approval record](phase-1-external-approval-record.md) remains open.

| Topic | Founder direction usable for synthetic design | Required before real use |
| --- | --- | --- |
| Market and service | Ferrara city; Home Cleaning only. Exclude Milano, Bologna, Car Wash, and Pet Care from active choices. | Confirm exact operational coverage and service qualifications. |
| Commercial model | Proposed disclosed intermediary; 15% of the VAT-exclusive base for CLEANZZA, remaining base for the professional. Platform bears Stripe processing cost. | Counsel and commercialista confirm actual supplier, fee recipient, worker independence, platform obligations, contracts, and fee basis. |
| Professional pricing | Professional chooses an hourly rate within an experience-tier range; no penalty for declining an unaccepted offer. EUR 25–28/h is illustrative. | Founder sets tier ranges and public price definition; counsel reviews constraints and worker classification. |
| Registration | Allow VAT ordinary, VAT forfettario, VAT other, no VAT number, or unsure self-declaration. Keep uncertain cases pending. | Adviser approves required identity/work/fiscal evidence, lawful non-VAT criteria, privacy notice, retention, and terms. |
| Tax declaration | Explicit unchecked, versioned Italian acknowledgement; retain submitted values, actor and timestamp. Declaration does not approve eligibility or tax. | Counsel reviews final copy and lawful basis. An authorized reviewer applies a written policy; public VAT lookup establishes registration only. |
| Admin fiscal review | Separate declaration, registry result, evidence review, eligibility, and effective fiscal treatment. Use `PENDING`, `NEEDS_INFORMATION`, `VERIFIED`, `REJECTED`, `SUSPENDED` as candidate states. Material changes trigger re-review and stop new binding quotes. | Adviser supplies a standard matrix, rate/nature codes, evidence list, recheck interval, and escalation rules. Real evidence access needs restricted storage and audit. |
| Quote | Synthetic example base, 15% simulated split, null tax and `acceptance_ready: false`; old snapshots immutable. If provider is unknown, show a provisional quote. | Approve professional/platform VAT treatment, any no-VAT basis, final tax-inclusive customer total, mandatory charges, rounding, and provider-unknown binding-price policy. No default 22% or zero. |
| Card payment | One card authorization, later capture after confirmation or approved support resolution, transfer after verified capture; fake provider in demo. | Stripe approves actual Italian account, Connect model, capture windows, SCA, liability, transfers, refunds, and payouts; founder confirms completion window. |
| Fiscal documents | Proposed professional cleaning document plus CLEANZZA intermediation document where the approved arrangement permits. | Adviser approves issuer, document type, XML/SdI, issuance timing, credit notes, and retention for B2C/B2B. |
| Refunds | Free pre-start cancellation; full refund for unperformed services; human review after start. Record adjustments without editing accepted snapshot. | Counsel/adviser/Stripe approve partial allocation, tax corrections, transfer reversal and chargeback liability. |
| Referrals | Direct attribution only; no payment for registration or downstream recruiting. Proposed 6%/8%/10% of platform fee with EUR 0.65/0.80/0.95 caps by tier. Existing customer invite storage pays nothing. | Founder sets collision, refund reversal, payout timing/minimum; adviser reviews partner classification, contracts, documents, withholding, and reporting. No real reward ledger or payout is approved. |
| Release | Synthetic demo with fake people, addresses, evidence and money. No real service, fiscal document, or payout. | Signed go/no-go after tax/legal/Stripe/privacy/support/hosting evidence and working reconciled operations. |

## Engineering sequence

1. Keep the active Ferrara catalog, example pricing, frontend contract, and historical version compatibility synchronized.
2. Complete the synthetic customer and professional journey behind the demo flags, with server-owned state, idempotency, ownership, and PostgreSQL tests.
3. Build a synthetic professional declaration and review flow only when it can be clearly labeled as unverified and contains no real fiscal/identity evidence. Do not turn its state into paid eligibility.
4. Obtain the written outside decisions above, update accepted quote/payment/fiscal contracts, and implement the live release through separate reviewed changes.

Full legal and account-specific responses belong in a restricted record. Do not put founder personal tax details, applicant documents, account IDs, or credentials in Git or Trello.
