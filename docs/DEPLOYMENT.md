# Deployment checklist

1. Build from the repository root with the supplied `Dockerfile` or build `backend/pom.xml` with Java 17.
2. Set `DB_URL`, `DB_USERNAME`, `DB_PASSWORD`, `APP_JWT_SECRET`, `CORS_ORIGINS`, `FRONTEND_BASE_URL`, and a persistent `UPLOAD_DIR`. The default profile fails fast when database variables are missing; it never falls back to H2.
3. Use a MySQL 8.x database and leave Flyway enabled. Run migrations against a disposable staging database before production.
4. Use persistent disk or replace the filesystem image adapter with object storage; local `uploads/` is not a production guarantee.
5. Expose `/api/health` through the platform health check. Configure TLS, logs, backups, and monitoring at the hosting layer.

`docker-compose.yml` is a local MySQL/app example. It sets the container port to 8080, requires `APP_JWT_SECRET`, and persists MySQL and uploaded images in named volumes; it is not a production hardening profile.
