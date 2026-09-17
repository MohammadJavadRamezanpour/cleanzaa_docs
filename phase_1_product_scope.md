# Phase 1 Product Scope

**Launch country:** Italy

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

The full customer address and private details become available only after the worker accepts the job. Job acceptance must remain safe when multiple workers respond at once, and the worker must not be assigned overlapping jobs.

## Customer app

The customer can:

1. Sign up or log in.
2. Select **Home Cleaning**, **Car Wash**, or **Pet Care**.
3. Select a date, time, address, and service type.
4. View the price before submitting a request.
5. Submit a service request.
6. View the order status: **Requested**, **Accepted**, **On the Way**, **In Progress**, or **Completed**.
7. View the assigned worker's information.
8. View the Before Photo.
9. View the After Photo.
10. Confirm the completed service.
11. Rate the worker.
12. View order history.

## Main journey

1. The customer chooses one of the three services, supplies the booking details, sees a price, and submits the request.
2. Eligible workers see a job request. One worker accepts it, and the customer sees who was assigned.
3. The worker travels to the job, checks in, and takes a Before Photo.
4. The worker performs the job, takes an After Photo, and marks the job complete.
5. The customer reviews the photos, confirms completion, and can rate the worker. Both users can review the relevant history; the worker can view earnings and rating.

The customer sees the five status labels listed above. Each label must be backed by a defined server-side event or state; the UI alone cannot declare a job accepted, in progress, or completed.

## Details to settle before implementation

- What action or event sets **On the Way**? The worker app list does not yet specify a departure action.
- Does check-in start **In Progress**, or does the Before Photo or another action start it?
- Does **Completed** mean the worker ended the job, the customer confirmed it, or both? What happens if the customer does not confirm?
- What information and photos may each user see at each point in the journey?
- Does **earnings** mean expected earnings, completed-job earnings, paid-out earnings, or separate totals?
- What is the price calculation for each of the three services, and what payment method and timing apply?

The [technical architecture](technical_architecture.md) records the remaining launch decisions. In particular, this scope does not decide currency, payment provider, cash or wallet availability, pricing rules, document retention, complaint eligibility, emergency behavior, or support staffing. Unlisted features are not automatically excluded because this is a minimum scope.
