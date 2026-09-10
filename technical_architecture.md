# Cleanzza Technical Architecture & Engineering Specification

**Document status:** Initial implementation specification\
**Audience:** Code agents, backend/frontend engineers, reviewers\
**Product basis:** Cleanzza Platform Features and Workflows\
**Architecture decision:** Modular monolith

------------------------------------------------------------------------

## 1. Purpose

This document defines the initial technical architecture for Cleanzza so
that code agents can implement the system consistently.

The product supports two primary user types:

-   Customers who request and pay for cleaning services.
-   Cleaners who register, become verified/approved, declare
    availability, receive matched service requests, perform services,
    and receive payment.

The source product specification also includes administrative review,
identity/document verification, matching, scoring, payments, complaints,
referrals, support chat, service evidence photos, and operational rules.

## 1.1 Specification status and unresolved product decisions

This document defines the technical direction, but it must not be used to
invent unresolved business or legal policy. The following decisions must be
recorded before the affected feature is implemented:

- operating country or countries, supported currencies, and applicable
  privacy, employment, marketplace, and payment regulations
- payment provider and its marketplace, payout, stored-value, refund, and
  off-session payment capabilities
- whether Cleanzza ever holds customer funds or whether all balances are
  provider-managed
- the exact meaning of the EUR 10 cash-commission rule
- authorization and customer consent required for any automatic debit
- pricing, duration, overtime, cancellation, no-show, refund, and repeat-
  cleaner surcharge rules
- retention periods and deletion rules for identity documents, selfies,
  service photographs, assessment responses, and financial records
- the supported behavior of the emergency-call feature in each operating
  region
- whether restricting complaints to customers who uploaded before-service
  photographs is legally and contractually valid

Until a decision is made, represent the rule as an explicit open decision in
the relevant module documentation. Do not silently choose a behavior in code.

### Engineering principle

Prefer the simplest architecture that can safely implement the business
rules.

**Do not introduce microservices or distributed infrastructure unless a
concrete requirement requires it.**

------------------------------------------------------------------------

# 2. Architecture Decision

## 2.1 Stack

  -----------------------------------------------------------------------
  Concern                             Decision
  ----------------------------------- -----------------------------------
  Frontend                            Next.js + TypeScript

  Backend                             Python + FastAPI

  Database                            PostgreSQL

  ORM                                 SQLAlchemy

  Validation                          Pydantic

  API style                           REST/JSON

  File storage                        S3-compatible object storage

  Cache/queue                         Redis, only where needed

  Background jobs                     Python worker

  Frontend deployment                 Vercel or equivalent

  Backend deployment                  Docker container on a managed
                                      container platform

  Database deployment                 Managed PostgreSQL

  CI/CD                               GitHub Actions

  Error monitoring                    Sentry or equivalent

  Local development                   Docker Compose
  -----------------------------------------------------------------------

## 2.2 Architecture style

Use a **modular monolith**.

There is one Next.js application and one FastAPI application. The
backend is internally divided into domain modules.

``` text
                         Internet
                            |
                     +------+------+
                     |   Next.js   |
                     | TypeScript  |
                     +------+------+
                            |
                         HTTPS
                            |
                     +------+------+
                     |   FastAPI   |
                     | Python API  |
                     +------+------+
                            |
          +-----------------+-----------------+
          |                 |                 |
          v                 v                 v
    PostgreSQL       Object Storage         Redis
    source of truth  documents/photos       optional
          |
          v
     Background Worker
     optional / async
```

Do not create separate deployable services for matching, payments,
notifications, users, orders, or administration in the initial version.

------------------------------------------------------------------------

# 3. Repository Structure

Use a monorepo.

``` text
cleanzza/
├── apps/
│   ├── web/
│   │   ├── app/
│   │   ├── components/
│   │   ├── lib/
│   │   ├── hooks/
│   │   ├── types/
│   │   └── tests/
│   │
│   └── api/
│       ├── app/
│       │   ├── api/
│       │   ├── core/
│       │   ├── db/
│       │   ├── modules/
│       │   ├── integrations/
│       │   └── workers/
│       └── tests/
│
├── docs/
│   └── adr/
├── infra/
├── docker-compose.yml
├── .env.example
└── README.md
```

Backend modules:

``` text
apps/api/app/modules/
├── auth/
├── users/
├── customers/
├── cleaners/
├── services/
├── orders/
├── matching/
├── payments/
├── wallets/
├── ratings/
├── complaints/
├── referrals/
├── notifications/
├── support/
└── admin/
```

Not every module needs to be implemented in the first milestone.

------------------------------------------------------------------------

# 4. Backend Architecture

Each module should have clear boundaries.

Recommended internal structure:

``` text
modules/orders/
├── router.py
├── schemas.py
├── models.py
├── service.py
├── repository.py
├── policies.py
└── tests/
```

### Responsibilities

**router.py** - HTTP endpoints only. - Authentication/authorization
dependencies. - Request parsing. - Response serialization.

**schemas.py** - Pydantic request/response models.

**models.py** - SQLAlchemy persistence models.

**service.py** - Business operations. - Transaction boundaries. - Domain
rules.

**repository.py** - Database queries. - No business policy decisions.

**policies.py** - Authorization and domain eligibility rules where
useful.

Do not put business logic inside React components or database
repositories.

Each module owns its persistence models and public application-service
interface. One module must not query another module's repository or tables
directly. Cross-module workflows call public services or consume internal
domain events. Keep transaction ownership explicit: the application service or
unit of work starts and commits the transaction; repositories never commit.

Reject cyclic module dependencies in review. Shared code is limited to genuine
technical primitives such as database/session handling, clocks, identifiers,
errors, and observability; business concepts stay in their owning module.

