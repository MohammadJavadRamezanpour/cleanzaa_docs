# Ferrara pilot: Italy tax, eligibility, invoicing, and Stripe review

**Status:** Review request, not an approved tax or payment policy. This blocks real quote acceptance, order submission, charge, transfer, payout, fiscal issuance, and professional activation. The synthetic demo in ADR 0009 can continue with `acceptance_ready: false` and a fake provider.

## Proposed flow for reviewers

- [ADR 0010](../adr/0010-ferrara-home-cleaning-pilot.md) limits the first pilot to Home Cleaning in the Comune di Ferrara. Professionals propose an hourly rate within a tier range still to be finalized. The founder proposes a disclosed intermediary model: customer purchases cleaning from the professional and intermediation from CLEANZZA, subject to legal review.
- Proposed commercial fee: 15% of a VAT-exclusive hourly base. The professional receives the remainder of that base before any applicable tax or statutory adjustments. Legally applicable VAT would be added by the relevant supplier; no rate or no-VAT treatment is presumed. The synthetic EUR 25/hour rule is an example only.
- Proposed card flow: authorize before service or nearer the appointment, capture after customer confirmation or an approved support outcome, then transfer after verified capture. Customer sees the final tax-inclusive price before commitment. No customer wallet, cash, automatic extra charge, or real referral payout is in the first pilot.
- Professional applicants may self-declare ordinary, forfettario, other VAT regime, no VAT number, or unsure. The declaration and a public VAT-number lookup do not approve work eligibility or determine a tax rate. A written standard eligibility and fiscal-treatment matrix is required before real activation.

## Italian adviser and counsel: written answers required

Record the reviewer, professional role, date, actual operator, assumptions, authority, exceptions, and a restricted memo reference. Review the actual contracts and operations, not only this summary.

- [ ] Confirm the lawful operating entity, registration and accounting prerequisites, supplier and fee recipient, principal/intermediary status, professional independence, cleaning-business and insurance requirements, and consumer/recruitment obligations.
- [ ] Approve a standard eligibility matrix for ordinary/forfettario/other VAT holders, lawful non-VAT work, and unresolved applicants. Specify minimal evidence, public-registry checks, identity/work-authorization checks, effective dates, recheck intervals, escalation, and reviewer authority. Identify when applicants must stay on the waitlist.
- [ ] Confirm the taxable person, VAT nature/rate or no-VAT basis for each professional supply and the platform fee. Approve the final tax-inclusive customer display, versioned quote snapshot, fee recipient, mandatory charges, and exact line/order rounding. Provide signed examples for EUR 40, EUR 60 without professional VAT, and EUR 60 with professional VAT or another representative approved treatment.
- [ ] Specify professional-to-customer and CLEANZZA-to-customer invoice/receipt issuers, B2C/B2B variants, SdI/XML where applicable, timing at authorization/capture/completion, correction and credit-note rules, immutable originals, numbering, and retention. Provide sample documents.
- [ ] Approve customer and professional terms, the Italian tax self-declaration acknowledgement, privacy notice/lawful basis, evidence access and retention, cancellation/refund wording, complaints, and any required reporting, withholding, stamp-duty, or DAC7 process.
- [ ] Decide the exact treatment of full/partial refund, dispute, chargeback, failed capture, and transfer reversal; preserve accepted snapshots and record adjustments separately.
- [ ] Review direct referral rewards, organization contracts, partner status, withholding, documents, reversals, and payout policy before any money is owed or paid.

## Stripe account team: account-specific written answers required

Review the actual Italian platform and intended connected-account type. Share account identifiers through a secure channel, never in Git or Trello.

- [ ] Confirm that this account may authorize one customer card payment, use manual capture at the proposed time, and make a later separate transfer to an as-yet-unknown professional for Home Cleaning. Confirm the payment business-of-record and legal supplier/invoice alignment, including `on_behalf_of` and `source_transaction` requirements.
- [ ] Confirm authorization lifetime and `capture_before`, distant-booking and reauthorization policy, SCA and saved-method consent, capture-failure handling, and the permitted fallback if a hold expires. Do not assume a hold lasts until service completion.
- [ ] Approve connected-account API/version, service agreement, onboarding/KYC, capabilities, country restrictions, fees, liability, negative balances, transfer and payout timing, refund and reversal mechanics, and restricted-account handling.
- [ ] Specify signed webhook events, idempotency, delayed/duplicate/out-of-order event handling, reconciliation, and evidence of captured funds and external payout. Walk through success, failed SCA, expired authorization, disputed completion, refund, transfer reversal, and unavailable payout in test mode.

## Approval and implementation gate

The owner records non-sensitive conclusions and references to restricted written responses in the [external approval record](phase-1-external-approval-record.md). Before live use, update ADR 0010, Phase 1 scope, terms, Trello D2/D3/B09/B11/B20, feature contracts, OpenAPI, migrations, tests, and CI together. Then verify real access control, eligibility, tax-inclusive immutable quotes, payment/reconciliation, support coverage, refund funding, fiscal documents, and a signed go/no-go. Leave every approval entry `Pending` until evidence exists.

The founder's V3 business/tax/payment brief dated 2026-11-14 is the product proposal being reviewed. It is not an adviser or Stripe decision.
