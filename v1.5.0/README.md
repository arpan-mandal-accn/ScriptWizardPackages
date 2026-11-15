# v1.5.0 - 2026-11-14

The biggest release since 1.4. It brings **platform backup & restore**, a reworked **script
editor**, **multi-trigger schedules** and **run limits**. It also fixes desktop-runner reliability
and makes the dashboard look consistent. It contains everything since 1.4.5; an internal 1.4.6
build was never released, and its changes ship here.

**Both the server AND the agent changed this cycle - apply both patches.**

> ### Read this before upgrading
>
> **1. A desktop device now serves one Script Wizard server.** Serving several servers from one
> device is no longer supported, and `add-server.ps1` is gone. `update-agent.ps1` keeps the first
> server on a device enrolled with several, or the one you name with `-KeepServer <url>`. Devices
> enrolled with a single server update without any prompt.
>
> **2. Every script now has a run limit of 30 runs per minute**, existing scripts included. It is
> applied once, on the first start of 1.5.0. A script that needs more can be set higher, or to 0
> (no limit), in the script editor under **⋮ → Settings**. API keys stay unlimited unless you set
> a limit on one.
>
> **3. IP Filtering rules now apply to API calls only.** The dashboard is governed by the
> public-access setting alone, and no longer checks the IP rules.
>
> **4. Backups need one new component, `pyzipper`.** `setup\update.ps1` installs it automatically,
> offline, from wheels bundled in the patch, so `-Deps` is not needed. If it cannot be installed,
> the server still starts normally; only Settings → Backup & restore shows the component as
> missing.

## Packages

| File | Size | Description |
|---|---|---|
| `ScriptWizard-1.5.0-Setup.zip` | ~62 MB | **Fresh server install.** Includes the offline Python cp312 wheelhouse, no internet required. Run `setup\install.ps1`. A detailed `setup\INSTALL.html` walkthrough is bundled. |
| `ScriptWizard-1.5.0-patch.zip` | ~2.5 MB | **Same-brand server patch** for an install that is already Script Wizard. In-place code overlay, plus the two wheels backups need. Run `setup\update.ps1`. |
| `ScriptWizard-Agent-1.5.0.zip` | ~38 MB | **Full agent installer** for a runner device. Bundles its own offline wheelhouse and the FreeRDP client. Run `install.ps1`. |
| `ScriptWizard-Agent-1.5.0-patch.zip` | ~4 MB | **Agent code + FreeRDP patch** for an already-installed agent. Stops the service, overlays new source, restarts. Run `update-agent.ps1`. |

> **Upgrading from 1.4.5?**
> 1. Apply the server patch with `setup\update.ps1`.
> 2. Apply the agent patch with `update-agent.ps1` on each device. **Both changed.**
>
> Try it on the dev server first.
>
> **Schema:** additive only. This release adds:
> - a new `schedule_triggers` table;
> - a `runs_per_minute` column on `api_keys` and `scripts`;
> - a `run_limit_period` column on `scripts`;
> - new setting rows for backups;
> - an `incident_number` column on `anomaly_alerts`.
>
> `update.ps1` lists what it is about to add, takes a copy-only database backup only when
> something is pending (`-NoBackup` skips it), applies the change and verifies it. Existing
> schedules keep their single trigger. The server restarts as part of `update.ps1`, which is
> required.
>
> **Rollback:** a 1.4.5 build on this database ignores the new table and columns, and a
> multi-trigger schedule then runs its first trigger only. Backups made by 1.5.0 restore on 1.5.0
> or later.

Password: shared separately.

<details>
<summary>SHA-256 checksums</summary>

```
1cd73c257d14d9904661f1d930fc41f4771010aacbdafb5bfa26542f5cbd330c  ScriptWizard-1.5.0-Setup.zip
ebe654622978e2fdab47b8e9c06268945d9cfa11dd458e0172d123471320ce73  ScriptWizard-1.5.0-patch.zip
a81e6de5f7019b4ce1104f58bc52df9679220af8dabdd747ec35eaca79f64848  ScriptWizard-Agent-1.5.0.zip
d7677b4a6376ddf525bbc3d0e773de7a6171bc5151905534a61404631c46b907  ScriptWizard-Agent-1.5.0-patch.zip
```

In PowerShell: `Get-FileHash .\ScriptWizard-1.5.0-patch.zip -Algorithm SHA256`.
</details>

## Added

### Backup & restore (Settings → Backup & restore)

- **What a backup is.** Each backup is one AES-256 password-protected zip; it opens in 7-Zip with
  the password. Inside, a `manifest.json` describes it: the format and app version, the server it
  came from, and the row count of every item.
