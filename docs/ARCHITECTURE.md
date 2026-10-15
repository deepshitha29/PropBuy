# PropBuy architecture

PropBuy is a Spring Boot 3.3 / Java 17 application with a small vanilla JavaScript frontend. The frontend source lives in `frontend/`; the Maven resource build copies it into the packaged Spring Boot classpath at `static/`. The backend owns authentication, authorization, persistence, workflow transitions, image metadata, notifications, and reports.

## Main boundaries

- `frontend/` — `index.html`, `assets/app.js`, and `assets/app.css`.
- `backend/src/main/java/com/propbuy/model` — JPA entities.
- `backend/src/main/java/com/propbuy/web` — REST controllers.
- `backend/src/main/java/com/propbuy/service` — workflow activity and notification services.
- `backend/src/main/resources/db/migration` — Flyway migrations for MySQL production.
- `backend/src/main/resources/application.yml` — production/default configuration; secrets are required through environment variables.
- `backend/src/main/resources/application-demo.yml` — explicitly local H2 demo configuration.

The browser is login-gated. Search APIs require authentication, and method-level security protects ownership and administrative operations. Demo listings are created only under the `demo` profile and the UI labels that environment as local development data.
