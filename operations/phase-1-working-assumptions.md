# Phase 1 working assumptions for synthetic development

**Status: Editable proposal, 2026-10-24.** These assumptions let the team design and test with synthetic data. They are not Italian legal or tax advice, a Stripe account approval, a staffing commitment, or permission to transact with real customers or workers. The [external approval record](phase-1-external-approval-record.md) remains pending.

## What the product has already chosen

- Milano only; Home Cleaning, Car Wash, and Pet Care at the gross EUR hourly prices and booking rules in [ADR 0004](../adr/0004-phase-1-pricing-direction.md).
- A nominal 20% platform / 80% worker split of the gross customer total, one card charge at booking, and a later separate transfer to the worker under [ADR 0006](../adr/0006-phase-1-payment-direction.md).
- No cash, wallet, saved-card debit, automatic cancellation fee, or guaranteed live chat in Phase 1. The [Phase 1 scope](../phase_1_product_scope.md) and ADR 0008 define the boundary.

These are product directions, not conclusions about the supplier, invoice issuer, VAT, employment status, Stripe account configuration, or actual worker net pay.

## Proposed assumptions to edit

| Topic | Starting assumption for design and tests | What must be decided before real use |
| --- | --- | --- |
| Customer-facing price | The listed EUR amount is the entire customer total; no tax or fee is added at checkout. Synthetic examples use the ADR 0004 arithmetic. | Adviser confirms each supply, tax basis/rate or exemption, invoice issuer, display and rounding. Change the price model if that conclusion requires it. |
| Legal and commercial roles | Model Cleanzza as the *intended* customer-charge business of record and the worker as the intended service performer. Keep contracting-party and employment labels out of production terms until reviewed. | Italian counsel and tax adviser identify the legal supplier and all contracting/invoicing parties for each service and the platform fee. |
| Quote snapshot | A draft may store gross lines, rule versions, gross total, nominal platform/worker amounts and `acceptance_ready: false`. Tax treatment and tax amount remain unknown, not zero. | Versioned, adviser-approved tax treatment and worked EUR-cent examples must populate accepted snapshots before acceptance is enabled. |
| Payment | Exercise a fake provider in automated tests. If Cleanzza provisions Stripe, use its test mode for an account-specific walkthrough; never use live keys or real cards to validate this draft. | Stripe confirms the Italian platform and connected-account setup, capabilities, permitted funds flow, liability, checkout/SCA, refunds, reversals and payout behavior in writing. |
| Transfer and refunds | Synthetic state-machine tests follow ADR 0006: no worker transfer before customer confirmation or authorized support outcome; cancellation/refund is a separate movement. | Counsel, tax adviser and Stripe approve actual liability, funding and invoice corrections; operations can reconcile failed, disputed and reversed movements. |
| Staging | Use separate Cleanzza-owned staging accounts and synthetic records only. The repository `.env.sample` files are field lists, not issued credentials; the owner will create local `.env` files. | Owner provisions hosting, managed database, Auth0 EU tenant, SMS, private storage, Stripe test account, secrets, monitoring and restore access; each gate in the external approval record is evidenced. |
| Support coverage | **Proposed pilot:** offer appointments only 09:00–18:00 Europe/Rome when an assigned duty owner is on shift; cover 08:30–18:30. Aim to acknowledge urgent active-job cases within 15 minutes and escalate unresolved cases to a named backup at 30 minutes. These are planning values, not a service promise. | Owner names primary/backup people, channels, actual roster, response targets and fallback, rehearses urgent-case handling, and publishes only the support promise it can meet. |
| Real identity and service photos | Use synthetic placeholders in development and staging; production collection remains off. | Privacy/legal review approves purpose, provider/region, access, retention including backups, deletion/hold handling and capture procedure under ADR 0002. |

The European Commission explains that Member States set the applicable VAT rates and categories within the EU framework. Its Italy listing of a standard rate does not determine the treatment of these particular supplies. [EU VAT rate framework](https://taxation-customs.ec.europa.eu/taxation/vat/vat-directive/vat-rates_en) and [Italy VAT rules](https://vat-one-stop-shop.ec.europa.eu/national-vat-rules/italy-vat-rules_en) are starting references for the adviser, not values to code into accepted quotes.

Stripe describes separate charges and transfers as a platform charge followed by transfers to connected accounts, and says the platform balance bears its Stripe fees, refunds and chargebacks. A refund does not automatically reverse a related transfer. These mechanics support the proposed test cases but do not approve Cleanzza's specific account or legal structure. See [Stripe's Connect guide](https://docs.stripe.com/connect/separate-charges-and-transfers).

## Synthetic development that can proceed

1. Complete deterministic draft-quote and order-draft behavior, including versioned snapshots and PostgreSQL tests. A draft cannot become a real accepted order or matching request.
2. Define and test payment/fulfillment transitions with fake provider events and synthetic users, keeping any public booking, charge, transfer and identity-capture path disabled.
3. Add Stripe test-mode integration only after Cleanzza has provisioned the relevant account and test credentials. Capture the walkthrough and open questions in the restricted project record.
4. Keep customer-visible copy accurate: show a draft price and disabled submission until the external gates are closed. Do not display provisional tax or worker-payout figures as finalized.

## Owner edit and approval path

For each row above, the owner may change the proposal and add a dated decision. The owner should put the non-sensitive conclusion and a reference to each restricted written adviser/Stripe response in the [external approval record](phase-1-external-approval-record.md). Do not replace `Pending` with an invented approver, approval date, account capability, VAT rate, or contract conclusion. If review changes the model, revise ADRs 0004/0006 and the product/API contracts before enabling real acceptance. A local `.env` file alone never clears a launch gate.
