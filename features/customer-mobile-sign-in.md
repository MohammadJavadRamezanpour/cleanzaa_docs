# Customer mobile sign-in and profile (B03 frontend)

## Purpose and scope

The Next.js customer screen provides phone-code sign-in, session recovery,
profile editing, and acceptance of the current customer terms. The layout is
mobile-first, with a maximum-width phone presentation on larger screens. It
uses the backend's B03 API and pins the exported OpenAPI document at
`front/contracts/openapi.json`; generated TypeScript types live at
`front/lib/api/generated/schema.ts`.

The screen does not offer booking, payment, address search, phone changes, or
account deletion. Booking remains a later feature. An active profile says it
is ready while making clear that booking is not yet available.

## Behavior

On load, the browser calls `GET /auth/customer/session` with credentials. A
`401` shows phone sign-in. Other failures show a retry state. The phone form
requires canonical E.164 input and requests a six-digit code through
`POST /auth/customer/request-otp`. The code form calls
`POST /auth/customer/verify-otp`, then reads `GET /customers/me/profile` and
`GET /terms/customer/current`. The customer can resend a code or change the
phone number before verification. The backend enforces code expiry, cooldown,
attempt limits, and rate limits.

The profile form saves name and service address through
`PATCH /customers/me/profile`. Both are required by this screen for customer
activation. If approved current terms are configured, a separate link opens
the immutable HTTPS document, and an explicit checkbox enables
`POST /customers/me/terms-acceptances`. The screen shows accepted status only
for the current version. If terms are unavailable (`503`), the customer may
save details but cannot accept terms or become active. A stale-version `409`
reloads the current terms and asks the customer to review again.

The sign-out action calls `POST /auth/logout` and returns to phone sign-in.
Loading, validation, network failure, rate-limit, and expired-session states
are visible. Form inputs have labels; feedback uses alert or status regions.

## Security and data

Every API call uses `credentials: include` and `cache: no-store`. The
HTTP-only session cookie is never read by JavaScript. Mutations send the CSRF
token returned by the session or verification endpoint. The frontend holds
that token only in component memory. Profile data is sent only to the
configured backend origin. No phone number, code, token, name, or address is
written to application logs or browser storage.

The backend still owns role, ownership, terms, and booking eligibility checks.
The configured `NEXT_PUBLIC_API_BASE_URL` must be an HTTPS origin outside local
development, and the backend must allow the frontend origin through CORS with
credentials. Deployed frontend/API cookie behavior needs a browser check in
the actual staging domain pair before real users sign in.

## Verification

From `front/`, run `npm ci`, `npm run lint`, `npm run format:check`,
`npm run typecheck`, `npm test`, `npm run contract:check`, `npm run build`, and
`npm audit --omit=dev --audit-level=high`. The tests cover session recovery,
credentialed and CSRF-protected profile writes, and rate-limit responses.
The pinned contract came from backend B03 OpenAPI export and generated types
are checked in CI.