------------------------------------------------------------------------

# 5. Frontend Architecture

Next.js is the presentation layer.

``` text
apps/web/
├── app/
│   ├── (marketing)/
│   ├── auth/
│   ├── customer/
│   ├── cleaner/
│   └── admin/
├── components/
│   ├── ui/
│   ├── forms/
│   ├── orders/
│   ├── payments/
│   └── profile/
├── lib/
│   ├── api/
│   ├── auth/
│   └── validation/
├── hooks/
└── types/
```

The frontend must not implement authoritative business rules.

Example:

``` text
BAD:
React component decides whether cleaner can accept order.

GOOD:
React asks API:
POST /cleaner/offers/{id}/accept

Backend validates:
- offer ownership and expiry
- cleaner eligibility
- availability
- time conflict
- order state
- race condition
```

The UI can perform convenience validation, but the API is authoritative.

------------------------------------------------------------------------

# 6. Core Domain Model

The initial conceptual model is:

``` text
User
 ├── UserRole
 ├── CustomerProfile
 └── CleanerProfile

Customer
 └── Order

Cleaner
 ├── CleanerServices
 ├── CleanerEquipment
 ├── CleanerAvailability
 ├── CleanerScore
 └── Orders

Order
 ├── OrderItems
 ├── Quote / PriceSnapshot
 ├── OrderOffer
 ├── ServiceSession
 ├── Payments
 ├── Rating
 └── Complaints
```

## 6.1 Users

A single user identity model should support different roles.

Suggested fields:

``` text
users
- id
- email
- phone
- username
- password_hash (nullable when passwordless authentication is used)
- authentication_status
- created_at
- updated_at

user_roles
- user_id
- role
```

Roles:

``` text
CUSTOMER
CLEANER
ADMIN
SUPPORT
```

Use a single `users.role` enum only if product policy guarantees that a person
can never hold more than one role. Otherwise use `user_roles`; this also avoids
forcing support and administration permissions into the customer/cleaner
profile model.

Authentication status must be separate from cleaner approval, suspension,
customer payment restrictions, and other domain-specific eligibility. The
exact authentication mechanism is an implementation choice, but authentication
must be separated from customer and cleaner profiles.

------------------------------------------------------------------------

# 7. Customer Domain

Customer registration requires:

-   first and last name
-   date/place of birth
-   current residence
-   home address
-   fiscal/tax code
-   phone or email
-   identity document front/back
-   document expiration
-   selfie
-   bank details
-   acceptance of terms

These requirements come from the product specification.

Recommended separation:

``` text
users
customer_profiles
identity_documents
verification_records
payment_customer_references
payment_method_references
terms_acceptances
```

Do not store document image bytes in PostgreSQL.

Store object-storage keys and metadata.

Do not store raw card data or online-banking credentials. Customer payment
references must be opaque identifiers issued by the selected payment provider.

------------------------------------------------------------------------

# 8. Cleaner Domain

Cleaner registration requires:

-   identity/personal information
-   username/password
-   phone verification
-   email confirmation
-   residence permit front/back
-   permit expiration
-   selfie
-   fiscal/tax code
-   bank details
-   terms acceptance

After approval, the cleaner provides:

-   employment/student/unemployed status
-   supporting documentation when applicable
-   service areas
-   service categories
-   professional certificates where required
-   home-cleaning experience level
-   equipment/supplies
-   psychological-assessment responses

Suggested entities:

``` text
cleaner_profiles
cleaner_verification
cleaner_employment
cleaner_service_areas
cleaner_services
cleaner_certifications
cleaner_equipment
cleaner_assessments
cleaner_scores
cleaner_suspensions
payout_account_references
```

Cleaner payout/bank details should be collected and hosted by the marketplace
payment provider where possible. Cleanzza stores only the minimum provider
account identifiers and status needed for payout eligibility.

------------------------------------------------------------------------

# 9. Order Domain

The order is the central business entity.

An order should contain enough information to describe:

-   customer
-   requested services
-   scheduled start and end timestamps
-   customer/service-location timezone
-   location
-   home/service details
-   number of rooms where applicable
-   pet details where applicable
-   car model where applicable
-   equipment requirements
-   cleaner type preference where applicable
-   assigned cleaner (nullable until acceptance)
-   accepted quote/price snapshot
-   fulfillment state

Use normalized relational data for important business fields.

An accepted order must reference an immutable price snapshot containing the
pricing-rule version, line items, taxes where applicable, discounts or
surcharges, gross amount, platform commission, cleaner amount, currency, and
calculated service duration. Historical orders must not change when the current
price catalogue changes.

The order must use a half-open time range `[scheduled_start, scheduled_end)`.
Store timestamps in UTC and retain the IANA timezone used for customer display
and calendar calculations.

------------------------------------------------------------------------

# 10. Domain State Machines

Do not encode fulfillment, matching, payment, complaints, and ratings as one
linear order status. They progress independently and have different invariants.

## 10.1 Order fulfillment

``` text
DRAFT -> SEARCHING -> ASSIGNED -> ARRIVED -> IN_PROGRESS -> COMPLETED
   |         |            |          |            |
   +---------+------------+----------+------------+----> CANCELLED

SEARCHING -> EXPIRED
```

`ASSIGNED` means one cleaner has transactionally accepted the order. An order
is never `RATED`, `PAID`, or `DISPUTED`; those states belong to their own
entities.

## 10.2 Offers

``` text
PENDING -> ACCEPTED
   |----> REJECTED
   |----> EXPIRED
   |----> WITHDRAWN
```

At most one offer can be accepted for an order. Accepting one offer withdraws
or invalidates the remaining pending offers in the same transaction.

## 10.3 Payments

