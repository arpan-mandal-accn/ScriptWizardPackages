# v1.4.2 - 2026-08-12

Security, reliability, and UX release on top of 1.4.1. Same brand, so no migration is needed:
patch both the server and the agent in place. Both the server AND the agent changed this cycle,
so update both.

## Packages

| File | Size | Description |
|---|---|---|
| `ScriptWizard-1.4.2-Setup.zip` | ~60 MB | **Fresh server install.** Includes the offline Python cp312 wheelhouse, no internet required. Run `setup\install.ps1`. A detailed `setup\INSTALL.html` walkthrough is bundled. |
| `ScriptWizard-1.4.2-patch.zip` | ~0.6 MB | **Same-brand server patch** for an install that is already Script Wizard. In-place code overlay only. Run `setup\update.ps1` (add `-Deps` if dependencies changed). |
| `ScriptWizard-Agent-1.4.2.zip` | ~39 MB | **Full agent installer** for a runner device. Bundles its own offline wheelhouse and the FreeRDP client. Run `install.ps1`. |
| `ScriptWizard-Agent-1.4.2-patch.zip` | ~3.9 MB | **Agent code + FreeRDP patch** for an already-installed agent. Stops the service, overlays new source, restarts. Run `update-agent.ps1` (add `-Deps` if dependencies changed). |

> **Upgrading from 1.4.1?** Apply the server patch with `setup\update.ps1`, then the agent patch
> with `update-agent.ps1`. Both changed in 1.4.2.
>
> **Schema / settings:** a new `desktop_runners.run_window_minimized` column is added
> **automatically** by the additive schema sync - `setup\update.ps1` checks/repairs the schema and
> the server also self-heals it at startup, so there is no manual migration and no data is dropped.
> The new `ARTIFACT_RETENTION_DAYS` setting seeds on the first restart (default `0` = no change).

Password: shared separately.

## What's new in 1.4.2

### New features and UX
- **Desktop run-window minimize (per device).** A toggle on the device page ("Minimize run window
  at start") opens the on-device mini run-window minimized to the taskbar at script start instead
  of in the foreground. Off by default; applies to both `normal` and `rdp` deployments. (Needs the
  updated agent on the device.)
- **Status dots for devices.** The Devices list and device details now show connection + activity
  as a colour-coded status dot - online, busy (pulsing), connecting, offline, revoked - replacing
  the old text pills. Honours `prefers-reduced-motion`.
- **14-day success-rate KPI on the Overview.** The all-time "Success rate" tile (pinned near 100%
  and never moving) is now a **14-day** windowed rate, colour-coded: green >=90, blue >=75,
  amber >=50, **red under 50** - so it actually reflects recent reliability.
- **Artifact retention, decoupled from job retention.** A new `ARTIFACT_RETENTION_DAYS` setting
  (Settings > Storage, right under Job retention) clears the on-disk run files/artifacts
  (`data/artifacts/<job_id>`) sooner than `JOB_RETENTION_DAYS`, while keeping the job record + logs
  for the full window. Default `0` = keep as long as the job (no behaviour change until you set it).
- **Settings page polish.** Each section header now carries a category logo; the sidebar order was
  tidied to Scripts, In Progress, Run History, Schedules, Libraries.

### Security
- **Stored + attribute-context XSS fixed** across the dashboard: action buttons pass opaque ids and
  resolve records in JS instead of interpolating display names into inline handlers, closing an
  apostrophe-breakout class that `escapeHtml` alone did not stop.
- **Server-side HTML sanitizer** (dependency-free, allowlist) applied on write to admin notification
  bodies and the editable Terms and Conditions, plus a sandboxed client render, so admin-authored
  HTML can never execute script, `on*` handlers, or `javascript:` URLs. The email-preview iframe is
  now sandboxed.
- **Security response headers** (X-Content-Type-Options, X-Frame-Options, Referrer-Policy, baseline
  CSP) on every response.
- **CSV formula-injection guard** on the audit export, and secret hygiene in the UI (enrollment
  token masked with reveal, no CyberArk password-length disclosure).

### Error handling and observability
- **Three-state list rendering** (loading / error + Retry / empty) across every list page, so a
  fetch failure no longer masquerades as a legitimate empty state.
- **API error contract:** 422 validation errors render as readable `field: message`; non-JSON proxy
  error pages and hung requests are detected instead of surfacing as `null` / permanent spinners; a
  30s request timeout was added.
- **Backend correlation ids:** a global 500 handler and a request-validation handler log and return
  a short reference code, so a user's screenshot maps to one line in `runner.log`.
- **Frontend error capture:** uncaught errors and unhandled rejections show one toast and POST to a
  new `/api/admin/client-errors` sink that writes to `runner.log`.
- **Session and network resilience:** a mid-session 401 preserves the destination via a validated
  `?next=` and a flash message; a network or 5xx blip shows a reachable "can't reach the server"
  panel with Retry instead of bouncing to login.

### Correctness and data-loss fixes
- **Desktop session creation** now refuses to run on a single-session device when a DIFFERENT user
  is signed in, failing with a clear, actionable message instead of stalling on a hidden Windows
  "another user is signed in" prompt.
- Editing a disabled schedule no longer silently re-enables it; missing-script schedule edits no
  longer wipe stored variables; boolean variables round-trip correctly.
- Numeric settings reject an emptied field instead of persisting `null`.
- The script editor no longer blanks a just-saved new script on "Reset Code"; the Monaco diff
  editor is disposed on modal close; IO renames no longer shred the undo stack.
- Admin-scoped API keys display with the correct role; the desktop-runner "test fetch" refreshes
  local state after its implicit save.
- **Double-submit guards** on create/save/send/cancel actions across the app.

### Accessibility
- Shared modal gets a focus trap, Escape to close, scroll-lock, initial focus, and focus restore;
  the account menu is keyboard reachable; toasts announce via `role=status`, de-dupe, and are
  dismissable.
- `prefers-reduced-motion` and `prefers-color-scheme` are honoured, list/pill/outline contrast
  raised, and a reveal fallback prevents a permanently blank page if a script fails to load.
- All em/en dashes and ellipses scrubbed from the dashboard for consistent rendering.
