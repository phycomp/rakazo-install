# Rakazo INSTALL

Rakazo is an open-source platform for persistent AI teammates with memory,
routines, and their own computers (browser, terminal, files, graphical desktop).
This guide documents end-to-end deployment on Railway using the official template.

## Prerequisites

- A Railway account (sign up at https://railway.com)
- (Optional) OpenRouter API Key (https://openrouter.ai) for out-of-the-box model access
- (Optional) Sandbox provider API Key (E2B, Daytona, or Box) for bot computer capabilities

## 1. Open the Railway Template

- Template page: https://railway.com/deploy/rakazo
- Deploy wizard: https://railway.com/new/template/rakazo

## 2. Sign In to Railway

Railway supports three login methods:

- Continue with GitHub (recommended — also grants repo access for template sources)
- Continue with Google
- Log in using email

## 3. Review the Template

The template deploys four services:

| Service | Image | Role |
|---------|-------|------|
| `web` | `ghcr.io/elie222/rakazo/app:v0.3.3` | Public web app (Vite preview, port 5173) |
| `api` | `monotykamary/railway-template-rakazo` | Hono API server (port from PORT env) |
| `worker` | `monotykamary/railway-template-rakazo` | Graphile Worker (routines, wakeups) |
| `postgres` | `railwayapp-templates/postgres-ssl:18` | PostgreSQL 18 datastore |

## 4. Configure Environment Variables

### Required (auto-generated — do not edit)
- `BETTER_AUTH_SECRET` — signing secret for auth sessions
- `ENCRYPTION_KEY` — AES key for stored credentials (never regenerate after first deploy)
- `SCREEN_PROXY_SECRET` — shared signing secret for browser-screen capabilities
- `POSTGRES_PASSWORD` — auto-generated Postgres password
- `DATABASE_URL` — auto-wired Postgres connection string

### Optional but Recommended
- `OPENROUTER_API_KEY` — model key so bots work out of the box
- `SANDBOX_PROVIDER` — `e2b`, `daytona`, `box`, or `none` (default: `e2b`)
- `E2B_API_KEY` / `DAYTONA_API_KEY` / `BOX_API_KEY` — matching key for the chosen provider
- `SIGNUP_ALLOWLIST` — comma-separated emails/domains allowed to sign up
- `SMTP_URL` — optional SMTP URL for password recovery
- `EMAIL_FROM` — optional From header for outgoing email
- `COMPOSIO_API_KEY` — optional Composio key for managed plugin catalog

### Auto-wired (do not edit)
- `API_URL`, `WEB_ORIGIN`, `BETTER_AUTH_URL`, `RAKAZO_HOST`
- `API_PROXY_TARGET`
- `DATABASE_URL`
- `DATA_DIR`

## Configuration Files (Examples)

### Railway environment variables (dashboard)

Set these in the Railway service variables (api + worker where noted):

```env
# Required (auto-generated — do not edit)
BETTER_AUTH_SECRET=<auto>
ENCRYPTION_KEY=<auto>
SCREEN_PROXY_SECRET=<auto>
POSTGRES_PASSWORD=<auto>
DATABASE_URL=<auto-wired>

# Model provider (recommended)
OPENROUTER_API_KEY=sk-or-...

# Sandbox provider (choose one)
SANDBOX_PROVIDER=e2b          # e2b | daytona | box | none
E2B_API_KEY=e2b_...
# DAYTONA_API_KEY=...
# BOX_API_KEY=...

# Access control (set before first API start)
SIGNUP_ALLOWLIST=you@example.com

# Optional email
SMTP_URL=smtp://user:pass@smtp.example.com:587
EMAIL_FROM=noreply@example.com

# Optional integrations
COMPOSIO_API_KEY=...
```

### OpenRouter

- Sign up at https://openrouter.ai and create an API key.
- Set `OPENROUTER_API_KEY` in Railway (or in `.env` for Docker Compose).
- Bots will use OpenRouter models out of the box; users can also connect per-user models in the UI.

### Box (sandbox provider)

If `SANDBOX_PROVIDER=box`:

```env
SANDBOX_PROVIDER=box
BOX_API_KEY=<your-box-api-key>
```

Without a sandbox key, Rakazo still boots and bots can chat, but computer tools (browser, terminal, files) are unavailable.

### Docker Compose (self-hosting)

`infra/compose/.env` example:

```env
POSTGRES_PASSWORD=change-me
OPENROUTER_API_KEY=sk-or-...
SANDBOX_PROVIDER=e2b
E2B_API_KEY=e2b_...
SIGNUP_ALLOWLIST=you@example.com
```

Then run:

```bash
docker compose -f docker-compose.prod.yml up -d --build
```

### Railway CLI (optional)

```bash
# Install Railway CLI
npm i -g @railway/cli

# Login
railway login

# Link to your project
railway link

# Set a variable
railway variables set OPENROUTER_API_KEY=sk-or-...
```

## 5. Deploy

Click the **"Deploy"** button. Railway will:

1. Provision all four services
2. Run database migrations (`prisma migrate deploy`)
3. Health-check the API at `GET /health`
4. Assign a public domain to the web service

## 6. Post-Deploy Verification

1. Register the first user immediately — the first registered user becomes the deployment owner.
2. Access the web URL provided by Railway (e.g., `https://xxx.railway.app`).
3. Sign up with your email.
4. Verify the API health endpoint: `https://xxx.railway.app/health` should return OK.
5. (Optional) Test bot creation and a simple conversation.

## 7. Important Notes

- **ENCRYPTION_KEY**: Never regenerate after the first start — stored credentials would become undecryptable.
- **First user is owner**: Register immediately after deploy to claim ownership.
- **Private deployment**: Set `SIGNUP_ALLOWLIST` before the first API start.
- **Sandbox provider**: Without a key, the product boots and bots chat, but computer tools are unavailable.
- **Railway trial**: Free tier provides $5.00 or 30 days. The free plan may not have enough resources to provision all 4 services. If you see "Free plan resource provision limit exceeded", upgrade to a paid plan (Hobby $5/mo or Pro) before deploying.
- **Resource requirements**: The template deploys 4 services. The free tier's resource limits may be insufficient. Consider the Hobby plan ($5/month) or higher.

## 8. Alternative: External Postgres (3 services)

If the free plan's 3-service limit is insufficient for all 4 services, use an external Postgres database.

Free external Postgres providers:
- [Neon](https://neon.tech) — free tier, 0.5 GB storage
- [Supabase](https://supabase.com) — free tier, 500 MB

Steps:
1. Create a database on the external provider
2. Get the connection string: `postgresql://user:pass@host:port/dbname?sslmode=require`
3. Deploy only `web`, `api`, `worker` on Railway (3 services)
4. Set `DATABASE_URL` in both `api` and `worker` to the external connection string
5. Ensure `ENCRYPTION_KEY` is shared between `api` and `worker`

Minimum resources:
| Service | Min RAM | Recommended |
|---------|---------|-------------|
| web (Vite) | 128 MB | 256 MB |
| api (Hono) | 256 MB | 512 MB |
| worker (Graphile) | 256 MB | 512 MB |

## 9. Alternative: Docker Compose Self-Hosting

For full control over resources, deploy on your own server:

```bash
git clone https://github.com/elie222/rakazo.git
cd rakazo/infra/compose
cp .env.example .env
# Edit .env: set POSTGRES_PASSWORD, OPENROUTER_API_KEY, etc.
docker compose -f docker-compose.prod.yml up -d --build
```

Requirements: Linux server, Docker + Docker Compose, 4 GB RAM / 2 CPU minimum (8 GB recommended).

## 10. Alternative: GitHub Codespaces as External Postgres

Use a GitHub Codespace to run Postgres as an external database for Railway.

Setup:
1. Go to https://github.com/phycomp/codespaces
2. Click "Code" → "Codespaces" → "Create codespace on main"
3. Choose 4+ cores, 8GB+ RAM machine
4. Wait for the setup script to complete (installs Rakazo + Postgres)
5. In the Codespace terminal, get the Postgres connection string:
   ```bash
   echo "postgresql://postgres:password@localhost:5432/rakazo?sslmode=disable"
   ```
6. Make port 5432 public in the Codespace "Ports" tab
7. Use the forwarded URL as `DATABASE_URL` in Railway

Limitations:
- Codespaces sleep after 30 min of inactivity
- Codespaces are deleted after 30 days of inactivity
- Free tier: 60 hours/month for 2-core, 120 hours for 4-core
- Not suitable for production; use for development/testing only

Recommended: Use Neon (https://neon.tech) or Supabase (https://supabase.com) for a permanent free Postgres database instead.

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Free plan resource limit exceeded | Railway free tier cannot provision 4 services; upgrade to Hobby ($5/mo) or Pro plan |
| API container exits immediately | Check `POSTGRES_PASSWORD` is set and Postgres healthcheck passes |
| Bots can't use computer tools | Set `SANDBOX_PROVIDER` and matching API key |
| Can't sign up | Check `SIGNUPS_ENABLED` is true and `SIGNUP_ALLOWLIST` if set |
| Stored credentials undecryptable | ENCRYPTION_KEY was regenerated — restore from backup |

## Quick Reference

```
Template URL:  https://railway.com/deploy/rakazo
Deploy URL:    https://railway.com/new/template/rakazo
Station:       https://station.railway.com/all-templates/49594aec-af0d-44fd-b143-ce05be3dd1e9
Source:        https://github.com/elie222/rakazo
Image:         ghcr.io/elie222/rakazo/app:v0.3.3
```
EOF
echo "written"; ls -l /home/sam/INSTALL.md
---

## Connecting from Your Computer

### Cloud Box (web interface)

Open in your browser:

```
https://box.simular.ai/box/cb-fbad7a7650b1/live
```

Log in with GitHub or Google to access the XFCE desktop (1280x800).

### Rakazo (Railway public URL)

After deployment, Railway assigns a public domain:

```
https://<your-project>.railway.app
```

Verify health:

```bash
curl -s https://<your-project>.railway.app/health
```

### Local Services (optional)

If your computer also runs Paperclip or OpenMausBot:

- **Paperclip**: managed CLI at `~/.local/bin/paperclipai`, service `paperclipai.service`, API at `http://127.0.0.1:3100`
- **OpenMausBot**: MCP server at `http://127.0.0.1:8799`

Refer to `~/.agents/skills/paperclip-launch/SKILL.md` for full setup.

## Quick Links

| Service | URL |
|---|---|
| Rakazo (deployed) | `https://<your-project>.railway.app` |
| Rakazo health | `https://<your-project>.railway.app/health` |
| Cloud Box | https://box.simular.ai/box/cb-fbad7a7650b1/live |
| Railway template | https://railway.com/deploy/rakazo |
| OpenRouter | https://openrouter.ai |
| Neon (free Postgres) | https://neon.tech |
| Supabase (free Postgres) | https://supabase.com |
