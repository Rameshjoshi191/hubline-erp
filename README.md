# HUB LINE ERP — NVOCC operations, agents & finance

A web-based ERP for an NVOCC / consolidator, with a modern interface (icon sidebar, dashboard charts, global search, light and dark themes, works on phones and tablets). Version 0.2 covers:

| Module | What it does |
|---|---|
| **Shipments / bookings** | Booking → confirmed (HBL number issued) → sailed → arrived → delivered → closed. FCL and LCL. Printable booking confirmation and house B/L. |
| **Master BL / consolidation** | Group LCL house shipments on one master B/L; sail and arrive them together (statuses and container events cascade). |
| **Container inventory** | Every box, its status, location and free-time (detention) clock, as-of-any-date. Event-based movement log, ISO 6346 check-digit validation, bulk add / bulk movement, **daily inventory report** (summary by type and location, movements, detention alerts, idle empties) with Excel export. |
| **Agents & agency agreements** | Agent master, agreements with validity, territory, exclusivity, payment terms, credit limit, editable clause text (standard clause set included), renewals, printable agreement. |
| **Agency commission** | Rules per agreement (per TEU / container / CBM / BL, % of freight, % of profit; by port pair and cargo type; min / max). Statements are previewed, adjusted, posted to the agent's account, and can be cancelled. |
| **Container enquiry & Excel updates** | One search box for container numbers (paste a list), booking / HBL / MBL, vessel or a place; a journey view for every box. The inventory and movements can be updated by uploading an Excel / CSV file: columns, container types and movement words are understood automatically, a preview shows exactly what will change, and re-loading the same file changes nothing. |
| **Agent portal** | Role *Agent (external)* tied to an agent company and a list of locations: agents update container movements at their own locations, view their own shipments and edit / complete their own bills of lading until sailing (edits are flagged for review), and send booking requests. |
| **Bills of lading tab** | Search, view, edit, download (PDF, Excel, combined PDF) and share every BL; filters for details missing and agent-edited. |
| **Vessel schedules** | Load schedules from Excel / CSV or from a carrier's web link (saved links, daily auto-refresh), date-change alerts that update bookings and master BLs in one click (or automatically), and a sailing picker on bookings and master BLs. |
| **MRG** | Month-wise freight rate, slot cost and margin per lane and container type, committed slots against shipped boxes, roll-forward from the previous month, fill from the tariff book, Excel load, year matrix. |
| **Sharing & automation** | Share any list as Excel or PDF, by the device's share sheet, or by e-mail (SMTP); action centre on the dashboard; the daily inventory report e-mailed to your list. |
| **Customers, agents, vendors, carriers** | One party master with role flags, credit terms, tax ids. |
| **Rates / tariffs** | Sell and buy rates by route, cargo type, container type and validity. "Fill from tariff" on invoices. |
| **Finance** | Invoices, debit notes, credit notes, vendor bills; receipts and payments with allocation to documents; netting (e.g. agent debit note against agent bill); multi-currency with base-currency reporting; shipment profit. |
| **Statement of account (SOA)** | Full-ledger (opening balance + running balance) or open-item statements for any customer, agent, vendor or carrier, with ageing. Print or export to Excel. |
| **Reports** | Ageing, shipment register and profit, container inventory, dashboard. |
| **Administration** | Users and seven roles (Admin, Management read-only, Operations, Documentation, Accounts, Sales, and the external Agent), company settings, ports, container types, charge codes, currencies and rates, full audit log. |

**Documentation:** the full **[User & Administrator Guide](docs/USER_GUIDE.md)** explains setup, roles, every workflow and the accounting rules. A Word copy is in `docs/HUBLINE-ERP-User-Guide.docx`.

## Install and run

Requires **Node.js 20 or newer** (22 recommended). No other services — data is stored in one SQLite file.

```bash
npm install
npm start
```

Open <http://127.0.0.1:3000>. On the very first start the console prints the administrator login:

```
username: admin
password: <random>     (you must change it at first sign-in)
```

To choose the first password yourself: `ADMIN_PASSWORD='something-long' npm start` (first start only).

Then, in **Setup**:

1. *Company & settings* — company name, address, tax registration number, base currency, default VAT, bank details, B/L terms, invoice footer.
2. *Users & roles* — create one user per team member.
3. *Ports*, *Container types*, *Charge codes*, *Currencies & rates* — a starter set is pre-loaded; edit to suit.
4. Add your carriers, agents and customers under *Customers & parties* / *Agents*, then tariffs under *Rates / tariffs*.

### Try it with demo data first

```bash
npm run seed:demo        # empty database only
npm start
```

Demo logins: `admin / admin123` (full access), and `ops`, `docs`, `accounts`, `sales`, `manager` — all with password `demo1234`.
The demo data creates parties, three agents with agreements and commission rules, tariffs, 40 containers, shipments in every status, a consolidation, invoices, bills, receipts, a netted debit note and posted commission statements. To go live, stop the server and delete `data/nvocc.db` (or set a different `DATA_DIR`), then start again.

### Run the automated tests

```bash
npm test
```

