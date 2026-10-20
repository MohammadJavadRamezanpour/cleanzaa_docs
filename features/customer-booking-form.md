# Customer booking form (B10)

## Implemented form

The mobile-width `/book` page loads the typed `GET /catalog/launch` response and presents shared booking inputs: one or more of the three launch services, service date, start time, fixed Milano area, street, house number, CAP, and recipient type. It provides loading, failure/retry, and ready states, with explicit labels for keyboard and assistive technology. The page is linked from `/services`; its 440-pixel frame also applies on desktop. Dates and times are entered for the catalog's `Europe/Rome` timezone. The city and area are controlled values from the server catalog, not arbitrary customer text.

Each selected service gets 1–8 whole-hour duration and explicit yes/no choices for worker equipment and supplies. Home Cleaning additionally asks for beginner/professional level, rooms or areas as text, and a positive room count. Car Wash asks for make/model. Pet Care asks for dog/cat and breed. For a service for someone else, the form asks for that person's first name, last name, and phone. The `Check details` action runs client-side validation for the required fields and Milano CAP range and displays field and service errors. Editing any field clears a previous validation result. The browser does not claim worker availability or exact street boundary coverage from these checks.

The form keeps field values in browser component state only. It sends no address, recipient, or schedule to a server and has no order submission action. The booking gate is visible. Live quote change handling, customer activation checks, and order creation are not implemented here. The public price preview remains at `/services` as documented in [pricing-quotes.md](pricing-quotes.md). The next B10 slice will connect the selected services and hours to a live price preview; B11 will own the server-side order command. No account or accepted quote is inferred from these local fields.

## Contract and verification

This slice uses the existing public catalog API and pinned generated TypeScript types. It adds no backend endpoint, OpenAPI change, database migration, idempotency key, audit event, or asynchronous effect. The frontend does not decide booking eligibility; a future backend order endpoint must validate location, inputs, quote, ownership, and account activation. The form currently collects address and recipient data only within the page and does not store it across reloads. Pure validation tests cover shared, service-specific, combined-service and recipient rules. From `front/`, run `npm run lint`, `npm run format:check`, `npm run typecheck`, `npm test`, `npm run contract:check`, and `npm run build`. Review `/book` in a browser at phone width and on desktop.

The product inputs and area rules are in [ADR 0003](../adr/0003-phase-1-launch-services.md). The quote and legal gates are in [ADR 0004](../adr/0004-phase-1-pricing-direction.md) and [ADR 0006](../adr/0006-phase-1-payment-direction.md).
