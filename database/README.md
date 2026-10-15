# Database notes

Production schema changes are in `backend/src/main/resources/db/migration` and are applied by Flyway. The intended production database is MySQL 8.x. The demo profile disables Flyway and uses an in-memory H2 database with local sample records, so demo behavior does not prove MySQL compatibility.

Before production launch, run all migrations against a clean MySQL 8.x staging database and verify the existing schema with a backup. The `V8__audit_details_compatibility.sql` migration reconciles the historical `reason` column with the application's `details` activity field without deleting existing rows.
