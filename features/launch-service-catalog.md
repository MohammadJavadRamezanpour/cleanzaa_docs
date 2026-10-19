# Phase 1 launch service catalog (B08)

## Purpose and scope

The backend publishes one versioned, read-only catalog for the approved
Milano launch. It contains Home Cleaning, Car Wash, and Pet Care only. The
customer and worker browser can read the same API; the mobile `/services`
page shows the choices and their essential inputs. There is no catalog editor,
pricing, quote, or booking API in this original catalog slice. The later public
draft price endpoint is documented in [pricing-quotes.md](pricing-quotes.md).

[ADR 0003](../adr/0003-phase-1-launch-services.md) sets the product rules.
`milano-v1` is a stable identifier. Changing coverage, fields, or
qualifications requires a new version; future accepted quotes and orders must
retain the version they used.

## API contract

`GET /catalog/launch` is public and takes no path, query, headers, body,
pagination, ordering, or filtering parameters. It returns `200` with the
`LaunchCatalog` OpenAPI schema, `Cache-Control: public, max-age=300`, and the
standard `X-Request-ID`. There are no domain error responses or authentication
checks. Standard infrastructure errors may still occur.

The response includes:

- `version: "milano-v1"`.
- One area: `IT-MI-MILANO`, city `Milano`, country `IT`, timezone
  `Europe/Rome`, inclusive five-digit CAP range `20121`–`20162`.
- Whole-hour duration from 1 through 8, step 1, for each selected service.
  Multiple selected services reserve the sum of their durations in a future
  booking flow.
- Shared required fields: service date, start time, area code, street, house
  number, city, postcode, and recipient type. A service for someone else also
  requires recipient first name, last name, and phone.
- Three service definitions with machine-readable required field names,
  display name and summary, customer notes, worker qualification summary,
  certificate requirement, and manual approval requirement.

Home Cleaning requires room selection, a positive room count, and beginner or
professional preference. A worker needs an approved Home Cleaning level; no
certificate is required by ADR 0003. Car Wash requires make/model and an
approved service certificate. Pet Care requires dog/cat species and breed and
an approved service certificate. Every service also asks for hours and
explicit equipment and supplies choices. Workers must be approved for the
chosen service and area and able to bring requested items.

The backend's `is_supported_service` and `is_supported_address` functions
reject codes outside these three and city/CAP combinations outside the launch
gate. The city comparison is case-insensitive after trimming. The CAP gate is
coarse and does not prove that an exact street belongs to the municipality.
The draft quote endpoint calls these checks within server-side validation; a
frontend selection alone cannot authorize booking. There is currently no
booking endpoint, so this catalog does not claim an end-to-end
unsupported-booking rejection yet.

## Browser behavior

`/services` loads the catalog from the API and displays a mobile-width view
on phones and desktops. It has loading, failure/retry, and ready states. It
shows all three services and the Milano area, with a draft price preview and a
clear message that booking is still forthcoming. Links from the customer and worker
profile screens open the same view. The browser uses the generated TypeScript
schema pinned to the backend OpenAPI export; it does not decide eligibility.

## Verification and rollout

The catalog is code configuration with no database migration or sensitive
data. Backend tests assert the public response and accepted/rejected area and
service values. Run `.venv/bin/pytest -q tests/test_launch_catalog.py`,
`.venv/bin/ruff check .`, `.venv/bin/ruff format --check .`,
`.venv/bin/mypy app scripts`, and
`.venv/bin/python -m scripts.export_openapi --check` from `back/`.
Frontend tests assert the typed public request; run `npm run lint`,
`npm run format:check`, `npm run typecheck`, `npm test`,
`npm run contract:check`, and `npm run build` from `front/`.
The backend OpenAPI and frontend pinned contract must be updated together.