``` text
CREATED -> REQUIRES_ACTION -> AUTHORIZED -> CAPTURED
   |              |              |
   +--------------+--------------+----> FAILED / CANCELLED / EXPIRED

CAPTURED -> PARTIALLY_REFUNDED -> REFUNDED
   |
   +----------------------------> REFUNDED

CASH_DUE -> CASH_REPORTED -> CASH_COMMISSION_DUE -> SETTLED
                                      |
                                      +----> OVERDUE
```

Provider event names must be mapped to these provider-neutral states. Payouts
to cleaners are separate settlement records; payment capture does not imply
that a cleaner payout has completed.

## 10.4 Complaints and ratings

Complaint state:

``` text
OPEN -> UNDER_REVIEW -> RESOLVED
                    -> REJECTED
```

A rating is a separate record associated with an eligible completed order. It
is optional and must not block any order or payment transition.

Each aggregate must expose named operations rather than arbitrary status
updates. Every operation validates the current state, actor, business
preconditions, and idempotency before recording a history entry.

------------------------------------------------------------------------

# 11. Concurrency and Order Acceptance

This is a critical requirement.

Multiple cleaners may see the same order. Only one cleaner may
successfully accept it.

The acceptance operation must be transactional.

Conceptually:

``` text
BEGIN TRANSACTION

lock order

verify order is still available

verify offer is pending and unexpired

verify cleaner is eligible

verify cleaner has no conflicting booking

assign cleaner

accept winning offer and invalidate remaining offers

change fulfillment state to ASSIGNED

create order history record

COMMIT
```

If another cleaner already accepted the order, the offer expired, or the same
request is repeated with incompatible data, return a machine-readable conflict
response. Retrying an already-successful request with the same idempotency key
must return the original result.

Use PostgreSQL transactional locking/constraints rather than relying on
frontend behavior.

------------------------------------------------------------------------

# 12. Cleaner Time Conflicts

A cleaner cannot accept two orders occupying overlapping time ranges.

The backend must check this transactionally.

Do not rely solely on:

``` text
SELECT ...
then INSERT ...
```

because two concurrent requests can pass the check simultaneously.

Prefer a PostgreSQL exclusion constraint over the cleaner identifier and a
`tstzrange(scheduled_start, scheduled_end, '[)')` for active assignments. If an
exclusion constraint is not used, document and test the exact locking strategy.
The constraint must ignore cancelled or expired assignments.

Service duration must be produced by the accepted price quote before matching
begins. Buffer/travel time is a product decision and, when enabled, must be
included consistently in both matching and the database constraint.

------------------------------------------------------------------------

# 13. Matching

Matching starts as deterministic business logic.

Do not introduce machine learning infrastructure initially.

Eligibility:

``` text
1. Cleaner is approved.
2. Cleaner is not suspended.
3. Cleaner provides requested service.
4. Cleaner covers requested service area.
5. Cleaner satisfies required qualification.
6. Cleaner satisfies cleaner-type requirement.
7. Cleaner has required equipment when necessary.
8. Cleaner is available.
9. Cleaner has no conflicting order.
```

Ranking can then consider:

``` text
cleaner score
service quality
reliability
availability
distance/proximity
equipment match
other product-defined priority rules
```

Keep matching behind an interface:

``` python
class Matcher:
    def find_candidates(self, order): ...
    def rank(self, order, candidates): ...
```

This allows future improvements without changing the order domain.

Matching results must be materialized as offers rather than by changing the
order to a global `OFFERED` state:

``` text
order_offers
- id
- order_id
- cleaner_id
- round_number
- rank
- status
- offered_at
- expires_at
- responded_at
```

Add a unique constraint on `(order_id, round_number, cleaner_id)` and a partial
unique index permitting at most one accepted offer per order. A cleaner may be
offered an order again in a later round if product policy allows it. Rejecting
an offer does not cancel the underlying order.

Before acceptance, expose only the information needed to evaluate the job.
Customer identity, exact address, phone number, photographs, and other private
details must remain hidden unless a documented operational rule requires them.

------------------------------------------------------------------------

# 14. Cleaner Availability

Cleaner availability has two different concepts:

### General availability

The cleaner indicates:

``` text
I am available for work
```

### Service-specific availability

The cleaner must not be offered a job that conflicts with an existing
accepted service.

When the cleaner activates availability, the system also requires a
selfie according to the product workflow.

Availability should therefore be modeled as business state, not merely a
boolean on the frontend.

------------------------------------------------------------------------

# 15. Arrival and Service Tracking

When the cleaner arrives:

1.  Cleaner selects `I have arrived`.
2.  Cleaner takes a selfie.
3.  Backend records arrival.
4.  Service timing begins.

Suggested entity:

``` text
service_sessions
- id
- order_id
- cleaner_id
- arrived_at
- started_at
- completed_at
- arrival_selfie_object_key
- completion_notes
```

The backend timestamp is authoritative.

Do not trust a client-provided `started_at`.

------------------------------------------------------------------------

# 16. Service Completion

When work is complete:

1.  Cleaner selects Complete.
2.  Cleaner uploads photographs of cleaned areas.
3.  Cleaner specifies settlement method.
4.  Order moves into payment settlement.

Completion photographs should be stored in object storage.

Suggested:

``` text
service_evidence
- id
- order_id
- file_id
- evidence_type
- uploaded_by
- created_at
```

Service evidence types:

``` text
BEFORE_SERVICE
AFTER_SERVICE
ARRIVAL_SELFIE
COMPLETION
```

Identity documents and professional certificates are not service evidence and
must not require an `order_id`. Store file metadata in a common `stored_files`
table and link it from purpose-specific entities such as `identity_documents`,
`cleaner_certifications`, and `service_evidence`.

------------------------------------------------------------------------

