# ADR 0001: Staging hosting for the application foundation

**Status:** Proposed for B01 review

## Context

The architecture requires separate Next.js and FastAPI repositories with
independent deployments. It allows Vercel or an equivalent frontend host and
a managed Docker container host for the backend. B01 needs a concrete staging
configuration without choosing payment, identity, or production data policy.

## Proposed decision

Use Vercel preview deployments of the frontend `develop` branch and a Render
Docker web service for the backend `develop` branch. Provision managed
PostgreSQL separately and supply the URL through Render's secret store. The
backend Blueprint uses the Frankfurt region for staging and gates traffic on
`/health/ready` after Alembic migrations. Configure Sentry projects and
staging origins through provider environment settings.

This is a staging deployment choice only. Production hosting, data residency,
backup retention, recovery objectives, account ownership, and spending require
their own review before connection or launch. No service is provisioned by
this ADR or the B01 code change.

## Consequences

The repositories can build and deploy independently. Cross-origin access
requires the Vercel staging origin in backend CORS settings. Provider accounts,
secrets, managed database, alerts, and a restore drill remain prerequisites for
claiming a functioning staging environment. If the review selects another
managed host, replace the staging configuration and update this ADR before
deployment.
