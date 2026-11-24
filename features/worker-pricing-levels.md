# Worker pricing levels

Worker onboarding requires a pricing level and EUR hourly rate after phone verification. Workers can update both together from Worker Settings. Existing worker profiles have no pricing until their owner chooses it. The preview uses the requirements as self-selection guidance. Automated qualification decisions need measurable thresholds and verified inputs.

| Level | Hourly range | Requirements shown |
| --- | --- | --- |
| Starter | €18–€21 | New to Cleanzza, no Cleanzza work history, limited or no professional cleaning experience, few or no ratings and reviews |
| Standard | €22–€25 | Verifiable professional experience, completed verification, several successful Cleanzza orders, good customer rating |
| Expert | €26–€30 | Strong professional cleaning experience, recognized valid cleaning certificate, high rating, many successful orders, low cancellation rate, consistently high service quality, own basic professional cleaning equipment |
| Premium | €31–€35 | Extensive professional cleaning experience, recognized valid cleaning certificate, excellent rating, many successful orders, very low cancellation rate, consistently exceptional service quality, own professional-grade cleaning equipment |

## API and persistence

- `GET /cleaners/pricing-levels` returns the four level codes, display labels, inclusive bounds, requirements, and EUR currency.
- `GET /cleaners/me/pricing` returns the signed-in worker's current level, hourly rate, and currency. Both values are null until selected.
- `PUT /cleaners/me/pricing` replaces both fields atomically using the worker session cookie and `X-CSRF-Token`. Send a level code and a two-decimal string, for example `{"level":"STARTER","hourly_rate":"19.50"}`. Valid saves return 200 and can be retried with the same desired values. PostgreSQL also checks the pair and rate range. Worker endpoints use 401 for missing sessions, 403 for invalid role or write authorization, 404 for a missing worker profile, 422 for invalid values, and 503 for unavailable storage.
- `GET /cleaners/demo-directory` lists priced synthetic workers, with opaque cleaner ID, first name when set, level, hourly rate, and currency. It returns `items` and `next_cursor`, ordered by cleaner ID, with optional UUID `cursor` and `limit` from 1 to 50 (default 50). Filtering is limited to workers with pricing and pending or approved review status. Contact information, evidence, and user IDs stay private. The directory requires the booking demo and inherits the hosted preview access boundary. Unavailable demo configuration returns 503.
- `POST /quotes/drafts` and `POST /customers/me/orders/drafts` accept an optional `worker_id` from the demo directory. The server looks up the current rate and returns 404 for an unavailable worker, or 503 when worker booking is disabled. A request without a worker uses the existing base estimate and open request flow. The usual quote validation, ownership, CSRF, and draft idempotency rules apply. Check the saved draft's total before submitting it.
- A targeted draft stores the worker ID and quote amount in the snapshot. Rate changes affect new quotes and drafts. A saved draft keeps its quoted amount through expiry and retry, and only the selected worker can see or accept a targeted request. Existing untargeted draft fingerprints and quote snapshots stay compatible.

All pricing is synthetic in the hosted demo. The existing fake-payment flow and demo commission apply. The migration adds nullable pricing columns without assigning a rate to existing workers. Deploy the API first. Verify with PostgreSQL migration and integration tests, OpenAPI export, frontend generated-client checks, and a hosted synthetic request using a selected worker.

See [versioned quote pricing](pricing-quotes.md) and [synthetic worker request acceptance](demo-worker-request-acceptance.md).

With [worker verification](worker-document-verification.md) enabled, the directory and targeted estimates require approved verification in the Ferrara service area as well as a valid price.