# 17. Peace of Mind / Complaint Evidence

Before service, a customer can activate the Peace of Mind feature.

If activated:

``` text
customer uploads before-service photographs
```

Subject to the legal/product decision in Section 1.1, those photographs may
create eligibility for a specific platform guarantee or evidence-backed
complaint workflow.

The backend---not the frontend---must enforce:

``` text
evidence-backed complaint can be created
IF
before-service evidence exists
AND
complaint is within allowed period
```

Do not allow clients to bypass an approved eligibility rule by calling the API
directly. Conversely, do not use this rule to suppress statutory rights,
safety reports, payment disputes, theft/damage reports, or access to support.
Those may require separate case types and deadlines.

------------------------------------------------------------------------

# 18. Complaint Domain

Suggested:

``` text
complaints
- id
- order_id
- customer_id
- type
- description
- status
- created_at
- resolved_at
```

Possible statuses:

``` text
OPEN
UNDER_REVIEW
RESOLVED
REJECTED
```

Complaints should be visible to admin/support users.

The product specification states that complaints may be submitted within 24
hours after service and that Peace of Mind photographs are required. Apply that
deadline only to the approved product complaint type; it must not silently
close other support, safety, payment, or legally required reporting paths.

------------------------------------------------------------------------

# 19. Payments

Treat payments as an integration boundary.

Core domain should use provider-neutral concepts:

``` text
Payment
PaymentTransaction
PaymentMethod
PaymentStatus
Commission
CleanerPayout
Refund
```

Use an external payment provider for online payment processing.

Do not build payment processing internally.

The provider must support the operating jurisdiction and required marketplace
flow. Cleanzza must not store raw card details or online-banking credentials.
Store provider customer, payment-method, connected-account, payment, refund,
and payout identifiers.

Before implementation, document:

- when payment is authorized and captured
- whether an off-session charge is permitted and how customer consent is
  recorded
- who is merchant of record
- how cleaner identity/KYC and payout onboarding are handled
- refunds, partial refunds, chargebacks, payout reversals, and negative balances
- webhook ordering, duplicate delivery, and reconciliation behavior

All client-initiated payment creation, capture, refund, top-up, and settlement
operations require idempotency keys. Webhooks remain authoritative for
provider-side completion.

------------------------------------------------------------------------

# 20. Cash Payment

The product has special cash-settlement rules.

When a customer selects cash:

``` text
service completed
      |
      v
cash payment due
      |
      v
commission due
      |
      +---- settled ----> account usable
      |
      +---- not settled -> account restricted
```

The product specification says that if the platform commission is not settled,
the customer cannot continue using the system or select Start work. It is not
currently clear whether the referenced EUR 10 value is a maximum commission,
a debt threshold, or a payment limit. This is a blocking product decision; do
not encode an interpretation until it is resolved.

If payment settlement remains incomplete for 48 hours, the account is
blocked and receives a negative payment record.

These rules must be implemented in the backend domain layer after the policy is
resolved. Model restrictions as explicit records with reason, source debt,
effective time, expiry where applicable, and the actor or job that created or
removed the restriction.

------------------------------------------------------------------------

# 21. Wallet / Customer Balance

Customers may add funds and use their balance for payments only if the selected
provider and operating jurisdiction permit the intended stored-value model.

Do not represent a wallet merely as:

``` text
customer.balance += 10
```

Use a ledger.

``` text
wallets
- id
- customer_id

wallet_transactions
- id
- wallet_id
- type
- amount
- currency
- reference_type
- reference_id
- idempotency_key
- effective_at
- reversal_of_transaction_id (nullable)
- created_at
```

Balance should be derived from ledger transactions or maintained with
transactional consistency.

Every money movement needs an auditable record.

Prefer provider-managed customer balances where possible. Do not create an
internally custodied wallet merely because the product calls the feature a
"balance." A top-up is not spendable until the provider confirms it. Never
delete or mutate posted ledger entries; correct them with compensating entries.

------------------------------------------------------------------------

# 22. Commission

Commission should be represented explicitly.

Do not calculate it only in the frontend.

Example:

``` text
order
├── gross_amount
├── platform_commission
├── cleaner_amount
└── currency
```

For every payment:

``` text
gross
  - commission
  = cleaner settlement
```

All monetary values should use decimal-safe numeric types, never
floating-point arithmetic.

Commission and cleaner settlement amounts are copied into the accepted price
snapshot. Recalculating historical commission from the current pricing rules is
forbidden. Define rounding per currency and require that line items, discounts,
tax, gross amount, commission, and cleaner amount reconcile exactly.

------------------------------------------------------------------------

# 23. Ratings

After order completion:

``` text
customer -> cleaner rating
customer -> feedback
```

Suggested:

``` text
ratings
- id
- order_id
- customer_id
- cleaner_id
- score
- feedback
- created_at
```

Enforce one rating per eligible order unless a future business rule
explicitly allows updates.

------------------------------------------------------------------------

# 24. Cleaner Score

Cleaner score is a domain concept.

Do not hard-code score manipulation inside controllers.

Create:

``` text
ScoreService
```

Responsibilities:

``` text
record_positive_event(...)
record_negative_event(...)
calculate_score(...)
apply_suspension(...)
```

The product specification says cleaner score affects access/timing for
future service requests and that certain violations result in negative
scoring or suspension.

Keep scoring rules configurable in code so they can evolve.

------------------------------------------------------------------------

# 25. Referrals

Two referral concepts exist:

### Customer introduces an existing cleaner

The customer receives commission for services completed by the referred
cleaner.

### Customer refers another customer

The referring customer receives rewards based on referred customer
orders/payments.

Keep these separate:

``` text
cleaner_referrals
customer_referrals
rewards
```

