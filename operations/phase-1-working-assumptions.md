# Phase 1 working assumptions for synthetic development

**Status: Editable demo proposal, updated 2026-11-15.** These values support synthetic development and owner review. They do not constitute Italian legal or tax advice, Stripe account approval, a staffing commitment, or permission to transact with real customers or professionals. The [external approval record](phase-1-external-approval-record.md) remains pending.

## Current product direction

- The pilot area is the Comune di Ferrara, with Home Cleaning as its only active service. `ferrara-v1` and `IT-FE-FERRARA` replace the earlier Milano active catalog; historical Milano data keeps its original version.
- The proposed platform commission is 15% of the VAT-exclusive base. The customer is intended to buy cleaning from the professional and intermediation from CLEANZZA, subject to adviser review. Actual supplier and fee-recipient classification is open.
- The demo rule `ferrara-demo-v1` uses EUR 25.00 per hour as an **illustrative base only**. A two-hour example is EUR 50.00, with a simulated EUR 7.50 platform share and EUR 42.50 professional amount before any applicable tax or statutory adjustments. The EUR 25–28 professional range in the founder brief is illustrative and not approved pricing. The API retains the older `gross_total` field name for contract compatibility, but its current value is a synthetic example base, not an approved tax-inclusive final total.
- The preferred later live payment model is authorization before service, capture after a confirmed or supported outcome, then a separate transfer. Account-specific timing, merchant/funds-flow treatment, and manual-capture feasibility remain open.
- Professional tax self-declaration, restricted review, and direct referral rewards are founder proposals. No checkbox, VAT-number format check, or valid public registry result authorizes paid work or establishes a tax rate. Existing customer invite attribution pays no reward.

## Synthetic development that can proceed

1. Use the Ferrara Home Cleaning catalog and draft quote rule with synthetic addresses and accounts. Drafts remain `acceptance_ready: false`; a saved old Milano draft remains readable but cannot be newly submitted through the active pilot flow.
2. Exercise the controlled fake-provider booking, cancellation, refund, transfer, and payout story. All monetary amounts are simulated and all tax fields remain unknown.
3. Model onboarding declarations and admin review with synthetic data if needed for the demo, while keeping real identity/fiscal evidence collection and paid eligibility closed.
4. Use Stripe test mode only in a separately configured experiment. It is not a dependency of the Phase 1 demo and does not clear live approval.
5. Show a clear demo notice. No real worker is dispatched, service is booked, card is charged, invoice is issued, or referral reward is paid.

## Before real use

The founder must establish the lawful operator and finalize hourly tier ranges and operational policies. Qualified Italian advisers must approve the role of each party, professional eligibility matrix and evidence, tax treatment, exact final-price and rounding examples, fiscal documents, terms, privacy/retention, refunds, referrals, and reporting duties. Stripe must review the actual Italian account, connected-account type, authorization/capture windows, transfers, liability, refunds, payouts, and reconciliation. Operations must prove support coverage, hosting, access, and recovery. Record non-sensitive conclusions and restricted written response references in the [external approval record](phase-1-external-approval-record.md); leave each entry `Pending` until there is real evidence.

The proposal derives from [ADR 0010](../adr/0010-ferrara-home-cleaning-pilot.md) and the founder's workspace V3 brief dated 2026-11-14. The original Milano/20% assumptions are historical only.
