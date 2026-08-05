# Script Wizard — Packages

> Internal distribution repository for **Script Wizard** installation packages.
> All zips are AES-256 encrypted. [Contact the maintainer](#contact) to obtain the password.

---

## What is Script Wizard?

**Script Wizard** is a self-hosted Windows automation platform for enterprise environments.
It lets teams register, organize, run, schedule, and monitor **Python / PowerShell / VBScript**
automations from a web dashboard or a REST API — with team-scoped access control, Active Directory
sign-in, scheduling, failure alerting, and full run history.

### Key capabilities

- **Script library** — organized in a nested folder tree with a rich in-browser Monaco editor,
  input/output variable schemas, tags, timeouts, versions, and JSON import/export.
- **Execution engine** — a priority queue runs scripts as sandboxed subprocesses with concurrency
  limits, timeouts, retries, live log streaming, and artifact collection.
- **Desktop automation runners** — register separate Windows devices as runner agents. Admins flag
  a script "Use Desktop" and it is deployed over a secure outbound WebSocket to run in a real
  interactive session on the runner device. Supports **Normal (console auto-login)** and **RDP**
  deployment types, CyberArk credential integration, and live log/progress streaming back to the
  dashboard. Modeled on the Automation Anywhere A360 device model.
- **Scheduling** — cron, interval, daily, weekly, and monthly schedules with per-schedule
  IANA timezone support, retry config, and failure alerts.
- **Teams and RBAC** — `admin`, `editor`, and `viewer` roles. Non-admins see and run only scripts
  in their team's granted folders, enforced across the UI, public API, and nested script calls.
- **Active Directory** — AD/LDAP sign-in, user provisioning, group-to-role mapping, and a
  background reconciling sync.
- **Public REST API** — submit jobs, poll status, fetch logs and artifacts, and receive
  HMAC-signed callback webhooks. Team-scoped API keys.
- **Notifications and email** — admin in-app notices, email blasts, branded failure alerts,
  welcome emails, and a first-login Terms and Conditions gate.
- **Audit log, IP filtering, TLS** via a local IIS reverse proxy.

### Technology

| Layer | Stack |
|---|---|
| API / server | FastAPI (async), single uvicorn process |
| Database | SQL Server via aioodbc (named `ScriptWizard`) |
| Frontend | Vanilla JS + HTML, Monaco editor |
| Auth | Cookie sessions + AD/LDAP + API keys |
| Desktop runners | Outbound-WebSocket agent — LocalSystem Windows service, Win32 token APIs |
| Schema | Model-driven, additive-only — never drops data |

---

## Platform requirements

| Component | Requirement |
|---|---|
| OS | Windows Server 2016+ or Windows 10/11 (64-bit) |
| Python | 3.11 or 3.12 (bundled offline in the Setup package) |
| Database | SQL Server (any edition, including Express) + ODBC Driver 17 or 18 |
| IIS | Optional — required only for HTTPS (ARR + URL Rewrite, installers bundled) |
| Runner device | Windows 10/11 64-bit, Python 3.9–3.13, pywin32 |
| FreeRDP | Required only for RDP deployment type on runner devices |

---

## Packages

All packages are AES-256 encrypted. Use [7-Zip](https://www.7-zip.org) to extract.

### Current release

| Version | Release date |
|---|---|
| [v1.4.0](v1.4.0/) | 2026-08-05 |

### Package types

| Filename | Size | Use case |
|---|---|---|
| `ScriptWizard-<ver>-Setup.zip` | ~60 MB | Fresh server install. Includes offline Python wheelhouse — no internet required on the target machine. |
| `ScriptWizard-<ver>-migrate.zip` | ~1 MB | Migrate an existing Script Manager install to Script Wizard, or apply a same-brand server code patch. |
| `ScriptWizard-Agent-<ver>.zip` | ~5 MB | Fresh agent install on a runner device. Includes offline wheels for Python 3.9–3.13 + NSSM service host. |
| `ScriptWizard-Agent-<ver>-patch.zip` | ~100 KB | Agent code-only update for an already-installed agent. No reinstall, no wheelhouse — just the source files. |

---

## How to extract

Standard Windows Explorer cannot open AES-256 encrypted zips. Use **7-Zip** (free):

```
Right-click the zip -> 7-Zip -> Extract Here -> enter password when prompted
```

Or from an elevated PowerShell if 7-Zip is on PATH:

```powershell
7z x ScriptWizard-1.4.0-Setup.zip
```

---

## Quick start

### Fresh server install

```powershell
# Extract ScriptWizard-<ver>-Setup.zip, then from an elevated PowerShell:
cd ScriptWizard
.\setup\install.ps1
```

Installs to `C:\Program Files\Script Wizard` (code) and `C:\ProgramData\Script Wizard`
(config, data, logs, scripts). Registers the `ScriptWizard` Windows service and a SYSTEM
watchdog task. Optionally configures HTTPS via IIS.

The dashboard is available at `https://<host>/dashboard/` after install.

### Migrate from Script Manager (one-time)

```powershell
# Extract ScriptWizard-<ver>-migrate.zip, then from an elevated PowerShell:
cd ScriptWizard
.\setup\migrate.ps1
```

Renames folders, tasks, firewall rules, the IIS site, and the SQL database from
Script Manager to Script Wizard, then applies the 1.4.0 code update. Works from
both Script Manager 1.4.0 and 1.2.8.

### Same-brand server code patch (existing Script Wizard install)

```powershell
# Extract ScriptWizard-<ver>-migrate.zip, then from an elevated PowerShell:
cd ScriptWizard
.\setup\update.ps1
```

Overlays new code, runs additive schema migration, re-registers tasks, restarts the service.
Add `-Deps` if the requirements.txt changed between versions.

### Fresh agent install on a runner device

```powershell
# Extract ScriptWizard-Agent-<ver>.zip, then from an elevated PowerShell:
cd ScriptWizard-Agent
.\install.ps1 -ServerUrl https://your-server -EnrollToken rre_xxx -ProfileName prod
# Or run without arguments for an interactive prompt
.\install.ps1
```

Installs to `C:\Program Files\ScriptWizard Agent` and `C:\ProgramData\ScriptWizard Agent`.
Registers the `ScriptWizardAgent` Windows service (LocalSystem, auto-start, auto-restart).
The device appears online in the server's Devices page within a few seconds.

### Update an existing agent (no reinstall)

```powershell
# Extract ScriptWizard-Agent-<ver>-patch.zip, then from an elevated PowerShell:
cd ScriptWizard-Agent-patch
.\update-agent.ps1
# Add -Deps if the requirements.txt changed
.\update-agent.ps1 -Deps
```

Stops the service, overlays only the Python source files (vendor binaries are untouched),
restarts the service. Takes under 30 seconds.

---

## Contact

This repository is for **internal distribution only**. Packages are encrypted and not intended
for public use.

To obtain the zip password, or for any questions about deployment or the platform:

**Arpan Mandal** — [arpan.mandal@accenture.com](mailto:arpan.mandal@accenture.com)