Do not create one ambiguous `referrals` table with unrelated semantics
unless its model is explicitly polymorphic and well constrained.

------------------------------------------------------------------------

# 26. Authentication

Authentication requirements differ by actor.

Cleaner:

``` text
username + password
```

Customer:

``` text
mobile number + verification code
```

The exact provider/library can be selected during implementation.

Regardless of mechanism:

-   Passwords must never be stored plaintext.
-   OTPs must be stored hashed, expire, and be single-use.
-   OTP attempts must be rate-limited.
-   Authentication endpoints must be rate-limited.
-   Authorization must be checked server-side.
-   Email addresses and phone numbers must be normalized and uniquely
    constrained according to the chosen identity policy.
-   Sessions and refresh tokens must support rotation, revocation, and device-
    level logout.
-   Account recovery and contact-detail changes must require re-verification.

------------------------------------------------------------------------

# 27. Authorization

Every protected API operation must establish:

``` text
authenticated user
+
required role
+
resource ownership/permission
```

Example:

``` text
Customer A cannot retrieve Customer B's order.
Cleaner A cannot modify Cleaner B's profile.
Cleaner cannot access another cleaner's verification documents.
Customer cannot access admin endpoints.
```

Use dependency/policy functions such as:

``` python
require_authenticated_user()
require_role(...)
require_order_owner(...)
require_order_cleaner(...)
```

------------------------------------------------------------------------

# 28. Files and Object Storage

Use S3-compatible object storage.

Never store large images directly in PostgreSQL.

Upload flow:

``` text
Browser
   |
   | request upload URL
   v
FastAPI
   |
   | presigned URL
   v
Browser
   |
   | upload
   v
Object Storage
```

Then save metadata:

``` text
object_key
content_type
size
owner_id
purpose
created_at
```

Use private buckets.

Files containing identity, financial, or private service information
must not be publicly accessible.

The upload authorization must constrain object key, maximum size, allowed media
type, checksum, owner, and purpose. A successful browser upload does not make a
file trusted. The backend must finalize the upload by verifying the object
exists, validating its actual type and size, recording its checksum, and
queuing malware/content-safety checks where required. Unfinalized uploads must
expire and be removed.

Use short-lived signed download URLs or authenticated streaming endpoints.
Record access to identity, financial, and in-home service files. Configure
encryption, retention, deletion, backup behavior, and environment-specific
buckets. Test that identifiers cannot be changed to access another user's file.

------------------------------------------------------------------------

# 29. Notifications

Notifications can initially be handled inside the monolith.

Supported channels may include:

``` text
SMS
Email
In-app
Push (future)
```

Create a notification abstraction:

``` python
notification_service.send(...)
```

Provider-specific code belongs in:

``` text
integrations/
```

not in order/business modules.

------------------------------------------------------------------------

# 30. Background Jobs

Use background processing for:

-   sending notifications
-   OTP delivery
-   email delivery
-   payment reconciliation
-   payment deadline checks
-   expiring offers
-   cleanup tasks
-   future verification workflows

Initial implementation can use a simple Python worker + Redis.

Do not introduce a queue system if synchronous processing is sufficient
for an operation.

The worker is optional only while no required asynchronous workflow exists. It
becomes a required production component as soon as OTP/notification delivery,
offer expiry, reconciliation, or deadline enforcement is asynchronous.

Any database transaction that must cause asynchronous work must also insert an
outbox record in the same PostgreSQL transaction. A dispatcher publishes
unprocessed outbox records to the worker and marks them delivered. Job handlers
must be idempotent and define retry limits, exponential backoff, terminal-
failure handling, and operational alerts.

Deadline and reconciliation jobs must use durable database state, not Redis
timers as the source of truth. Re-running a job after a crash must be safe.

------------------------------------------------------------------------

# 31. Real-Time Order Feed

The cleaner order list continuously updates.

Initial implementation:

``` text
GET /cleaner/offers
```

with polling.

Do not implement WebSockets in the first iteration unless the product
demonstrates a real need.

Future option:

``` text
WebSocket /orders
```

or Server-Sent Events.

The database remains authoritative.

------------------------------------------------------------------------

# 32. Support Chat

The product requires always-available online support chat.

For the first version, model conversations:

``` text
support_conversations
support_messages
```

Example:

``` text
support_conversations
- id
- customer_id / cleaner_id
- assigned_agent_id
- status
- created_at

support_messages
- id
- conversation_id
- sender_id
- body
- created_at
```

Real-time transport can be added later.

"Always available" is an operational service-level requirement, not something
the data model guarantees. Define staffing, response-time expectations,
escalation, retention, attachment handling, abuse controls, and behavior when no
agent is online before promising 24-hour live support.

------------------------------------------------------------------------

# 33. Admin / Back Office

Admin should be part of the same Next.js application.

Routes:

``` text
/admin
/admin/cleaners
/admin/customers
/admin/orders
/admin/payments
/admin/complaints
/admin/verifications
/admin/support
```

Admin functions should include, as required by product workflows:

-   cleaner approval/review
-   document review
-   complaint review
-   account restrictions
-   suspension management
-   payment issue review
-   support

Do not create a separate admin backend initially.

A minimal verification and support console is required in the same delivery
slice as cleaner verification. Do not postpone operational tooling while making
manual approval a prerequisite for marketplace supply.

------------------------------------------------------------------------

# 34. API Design

Use REST.

Example endpoints:

