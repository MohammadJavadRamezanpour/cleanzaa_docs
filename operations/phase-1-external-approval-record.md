# Phase 1 external approval record

**Status:** Open review template. No Italian tax/legal conclusion or account-specific Stripe approval has been received or recorded here. This document does not authorize real quotes, bookings, charges, transfers, or identity collection.

Use the detailed [Italy tax and Stripe review checklist](italy-tax-stripe-review-checklist.md) as the question set. This record captures the written responses and the owner's resulting decisions without putting account identifiers, credentials, identity evidence, or private contracts in Git. Keep full responses in a restricted project record and put only non-sensitive references below.

The [editable Phase 1 working assumptions](phase-1-working-assumptions.md) provide concrete starting values for synthetic development and owner review. They do not change any `Pending` response or clear this record's hold on real use.

## Product proposal submitted for review

Cleanzza proposes a Comune di Milano launch for Home Cleaning, Car Wash, and Pet Care. [ADR 0004](../adr/0004-phase-1-pricing-direction.md) sets tax-inclusive displayed hourly totals of EUR 20, EUR 25, and EUR 20, with 1–8 whole hours per service. [ADR 0006](../adr/0006-phase-1-payment-direction.md) proposes a Stripe Connect platform charge at booking and a later worker transfer, with a nominal 20% platform and 80% worker split of the gross customer total. These are product proposals, not verified tax, invoice, merchant, or Stripe-account rules.

## Italian tax and legal review

| Required record | Response or restricted-record reference |
| --- | --- |
| Adviser name, professional role, firm, review date, legal entity, and assumptions | Pending |
| Legal supplier, contracting parties, principal/agent status, and invoice/receipt issuer for each service | Pending |
| VAT treatment for each service and platform/worker supply, with source or authority | Pending |
| Approved customer totals, platform/worker split, tax allocation, and EUR-cent rounding examples | Pending |
| Invoice/receipt issuance, corrections, refunds, disputes, and retention | Pending |
| Reviewed customer and worker terms, cancellation/refund notices, and consent text | Pending |
| Italian employment/self-employment and marketplace obligations affecting this flow | Pending |
| Written conclusion, exceptions, and restricted memo reference | Pending |

## Stripe account review

| Required record | Response or restricted-record reference |
| --- | --- |
| Stripe reviewer/team, review date, Italian platform account and connected-account type (private reference only) | Pending |
| Approval or rejection of the proposed platform charge and separate transfer for these services and parties | Pending |
| Merchant/funds-flow configuration, account capabilities, service agreement, country and transfer restrictions | Pending |
| Checkout/SCA, capture proof, signed webhook events, duplicate and delayed-event behavior | Pending |
| Transfer and payout timing, restricted accounts, refunds, reversals, disputes, and reconciliation | Pending |
| Controlled test-mode walkthrough and account-specific written response reference | Pending |

## Other Phase 1 launch gates

The payment review does not clear these separate gates:

| Gate | Evidence required before use |
| --- | --- |
| Private identity and service evidence ([ADR 0002](../adr/0002-phase-1-personal-data.md)) | Reviewed legal basis, regional provider/contracts, purpose-specific retention and backup deletion, access/hold procedure, and passport capture procedure before real evidence is collected. |
| Reviewer access ([ADR 0005](../adr/0005-admin-managed-mfa.md)) | Cleanzza-owned Auth0 EU tenant, MFA, allowlisted reviewer accounts, and a tested reviewer flow. |
| Staging ([ADR 0001](../adr/0001-staging-hosting.md)) | Approved host/account owner, separate managed database and secret stores, external health checks, alert delivery, and the [restore drill](backup-restore.md). Use synthetic data until personal-data gates clear. |
| Customer and worker activation ([ADR 0002](../adr/0002-phase-1-personal-data.md)) | Approved immutable terms versions and public HTTPS URLs; reviewed SMS provider credentials and browser cookie/CORS validation. |
| Active-job support ([ADR 0008](../adr/0008-phase-1-boundaries-and-support.md)) | Named duty owner, coverage roster, contact and escalation channels, response target, fallback, and urgent-case rehearsal. |

## Owner decision and implementation handoff

**Current decision:** Hold real acceptance, order submission, payment, payout, and production identity collection. Draft quotes remain `acceptance_ready: false`; the existing order table permits `DRAFT` only.

When written responses arrive, the Cleanzza owner records the response references, decision date, approvers, accepted assumptions, unresolved exceptions, and effective date here. If the proposed model changes, update ADRs 0004 and 0006, the product scope, relevant Trello cards, and the API/feature contracts before implementing acceptance. Implement and test the approved quote snapshot, order, payment, and payout path in dependency order, then complete staging and operational verification. Do not mark this gate cleared solely because this template exists or a local `.env` file has been populated.
