# eDelphi Setup Guide

This guide covers two deployment scenarios:
- **Local macOS** — run everything on your Mac with Docker
- **Railway** — deploy to the cloud at [railway.app](https://railway.app)

---

## Local macOS Setup

### Prerequisites
- [Docker Desktop for Mac](https://www.docker.com/products/docker-desktop/)
- [Git](https://git-scm.com/download/mac) (or install via `brew install git`)

### Steps

```bash
# 1. Clone the repo
git clone https://github.com/lekejo/edelphi.git
cd edelphi

# 2. Copy the local env file
cp .env.local .env

# 3. Create a placeholder Google service account file
echo '{}' > google-service-account.json

# 4. Build and start all services
docker compose up --build
```

### Access
| Service | URL |
|---|---|
| eDelphi App | http://localhost:8081 |
| Keycloak Admin | http://localhost:8080 |
| MySQL | localhost:3306 |

### Stop
```bash
docker compose down
```

### Full reset (wipes database)
```bash
docker compose down -v
```

---

## Railway Deployment

### Prerequisites
- A [Railway account](https://railway.app) (free tier available)
- Your GitHub repo connected to Railway

### Step 1 — Create a new Railway project
1. Go to [railway.app/new](https://railway.app/new)
2. Choose **Deploy from GitHub repo**
3. Select `lekejo/edelphi`
4. Railway will detect the Dockerfile automatically

### Step 2 — Add MySQL database
1. In your Railway project, click **+ New**
2. Choose **Database → MySQL**
3. Railway creates a managed MySQL instance
4. Click on the MySQL service → **Connect** tab → copy the connection variables

### Step 3 — Deploy Keycloak
1. In your Railway project, click **+ New → Docker Image**
2. Image: `quay.io/keycloak/keycloak:21.1`
3. Start command: `start-dev`
4. Set these environment variables on the Keycloak service:
   ```
   KC_DB=mysql
   KC_DB_URL=jdbc:mysql://YOUR_MYSQL_HOST:YOUR_MYSQL_PORT/keycloak?useSSL=false&allowPublicKeyRetrieval=true
   KC_DB_USERNAME=YOUR_MYSQL_USER
   KC_DB_PASSWORD=YOUR_MYSQL_PASSWORD
   KC_PROXY_HEADERS=xforwarded
   ```
5. Expose port `8080` and note the public Railway domain assigned to Keycloak

### Step 4 — Set environment variables on the eDelphi service
Open the `.env.railway` file in this repo as a reference.
In Railway → your eDelphi service → **Variables**, add each variable.

Key values to fill in:
- `HOST` = your Railway app domain (e.g. `edelphi-production.up.railway.app`)
- `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USERNAME`, `DB_PASSWORD` = from the MySQL plugin Connect tab
- `KEYCLOAK_URL` = the public Railway domain of your Keycloak service
- `KEYCLOAK_SECRET` = client secret from Keycloak admin console
- `KUBERNETES_NAMESPACE` = leave **empty**

### Step 5 — Deploy
Railway auto-deploys on every push to `master`. Trigger a manual deploy from the Railway dashboard if needed.

### Notes
- Railway handles HTTPS automatically — no nginx needed
- The `nginx` service from the original `docker-compose.yml` has been **removed** — it was only needed for local SSL and is not required on Railway
- The `google-service-account.json` file is mounted as a volume locally. On Railway, you can upload it as a file volume in Railway's **Volumes** feature or encode it as a base64 environment variable

---

## Environment Files Reference

| File | Purpose |
|---|---|
| `.env.local` | Copy to `.env` for local macOS Docker development |
| `.env.railway` | Reference for variables to set in Railway dashboard |
| `.env.example` | Generic reference with all variable names |
