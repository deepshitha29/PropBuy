# PropBuy Cloud Deployment Guide (Render + Vercel)

This guide walks you through deploying **PropBuy** completely free without errors:
- **Backend (Spring Boot + Java 17)** deployed to **Render** (as a Docker Web Service)
- **Frontend (Static HTML/CSS/JS)** deployed to **Vercel**

---

## Part 1: Deploy Backend on Render

Render builds the backend using the root `Dockerfile`.

### Method A: One-Click Blueprint Deployment (Recommended)
1. Go to [dashboard.render.com](https://dashboard.render.com) and log in.
2. Click **New +** > **Blueprint**.
3. Connect your GitHub repository (`PropBuy`).
4. Render will detect `render.yaml` automatically and configure:
   - Environment: `Docker`
   - Health Check Path: `/api/health`
   - In-memory demo mode enabled out-of-the-box (`SPRING_PROFILES_ACTIVE: demo`)
   - Auto-generated 32+ character JWT secret (`APP_JWT_SECRET`)
   - Pre-configured CORS permissions for `https://*.vercel.app`
5. Click **Apply**.
6. Wait 3–5 minutes for the build to complete. Once deployed, Render will provide a service URL (e.g., `https://propbuy-backend-xxxx.onrender.com`).
7. Verify by opening in your browser:
   `https://propbuy-backend-xxxx.onrender.com/api/health`
   It should return: `{"demoMode":true,"service":"propbuy","status":"UP"}`.

### Method B: Manual Web Service Setup
If you prefer not using Blueprints:
1. Click **New +** > **Web Service**.
2. Select your GitHub repository.
3. Configure the following settings:
   - **Name**: `propbuy-backend`
   - **Language / Runtime**: `Docker`
   - **Region**: Oregon (or your preferred region)
   - **Branch**: `main`
   - **Plan**: `Free`
4. Expand **Advanced** and set **Health Check Path** to `/api/health`.
5. Under **Environment Variables**, add:
   - `SPRING_PROFILES_ACTIVE` = `demo`
   - `APP_JWT_SECRET` = `any-random-string-with-at-least-32-characters`
   - `CORS_ORIGINS` = `https://*.vercel.app,http://localhost:8080,http://localhost:5500`
   - `PORT` = `8080`
6. Click **Create Web Service**.

> [!NOTE]
> **Switching to a Real MySQL Database:**
> To use persistent cloud MySQL (such as free TiDB Cloud Serverless or Aiven MySQL) instead of the in-memory demo database:
> 1. In Render, go to **Environment**.
> 2. Delete or set `SPRING_PROFILES_ACTIVE` to `default`.
> 3. Add:
>    - `DB_URL` = `jdbc:mysql://<host>:<port>/<dbname>?serverTimezone=UTC&useSSL=true`
>    - `DB_USERNAME` = `<your-db-user>`
>    - `DB_PASSWORD` = `<your-db-password>`
> Flyway will automatically run all database migrations on the next start.

---

## Part 2: Deploy Frontend on Vercel

1. Go to [vercel.com](https://vercel.com) and log in.
2. Click **Add New…** > **Project**.
3. Import your `PropBuy` GitHub repository.
4. Settings:
   - **Framework Preset**: `Other`
   - **Root Directory**: Leave as `./` (the root `vercel.json` automatically serves `frontend/`) or set to `frontend`.
   - **Build and Output Settings**: Leave empty / default.
5. Click **Deploy**.
6. In about 30 seconds, your site will be live at `https://propbuy-xxxx.vercel.app`.

---

## Part 3: Connect Frontend to Backend

You have two simple ways to connect your Vercel frontend to your Render backend:

### Option 1: In the UI (Zero Configuration)
1. Open your deployed Vercel site in your browser.
2. Scroll to the footer and click **⚙ API Settings** (or it will prompt you if unconfigured).
3. Paste your Render backend URL (e.g. `https://propbuy-backend-xxxx.onrender.com`).
4. Click **Test connection** (it will test `/api/health` and turn green).
5. Click **Save URL**.
The frontend will store the URL in your browser and connect immediately.

### Option 2: Pre-configure in Code (Optional)
In `frontend/index.html`, set:
```html
<script>
  window.PROPBUY_API_BASE = 'https://propbuy-backend-xxxx.onrender.com';
</script>
```
Commit and push to GitHub, and Vercel will automatically redeploy with the backend pre-linked.

---

## Troubleshooting & Tips

- **Render Cold Starts**: On Render's free plan, the backend sleeps after 15 minutes of inactivity. When you send the first request, it may take 30–50 seconds to wake up.
- **CORS Errors**: The backend is configured with `setAllowedOriginPatterns` supporting wildcard domains `https://*.vercel.app`. As long as your frontend is on Vercel or localhost, requests will be accepted.
- **Demo Accounts**:
  - Email: `seller@propbuy.demo`, `buyer@propbuy.demo`, or `admin@propbuy.demo`
  - Password: `DemoPass123!`
