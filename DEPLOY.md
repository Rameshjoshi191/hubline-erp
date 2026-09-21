# Put HUB LINE ERP online

Two very different goals — pick the one you need.

| Goal | Best option | Cost | Your data |
|---|---|---|---|
| **Show it to colleagues / try it out** | Render free web service (below, ~10 minutes, no coding) | Free | Demo data only. Wiped whenever it restarts |
| **Run the real company on it** | A server with a persistent disk (Render paid plan + disk, a small VPS, or Oracle Cloud "Always Free" VM) | From a few dollars/month (or free on Oracle) | Kept safely — with daily backups |

> **Why the split?** The system stores everything in one database file. Free web hosts sleep after a period of no use and **erase their disk when they restart**, so anything you type into a free instance eventually disappears. That is fine for a demo, but never enter real customers, invoices or bank details there.

---

## A. Free online demo on Render (about 10 minutes)

You need two free accounts: **GitHub** (to hold the files) and **Render** (to run them).

1. **Unzip** `hubline-nvocc-erp.zip` on your computer. Open the `nvocc-erp` folder.
2. **GitHub:** sign in at github.com → **New repository** → name it `hubline-erp` → keep it **Private** → *Create repository*. Click **uploading an existing file**, drag in everything inside the `nvocc-erp` folder (do **not** upload a `node_modules` or `data` folder if you have one), and press **Commit changes**. Make sure `render.yaml` is at the top level of the repository.
3. **Render:** sign in at render.com with your GitHub account → **New +** → **Blueprint** → choose the `hubline-erp` repository → **Apply**.
4. Wait 3–6 minutes while it builds. When it says *Live*, click the address at the top (like `https://hubline-erp.onrender.com`).
5. **Sign in.** In Render open the service → **Environment** → click the eye icon on `ADMIN_PASSWORD`. Username `admin`, password = that value. (The other demo users `ops`, `docs`, `accounts`, `sales`, `manager` all use `DEMO_PASSWORD`.)

Good to know: the first visit after 15 idle minutes takes about a minute to wake up. Share the link and the passwords only with people you trust. Render's terms and limits change, so check render.com/pricing for the current free-plan rules.

---

## B. Going live with real data

The application is the same; the difference is that the database file must live on a disk that survives restarts, and you must back it up.

**Option 1 — Render paid plan with a disk.** In your `render.yaml`, change `plan: free` to a paid plan (for example `starter`), delete the `SEED_DEMO`, `DEMO_PASSWORD` lines, and add:

```yaml
    disk:
      name: hubline-data
      mountPath: /var/data
      sizeGB: 1
    # and under envVars:
      - key: DATA_DIR
        value: /var/data
```

Keep `ADMIN_PASSWORD` (generated) for the first sign-in; you will be asked to choose your own password immediately.

**Option 2 — Any Docker host or VPS** (Hetzner, DigitalOcean, Oracle Cloud Always Free, your own PC or server):

```bash
docker build -t hubline-erp .
docker run -d --name hubline --restart unless-stopped \
  -p 127.0.0.1:3000:3000 -v hubline-data:/data \
  -e ADMIN_PASSWORD='choose-a-long-password' \
  -e TRUST_PROXY=1 -e COOKIE_SECURE=1 \
  hubline-erp
```

Then put HTTPS in front of port 3000 with Caddy or nginx (Caddy needs only: `erp.yourcompany.com { reverse_proxy 127.0.0.1:3000 }`). Without Docker: install Node 22, run `npm ci --omit=dev` and `npm start` under systemd/PM2, as described in the README.

**Backups (do this from day one).** *Setup → Company & settings → Download database backup* saves a complete copy of your data. Do it daily (or automate it on the server with `sqlite3 /data/nvocc.db ".backup '/backups/nvocc-$(date +%F).db'"`) and keep copies off the server. To restore, stop the app, replace `nvocc.db` with the backup (and delete any `nvocc.db-wal` / `nvocc.db-shm` files beside it), and start it again.

**Security checklist:** HTTPS only · a strong administrator password · one login per person (Setup → Users) · remove the demo users and demo data before entering real data · keep the server and Node updated.

---

## Environment variables used online

| Variable | Purpose |
|---|---|
| `PORT` | Set by most hosts automatically |
| `HOST` | `0.0.0.0` on hosted platforms (the Docker image sets this) |
| `TRUST_PROXY=1` | The host terminates HTTPS in front of the app |
| `COOKIE_SECURE=1` | Session cookie only over HTTPS |
| `ADMIN_PASSWORD` | First administrator password |
| `DATA_DIR` | Folder for the database — point it at the persistent disk |
| `SEED_DEMO=1` | Load demo data when the database is empty (demo instances only) |
| `DEMO_PASSWORD` | Password for the other demo users when `SEED_DEMO=1` |
| `DISABLE_JOBS=1` | Switch off the background jobs (daily e-mailed inventory report, daily schedule-link refresh) |

---

After the system is online, hand your team the [User & Administrator Guide](docs/USER_GUIDE.md) (Word copy: `docs/HUBLINE-ERP-User-Guide.docx`).

## After deploying version 0.2

* **E-mail:** sign in as admin, open *Setup → Company & settings*, fill in *Sharing by e-mail* (SMTP server, port, user, password, from address) and press *Send test e-mail*. Then fill in *Automation* to e-mail the daily inventory report.
* **Agents:** create a user with the role *Agent (external)*, choose the agent company and list its locations.
* **Background jobs** run inside the same process (checked every 10 minutes). On a host that sleeps when idle, the daily report is sent at the first check after the host wakes up past the chosen hour; a paid always-on plan sends it on time.
* **Vessel schedule links** are fetched from the server, so the server needs outbound internet access; addresses on private networks are refused.