``` text
POST   /auth/customer/request-otp
POST   /auth/customer/verify-otp
POST   /auth/cleaner/register
POST   /auth/cleaner/login
POST   /auth/logout

GET    /me
PATCH  /me

POST   /customers/profile
GET    /customers/profile
PATCH  /customers/profile

GET    /cleaners/profile
PATCH  /cleaners/profile
POST   /cleaners/availability
POST   /cleaners/availability/selfie

POST   /orders
GET    /orders
GET    /orders/{order_id}
POST   /orders/{order_id}/quote
POST   /orders/{order_id}/start-search
POST   /orders/{order_id}/arrive
POST   /orders/{order_id}/start
POST   /orders/{order_id}/complete
POST   /orders/{order_id}/cancel

GET    /cleaner/offers
POST   /cleaner/offers/{offer_id}/accept
POST   /cleaner/offers/{offer_id}/reject

POST   /orders/{order_id}/peace-of-mind
POST   /orders/{order_id}/evidence

POST   /orders/{order_id}/payments
POST   /payments/{payment_id}/confirm
POST   /orders/{order_id}/rating
POST   /orders/{order_id}/complaint

GET    /cleaner/orders/history

GET    /wallet
GET    /wallet/transactions
POST   /wallet/top-up

GET    /support/conversations
POST   /support/conversations
POST   /support/conversations/{id}/messages
```

Wallet endpoints are conditional and must not be implemented until the wallet
gate in Sections 1.1 and 21 is resolved.

Exact endpoint naming may evolve, but maintain consistent
resource-oriented REST semantics.

Publish an OpenAPI contract and generate or validate the TypeScript API client
from it. Choose and document an API versioning strategy before the first public
release.

------------------------------------------------------------------------

# 35. API Rules

All API responses should use predictable structures.

Errors should be machine-readable.

Example:

``` json
{
  "error": {
    "code": "ORDER_ALREADY_ACCEPTED",
    "message": "This order is no longer available."
  }
}
```

Do not expose internal exception messages to clients.

Use appropriate HTTP statuses:

``` text
400 invalid request
401 unauthenticated
403 unauthorized
404 not found
409 state/concurrency conflict
422 validation failure
429 rate limited
500 unexpected server error
```

Require an `Idempotency-Key` for retryable commands that create financial
effects or perform one-time transitions, including offer acceptance, order
creation, payment creation/confirmation, refunds, top-ups, cash settlement, and
file finalization. Persist the key with the authenticated actor, operation,
request fingerprint, response, and expiration. Reuse with different request
data returns `409`.

List endpoints must define cursor pagination, stable ordering, filtering, and a
maximum page size. Dates in JSON use ISO 8601 with an explicit offset; money is
serialized as decimal strings plus currency, never JSON floating point.

------------------------------------------------------------------------

# 36. Database Rules

PostgreSQL is the source of truth.

Requirements:

-   UUID primary keys are recommended.
-   Foreign keys should be explicit.
-   Important business invariants should be enforced at the database
    level where practical.
-   Money uses NUMERIC/Decimal.
-   Timestamps are stored consistently in UTC.
-   All important state changes are auditable.

Avoid JSON blobs for core relational business data.

JSON can be used for flexible metadata where justified.

------------------------------------------------------------------------

# 37. Auditability

Important operations should produce audit/history records.

At minimum:

``` text
order_fulfillment_history
order_offer_history
payment_transaction_history
cleaner_payout_history
account_restriction_history
cleaner_score_events
admin_actions
file_access_events
```

For example:

``` text
order_fulfillment_history
- id
- order_id
- from_status
- to_status
- actor_id
- reason
- created_at
```

This is especially important for payment, complaint, suspension, and
account-blocking workflows.

------------------------------------------------------------------------

# 38. Security Requirements

Minimum requirements:

-   HTTPS everywhere outside local development.
-   Secure HTTP-only cookies if cookie-based sessions are used.
-   CSRF protection where applicable.
-   Strong password hashing.
-   OTP expiration and rate limiting.
-   API rate limiting.
-   Private object storage.
-   Server-side authorization.
-   Input validation.
-   SQL injection protection through ORM/parameterized queries.
-   Never log passwords, OTPs, payment secrets, or identity-document
    contents.
-   Never expose private object-storage URLs permanently.
-   Secrets only through environment/secret management.
-   Data is classified by sensitivity and access is least-privilege.
-   Production data is never copied into development environments.
-   Encryption at rest is enabled for the database, backups, and object
    storage.
-   Security-sensitive admin actions require stronger authentication and are
    audited.
-   Dependency, container, and secret scanning run in CI.

Sensitive document access should be auditable.

Define privacy and data-governance requirements before collecting production
identity data: purpose and consent/legal basis, retention, deletion and legal
holds, subject-access/export, breach response, subprocessors, and geographic
residency. Psychological-assessment responses require an explicit necessity and
access review; collect the minimum data required.

The emergency-call feature must not promise guaranteed police contact from a
web application. Its implementation and wording must be approved per operating
region, degrade safely when location or telephony is unavailable, and clearly
direct users to local emergency services.

------------------------------------------------------------------------

# 39. Infrastructure

Initial production topology:

``` text
                 Cloudflare / DNS
                       |
          +------------+------------+
          |                         |
          v                         v
       Vercel                  Backend host
       Next.js                   FastAPI
                                     |
                      +--------------+--------------+
                      |              |              |
                      v              v              v
                  PostgreSQL    Object Storage     Redis
                                  (private)       optional
```

Backend should run in Docker.

No Kubernetes.

No service mesh.

No self-hosted database initially.

No self-hosted object storage.

Production infrastructure must include automated encrypted database backups,
point-in-time recovery, object-versioning or equivalent recovery appropriate to
retention policy, and restore testing. Define recovery point and recovery time
objectives before launch. Separate development, staging, and production
accounts/resources and restrict production access.

------------------------------------------------------------------------

# 40. Local Development

`docker-compose.yml` should provide:

``` text
postgres
redis
```

The applications can run locally:

