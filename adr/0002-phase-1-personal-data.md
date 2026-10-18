# ADR 0002: Phase 1 personal-data and account policy

**Status:** Phase 1 product direction approved 2026-10-18. The interpretations
below are recorded for implementation and may be changed by a later ADR. Legal,
provider, and retention review remains required before production collection.

## Decision

### Accounts and access

- A successful, single-use, rate-limited phone code creates a customer account
  and permits sign-in. It is **pending activation** until the customer provides
  their name and service address and accepts the current terms. A pending
  customer may complete their profile but cannot place an order. Phone
  verification proves control of the number, not the person's legal identity.
- A cleaner cannot receive or accept work until an authorized administrator
  manually approves identity and service qualifications. The cleaner must
  submit a live in-app selfie and live captures of both sides of an ID card, or
  the identity page and other required side of a passport, plus evidence needed
  for the services they offer. The reviewer records the decision, reason, actor,
  and time. Work authorization is checked where applicable.
- Phase 1 cleaner registration and sign-in use the same single-use phone-code
  mechanism and revocable session model as customers, with a distinct `CLEANER`
  role and pending worker profile. Username/password and email confirmation are
  not Phase 1 gates. A person may hold both customer and cleaner roles, but a
  phone code cannot grant job eligibility; manual approval is always required.
  An email may be added later for notices, with verification before it is used
  for sensitive communications.
- Collect the minimum worker profile needed for review: legal first and last
  name, verified phone, live selfie, live ID/passport captures, work-authorization
  evidence where applicable, offered services, service areas, and qualifications
  required for those services. Capture a versioned worker terms acceptance once
  approved terms are published. Service-specific fields and exact area choices
  are defined in [ADR 0003](0003-phase-1-launch-services.md). Defer separate
  birth details, residential address,
  employment/student status, psychological assessments, fiscal code, and bank
  details until a documented legal or provider need is approved; do not copy
  payment account details into Cleanzza storage.
- The assigned cleaner receives the exact service address only after accepting
  the order. Before acceptance, the listing omits customer identity, exact
  address, phone number, and private images. Private data is available only to
  the actor who needs it, with access logged.

### Hosting, service data, and publication

- “Internal servers” means infrastructure and private storage controlled by
  Cleanzza in its own cloud account, with provider contracts and regional
  controls reviewed before launch. A managed cloud service can operate that
  infrastructure; it does not require Cleanzza-owned physical hardware. A
  provider receives only the data needed for an approved SMS or payment task.
  Identity images and service media remain in Cleanzza-controlled private
  storage, rather than a third-party identity collection service.
- Raw service photographs are available only to authorized administrators and
  case-scoped support staff. The cleaner can capture and submit them but cannot
  browse the stored originals; the customer cannot retrieve them. Customer
  completion confirmation and complaints therefore cannot depend on viewing
  those photographs. Access is logged.
- Worker identity images, selfies, and qualification evidence are available to
  the submitting worker only during the active capture/upload flow and to
  designated reviewers for a recorded review purpose. The worker sees evidence
  status afterward, not a raw download. General support access does not include
  identity evidence. Every reviewer read is audited. Before job acceptance the
  worker sees only a redacted offer; afterward only the assigned worker receives
  the exact address and needed job details. The customer sees the assigned
  worker's public profile, not private identity documents.
- Raw service photographs and temporary service-location notes are deleted
  seven days after completion, unless a documented complaint, dispute, or legal
  hold requires a longer, separately recorded period. This seven-day rule does
  not erase transaction records, ratings, required audit records, or the
  account data needed for the exit process. Backup expiry and legally required
  financial retention need their own schedules before launch.
- After completion, administrator approval, and recorded customer satisfaction,
  a cleaner may show the rating and non-private achievements on their profile.
  With the homeowner's separate, revocable photo-publication consent, the
  cleaner may also show approved non-private satisfaction feedback and home
  photographs. An authorized reviewer approves a separate publication copy
  after removing metadata and checking that no person, document, address, contact
  detail, or other identifying information is visible. The homeowner's name,
  address, and contact details are never shown. The publication item also omits
  the cleaner's legal name, private address, and contact details; the cleaner's
  general profile identity remains visible. The approved copy is distinct from raw
  service evidence and remains only while consent and the profile publication
  remain active; withdrawal removes it. Without photo consent, only the rating
  and non-private achievements may appear.

### Decisions, corrections, and exit

