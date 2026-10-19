# Customer booking form (B10)

## Implemented scaffold

The mobile-width `/book` page loads the typed `GET /catalog/launch` response and presents the first shared booking inputs: one launch service, service date, start time, fixed Milano area, street, house number, and CAP. It provides loading, failure/retry, and ready states, with explicit labels for keyboard and assistive technology. The page is linked from `/services`; its 440-pixel frame also applies on desktop. Dates and times are entered for the catalog's `Europe/Rome` timezone. The city and area are controlled values from the server catalog, not arbitrary customer text.

This first scaffold keeps field values in browser component state only. It sends no address or schedule to a server and has no submission action. The booking gate is visible in the page. Service-specific inputs, recipient fields, multiple services, validation, live quote change handling, customer activation checks, and order creation are not implemented here. The public price preview remains at `/services` as documented in [pricing-quotes.md](pricing-quotes.md). Later B10 slices will add the remaining inputs and preview behavior; B11 will own the server-side order command. No account or accepted quote is inferred from these local fields.

## Contract and verification

This slice uses the existing public catalog API and pinned generated TypeScript types. It adds no backend endpoint, OpenAPI change, database migration, idempotency key, audit event, or asynchronous effect. The frontend does not decide booking eligibility; a future backend order endpoint must validate location, inputs, quote, ownership, and account activation. The form currently collects address data only within the page and does not store it across reloads. From `front/`, run `npm run lint`, `npm run format:check`, `npm run typecheck`, `npm test`, `npm run contract:check`, and `npm run build`. Review `/book` in a browser at phone width and on desktop.

The product inputs and area rules are in [ADR 0003](../adr/0003-phase-1-launch-services.md). The quote and legal gates are in [ADR 0004](../adr/0004-phase-1-pricing-direction.md) and [ADR 0006](../adr/0006-phase-1-payment-direction.md).