- **When and where.** Backups run daily at a set time on chosen days. They go to a folder you pick
  with a folder browser that shows the server's own drives, with its OneDrive folders listed
  first.
  - A OneDrive folder copies every backup off the server.
  - The newest 14 are kept.
  - **Back up now** makes one at once.
- **What it contains.** Scripts and folders, libraries, schedules, users and teams, API keys and IP
  rules, desktop devices, settings, secrets, notifications, the audit log and run history.
  - Secrets means the HMAC key and the SMTP and LDAP credentials. The database connection is never
    included.
  - Run files (artifacts) and the TLS certificate are optional.
- **Restore** works on this server or a new one.
  1. Pick a backup from the list, or upload one.
  2. Enter its password to see what is inside.
  3. Untick anything you don't want back, and type `RESTORE`.

  The server then restarts and applies it before anything else runs, in one transaction, after a
  safety backup of what it replaces.
  - This server's own network, HTTPS and folder settings are kept.
  - Runs that were queued in the backup never re-run.
  - Schedules continue from now.
  - Everyone signs in again.
  - A backup without an active administrator is refused.
  - Restores are audited and emailed to the alert recipients.
- **Compatibility.** A newer version can always restore an older backup: columns the backup lacks
  get their defaults, and unknown ones are skipped and reported. A backup from a newer version is
  refused until the server is updated.
- **Without the dashboard.** If the dashboard cannot start, stop the service and run
  `scripts\restore_backup.py --file <backup.zip>`. `--check` only shows what is inside.

### Scheduling and limits

- **Several triggers under one schedule.** Each trigger has its own type, time, timezone and
  inputs, for example 07:00 IST with `region=EU` and weekdays 18:30 EST with `region=US`.
  - **Run now** asks which trigger's inputs to use.
  - The schedule form has three tabs: Details · Triggers · On failure.
- **Run limits.**
  - **API keys:** per minute, counting job submissions only. Polling and logs are not limited.
  - **Scripts:** per minute, hour or day, counting API calls, Run now, re-runs and schedules. Tests
    and automatic retries don't count.
  - **Over the limit:** the API answers `429` with `Retry-After`, and a scheduled run is skipped
    and audited as `schedule.skipped`.
  - **Hourly and daily limits** carry on after a restart, rebuilt from Run History.

### Script editor

- **The Test panel slides in from the right**, next to the code. Drag its edge to resize it; it
  always starts closed. The result is one line (status, exit code, duration), then the outputs and
  logs, each with a copy button.
- **The input and output variables fold to one line of chips**: name, a dot in the type's colour,
  the type, and `*` when required. **Edit variables** opens the full editor.

### Dashboard

- **A new sign-in page.** The Accenture and Script Wizard logos are drawn in particles
  that blow away and reform as each other; the sign-in card sits on the right, with a theme
  switch. The animation is built for Cloud PCs and stops as soon as you sign in.
- **API key Usage tab.** Runs per day for a week, with failed runs and rejected submissions,
  paging back week by week. A disabled key can be **enabled again**.
- **Multi-select.**
  - Run History and Audit export just the ticked rows.
  - In Progress cancels the ticked jobs together.
  - The Recycle Bin restores or deletes the ticked items in bulk.
- **A sun / moon theme button.** Switching grows the new theme as a circle from the button.
- **Libraries** get a full-screen code view. Each **Scripts** folder has a ⋮ menu with **Add
  script** and **Edit folder**.

### Summary email (Settings → Summary email)

- **A scheduled report for leadership.** Daily (the previous day), weekly on a chosen weekday (the
  previous 7 days) or monthly on the 1st (the previous calendar month), at a set time in a chosen
  time zone, to To and Cc recipients.
- **You choose the sections:** highlights written as sentences, headline numbers with their change
  on the previous period, an activity chart, the most-run scripts, the scripts that need attention
  with their latest error, automation by team, new automations live, how runs were started, alerts
  and incidents, platform health and growth. Editor test runs appear only in their own optional
  section.
- **Preview** opens the email in a new tab; **Send now** sends it straight away. It is built for
  Outlook, up to 720 px wide, and rearranges itself on phones.

### Alerts

- **ServiceNow incidents for anomaly alerts.** Set up the connection once: instance, endpoint,
  incident fields, and basic, OAuth 2.0 or API-key sign-in. The credentials are stored encrypted and
  never shown again. Then turn incidents on per check, each with its own impact, urgency and
  prefilled texts. One incident is raised per problem; repeat alerts go by email. **Test
  connection** and **Send test incident** check the setup.
- **Three new anomaly checks:** **Unusual run volume**, **Low disk space** and **Backup failed**.
  While Backup failed is on, it replaces the standard backup-failure email.
