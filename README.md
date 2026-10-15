# PropBuy

PropBuy is a Spring Boot 3.3 / Java 17 application for property acquisition and resale. It has a vanilla HTML/CSS/JavaScript frontend and a MySQL-backed REST API with JWT authentication, Flyway migrations, seller acquisition workflows, buyer resale workflows, admin operations, image uploads, notifications, and reports.

## Project structure

```text
PropBuy/
|-- frontend/                  # Static browser application
|   |-- index.html
|   `-- assets/                # app.css and app.js
|-- backend/                   # Spring Boot application
|   |-- pom.xml
|   `-- src/
|       |-- main/java/         # Controllers, services, security, entities
|       |-- main/resources/    # Configuration and Flyway migrations
|       `-- test/              # Backend tests
|-- docs/                      # Setup, architecture, and deployment notes
|-- database/                  # Database-specific operational notes
|-- uploads/                  # Local uploaded images; do not commit private data
|-- tools/                     # Optional local Maven installation
|-- Dockerfile                 # Root build: copies frontend into Spring Boot static resources
|-- docker-compose.yml         # MySQL + application local deployment
|-- .env.example               # Safe environment-variable template
`-- .gitignore
```

The frontend source is not duplicated in the backend. Maven copies `frontend/` into the packaged JAR during the resource phase so Spring Boot can serve it at `/`.

## Prerequisites

Install:

- Java 17. Check with `java -version`.
- MySQL 8.x for the normal profile, or Docker Desktop for Compose. Check with `mysql --version` or `docker --version`.
- Maven, unless you use the included Maven installation. Check with `mvn -version`.
- Python 3 only if you want to serve the static frontend separately. Check with `py --version`.

This frontend does not use npm, Vite, React, or a `package.json`; do not run `npm install` or `npm run dev`.

## Configure the backend

From PowerShell at the project root:

```powershell
Copy-Item .env.example .env
notepad .env
```

Set real local values in `.env` for `DB_URL`, `DB_USERNAME`, and `DB_PASSWORD`. Set `APP_JWT_SECRET` to a random secret of at least 32 UTF-8 characters. Set `CORS_ORIGINS` to the browser origin you use, such as `http://localhost:5500` or `http://localhost:8080`.

The default profile requires the database variables and never silently switches to H2. Flyway runs automatically at startup. Do not set `JPA_DDL_AUTO=create` in a real environment; the configured value is `validate`.

## Run with Docker Compose (simplest)

```powershell
Copy-Item .env.example .env
# Edit .env and set APP_JWT_SECRET and DB_PASSWORD.
docker compose up --build
```

Open `http://localhost:8080`. Stop it with `Ctrl+C`, or use `docker compose down`. Database data is stored in `mysql_data` and uploaded images in `uploads_data`.

## Run the backend manually

Start MySQL, create a database named `propbuy`, configure `.env`, then open a PowerShell terminal in `backend/`:

```powershell
cd backend
..\tools\apache-maven-3.10.0\bin\mvn.cmd spring-boot:run
```

If Maven is installed globally, use `mvn spring-boot:run` instead. The backend serves the frontend too and listens on `http://localhost:8081` by default. Override the port with `$env:PORT = '8080'` before starting.

Build and test:

```powershell
cd backend
..\tools\apache-maven-3.10.0\bin\mvn.cmd test
..\tools\apache-maven-3.10.0\bin\mvn.cmd package -DskipTests
```

## Run the frontend separately

The frontend is plain static HTML/CSS/JavaScript. In a second PowerShell terminal from the project root, run:

```powershell
py -m http.server 5500 --directory frontend
```

Open `http://localhost:5500`. Its API calls use relative `/api/...` paths, so a separately served frontend needs a reverse proxy to the backend. For the simplest local setup, use the packaged frontend at `http://localhost:8081` instead. If a reverse proxy is used, include `http://localhost:5500` in `CORS_ORIGINS`.

For direct local frontend-to-backend requests, edit the small configuration line near the bottom of `frontend/index.html` to `window.PROPBUY_API_BASE = 'http://localhost:8081';`, then include `http://localhost:5500` in `CORS_ORIGINS` before starting the backend.

## Run both together

1. Open Terminal 1 in VS Code, run the backend command above, and keep it open.
2. Open Terminal 2, run the Python static-server command above, and keep it open.
3. Open `http://localhost:5500` through your reverse proxy, or use `http://localhost:8081` for the built-in frontend.
4. Check `http://localhost:8081/api/health` returns JSON with `status: UP`.
5. Stop each process with `Ctrl+C`.

## Demo mode (local UI testing only)

When MySQL is unavailable, explicitly start the isolated demo profile:

```powershell
cd backend
$env:SPRING_PROFILES_ACTIVE = 'demo'
$env:APP_JWT_SECRET = 'local-demo-secret-that-is-at-least-32-characters'
..\tools\apache-maven-3.10.0\bin\mvn.cmd spring-boot:run
```

Demo mode uses temporary H2 data and clearly labeled sample listings. These records are not real inventory and are never created by the normal production profile.

## Troubleshooting

- **Java version error:** install Java 17 and confirm `java -version` reports version 17.
- **Maven not found:** run the included `..\tools\apache-maven-3.10.0\bin\mvn.cmd` command from `backend/`.
- **Port already in use:** run `Get-NetTCPConnection -LocalPort 8081`; stop the owning development process or set another `$env:PORT`.
- **Database connection failed:** confirm MySQL is running, the database exists, and `DB_URL`, username, and password are correct. In Compose, use `docker compose logs mysql`.
- **Flyway migration failed:** inspect the first migration error and do not delete migration files or edit Flyway history in production.
- **Frontend cannot reach the API:** use the packaged frontend on port 8081, or configure a reverse proxy for `/api` and include the frontend origin in `CORS_ORIGINS`.
- **CORS error:** restart the backend after changing `CORS_ORIGINS`; values are comma-separated origins with no trailing path.
- **Login/JWT error:** confirm `APP_JWT_SECRET` is present and at least 32 characters, then restart the backend and log in again.
- **Missing images:** verify `UPLOAD_DIR` exists and is writable. Docker uses the persistent `/app/uploads` volume.

## Production notes

Use MySQL 8.x, keep Flyway enabled, configure a strong JWT secret and restricted CORS origins, and place `UPLOAD_DIR` on persistent storage or replace the filesystem adapter with object storage. Review [docs/DEPLOYMENT.md](docs/DEPLOYMENT.md), [docs/SETUP.md](docs/SETUP.md), and [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) before deployment. Never commit `.env`, credentials, logs, build output, or private uploaded files.
