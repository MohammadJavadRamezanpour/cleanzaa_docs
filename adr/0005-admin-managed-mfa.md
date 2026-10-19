# ADR 0005: Managed MFA for reviewers

**Status:** Approved direction, 2026-10-18. Auth0 EU is the initial managed identity provider. Tenant creation, account ownership, contracts, region and live factor configuration remain operational launch gates.

## Decision

Designated administrators use a separate Auth0 EU tenant and Universal Login with MFA. The existing customer/worker phone-code session cannot authorize an admin operation, even if its user has an `ADMIN` row. The admin frontend requests a short-lived RS256 access token for the Cleanzza admin API audience. No Auth0 client secret belongs in browser code. The backend verifies the token signature from the configured tenant JWKS, issuer, audience, expiry, issued-at time, and the exact admin application client ID (`azp`). It also requires the namespaced Boolean `https://cleanzza.it/claims/admin_mfa` claim.

The token subject must map through an explicit server-held allowlist to an existing Cleanzza user ID, and that user must still have a database `ADMIN` role. Provisioning and removing reviewers is a controlled operator action, never public registration. Revoking the database role takes effect on the next request; allowlist edits take effect after the reviewed configuration is rolled out. Each file access or decision continues to audit the local user ID, reason and time. General support users do not inherit identity-review access.

Configure Auth0 MFA to challenge on the admin application and add the MFA claim **only after** a completed factor. The Auth0 Post-Login flow must use an MFA challenge Action followed by a separate claim Action, because Auth0 documents that the older `api.multifactor.enable()` path can run before the MFA method appears on first login. The second Action checks `event.authentication.methods` for `mfa` and sets the access-token claim only when present. The backend remains the final gate; a missing/false claim is denied. Configure access-token lifetime at most five minutes for reviewer operations, and obtain a new managed token when it expires.

Admin API calls use `Authorization: Bearer <access token>`. The backend still checks allowed origins on write requests. Bearer tokens are not carried in cookies, so the customer/worker CSRF token is not an admin credential. Private evidence stays behind one-use, actor-bound review grants and file-read audit from B06.

## Rollout and limits

Backend admin endpoints fail closed without complete issuer, audience, client ID, allowlist and local `ADMIN` role. A tenant must be created in a reviewed EU region and a dedicated public SPA admin application plus API audience configured; verify tenant ownership, subprocessors, factor recovery and access logs before real identity documents are enabled. The browser uses Auth0's authorization-code-with-PKCE SDK and in-memory token cache; it contains no client secret. The Auth0 application must allow the deployed `/admin/reviews` callback and logout URL and the admin API audience. Development and staging use synthetic evidence. The later B07 slices add audited worker decisions and a mobile review screen; live operation still depends on tenant provisioning and the ADR 0002 gates.

Do not treat Auth0 organization email-domain discovery as an allowlist. Auth0 says it routes users for discovery but does not restrict who can sign in. Use exact subject mapping and the local role as independent checks.

## References

- [Phase 1 personal-data policy](0002-phase-1-personal-data.md)
- [Auth0 JWKS](https://auth0.com/docs/secure/tokens/json-web-tokens/json-web-key-sets)
- [Auth0 access-token custom claims](https://support.auth0.com/center/s/article/Set-access-token-claims-using-actions)
- [Auth0 MFA method timing on first login](https://support.auth0.com/center/s/article/using-actions-mfa-authentication-method-is-missing-on-first-login)
- [Auth0 organization domain limitation](https://support.auth0.com/center/s/article/organization-domains-access-restrictions-in-auth0)
