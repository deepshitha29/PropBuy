# Local setup

## Demo mode

Use Java 17 and the Maven wrapper installation in `tools/apache-maven-3.10.0` when Maven is not on PATH:

```powershell
$env:APP_JWT_SECRET = 'local-demo-secret-that-is-at-least-32-characters'
$env:SPRING_PROFILES_ACTIVE = 'demo'
..\tools\apache-maven-3.10.0\bin\mvn.cmd package -DskipTests
java -jar backend\target\propbuy-0.1.0.jar
```

Open `http://localhost:8081`. Demo users use `DemoPass123!`; they are local-only accounts and the displayed properties are explicitly sample data.

## MySQL mode

Copy `.env.example` to `.env`, provide a MySQL 8.x database, a strong `APP_JWT_SECRET` with at least 32 characters, and the intended frontend origin. Run the application without the `demo` profile. Flyway applies the migrations; do not use `ddl-auto=create` in production.

The browser application is packaged by Maven from `frontend/`. No separate frontend server is required for the packaged deployment.
