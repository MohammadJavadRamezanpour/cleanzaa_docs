# Phase 1 Product Scope

**Launch country and area:** Italy; Comune di Milano only (ADR 0003)

**Pricing direction:** EUR hourly gross prices and booking policies in [ADR 0004](adr/0004-phase-1-pricing-direction.md); accepted real quotes remain gated by tax, legal and provider validation.

**Payment direction:** One card charge at booking through Stripe Connect, with Cleanzza intended as business of record and a nominal 20%/80% platform/worker split. [ADR 0006](adr/0006-phase-1-payment-direction.md) sets the product rule; Italian tax/legal, provider approval and invoicing remain gates before real acceptance or payment.

**Status:** Confirmed feature scope; operating and payment rules linked below remain open.

This is the customer- and worker-facing scope for the first product release. “Worker” here refers to the **cleaner** role used in the [technical architecture](technical_architecture.md). The architecture's numbered implementation phases describe build order; they are not product release numbers.

The Next.js frontend and FastAPI backend are developed in separate Git
repositories and deployed independently.

## Worker app

The worker can:

1. Sign up or register.
2. Create a profile.
3. Select the services they provide.
4. Add work experience.
5. Upload certificates or evidence of skills.
6. View available job requests.
7. Accept a job.
8. View the accepted job's address and details.
9. Check in at the job.
10. Take a Before Photo.
11. Perform the job.
12. Take an After Photo.
13. Complete or end the job.
14. View earnings.
15. View their rating.
16. Show approved completed-work highlights on their profile, with home photos
    only when the homeowner has separately consented to publication.

The full customer address and private details become available only after the worker accepts the job. Job acceptance must remain safe when multiple workers respond at once, and the worker must not be assigned overlapping jobs.
Before receiving or accepting jobs, the worker completes live identity capture
and service-qualification review and is manually approved by an authorized
administrator. Service photos can be captured and submitted but their stored
originals are visible only to authorized administrators and case-scoped support.
Worker registration and sign-in use a single-use phone code and revocable
session. The worker remains pending until required profile data, private
identity evidence, qualifications, and published worker terms acceptance are
reviewed and approved. [ADR 0003](adr/0003-phase-1-launch-services.md) sets the
Milano service-area choice and service-specific qualifications. Production
collection of real identity evidence remains subject to ADR 0002's legal,
retention, and provider gates.

## Customer app

The customer can:

1. Sign up or log in.
2. Select **Home Cleaning**, **Car Wash**, or **Pet Care**.
3. Select a date, time, address, and service type.
4. View the price before submitting a request.
5. Submit a service request.
6. View the order status: **Requested**, **Accepted**, **On the Way**, **In Progress**, or **Completed**.
7. View the assigned worker's information.
8. Confirm the completed service.
9. Rate the worker and record satisfaction feedback.
10. Decide separately whether approved home photos may appear on the worker's
    profile; withdrawing consent removes the published copy.
11. View order history.

Phone verification creates an account. Before booking, the customer provides
a name and service address and accepts the current terms. Until then the account
is pending activation and cannot place an order. See
[ADR 0002](adr/0002-phase-1-personal-data.md) for the Phase 1 data policy.
The launch catalogue permits only Milano addresses and requires whole-hour
duration (1–8 hours) and the service-specific fields in
[ADR 0003](adr/0003-phase-1-launch-services.md). Unsupported cities cannot be
quoted or booked.

## Main journey

1. The customer chooses one of the three services, supplies the booking details, sees a price, and submits the request.
2. Eligible workers see a job request. One worker accepts it, and the customer sees who was assigned.
3. The worker travels to the job, checks in, and takes a Before Photo.
4. The worker performs the job, takes an After Photo, and marks the job complete.
5. The customer confirms completion and can rate the worker without access to raw service photos. Both users can review the permitted history; the worker can view earnings and rating. An approved, anonymized photo may appear on the worker's profile only with the homeowner's separate consent.

The customer sees the five status labels listed above. Each label must be backed by a defined server-side event or state; the UI alone cannot declare a job accepted, in progress, or completed.

## Details to settle before implementation

- What action or event sets **On the Way**? The worker app list does not yet specify a departure action.
- Does check-in start **In Progress**, or does the Before Photo or another action start it?
- Does **Completed** mean the worker ended the job, the customer confirmed it, or both? What happens if the customer does not confirm?
- Which job details may each user see at each point in the journey, and what
  separate preview, if any, can the customer see without exposing raw evidence?
- Which Italian tax and invoice treatment applies to each service and the platform share, and does the intended business-of-record model pass legal/provider review? ADR 0006 fixes the product payment and earnings direction but leaves these launch gates open.

The [technical architecture](technical_architecture.md) records the remaining launch decisions. ADR 0004 selects EUR and Phase 1 customer-facing prices; ADR 0006 selects the payment flow, nominal split, and earnings states. Tax/invoicing, provider/legal validation, document retention, complaint eligibility, emergency behavior, and support staffing remain open. Unlisted features are not automatically excluded because this is a minimum scope.
