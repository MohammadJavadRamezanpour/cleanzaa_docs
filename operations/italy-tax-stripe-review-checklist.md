# Italy tax, invoicing, and Stripe review for Phase 1

**Status:** Review request, not an approved tax or payment policy. No real quote acceptance, order submission, or charge is authorized by this checklist.

**Blocks:** D2's tax decision; B09 accepted price snapshots; B11 real service requests; B20 payment and payout. The draft `POST /quotes/drafts` preview remains available with `acceptance_ready: false`.

## Give reviewers this proposed flow

- Cleanzza launches in Comune di Milano with Home Cleaning, Car Wash, and Pet Care. Published hourly customer totals are EUR 20, EUR 25, and EUR 20 respectively, inclusive of any applicable tax. One to three services may be combined, each for 1–8 whole hours. See [ADR 0003](../adr/0003-phase-1-launch-services.md) and [ADR 0004](../adr/0004-phase-1-pricing-direction.md).
- [ADR 0006](../adr/0006-phase-1-payment-direction.md) proposes one card charge to the Cleanzza Stripe platform account at booking, a private `PAYMENT_PENDING` order until capture is confirmed, and a later separate transfer to the assigned worker's connected account after customer confirmation or support resolution. Cleanzza is intended to be the business of record for the payment; this does **not** establish the legal supplier or invoice issuer.
- The nominal commercial split is 20% of the tax-inclusive customer total for Cleanzza and 80% for the worker. Stripe fees come from Cleanzza's share. No wallet, cash, off-session debit, automatic penalty, or automatic overtime charge is proposed.
- A customer cancellation before service starts is free. Worker/platform cancellation or worker no-show calls for a full refund of any captured charge. After service starts, a partial or full refund requires human review. See ADRs 0004 and 0006.

## Italian tax and legal adviser: written answers required

Record the adviser's name, professional role, date, legal entity reviewed, assumptions, and the source or authority for each conclusion. Review the actual customer and worker terms, not only this summary.

- [ ] Identify the legal supplier of each service and the invoice/receipt issuer to the customer. State whether Cleanzza acts as principal, agent, or another role for each transaction, and whether the proposed Stripe payment configuration and worker contract fit that role. Check worker employment/self-employment and marketplace obligations with appropriate Italian counsel.
- [ ] For **each** of Home Cleaning, Car Wash, and Pet Care, confirm the applicable VAT treatment, rate or exemption, taxable person, place and time of supply, and the evidence needed to support it. Do the same for Cleanzza's platform fee and any worker-to-platform or worker-to-customer supply. Do not infer treatment from Italy's standard rate.
- [ ] Confirm whether the EUR 20/25/20 tax-inclusive customer totals and 20%/80% gross split can be used as proposed. If not, provide corrected customer/worker/platform examples and revised terms before code or published pricing changes.
- [ ] Give an exact allocation and EUR-cent rounding rule for a single service and a mixed-service order. Include taxable base, rate, tax amount, gross line, gross order total, platform fee/share, worker amount, and how totals reconcile. Use 2h Home Cleaning = EUR 40.00; 2h Car Wash = EUR 50.00; 3h Pet Care = EUR 60.00; and 2h Home Cleaning + 1h Car Wash = EUR 65.00 as worked examples.
- [ ] Specify which invoice, receipt, electronic-invoice or fiscal-document process applies to B2C and any B2B case; who issues it; required customer/worker identifiers; issuance timing at charge, completion, refund, or transfer; Sistema di Interscambio use if applicable; corrections/credit notes; and accounting retention. Give a sample customer document and any worker/platform document.
- [ ] Confirm how a full cancellation refund, a post-start partial refund, chargeback, failed charge, and transfer reversal affect tax documents, VAT reporting, and the worker/platform split. State whether the accepted price snapshot stays unchanged while adjustments are recorded separately.
- [ ] Identify the exact customer and worker terms, consent text, cancellation/refund notices, and record-retention obligations required before live booking. Provide the approved text/version or assign an owner and due date.

## Stripe account team: account-specific written answers required

Ask Stripe to review the **actual Italian platform account and intended connected-account type**, with account IDs shared through a secure channel. Do not put secret keys or personal account documents in Git or Trello.

- [ ] Confirm that this Cleanzza account may use a platform charge followed by a separate transfer to an as-yet-unknown worker, for these service categories and Italian parties. Confirm whether omitting `on_behalf_of` gives the intended payment business-of-record behavior for this account and whether the proposed legal supplier/invoice model is compatible. Record any country, timing, source-transaction, or balance restrictions.
- [ ] Select the supported connected-account API/version, dashboard/hosted-onboarding model, service agreement, required transfer and payout capabilities, KYC requirements, and who bears fees, negative balances, disputes, and chargebacks. Obtain the specific configuration and approval status for this account; general documentation is not account approval.
- [ ] Confirm the card checkout and Strong Customer Authentication flow, the event that proves a charge is captured, and what to do with `requires_action`, delayed, failed, duplicate, or out-of-order outcomes. Confirm the webhook events and signed-event verification needed for charge, refund, transfer, payout, dispute, and capability changes.
- [ ] Confirm the allowed timing and conditions for a worker transfer after customer confirmation/support resolution, how long funds may remain on the platform, what happens when the connected account is restricted, and how payout confirmation differs from transfer creation.
- [ ] Confirm refund, partial refund, transfer reversal, dispute, fee and reconciliation mechanics. Provide a test-mode walkthrough for a successful EUR 40 booking, SCA delay, customer cancellation/full refund, disputed charge, and unavailable worker payout; include provider IDs and expected ledger events in a controlled test record.

## Approval packet and implementation gate

The owner records links to the written tax/legal memo, Stripe account response, approved customer and worker terms, and worked examples in a restricted project record. The public ADR records only decisions and non-sensitive references. Before enabling acceptance:

- [ ] Amend ADRs 0004 and 0006 with the reviewed legal supplier, tax matrix, invoice flow, payment-account configuration, refund/transfer rules, effective date, and named approvers. If the proposed model changes, update the product scope and Trello D2/D3/B09/B11/B20 cards together.
- [ ] Version the tax treatment and pricing/commission rules. Define the accepted snapshot's non-null tax basis/rate/amount fields and deterministic allocation/rounding examples. Keep historical snapshots immutable; use separate refund and transfer records.
- [ ] Implement and test the production acceptance gate, order/payment idempotency, customer activation/terms checks, Stripe webhook verification and reconciliation, and PostgreSQL money/concurrency invariants. Verify OpenAPI and pinned frontend contracts, migrations, and hosted CI before any live charge.
- [ ] Complete a signed go/no-go review for the actual Stripe account, terms, tax documents, and operational refund/payout handling. Until then, keep `acceptance_ready: false` and real submission/payment unavailable.

## Reference material for reviewers

These are starting points for review, not a decision about Cleanzza's tax or contractual status: [EU VAT charging and invoices](https://europa.eu/youreurope/business/finance-and-tax/vat/charging-deducting-vat/index_en.htm), [Italy e-invoicing country sheet](https://ec.europa.eu/digital-building-blocks/sites/spaces/einvoicingCFS/pages/956171290/2026%2BItaly%2B2026%2BeInvoicing%2BCountry%2BSheet), [Stripe separate charges and transfers](https://docs.stripe.com/connect/separate-charges-and-transfers), [Stripe connected accounts](https://docs.stripe.com/connect/accounts-v2), [Stripe service agreements](https://docs.stripe.com/connect/service-agreement-types), and [Stripe webhooks](https://docs.stripe.com/webhooks).