`npm test` runs about 200 end-to-end checks against throw-away databases: finance and allocation rules, SOA and ageing, container events and free time, commission maths, permissions and security headers (`test/e2e.js`), and Excel imports, enquiry, agent scoping, BLs, MRG, vessel schedules (including a local web-link server and the SSRF guard), Excel / PDF exports, e-mail attachments and dashboards (`test/features.js`).

## Configuration (environment variables)

| Variable | Default | Meaning |
|---|---|---|
| `PORT` | `3000` | HTTP port |
| `HOST` | `127.0.0.1` | Bind address. Use `0.0.0.0` to let other computers on your network connect. |
| `DATA_DIR` | `./data` | Folder holding the database (`nvocc.db`) |
| `DB_FILE` | `$DATA_DIR/nvocc.db` | Full path of the database file |
| `ADMIN_PASSWORD` | random | First administrator password (first start only) |
| `COOKIE_SECURE` | off | Set to `1` when served over HTTPS so the session cookie is HTTPS-only |
| `TRUST_PROXY` | off | Set to `1` when behind a reverse proxy (nginx, Caddy, IIS) so client IPs are read from `X-Forwarded-For` |
| `SEED_DEMO` | off | `1` = load the demo data on start if the database is empty (for throw-away demo instances only) |
| `DISABLE_JOBS` | off | `1` = do not start the background jobs (daily e-mailed inventory report and daily refresh of saved schedule links) |
| `DEMO_PASSWORD` | `demo1234` | Password for the demo users other than `admin` when the demo data is loaded (`ADMIN_PASSWORD` sets `admin`) |

## Hosting online

See **[DEPLOY.md](DEPLOY.md)** — a step-by-step guide to a free online demo (Render, `render.yaml` included) and to running it for real on a server with a persistent disk (a `Dockerfile` is included). Free hosts erase their disk on restart, so use them for demos only.

## Deploying for the team

* Run on a small server (Windows, Linux or a VM) and keep it running with a service manager (systemd, PM2, NSSM).
* **Put it behind HTTPS** (Caddy, nginx or IIS reverse proxy) and set `COOKIE_SECURE=1`, `TRUST_PROXY=1`, `HOST=127.0.0.1`. Do not expose plain HTTP to the internet.
* **Backups:** *Setup → Company & settings → Download database backup* (administrators) saves a consistent copy at any time. The whole system is `data/nvocc.db` (plus `-wal` / `-shm` files while running). Back it up daily with SQLite's online backup — `sqlite3 data/nvocc.db ".backup '/backups/nvocc-$(date +%F).db'"` — or copy the folder while the server is stopped. Test a restore.
* Sessions last 12 hours; repeated failed sign-ins are throttled (8 per 15 minutes per user and address).
* Sign-in passwords are stored as bcrypt hashes. Every create / change / cancel is recorded in the audit log (*Setup → Audit log*).

## Accounting conventions (read before going live)

* **Documents.** Invoices (INV) and debit notes (DN) are debits — the party owes you. Credit notes (CN), vendor bills (BILL) and commission statements (COMM) are credits. A receipt is a credit on the party; a payment is a debit. Documents are drafted, then **posted**; posted documents are not edited — cancel and re-issue, or issue a credit note. A document with allocations cannot be cancelled until they are removed.
* **Settlement.** Receipts settle debit documents, payments settle credit documents, and a debit can be netted against a credit for the same party and currency. Allocation is in the document's own currency; there is **no FX gain/loss posting** — settle each currency in its own currency.
* **Base currency** defaults to **AED**, with USD pegged at 3.6725. Other exchange rates in the demo are illustrative — enter your own. The base currency is locked once documents are posted.
* **VAT.** The default charge codes apply **5% UAE VAT to local charges and 0% to ocean freight** as a starting point. Confirm the treatment of each charge with your accountant / FTA guidance and edit the charge codes.
* **Shipment profit** = posted income minus posted costs on the shipment, converted to base currency, ex-VAT.
* **Commission** is calculated on shipments that are sailed or later, with ETD inside the statement period and the agreement dates, and not already on another statement. The most specific matching rule wins (port pair, then cargo type, then general).
* This system is an operational sub-ledger, not a general ledger. Export or integrate with your accounting package for statutory books.

## Templates that need your review

The standard agency-agreement clauses, the house B/L layout and its terms, and the invoice / SOA layouts are **starting-point templates**. Have your legal adviser review the agreement wording and check the B/L against your registered form before issuing to customers.

## Project layout

```
src/            server (Express 5, SQLite via better-sqlite3)
  db.js           schema, settings, numbering, audit
  logic.js        container status engine, ledger/SOA/ageing, commission engine
  services.js     document create / post / cancel
  routes/         REST API by module
  seed-demo.js    demo data loader
public/         single-page web app (vanilla ES modules, no build step)
test/e2e.js     end-to-end tests
docs/           user & administrator guide
```

## Roadmap ideas (not in Phase 1)

Quotations and rate requests · vessel / voyage schedule master · carrier booking and tracking integrations (EDI / API) · agent portal · multi-branch and multi-company · attachments and document storage · email delivery of invoices and SOAs · server-side PDF generation (today: browser *Print / save as PDF*) · dangerous-goods and reefer details · FX revaluation and accounting-package export.
