# v1.4.1 - 2026-08-08

Quality, reliability, and observability release on top of 1.4.0. Same brand, so no
migration is needed: patch both the server and the agent in place.

## Packages

| File | Size | Description |
|---|---|---|
| `ScriptWizard-1.4.1-Setup.zip` | ~60 MB | **Fresh server install.** Includes the offline Python cp312 wheelhouse, no internet required. Run `setup\install.ps1`. A detailed `setup\INSTALL.html` walkthrough (pre-steps, install, post-steps) is bundled. |
| `ScriptWizard-1.4.1-patch.zip` | ~0.6 MB | **Same-brand server patch** for an install that is already Script Wizard. In-place code update only. Run `setup\update.ps1` (add `-Deps` if dependencies changed). |
| `ScriptWizard-Agent-1.4.1.zip` | ~35 MB | **Full agent installer** for a runner device. Bundles its own offline wheelhouse. Run `install.ps1`. |
| `ScriptWizard-Agent-1.4.1-patch.zip` | ~110 KB | **Agent code-only patch** for an already-installed agent. Stops the service, overlays new source, restarts. Run `update-agent.ps1` (add `-Deps` if dependencies changed). |

> Upgrading from 1.4.0? Apply the server patch with `setup\update.ps1` and the agent patch
> with `update-agent.ps1`. No migrate step is required (migrate only existed for the
> Script Manager to Script Wizard rename in 1.4.0; if you still run a legacy Script Manager
> box, use the migrate zip from v1.4.0 first, then this patch).

Password: shared separately.

## What's new in 1.4.1

**Added**
- **Anomaly detection & alerting** (admin): a background engine watches automated (scheduled / API) jobs for consecutive failures, failure-rate and runtime/duration spikes, queue backlog, worst queue-wait, and stalls. Each detector has its own thresholds on the Settings page; alerts go out as in-app notices + branded emails with a cooldown and auto-resolve.
- **Re-run with edited inputs**: a single Re-run button reopens a job's inputs prefilled (the dialog title shows the script id); tweak a value and run, or run as-is.
- **Email blasts to typed addresses**: Send Notification can target a raw comma/semicolon list of email addresses, in addition to system/team/user/role.
- **Fresh-install HTML guide** bundled in the Setup package (`setup\INSTALL.html`).

**Changed**
- **Devices list**: clearer connected / disconnected status and an idle / busy Activity column (dropped the attended/unattended tags).
- **Uniform list density** across Run History, Audit, and the Overview (Recent jobs + Needs attention); Recent jobs now shows the script name.
- **Script-card polish** (cursor-follow dot reveal, gentle hover lift, lighter blue hover border) and sensible random icon/color defaults for new folders.
- **Run dialog** decluttered (removed the inline "Copy as API call" button; the REST snippet still lives with the API docs).

**Fixed / hardened**
- **Runner and server resilience**: reconnect grace + pending re-attach on transient drops, in-flight result de-duplication, backoff reset with jitter, graceful agent shutdown, socket close on delete, interrupted desktop-job recovery, and cleaner deploy/interrupt handling across drops and server restarts.
- **Input-variable injection** fixed for Python and PowerShell scripts.
- **Schedule/cron edge cases** (OR terms, leap-year cap, weekly guard), the async-job callback-delivered flag, a child-process output-file leak, a larger 16 MB script-output cap, and script test/registration guards.
