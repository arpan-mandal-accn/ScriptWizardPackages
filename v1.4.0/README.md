# v1.4.0 — 2026-08-05

Rebrand from Script Manager to Script Wizard + desktop runner agent.

> **Rebuilt 2026-08-05** — server packages refreshed so the on-device run-window shows the
> **script name** instead of the job UUID (the server now sends `script_name` in the deploy
> frame). Server-side only; the agent needs no update. A dedicated same-brand
> `ScriptWizard-1.4.0-patch.zip` was added for in-place updates of existing Script Wizard boxes.

## Packages

| File | Size | Description |
|---|---|---|
| `ScriptWizard-1.4.0-Setup.zip` | ~60 MB | **Fresh server install.** Includes the offline Python cp312 wheelhouse — no internet required. Run `setup\install.ps1`. |
| `ScriptWizard-1.4.0-migrate.zip` | ~0.6 MB | **Migrate** an existing Script Manager install (1.4.0 or 1.2.8) **to** Script Wizard 1.4.0. Run `setup\migrate.ps1`. Renames folders, tasks, firewall, IIS site, and the SQL database, then applies the latest code. |
| `ScriptWizard-1.4.0-patch.zip` | ~0.6 MB | **Same-brand server patch** for an install that is *already* Script Wizard 1.4.0. In-place code update only (no rebrand). Run `setup\update.ps1` (add `-Deps` if dependencies changed). |
| `ScriptWizard-Agent-1.4.0.zip` | ~35 MB | **Full agent installer** for a runner device. Bundles its own offline wheelhouse. Run `install.ps1`. |
| `ScriptWizard-Agent-1.4.0-patch.zip` | ~110 KB | **Agent code-only patch** for an already-installed agent. Stops the service, overlays new source files, restarts. Run `update-agent.ps1` (add `-Deps` if dependencies changed). |

> **Server vs. agent, migrate vs. patch:** use **migrate** to convert a *Script Manager* box to Script Wizard; use **patch** to update a box that is *already* Script Wizard. Both carry the same application code — only the lifecycle scripts differ.

Password: shared separately.

## What's new in 1.4.0

- **Desktop runner agent** — separate Windows service on runner devices; executes scripts in a real interactive session dispatched over an outbound WebSocket. No Task Scheduler.
- **RDP deployment** — loopback FreeRDP for on-demand session creation (A360-style). ForceAutoLogon re-arms after every sign-out.
- **Run window** — native Script Wizard icon, script name + profile badge, proper taskbar minimize, clean Stop/close-as-cancel.
- **NoSignOut boot task** — hides the Sign Out UI on runner devices; survives Group Policy cycles.
- **Input variable injection** — desktop scripts now correctly receive all declared input variables.
- **Users: email column** — stored email shown in user list; email override setting for non-prod environments.
- **API key validator** — validate a full key string from the Keys page.
- **Rebrand** — all references updated from Script Manager to Script Wizard (UI, docs, folder names, DB name).
- **Migration hardening** — stale-target detection, preflight integrity check, DB backup step, repair script for half-done migrations.