``` text
Next.js -> localhost
FastAPI -> localhost
Postgres -> Docker
Redis -> Docker
```

Optional full-container development can be supported, but it should not
be required.

------------------------------------------------------------------------

# 41. Configuration

Use environment variables.

Example:

``` text
DATABASE_URL=
REDIS_URL=

S3_ENDPOINT=
S3_BUCKET=
S3_ACCESS_KEY=
S3_SECRET_KEY=

AUTH_SECRET=

PAYMENT_PROVIDER_SECRET=
PAYMENT_WEBHOOK_SECRET=

SMS_PROVIDER_KEY=
EMAIL_PROVIDER_KEY=

SENTRY_DSN=
```

Provide:

``` text
.env.example
```

Never commit secrets.

------------------------------------------------------------------------

# 42. Webhooks

External payment and verification providers may call the API.

Webhook requirements:

1.  Verify signature.
2.  Parse event.
3.  Check idempotency.
4.  Update domain state transactionally.
5.  Record event.
6.  Return success.

Suggested:

``` text
webhook_events
- id
- provider
- external_event_id
- event_type
- payload_hash
- received_at
- processing_attempts
- last_error_code
- processed_at
- status
```

Enforce a unique constraint on `(provider, external_event_id)`. Verify signatures
against the original request bytes before parsing. External webhook processing
must be idempotent, tolerate out-of-order events, and acknowledge only after the
event is durably recorded. Unexpected transitions go to reconciliation rather
than forcing local state backward.

------------------------------------------------------------------------

# 43. Testing Strategy

Use three levels.

## Unit tests

Business rules:

``` text
matching
scoring
order transitions
payment calculations
commission calculations
eligibility
complaint eligibility
pricing snapshots and rounding
idempotency behavior
```

## Integration tests

API + PostgreSQL:

``` text
registration
authentication
order lifecycle
concurrent acceptance
overlapping schedule acceptance
payment state
permissions
cross-user file access denial
outbox publication and retry
out-of-order and duplicate webhooks
```

## End-to-end tests

Critical user journeys:

``` text
customer registration -> request service
cleaner registration -> approval -> availability
customer request -> cleaner acceptance
arrival -> completion
payment -> rating
complaint workflow
```

Do not aim for 100% coverage.

Prioritize business-critical rules.

------------------------------------------------------------------------

# 44. Observability

Every production request should have a correlation/request ID.

Log structured events.

Good:

``` json
{
  "event": "order.accepted",
  "order_id": "...",
  "cleaner_id": "...",
  "request_id": "..."
}
```

Bad:

``` text
something went wrong
```

Monitor:

``` text
API errors
latency
database errors
failed payments
failed webhooks
background-job failures
authentication failures
```

Use Sentry or equivalent for application exceptions.

------------------------------------------------------------------------

# 45. Deployment Pipeline

GitHub Actions:

``` text
Pull Request
    |
    +--> lint
    +--> typecheck
    +--> unit tests
    +--> integration tests
    |
    v
merge
    |
    +--> build
    +--> deploy staging
    |
    v
production approval/deploy
```

Database migrations must run through a controlled migration system.

Use Alembic for SQLAlchemy migrations.

Never manually modify production schema without a migration.

Production migrations must be backward-compatible with the currently deployed
application during rollout. Every release needs a rollback or forward-fix plan;
destructive schema cleanup occurs only after old application versions no longer
depend on it.

------------------------------------------------------------------------

# 46. MVP Implementation Order

Implement vertical slices in this order. Each phase must satisfy the relevant
definition of done and leave a deployable, observable system.

## Phase 0 --- Resolve launch-critical decisions

-   operating jurisdiction, currency, and privacy/retention policy
-   payment provider and merchant/marketplace funds flow
-   exact cash, EUR 10, automatic-debit, refund, and payout rules
-   service duration, price, surcharge, overtime, cancellation, and no-show
    rules
-   complaint eligibility and emergency-feature review

Produce short ADRs for these decisions. Financial, identity, and emergency
features remain gated until their applicable decision is approved.

## Phase 1 --- Foundation and security baseline

-   monorepo, Next.js, FastAPI, PostgreSQL, SQLAlchemy, and Alembic
-   Docker Compose, CI, staging deployment, and controlled migrations
-   structured logging, request IDs, error monitoring, and health checks
-   secrets management, dependency scanning, backups, and restore procedure
-   base authorization, audit-event, idempotency, and transactional-outbox
    infrastructure

## Phase 2 --- Identity, onboarding, and approval slice

-   customer and cleaner authentication
-   SMS/email delivery required by authentication
-   user roles, sessions, recovery, and contact verification
-   customer and cleaner profiles
-   object storage, secure upload finalization, and document metadata
-   terms versioning and acceptance
-   cleaner verification/approval workflow
-   minimal admin console required to review and approve cleaners

## Phase 3 --- Catalogue, quote, and order slice

-   service categories and service-specific inputs
-   versioned pricing and duration calculation
-   immutable quote/price snapshot
-   customer service request and order creation
-   fulfillment state machine and history
-   cancellation and expiry rules included in the approved MVP scope

No order enters matching without an accepted quote, currency, scheduled UTC
time range, and display timezone.

## Phase 4 --- Marketplace assignment slice

-   cleaner services, qualifications, equipment, areas, and availability
-   deterministic eligibility and ranking
-   privacy-safe cleaner offer feed with polling
-   offer expiration and rejection
-   transactionally safe acceptance
-   database-enforced overlapping-schedule prevention
-   matching, offer, concurrency, and authorization tests

## Phase 5 --- Service execution and evidence slice

-   arrival action and selfie
-   customer identity-match confirmation where approved
-   authoritative service session timing
-   start, completion, and completion photographs
-   Peace of Mind before-service evidence where approved
-   secure evidence access, retention, and audit behavior
-   minimal complaint intake and admin review if the feature is advertised

