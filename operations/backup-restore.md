# Database backup and restore procedure

**Applies to:** Cleanzza managed PostgreSQL in staging and production.

## Before enabling an environment

1. Select a managed PostgreSQL plan that supports encrypted automated backups
   and point-in-time recovery. Record the provider, backup schedule, retention,
   encryption settings, access owners, recovery point objective, and recovery
   time objective in the environment's private operations record. Production
   objectives must be approved before launch.
2. Keep development, staging, and production databases in separate resources
   and credentials. Keep connection strings in the hosting platform's secret
   store. Restrict backup and restore permissions to named operators. Never
   commit database exports, URLs, passwords, or provider tokens.
3. Configure backup failure, database availability, and restore-test alerts to
   an owned operations channel. Verify that the alert reaches its owner.

## Restore drill (staging, and before a production launch)

1. Record the source environment, backup or point-in-time timestamp, operator,
   and intended recovery target. Verify the backup exists and is within the
   recorded recovery point objective.
2. Restore to a **new isolated database**, never over the live database. Use
   the provider's encrypted restore operation and a new secret. Do not connect
   the public application to the restored copy yet.
3. Restrict network access to the restored database. Run `alembic current`
   against it, then `alembic upgrade head` if the application version being
   tested requires a forward migration. Do not run a destructive downgrade.
4. Point a temporary backend instance at the restored database. Verify
   `GET /health/ready` returns 200, inspect the Alembic revision, and compare
   expected schema and record counts with the source's documented checkpoint.
   For B01, the expected schema contains only Alembic's version table.
5. Record elapsed time, observed data age, any migration or compatibility
   issue, and whether the recovery objectives were met. Remove the temporary
   instance and restored database under the approved retention policy.

## Incident restore

1. Declare the incident and stop writes to the affected environment. Preserve
   logs and the original database for investigation.
2. Choose a recovery point with the incident owner. Explain the expected data
   loss window before switching traffic.
3. Restore to a new managed database, verify it using the staging drill steps,
   then update the backend's `DATABASE_URL` secret. Deploy a compatible backend
   version and wait for `/health/ready` to pass before routing traffic.
4. Verify application behavior and monitoring. Keep the former database
   isolated until the incident owner approves its disposal. Record the actual
   recovery point and time, affected data, and follow-up actions.

No provider account is connected by B01's local code change, so the first
staging restore drill and alert verification remain delivery gates. Later
features that store personal or financial data must add their approved
retention, legal-hold, and reconciliation checks to this procedure.