- **To and Cc** for failure alerts, anomaly alerts and the summary email.
- **The SMTP and Active Directory service-account passwords are set from Settings.** They are
  written to the server's `conf\.env` encrypted and used at once, with no restart.

### Update script

- **`setup\update.ps1` previews, backs up and verifies the database migration.** It takes a
  copy-only backup only when something is pending. `scripts\ensure_schema.py --check` runs the
  preview on its own.

## Changed

- **Script settings save automatically.** In the script editor, the run settings live under
  **⋮ → Settings**: Enabled, Timeout, Max runs per minute / hour / day, Long-running and Use
  Desktop. They save as you change them, with no Save needed, and a settings change never creates
  a new version. The ⋮ menu is grouped (change · look · copy · delete), with dividers.
- **The Save button shows when there is something to save.** It is a blue outline normally and
  filled soft blue while something is unsaved.
- **Recovered drafts.** A recovered draft is one amber line in the header with Restore and ✕. It
  shortens on narrow windows and moves to its own row if needed; a long description never hides
  its date.
- **Edit details saves on its own.** Save details stores the name and description straight away.
- **More room for the code.**
  - The editor's tools share one compact row with the "Source code" title.
  - The editor is at least 380 px tall.
  - "How this script runs" starts folded.
  - **Add input / Add output** puts the cursor straight into the new name.
- **Script export includes everything**: every setting, the run-limit period and the desktop
  target, plus a header that says what the file is. Import restores them, and puts the desktop
  device back when it exists on this server.
- **One consistent list style** across Schedules, API Keys, Teams (each team gets its own colour),
  IP Filtering and the Recycle Bin.
  - **In Progress** is a single table with a state filter and the running time.
  - **Run History** shows the script name, Script ID, source and where each run executed.
  - The Run History **CSV** adds the language and execution place, and neutralises cells a
    spreadsheet would run as a formula.
- **Settings.** Security and Active Directory settings can now be changed from any admin session.
  Network, HTTPS, script-path and backup-folder settings still need a session on the server
  itself.
- **Desktop jobs.**
  - A desktop job waits for its busy device for up to `DESKTOP_QUEUE_WAIT_MAX_MINUTES` (default
    30) instead of timing out before its script starts.
  - Desktop jobs no longer share the generic concurrency cap.
- **Anomaly detection settings are reorganised.** Each check is its own panel with a switch, a
  one-line summary of its values and labelled fields with units. The repeat interval is one field,
  **Repeat alert every**. A repeated-failures alert now includes the error from the latest failed
  run.
- **The Alerting settings are grouped** into **Mail server** and **Failure alerts**, with plain
  names for every field.
- **Run History shows finished runs only.** A queued or running job opens in **In Progress**, where
  its log streams live.
- **Backups carry the new settings.** The ServiceNow connection travels with Settings and its
  credentials with Secrets.

## Fixed

### Desktop runs (agent-side - needs the agent patch)

- **Cancelling a run while its session was still connecting stalled the device.** The next job
  could then wait up to 5 minutes. Connecting, signing out and waiting for the worker are all
  cancellable now. A run that ignores STOP is force-stopped after 15 seconds, together with its
  whole process tree.
- **Someone watching with the same RunAs account broke the run.** The runner now neither signs
  them out ("always new" included) nor fights them for the session.
- **Sign-out and auto-logon contradicted each other.** Auto-logon is suspended while the runner
  signs a session out, and `ForceAutoLogon` is no longer set.
- **The runner could stop responding.**
  - Pre-warming a session now waits behind real jobs.
  - A heartbeat goes out on every job state change.
  - On reconnect, the runner reports the jobs it still holds, so the server finishes or re-queues
    them instead of leaving them stuck.

### Server and dashboard

- **"Run now" on the Schedules page always failed with HTTP 500**, because of a missing import. A
  run that hits its limit now gets a readable message.
- **A key that had ever been used could not be permanently deleted** (a foreign-key error). Its
  runs are now detached and stay in Run History.
- **Saving a script created a new version even when only its settings or description changed.**
  A version is now created only when the code or the name changes.
- **Dark mode flashed white** while the script editor, Libraries and sign-in pages loaded.
- **OneDrive could not sync backups** ("No access permissions to the item") when the server ran as
  a service. A backup is now copied into its folder, so it takes that folder's permissions.
- **On the Users page, the "+N" list of a user's other teams was cut off.**
- **The Refresh button's tick could get stuck** (and spin) when Refresh was pressed again.
- **A broken stylesheet comment** silently dropped the script editor toolbar's layout rule.

## Removed

- **`agent\add-server.ps1`**: one desktop device now serves one Script Wizard server.