## Phase 6 --- One complete payment and payout slice

-   one provider-backed online payment flow
-   authorization/capture timing and customer consent
-   commission snapshot, refund, and reconciliation
-   cleaner connected-account onboarding and payout tracking
-   signed, duplicate, and out-of-order webhook handling
-   payment support/admin tooling and failure recovery

Do not launch an internally custodied wallet, cash collection, automatic debit,
or additional payment methods in this phase unless their Phase 0 decisions and
provider capabilities are complete.

## Phase 7 --- Trust, quality, and operations

-   ratings and feedback
-   cleaner score events and configurable calculation
-   suspension and account-restriction workflows
-   complaint resolution and evidence review
-   notification preferences and operational alerts
-   support conversations plus a documented staffing/escalation model
-   admin dashboards for orders, payments, complaints, and audit history

## Phase 8 --- Approved commercial extensions

-   cash payment and commission-debt workflow
-   provider-supported customer balance or legally approved wallet ledger
-   customer and cleaner referrals
-   rewards and repeat-cleaner pricing
-   push or real-time transports only if polling metrics demonstrate a need

------------------------------------------------------------------------

# 47. Explicit Non-Goals for V1

Do not implement unless required by a concrete product requirement:

-   microservices
-   Kubernetes
-   Kafka
-   GraphQL
-   event sourcing
-   CQRS
-   Elasticsearch
-   custom payment processing
-   custom identity/KYC verification
-   ML-based matching
-   WebSocket infrastructure
-   separate admin application
-   separate matching service
-   separate notification service

The default assumption is that all of these are unnecessary.

------------------------------------------------------------------------

# 48. Code-Agent Rules

Code agents working on this repository must follow these rules.

### Rule 1 --- Preserve architecture

Do not introduce a new framework or infrastructure component without
documenting why it is necessary.

### Rule 2 --- Backend owns business rules

Any rule involving money, authorization, order state, eligibility,
availability, scoring, complaints, or account restrictions must be
enforced in the backend.

### Rule 3 --- Database is authoritative

Do not use frontend state, Redis, or cached values as the authoritative
source for business state.

### Rule 4 --- Transactions matter

Any operation involving multiple related state changes must use an
appropriate database transaction.

### Rule 5 --- Design for concurrency

Order acceptance, payment processing, wallet updates, and scheduling
must be safe under concurrent requests.

### Rule 6 --- Keep integrations isolated

Third-party providers belong under:

``` text
app/integrations/
```

Business modules should depend on provider-neutral interfaces.

### Rule 7 --- No speculative abstraction

Do not create abstraction layers without a current use case.

### Rule 8 --- Prefer boring technology

A simpler implementation is preferred when it satisfies the
requirements.

### Rule 9 --- Every feature needs tests

Business-critical behavior must have automated tests.

### Rule 10 --- Update documentation

When an architectural decision changes, update this document and the
relevant module documentation.

### Rule 11 --- Do not invent product policy

If a rule listed in Section 1.1 remains unresolved, stop implementation of the
affected behavior and record the decision needed. Do not convert ambiguous
prose into financial, legal, safety, or account-blocking behavior.

### Rule 12 --- Preserve module ownership

Modules interact through public application-service interfaces or internal
events. Do not reach into another module's repository, mutate its models, or
create cyclic dependencies.

### Rule 13 --- Minimize sensitive data

Collect, expose, retain, and log only the sensitive data required for an
approved workflow. Use provider-hosted identity and payment collection where
possible.

------------------------------------------------------------------------

# 49. Definition of Done for Backend Features

A backend feature is complete when:

-   API endpoint exists.
-   Request/response schemas exist.
-   Authorization is enforced.
-   Business rules are in the service/domain layer.
-   Database changes have a migration.
-   Errors are explicit.
-   Important state changes are auditable.
-   Retryable commands define and test idempotency behavior.
-   Concurrency-sensitive invariants are database-enforced where practical.
-   Required asynchronous effects use the transactional outbox and idempotent
    handlers.
-   Sensitive-data access, retention, and authorization are covered.
-   Unit tests cover core rules.
-   Integration tests cover the API behavior.
-   Operational/admin handling exists when the feature can require manual
    intervention.
-   No secrets are committed.
-   Documentation is updated where behavior is non-obvious.

------------------------------------------------------------------------

# 50. Definition of Done for Frontend Features

A frontend feature is complete when:

-   Loading state exists.
-   Empty state exists.
-   Error state exists.
-   API errors are displayed appropriately.
-   Forms have client-side validation.
-   Server-side validation remains authoritative.
-   Unauthorized states are handled.
-   Mobile/responsive behavior is considered.
-   Accessibility basics are respected.
-   Tests exist for important interactions.

------------------------------------------------------------------------

# 51. Final Architecture

The intended initial system is deliberately small:

``` text
                         +----------------+
                         |    Next.js     |
                         |   TypeScript   |
                         +-------+--------+
                                 |
                              REST API
                                 |
                         +-------v--------+
                         |    FastAPI     |
                         |  Modular App   |
                         +-------+--------+
                                 |
              +------------------+------------------+
              |                  |                  |
       +------v------+    +------v------+    +------v------+
       | PostgreSQL  |    | Object      |    | Redis       |
       |             |    | Storage     |    | optional    |
       +-------------+    +-------------+    +-------------+

                         +----------------+
                         | Python Worker  |
                         | when async     |
                         +----------------+
```

The key architectural principle is:

> **One frontend, one backend, one database, with external services only
> where they solve a problem we should not build ourselves.**

This architecture is sufficient to implement the product requirements
while keeping the system understandable to both humans and code agents.
