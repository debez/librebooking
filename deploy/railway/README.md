# Railway provisioning runbook

Per-tenant deployment of LibreBooking on Railway. One Railway project per tenant.

## Per-tenant resources

| Resource | Type | Notes |
|---|---|---|
| Project | Railway project | Named `tenant-<slug>` |
| Service: `librebooking-web` | Built from this fork's `deploy/Dockerfile` | 5 GB volume at `/var/www/html/Web/uploads` |
| Service: `mysql` | Railway MySQL template | 5 GB volume at `/var/lib/mysql` |
| Custom domain | `<slug>.<your-domain>` | Set on `librebooking-web` service |

## Prerequisites

- Railway Hobby plan or higher (for 5 GB volumes)
- Custom domain DNS access to add a CNAME

## Steps

### 1. Create Railway project

Empty project, named `tenant-<slug>`.

### 2. Add MySQL service

- **+ Create** → **Database** → **MySQL**
- Service Settings → **Region** → EU West (Amsterdam)
- Verify volume auto-mounted at `/var/lib/mysql` (5 GB)
- Optional: edit `MYSQL_DATABASE` variable to a semantic name (e.g. `librebooking`)
- Settings → Networking → **DO NOT enable the public TCP proxy** (keep DB private)
- Rename the service to `mysql` (lowercase) so env var refs work consistently across tenants

### 3. Add LibreBooking web service

- **+ Create** → **Empty Service**
- Settings → Source → **GitHub Repo** → connect `debez/librebooking`
- **Dockerfile path**: `deploy/Dockerfile`
- **Branch**: `develop`
- Region → EU West (Amsterdam)
- Rename service to `librebooking-web`
- Volumes tab → **+ New Volume** → mount `/var/www/html/Web/uploads`, size 5 GB

### 4. Configure environment variables

On the `librebooking-web` service, Variables tab:

```
LB_DATABASE_NAME=<the value of MYSQL_DATABASE on the mysql service>
LB_DATABASE_USER=${{mysql.MYSQL_USER}}
LB_DATABASE_PASSWORD=${{mysql.MYSQL_PASSWORD}}
LB_DATABASE_HOSTSPEC=${{mysql.RAILWAY_PRIVATE_DOMAIN}}
LB_INSTALL_PASSWORD=<generate strong random password — used once for installer>
LB_DEFAULT_TIMEZONE=Europe/Paris
LB_LOGGING_LEVEL=debug
LB_LOGGING_SQL=false
```

Hover over each `${{mysql.*}}` value in the UI to confirm it resolves to the actual value (not the literal string). If blank, the MySQL service name or var name is wrong.

### 5. Custom domain

- `librebooking-web` → Settings → Networking → Public Networking → **Generate Domain** (gives a temporary `*.up.railway.app` URL — useful for the initial install before DNS is wired)
- Then add **Custom Domain** → `<slug>.<your-domain>`
- Add CNAME at your DNS registrar pointing to Railway's provided target
- Railway provisions Let's Encrypt automatically

### 6. Run the LibreBooking installer

- Visit the live URL → the installer prompts for the `LB_INSTALL_PASSWORD`
- Step through the installer (DB connection check, schema creation, admin account)
- After install completes, **change `LB_LOGGING_LEVEL` from `debug` to `none`** to reduce log volume

### 7. Smoke test

- Log in as admin, create a resource, create a booking
- Trigger a redeploy of `librebooking-web` from the dashboard, confirm uploads + DB persist
- Configure SMTP env vars (separate, see LibreBooking docs) and trigger a confirmation email

## Pinned image / Dockerfile state

See `CHANGELOG.md` for the current pinned upstream image tag and any active workarounds.

## Upstream sync

Run monthly or when upstream announces a security patch:

```bash
cd C:\Users\bezen\librebooking
git fetch upstream
git checkout develop
git merge upstream/develop      # should be clean — only deploy/ files diverge
git push origin develop
```

Railway auto-deploys on push. Test on one tenant first, then roll forward to others.
