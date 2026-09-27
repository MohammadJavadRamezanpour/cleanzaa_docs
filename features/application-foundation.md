# Application foundation (B01)

## Purpose and scope

B01 creates the separate Next.js and FastAPI application repositories needed
for the Phase 1 customer and cleaner journeys. It provides runnable shells,
local PostgreSQL, an Alembic baseline, independent CI and staging configuration,
health checks, request IDs, structured request logs, optional Sentry error
reporting, and an exported OpenAPI contract. The backend and PostgreSQL remain
authoritative for later business rules.

The foundation contains no registration, profiles, catalogue, orders, money,
file storage, or authorization policy. Those are later B-series changes. It
does not create user tables or store personal data.

## Repositories and configuration

- `front/` is a standalone Next.js/TypeScript repository. Its public
  `NEXT_PUBLIC_API_BASE_URL` must be a bare HTTPS origin in staging or
  production; `http://localhost` and `http://127.0.0.1` are allowed locally.
  The URL is embedded at build time. `.env.local` is ignored by Git.
- `back/` is a standalone FastAPI/Python repository. It owns Docker Compose
  PostgreSQL for local development, SQLAlchemy connectivity, Alembic, its
  Docker image, and `render.yaml`. `DATABASE_URL` is required for readiness and
  migrations but not liveness or OpenAPI export. A managed PostgreSQL URL using
  `postgresql://` is normalized to the Psycopg 3 driver.
- Each repository has its own README, environment example, lock or dependency
  declaration, CI workflow, and feature branch. The documentation repository
  remains separately versioned.
- Sentry DSNs are optional locally and supplied only through environment
  configuration in staging. The frontend uses client and server DSNs; the
  backend uses one server DSN. The integrations do not enable default PII
  collection or request tracing.

## Health API contract

Health endpoints are unauthenticated operational probes. They accept no path,
query, or body parameters. `X-Request-ID` is optional on backend requests; a
safe value of up to 128 ASCII characters is preserved, and other values are
replaced with a UUID. Both backend health responses return the ID in that
header. The frontend proxy applies the same rule to frontend requests.

| Service | Method and path | Success | Failure |
| --- | --- | --- | --- |
| Frontend | `GET /health` | `200 {"status":"ok"}` | Standard platform HTTP failure |
| Backend | `GET /health/live` | `200 {"status":"ok"}` | Standard platform HTTP failure |
| Backend | `GET /health/ready` | `200 {"status":"ok"}` after `SELECT 1` succeeds | `503 {"error":{"code":"DATABASE_UNAVAILABLE","message":"Service temporarily unavailable."}}` |

The endpoints send `Cache-Control: no-store`. Health checks are read-only and
have no idempotency or concurrency effects. The readiness response does not
expose connection details. FastAPI routes and Pydantic schemas generate
`back/openapi/openapi.json`, including the 503 error schema. The frontend pins
a copy at `front/contracts/openapi.json`, generates TypeScript types, and checks
that generated types match the pinned contract in CI. B02 will extend this
contract to domain APIs and establish cross-repository compatibility handling.

## Observability and security

Backend request logs are JSON lines with timestamp, level, event, request ID,
method, path, status code, and duration. Frontend proxy logs are JSON lines
with timestamp, level, event, request ID, method, and path. Query strings,
request bodies, credentials, and database URLs are not logged. Backend
unexpected errors are reported through Sentry when configured. The frontend
initializes Sentry for client and server errors when configured.

Backend CORS currently permits only GET requests from configured origins, as
the foundation exposes only GET routes. Later API cards must expand methods and
headers deliberately. Health routes have no ownership requirement and reveal
no user or system data beyond availability. The initial Alembic revision has
no domain tables; migrations are applied by a predeploy command before the
backend readiness gate admits traffic.

## Staging and delivery state

`back/render.yaml` defines a Docker service on `develop` with a managed
database URL supplied as a secret. The frontend README specifies a Vercel
preview deployment for `develop`; both services deploy independently. Those
staging accounts, remote repositories, database, Sentry projects, and secrets
still need to be connected after code review. A live staging deployment, backup
policy, restore drill, monitoring alert delivery, branch protection, and CI
results cannot be claimed until those resources exist. See
[`../operations/backup-restore.md`](../operations/backup-restore.md) and
[`../adr/0001-staging-hosting.md`](../adr/0001-staging-hosting.md).

## Canonical verification

From `back/`, install `.[dev]` in `.venv` and run `ruff check .`,
`ruff format --check .`, `mypy app scripts`, `alembic upgrade head`,
`pytest -q`, `python -m scripts.export_openapi --check`,
`pip-audit`, and `docker build -t cleanzza-api:local .` using the commands in
its README. Both code repositories run Gitleaks in CI.

From `front/`, run `npm ci`, `npm run lint`, `npm run format:check`,
`npm run typecheck`, `npm test`, `npm run contract:check`,
`npm run build`, and `npm audit --omit=dev --audit-level=high`.
The same checks are in each repository's CI. With staging connected, verify
`GET /health` and `GET /health/ready` from outside the deployment platform.