- An authorized reviewer records acceptance, rejection, dismissal,
  reinstatement, fines, and bonuses with a reason and audit trail. One
  appropriately authorized approver is sufficient; a mandatory second approver
  is not assumed. Financial penalties and bonuses cannot be executed until
  their amounts, appeal process, and legal basis are separately approved.
- Customers and cleaners may edit ordinary profile details and request a copy
  or deletion of their data. A change to a verified identity record uses a
  formal correction request: fresh SMS verification, a new selfie, identity
  document images, and a signed request letter in private archival storage,
  followed by authorized manual review. This extra evidence is not required
  for ordinary profile edits. Changing a phone number always requires a fresh
  code for the new number. Correction evidence has a separate retention and
  access schedule before this workflow launches.
- The three-month exit review applies to **cleaners**, since they have former
  clients. At exit, the cleaner is immediately ineligible for new work and the
  account is restricted, not active. During the review period, Cleanzza sends
  former clients with an available verified email address a limited,
  non-repeated email asking about unresolved service issues. No reply is not
  evidence of a problem. After three months, delete
  ordinary cleaner account data unless an actual unresolved case or legal duty
  requires a documented hold. A reported issue keeps only relevant data in a
  restricted state until resolved; it never automatically reactivates the
  cleaner. Customer deletion requests use a separate prompt process, subject
  to documented legal holds. Referral to authorities occurs only after human
  review and an applicable legal basis, not automatically on an allegation.

## Consequences and implementation boundaries

The B03 phone authentication slice creates a customer and session. Its
`CUSTOMER` role means only that the user may access customer functions; it does
not mean the account is active or eligible to book. The following
[profile and activation slice](../features/customer-profile.md) records name,
address, and terms acceptance and exposes pending versus active state. It
provides a database-backed booking eligibility policy, which the future order
creation transaction must call; no order endpoint exists yet. Cleaner review,
private upload, admin approval, photo publication, offboarding, and correction
are separate delivery slices with authorization and PostgreSQL integration
tests. The Phase 1 scope, product description, and technical architecture use
this ADR for the policy.

The Render/Vercel arrangement in [ADR 0001](0001-staging-hosting.md) is a
proposed staging setup. It must use synthetic data until account ownership,
region, contracts, and production data handling have been reviewed against
this decision.

## Delivery and backlog mapping

- **B03, customer onboarding:** Keep OTP account creation. In the profile and
  terms checklist item, add pending/active state and a server-enforced order
  gate with tests. The current authentication slice does not implement these.
- **B04–B07, cleaner onboarding and administration:** Add live identity capture,
  service-qualification evidence, private storage, a designated reviewer,
  recorded decisions, and job-eligibility checks.
- **B12–B18, orders and service evidence:** Redact pre-acceptance private details;
  restrict original-photo reads to admin/support; delete raw media and temporary
  notes after seven days, with case holds and deletion tests.
- **B25, cleaner profile:** Add satisfaction and rating publication, separate
  homeowner photo consent, anonymized approved copies, and withdrawal.
- **B26–B27, account rights and exit:** Add export/deletion requests, formal
  verified identity corrections, restricted cleaner exit, client-email review,
  and three-month deletion with case holds.
- **B28, operations and money:** Restrict and audit authorized administrative
  actions; implement fines and bonuses only after their financial policy is
  approved. Review production hosting and third-party data flows separately.

## Revisit before affected implementation or launch

- Use the service-specific qualifications and Milano area choice in ADR 0003.
  The accepted live capture instructions are ID front and back, or the
  passport identity page plus any legally required second page, with no gallery
  substitution. Confirm the exact extra passport page and reviewer procedure
  before enabling real identity collection.
- Approve and publish an immutable customer terms document, its version and
  HTTPS URL before configuring activation in staging or production. Confirm how
  all API instances switch to the new version before enabling order creation.
- Validate the seven-day deletion scope, complaint and legal holds, published
  copy lifetime, backup expiry, and statutory transaction/identity retention.
- Decide the approved cloud region, infrastructure owner, SMS and payment
  providers, and contracts for any cross-border transfer.
- Approve a purpose-specific retention and deletion schedule for identity,
  selfie, qualification, and work-authorization evidence, including backups,
  before real worker registration is enabled. Until then, use synthetic data
  in controlled development or staging only.
- Define customer satisfaction, publication consent/withdrawal, anonymization
  review, and whether a customer can see any separate non-evidence preview.
- Define which former clients receive exit emails, the permitted message and
  frequency, response handling, and the case-resolution process.
- Approve the financial policy for fines and bonuses and the access/retention
  schedule for correction evidence before building those workflows.
