# Admin reviewer authentication (B07 security slice)

## Purpose and scope

The B06 private reviewer endpoints require managed Auth0 MFA plus an explicit subject allowlist and local `ADMIN` role. This backend slice closes the prior path in which a Cleanzza phone-code session with an `ADMIN` role could read private worker evidence. It does not yet provide the B07 review screen or approval decisions. [ADR 0005](../adr/0005-admin-managed-mfa.md) records the provider decision and deployment gates. Only synthetic identity data may be used until ADR 0002's retention and provider review is complete.

## Request contract

`POST /files/{file_id}/review-access` and `GET /files/review-access/content` now require `Authorization: Bearer <Auth0 access token>` instead of the `cleanzza_session` cookie. The bearer token must be an RS256 JWT for the configured issuer, API audience and admin application client ID, with required `exp`, `iat`, `sub` and `https://cleanzza.it/claims/admin_mfa: true`. The `sub` must be present in `AUTH0_ADMIN_SUBJECT_USER_IDS` and its mapped local user must have the database `ADMIN` role. The API never trusts an email address or a token role claim alone.

`POST` also requires an allowed `Origin` when supplied and a JSON body `{"reason":"worker identity review"}`. It no longer requires `X-CSRF-Token`; the admin bearer token is not stored in a cookie. `GET` additionally requires the actor-bound, one-use `X-File-Access-Token` header returned by the POST. Both responses use `Cache-Control: no-store` and the existing standard error envelope. `401` means absent/invalid/expired bearer; `403` means valid token without the required MFA, application, allowlist or local role, or a disallowed write origin; `404` means missing/inaccessible file or a grant bound to a different actor; `409` means file state changed; `422` means invalid request or missing grant header; `503` means admin Auth0 configuration, database or JWKS availability is missing. CORS includes the `Authorization` header for configured frontend origins. The OpenAPI security scheme is HTTP bearer for these paths.

The file-read grant remains bound to the local actor ID for 60 seconds and can be consumed once. The successful byte read creates an audit row with local actor, file, reason and time. A phone-code session, including one with an `ADMIN` row, has no admin authority without the managed token. There is no public admin-signup endpoint.

## Configuration and operation

Set all four together in the backend secret manager:

```text
AUTH0_ADMIN_ISSUER=https://<tenant>.eu.auth0.com/
AUTH0_ADMIN_AUDIENCE=<registered admin API identifier>
AUTH0_ADMIN_CLIENT_ID=<admin frontend application client ID>
AUTH0_ADMIN_SUBJECT_USER_IDS={"auth0|<subject>":"<existing-admin-user-UUID>"}
```

The issuer must be an HTTPS origin. Empty or partial configuration fails closed. Add/remove the subject mapping via a reviewed configuration change, and provision/revoke the local `ADMIN` role separately. Configure an Auth0 Post-Login MFA challenge Action for the admin application, followed by a claim Action that checks for a completed `mfa` authentication method and sets the namespaced access-token claim to Boolean `true`. Set a short access-token lifetime (at most five minutes); never log or persist the bearer token or put it in a URL. Validate the exact EU tenant, operational recovery, privacy agreement and allowed factors before production use. Keep customer/worker OTP separate.

## Verification

From `back/`: `.venv/bin/pytest -q tests/test_admin_auth.py tests/test_private_worker_files.py`, `.venv/bin/ruff check .`, `.venv/bin/ruff format --check .`, `.venv/bin/mypy app scripts`, `.venv/bin/python -m scripts.export_openapi --check`, and the backend CI PostgreSQL migration/test/security/build jobs. Unit tests cover signature, issuer, audience, expiry, MFA, allowlist, client and local role rejection. PostgreSQL integration tests cover the phone-session denial, actor-bound one-use file grant and audit. A real Auth0 EU tenant sign-in and factor challenge is required before staging activation; synthetic JWTs prove the backend boundary, not tenant configuration.
