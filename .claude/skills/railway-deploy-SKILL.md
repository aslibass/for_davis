---
name: railway-deploy
description: Make any project Railway-ready and guide deployment via GitHub. Use when asked to deploy to Railway, prepare a project for Railway, or set up Railway config files.
---

Railway deploys directly from GitHub. Every service needs a `railway.toml` in its root directory (or subdirectory for monorepos). Railway's current build system is **Railpack** (replaced Nixpacks). Docs: https://docs.railway.com/reference/config-as-code

---

## Step 1 — Audit the project

Check for these issues before writing any config:

- [ ] Does the backend read `$PORT` from env? Railway sets this — hardcoded ports will fail.
- [ ] Are API/WebSocket URLs hardcoded to `localhost`? Must use env vars at build time.
- [ ] Does CORS allow `*` or a hardcoded origin? Should read from `ALLOWED_ORIGINS` env var.
- [ ] Is the database path hardcoded? SQLite is ephemeral on Railway (data lost on redeploy). Use PostgreSQL for persistence or accept ephemeral data.
- [ ] Does the frontend need a build step? Vite/React must be built; env vars are baked in at build time.

---

## Step 2 — Fix the code

### Backend (Python/FastAPI)

Read PORT from env in the start command, not in code:
```
uvicorn app.main:app --host 0.0.0.0 --port $PORT
```

Read CORS origins from env:
```python
import os
_raw = os.getenv("ALLOWED_ORIGINS", "*")
origins = ["*"] if _raw == "*" else [o.strip() for o in _raw.split(",")]
app.add_middleware(CORSMiddleware, allow_origins=origins,
                   allow_credentials=_raw != "*", allow_methods=["*"], allow_headers=["*"])
```

### Frontend (React/Vite)

Create `src/api/env.ts` (or equivalent):
```typescript
export const API_BASE = import.meta.env.VITE_API_URL ?? 'http://localhost:8000'
export const WS_BASE = API_BASE.replace(/^https:/, 'wss:').replace(/^http:/, 'ws:')
```

Import `API_BASE` and `WS_BASE` wherever `localhost` was hardcoded. Vite env vars must be prefixed `VITE_` to be exposed to the browser bundle. They are baked in at build time — a redeploy is required after changing them.

---

## Step 3 — Write railway.toml files

Place each `railway.toml` in the root of the service it configures.

### Python/FastAPI backend

```toml
[build]
builder = "RAILPACK"

[deploy]
startCommand = "uvicorn app.main:app --host 0.0.0.0 --port $PORT"
restartPolicyType = "ON_FAILURE"
restartPolicyMaxRetries = 3
```

### React/Vite frontend (static)

```toml
[build]
builder = "RAILPACK"
buildCommand = "npm install && npm run build"

[deploy]
startCommand = "npx serve dist -l $PORT --no-clipboard"
restartPolicyType = "ON_FAILURE"
restartPolicyMaxRetries = 3
```

### With health check (recommended for APIs)

```toml
[build]
builder = "RAILPACK"

[deploy]
startCommand = "uvicorn app.main:app --host 0.0.0.0 --port $PORT"
healthcheckPath = "/health"
healthcheckTimeout = 30
restartPolicyType = "ON_FAILURE"
restartPolicyMaxRetries = 3
```

### Complete field reference

```toml
[build]
builder = "RAILPACK"          # or "DOCKERFILE"
buildCommand = ""             # override detected build command
dockerfilePath = ""           # path to non-standard Dockerfile
railpackVersion = ""          # pin a Railpack version
watchPatterns = ["src/**"]    # only deploy when these paths change

[deploy]
startCommand = ""             # override detected start command
preDeployCommand = ""         # run before container starts (migrations etc)
healthcheckPath = "/health"
healthcheckTimeout = 30
restartPolicyType = "ON_FAILURE"   # ON_FAILURE | ALWAYS | NEVER
restartPolicyMaxRetries = 3
overlapSeconds = 20           # zero-downtime: keep old container running this long
drainingSeconds = 10          # grace period between SIGTERM and SIGKILL
cronSchedule = "0 * * * *"   # for cron jobs, not servers
```

---

## Step 4 — Monorepo setup

For a repo with `/backend` and `/frontend` subdirectories, create **two Railway services** from the same GitHub repo:

1. Service 1 → set Root Directory to `/backend` in Railway UI → Railway reads `backend/railway.toml`
2. Service 2 → set Root Directory to `/frontend` in Railway UI → Railway reads `frontend/railway.toml`

Each service deploys independently and gets its own URL and env vars.

---

## Step 5 — Environment variables to set in Railway dashboard

Set these in the Railway service settings after deployment, not in `railway.toml` (never commit secrets to git).

| Service | Variable | Value |
| ------- | -------- | ----- |
| Frontend | `VITE_API_URL` | `https://your-backend.railway.app` |
| Backend | `ALLOWED_ORIGINS` | `https://your-frontend.railway.app` |
| Backend | `DATABASE_URL` | Connection string (if using Railway Postgres) |
| Any | `NODE_ENV` | `production` |

**Order matters for Vite:** set `VITE_API_URL` *before* the first deploy, or trigger a redeploy after setting it. Vite bakes env vars at build time.

---

## Step 6 — Deploy steps in Railway dashboard

1. New project → Deploy from GitHub repo
2. Select the repo → Railway creates a service
3. For monorepos: Settings → Root Directory → set to `/backend` or `/frontend`
4. Variables tab → add env vars from Step 5
5. Deploy → Railway runs the build and start commands from `railway.toml`
6. Domains tab → Railway assigns a `.railway.app` URL; add a custom domain here

---

## PostgreSQL (when persistence is needed)

SQLite is fine for ephemeral workshops, prototypes, or single-session tools. Use PostgreSQL when data must survive redeployment.

1. Railway dashboard → New Service → Database → PostgreSQL
2. Railway auto-sets `DATABASE_URL` in any service you link to it
3. Update `database.py` (or equivalent) to read from env:

```python
import os
DATABASE_URL = os.getenv("DATABASE_URL", "sqlite:///./local.db")
# PostgreSQL URLs from Railway use postgres:// — SQLAlchemy needs postgresql://
if DATABASE_URL.startswith("postgres://"):
    DATABASE_URL = DATABASE_URL.replace("postgres://", "postgresql://", 1)
```

4. Add `psycopg2-binary` to `requirements.txt`

---

## Common failures

| Symptom | Cause | Fix |
| ------- | ----- | --- |
| Build succeeds, service crashes on start | Port not bound to `0.0.0.0` or not reading `$PORT` | Use `--host 0.0.0.0 --port $PORT` |
| Frontend calls fail in production | `VITE_API_URL` not set before build | Set env var, redeploy |
| CORS errors in browser | `ALLOWED_ORIGINS` missing or wrong | Set to exact frontend URL including `https://` |
| WebSocket connections fail | WS URL still using `ws://` on HTTPS site | Derive from API URL: `https://` → `wss://` |
| Static frontend shows blank page on refresh | No SPA fallback for client-side routing | Add `--single` flag: `npx serve dist -l $PORT --single` |
| Database data lost after redeploy | SQLite on ephemeral filesystem | Accept it (for short-lived data) or switch to Railway Postgres |
