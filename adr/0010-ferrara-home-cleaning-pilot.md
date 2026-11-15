# ADR 0010: Ferrara home-cleaning pilot direction

**Status:** Founder product direction, 2026-11-15. Synthetic demo implementation is approved; real operation remains gated.

## Decision

The first pilot is the Comune di Ferrara, Italy, for **Home Cleaning only**. Milano, Car Wash, and Pet Care are no longer active Phase 1 choices. The active demo area is `IT-FE-FERRARA`, city `Ferrara`, with a coarse CAP gate of `44121`–`44124` and timezone `Europe/Rome`. The CAP check does not establish that a street is inside the municipal boundary. Existing `milano-v1` and `milano-hourly-v1` identifiers and saved historical snapshots must not be silently reinterpreted as Ferrara.

The founder's proposed commercial arrangement is a disclosed intermediary model, with the professional supplying cleaning and CLEANZZA supplying intermediation. The platform's proposed commission is **15% of a VAT-exclusive base**, paid from that base, with legally applicable VAT added to the appropriate supplies after adviser approval. This replaces the earlier 20% gross split as product direction. The proposed arrangement, fee recipient, tax treatment, invoice issuers, and Stripe configuration are **not approved for live use**.

The current demo may use a versioned, clearly labeled example hourly base and a 15% simulated split. It must keep tax treatment unknown, `acceptance_ready: false`, and all orders/payments synthetic. No demo amount is a final customer price or worker net pay. Real binding quotes require an approved professional fiscal profile and tax-inclusive final price; if the provider is unknown, keep the quote provisional or obtain an approved fixed-price policy with explicit reacceptance of any change.

For a later live release, the preferred payment flow is authorization at booking or near the service date, capture after customer confirmation or an approved support outcome, then a separate transfer after verified capture. The proposed 60-minute confirmation window and 30-minute reminder remain open. Do not implement real manual capture or replace the existing live-use gate without account-specific Stripe and legal approval. Cash, customer wallets, and automatic extra charges remain excluded.

The founder also proposes versioned professional tax self-declaration, restricted admin review, and direct referral rewards. These can be modeled with synthetic data for the demo. A declaration or VAT-number lookup is never an approval of eligibility or a tax treatment; live activation needs a written adviser-approved eligibility matrix. Referral payout and fiscal treatment remain open. The storage-only customer invitation feature may continue, but no real reward may accrue or be paid.

## Consequences and compatibility

This ADR supersedes the Phase 1 market, service, price-split, and payment-timing product directions in ADRs 0003, 0004, and 0006. Their historical rule versions and accepted snapshots remain immutable. The active catalog and quote APIs move to new Ferrara versions. This is a breaking change for clients pinned to the previous catalog or service-area literal; update and deploy the backend OpenAPI and frontend client together. Existing Milano draft data remains readable but cannot be submitted as a new Ferrara request.

ADR 0009 remains the delivery target: a controlled synthetic end-to-end demo. The [external approval record](../operations/phase-1-external-approval-record.md) remains pending for real use. No real quote acceptance, service dispatch, payment, payout, fiscal document, or identity collection is unlocked by this ADR.

## Decisions still required

- Italian counsel and a commercialista must approve the actual operator, supplier/fee recipient, worker eligibility and evidence matrix, VAT treatment, final totals, rounding, fiscal documents, privacy notices, retention, and reporting duties.
- Stripe must review the actual Italian account and proposed authorization, capture, Connect transfers, onboarding, liability, refunds, and payout configuration.
- The founder must finalize hourly tier ranges, provider-unknown pricing policy, confirmation window, referral attribution collisions and reward adjustments, and payout schedule before those rules can become live behavior.

The source founder brief is `CLEANZZA_Phase_1_Business_Tax_Payment_Decisions_v3.md` in the workspace root, dated 2026-11-14. It is a proposal for review, not an external approval. Poste Italiane lists Ferrara city CAPs `44121`–`44124` in its [city CAP table](https://business.poste.it/business/files/1473806071814/elenco-cap-province-afferenti.pdf); the coarse application gate should be reviewed before live coverage is promised.
