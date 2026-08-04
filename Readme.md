# 🪄 Script Wizard 1.4.0

> **On-prem Windows automation platform** — author, schedule, and run Python / PowerShell / VBScript at scale, with a clean dashboard, team-scoped access, and now **desktop-automation runners**.

This is the first release under the **Script Wizard** name (formerly *Script Manager*). It bundles everything needed to **install fresh**, **migrate an existing install**, or **add a desktop runner** — fully offline.

---

## 📦 Downloads

| Asset | Size | Use it to… |
|---|---:|---|
| **`ScriptWizard-1.4.0-Setup.zip`** | ~61 MB | **Install fresh** *or* **migrate** an existing box (bundles the Python 3.12 wheelhouse → works **offline**) |
| **`ScriptWizard-1.4.0-migrate.zip`** | ~0.6 MB | Lightweight **migrate / patch** for boxes that have (proxied) internet |
| **`ScriptWizard-Agent-1.4.0.zip`** | ~35 MB | Install the **desktop-runner agent** on a runner device (bundles its own offline wheelhouse) |

> 🔐 All archives are **AES-256 encrypted** — enter the distribution password (shared separately) when extracting.

---

## 🚀 Quick start

**Fresh install** (elevated PowerShell, inside the extracted `Setup` package):
```powershell
powershell -ExecutionPolicy Bypass -File setup\install.ps1
```

**Migrate an existing Script Manager box → Script Wizard** (extract the `Setup` zip — it's offline-capable):
```powershell
powershell -ExecutionPolicy Bypass -File setup\migrate.ps1
```
`migrate.ps1` renames everything (folders, database, tasks, IIS, firewall, config paths), overlays the new code, repairs the schema, refreshes dependencies, and restarts — with a preflight check, confirmation gate, automatic DB backup, and end-to-end verification. **Snapshot the machine first.**

**Install a desktop runner** (on the runner device, elevated):
```powershell
powershell -ExecutionPolicy Bypass -File install.ps1 `
  -ServerUrl https://<your-server>:443 -EnrollToken <token> -ProfileName prod
```

---

## 🆕 What's new

### 🏷️ Rebrand — Script Manager → Script Wizard
- Full rename across the app, dashboard, services, database, and installers.
- One-command **migrate + patch** (`migrate.ps1`) for existing installs — brand paths (incl. `LOG_DIR`), the SQL database, scheduled tasks, IIS site, and firewall rules are all repointed; stored settings (T&C, welcome / alert emails) are rebranded in place.

### 🖥️ Desktop-automation runners
- Run GUI / desktop scripts in a **real interactive Windows session** on a dedicated **runner device**, dispatched over an **outbound WebSocket** (no inbound ports).
- Single-user **console auto-login** handled entirely by the agent (Windows `AutoAdminLogon` + CyberArk) — **no credential-provider DLL**, no Task Scheduler.
- Admin-only **"Use Desktop"** per script; A360-style device model with per-device CyberArk (server-side).

### 👥 Users & email
- **Email column** on the Users page — populated from Active Directory (sync + provisioning), searchable, alphabetically sorted.
- **Email address override** (Settings) — for dev/stage, rewrite outbound recipient addresses to a real domain (`<user>@yourdomain.com`) so non-prod never mails dead/dummy addresses. Sending-only; the stored/displayed email is untouched.

### 🔑 Security & administration
- **API key validator** — paste a key on the API Keys page to identify it and see its status (active / revoked / expired) + full details.
- SYSTEM **watchdog self-heals** the run-as account's local-admin membership.

### ⏰ Schedules
- **Per-schedule timezones** (all schedule kinds, incl. cron) — requires the bundled `tzdata`; an unknown zone is rejected rather than silently run in server-local time.
- Editor: hierarchical **folder-tree picker**.

<details>
<summary><b>Full changelog & database changes</b></summary>

**New tables (additive):** `desktop_runners`, `runner_enrollments`.
**New columns (additive):** `admin_users.email`, desktop flags on `scripts`, and more.

The schema step is **model-driven and additive-only** — it creates any missing tables and `ALTER-ADD`s any missing columns/indexes. **No table or column is ever dropped**, so upgrades from **1.2.x → 1.4.0** are safe.

Also includes a batch of production-hardening fixes: WebSocket compression drop-loop, agent worker-hang, a stored-XSS in the schedules table, enrollment double-use race, and dashboard responsiveness under long-running jobs.

</details>

---

## 🔧 Requirements

- **Windows 10 / 11 or Windows Server**, administrator rights.
- **Python 3.12** (the installer auto-detects / installs via winget; the offline wheelhouse targets **cp312**).
- **SQL Server** reachable, with a database + login (Mixed-Mode auth).
- **ODBC Driver 17 or 18 for SQL Server**.
- The desktop runner is a **separate device** — see `INSTALL.md` inside the agent package.

---

## ⚠️ Upgrade / migration notes

- **From Script Manager (1.2.x or 1.4.0):** use `migrate.ps1` from the **Setup** package. It handles the full brand rename **and** the version jump (creating the two new runner tables + new columns) in one run.
- **Dependencies are always refreshed** during migration (1.4.0 adds `tzdata`). The Setup package installs them **offline** from the bundled cp312 wheelhouse — no internet needed.
- Always **snapshot the machine** first; the folder + database rename is effectively one-way (an automatic DB backup is taken as a backstop).

---

<sub>Script Wizard 1.4.0 · Windows / on-prem · FastAPI + SQLAlchemy (SQL Server) + a vanilla-JS dashboard.</sub>
