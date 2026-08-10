<div align="center">

# 🪄 Script Wizard

### Packages &amp; Distribution

**The on-prem Windows automation platform** - author, schedule, and run
Python · PowerShell · VBScript at scale, from a clean dashboard, with team-scoped
access, Active Directory sign-in, and desktop-automation runners.

<br/>

![Version](https://img.shields.io/badge/version-1.4.2-6D28D9?style=for-the-badge)
![Platform](https://img.shields.io/badge/platform-Windows-0078D6?style=for-the-badge&logo=windows&logoColor=white)
![Python](https://img.shields.io/badge/python-3.11%20|%203.12-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Packages](https://img.shields.io/badge/packages-AES--256-16A34A?style=for-the-badge&logo=gnuprivacyguard&logoColor=white)

![FastAPI](https://img.shields.io/badge/FastAPI-async-009688?style=flat-square&logo=fastapi&logoColor=white)
![SQL Server](https://img.shields.io/badge/SQL%20Server-aioodbc-CC2927?style=flat-square&logo=microsoftsqlserver&logoColor=white)
![WebSocket](https://img.shields.io/badge/runners-WebSocket-010101?style=flat-square&logo=socketdotio&logoColor=white)
![IIS](https://img.shields.io/badge/HTTPS-IIS%20reverse%20proxy-5C2D91?style=flat-square&logo=microsoft&logoColor=white)
![Distribution](https://img.shields.io/badge/distribution-internal%20only-EA580C?style=flat-square)

</div>

---

> 🔐 **All archives are AES-256 encrypted.** [Contact the maintainer](#-contact) for the distribution password.

<br/>

## 📖 About

**Script Wizard** is a self-hosted automation control platform for enterprise Windows
environments. Teams register, organize, run, schedule, and monitor automations from a
web dashboard or a REST API - with role-based access, AD sign-in, failure alerting, and
full run history. Version 1.4 adds **desktop-automation runners**: dedicated devices that
execute GUI/desktop scripts in a real interactive session, dispatched over a secure
outbound WebSocket.

<table>
<tr>
<td width="50%" valign="top">

### 🧩 Automation
- 📂 Scripts in a **nested folder tree** with a Monaco editor
- ⚡ Priority queue + worker pool, timeouts, retries, artifacts
- 🖥️ **Desktop runners** - Normal (console auto-login) &amp; RDP
- ⏰ Cron / interval / daily / weekly schedules, per-schedule **timezones**

</td>
<td width="50%" valign="top">

### 🔐 Security &amp; access
- 👥 **Teams &amp; RBAC** - admin / editor / viewer, folder-scoped
- 🪪 **Active Directory** sign-in, provisioning, group→role sync
- 🔑 Team-scoped **API keys** + HMAC-signed callbacks
- 📨 Notifications, email blasts, first-login T&amp;C gate

</td>
</tr>
</table>

<br/>

## 🧱 Architecture

| Layer | Stack |
|:--|:--|
| 🚀 **API / server** | FastAPI (async), single uvicorn process |
| 🗄️ **Database** | SQL Server via `aioodbc` - database `ScriptWizard` |
| 🎨 **Frontend** | Vanilla JS + HTML, Monaco editor |
| 🔑 **Auth** | Cookie sessions + AD/LDAP + API keys |
| 🖥️ **Desktop runners** | Outbound-WebSocket agent - LocalSystem service, Win32 token APIs |
| 🧬 **Schema** | Model-driven, **additive-only** - never drops data |

<br/>

## 📦 Downloads

> Latest release: **[v1.4.2](v1.4.2/)** · released 2026-08-10

| Package | Size | Use it to |
|:--|--:|:--|
| 🟣 **`ScriptWizard-1.4.2-Setup.zip`** | ~60 MB | **Install a fresh server** - bundles the Python 3.12 wheelhouse, fully offline (guide in `setup\INSTALL.html`) |
| 🟡 **`ScriptWizard-1.4.2-patch.zip`** | ~0.6 MB | **Update an existing Script Wizard server** - same-brand, in-place code patch |
| 🟢 **`ScriptWizard-Agent-1.4.2.zip`** | ~39 MB | **Install a desktop runner** on a device - bundles its own offline wheelhouse |
| ⚪ **`ScriptWizard-Agent-1.4.2-patch.zip`** | ~3.9 MB | **Update an installed agent** - code + FreeRDP, no reinstall, no wheelhouse |

> Converting a legacy **Script Manager** box? Run the one-time `migrate.ps1` from
> **[v1.4.0](v1.4.0/)** (`ScriptWizard-1.4.0-migrate.zip`) first, then apply the 1.4.2 patch.
> 1.4.2 is same-brand, so it ships no migrate zip.

<br/>

## 🚀 Quick start

<details open>
<summary><b>🟣 Fresh server install</b></summary>

```powershell
# Extract ScriptWizard-1.4.2-Setup.zip (7-Zip → enter password), then elevated:
cd ScriptWizard
.\setup\install.ps1
```
Installs to `C:\Program Files\Script Wizard` + `C:\ProgramData\Script Wizard`, registers
the `ScriptWizard` service + SYSTEM watchdog, and optionally sets up HTTPS via IIS.
Dashboard: `https://<host>/dashboard/`.
</details>

<details>
<summary><b>🔵 Migrate from Script Manager</b></summary>

```powershell
cd ScriptWizard
.\setup\migrate.ps1
```
Renames folders, tasks, firewall, IIS site, and the SQL database, then applies the 1.4.0
update. Works from Script Manager 1.4.0 **and** 1.2.8. Snapshot the machine first.
</details>

<details>
<summary><b>🟡 Patch an existing Script Wizard server</b></summary>

```powershell
# Already running Script Wizard? Extract ScriptWizard-1.4.2-patch.zip, then elevated:
cd ScriptWizard
.\setup\update.ps1 -Deps
```
In-place code overlay only - **no rebrand/rename**. Stops the service, overwrites the app
code (nothing deleted), runs the additive schema migration, re-registers the SYSTEM
watchdog, restarts. Drop `-Deps` if `requirements.txt` is unchanged.
</details>

<details>
<summary><b>🟢 Install a desktop runner</b></summary>

```powershell
cd ScriptWizard-Agent
.\install.ps1 -ServerUrl https://your-server -EnrollToken rre_xxx -ProfileName prod
```
Installs the `ScriptWizardAgent` LocalSystem service (auto-start, auto-restart). The device
appears **online** in the server's Devices page within seconds.
</details>

<details>
<summary><b>⚪ Update an installed agent (no reinstall)</b></summary>

```powershell
cd ScriptWizard-Agent-patch
.\update-agent.ps1          # add -Deps if requirements.txt changed
```
Stops the service, overlays only the Python source (vendor binaries untouched), restarts.
Under 30 seconds.
</details>

<br/>

## 🗂️ Extracting

Windows Explorer can't open AES-256 zips. Use **[7-Zip](https://www.7-zip.org)**:

```text
Right-click the zip → 7-Zip → Extract Here → enter the password
```

<br/>

## 💻 Platform requirements

| Component | Requirement |
|:--|:--|
| 🪟 OS | Windows Server 2016+ or Windows 10/11 (64-bit) |
| 🐍 Python | 3.11 / 3.12 (bundled offline in Setup) |
| 🗄️ Database | SQL Server (any edition) + ODBC Driver 17/18 |
| 🌐 IIS | Optional - only for HTTPS (installers bundled) |
| 🖥️ Runner device | Windows 10/11, Python 3.9-3.13, pywin32 |
| 🖧 FreeRDP | Only for the RDP deployment type |

<br/>

---

<div align="center">

## 📬 Contact

**This repository is for internal distribution only.**
Packages are encrypted and not intended for public use.

Reach out for the zip password or any deployment questions:

**Arpan Mandal**
[![Email](https://img.shields.io/badge/arpan.mandal@accenture.com-D14836?style=for-the-badge&logo=gmail&logoColor=white)](mailto:arpan.mandal@accenture.com)

<sub>Script Wizard 1.4.2 · Windows / on-prem · FastAPI + SQLAlchemy (SQL Server) + vanilla-JS dashboard</sub>

</div>
