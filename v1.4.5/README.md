# v1.4.5 - 2026-09-05

Access-control release on top of 1.4.4. Two new dashboard roles, and the privilege boundary
between them made airtight. **Both the server AND the agent changed this cycle - apply both
patches.**

> ### Read this before upgrading
>
> **1. An unrecognised role used to receive full write access.** The old permission gate denied
> exactly one role name (`viewer`), so anything it did not recognise fell through with author
> rights. Permissions are now resolved from a capability table and an unknown role resolves to
> *no* capabilities. If you have ever written a role value directly into the database, check
> those accounts after upgrading: they will now be denied rather than allowed.
>
> **2. Existing accounts are unaffected.** Administrator, Editor and Viewer keep exactly the
> access they have today. The two new roles are opt-in, per user.
>
> **3. Enrolling a desktop runner would have failed without this release's schema step.** A
> column removed from the models was left behind as `NOT NULL` with no database default on any
> install built from a clean setup, so inserts into the devices table failed outright. The
> schema step now relaxes it. This is handled automatically by `setup\update.ps1`.

## Packages

| File | Size | Description |
|---|---|---|
| `ScriptWizard-1.4.5-Setup.zip` | ~63 MB | **Fresh server install.** Includes the offline Python cp312 wheelhouse, no internet required. Run `setup\install.ps1`. A detailed `setup\INSTALL.html` walkthrough is bundled. |
| `ScriptWizard-1.4.5-patch.zip` | ~0.7 MB | **Same-brand server patch** for an install that is already Script Wizard. In-place code overlay only. Run `setup\update.ps1`. |
| `ScriptWizard-Agent-1.4.5.zip` | ~40 MB | **Full agent installer** for a runner device. Bundles its own offline wheelhouse and the FreeRDP client. Run `install.ps1`. |
| `ScriptWizard-Agent-1.4.5-patch.zip` | ~4 MB | **Agent code + FreeRDP patch** for an already-installed agent. Stops the service, overlays new source, restarts. Run `update-agent.ps1`. |

> **Upgrading from 1.4.4?** Apply the server patch with `setup\update.ps1`, then the agent patch
> with `update-agent.ps1` on each device. **Both changed.**
>
> **Schema:** additive. A new `counters` table and a new `jobs.source` column, both created by
> the patch's schema step, which also seeds the job counter and backfills `jobs.source` on
> existing rows. No data migration and no manual action. No dependency changes, so `-Deps` is
> not needed. The server restarts as part of `update.ps1`, which is required.

Password: shared separately.

## Added

### Two new roles

- **Platform Manager.** Everything an administrator can do *except* creating or editing scripts.
  Intended for someone who runs the platform (users, teams, devices, settings, audit) but should
  not be able to author code that executes on it.
- **Operator.** Everything a viewer can do, plus permission to *run* scripts, cancel a running
  job and re-run a finished one.

## Fixed

### Permissions (server-side)

- **An unrecognised role received full write access.** Authorisation ran through two functions,
  one of which denied a single role name rather than granting a known set. Every gate now
  resolves one of three capabilities (`admin`, `author`, `run`) from a single table, and a role
  outside that table gets nothing.
- **A Platform Manager could act on an administrator.** Granting a role followed a subset rule
  (you may only grant what you already hold), but the *target* was never checked, so a manager
  could demote, disable or delete an account more powerful than their own. The target is now
  checked too, and the Users page locks those controls to match the server.
- **The AD group map was writable by non-administrators.** Editing the group to role mapping is
  granting roles in bulk, deferred to the next sync and applied with no actor to authorise it. A
  manager could remap the administrators' group and run a sync. It now requires a full
  administrator, and each mapped role is validated against the editor's own capabilities.
- **An LDAP sync could lock everyone out.** A sync that would change the last account able to
  grant Administrator back is now refused and logged instead of applied.
- **`POST /scripts/test` was reachable by non-authors.** That endpoint executes code posted in
  the request body, so it now requires `author` rather than run permission. Library writes and
  server module installs are administrator-only for the same reason.

### Desktop runs (agent-side - needs the agent patch)

- **Cancelling a desktop run did nothing.** A read timeout on the worker channel was treated as
  end-of-stream, which ended the frame generator and killed the STOP watcher thread, leaving the
  worker deaf to cancels for the rest of the run. Nothing is normally sent worker-ward during a
  run, so quiet was the *expected* case and this fired on almost every run.
- **A stop arriving while the process was still launching was lost.** Interpreter resolution and
  writing the script file both happen before the child is registered; a STOP in that window set
  a flag that nothing acted on until the script had already finished.
- **`session_reuse=always_new` did not start a new session.** Windows will not grant one account
  a second interactive session, so the existing one has to be signed off first. Without that,
  "always new" behaved identically to "reuse if available".

### Dashboard

- **A script that had ever run could not be permanently deleted.** "Delete permanently" in the
  Recycle Bin raised a foreign-key violation because the run history still pointed at the row
  being removed, so it failed for essentially every real script. Run history is now detached
  rather than deleted: a run record outliving its script is the point of keeping one.
- **`Total jobs` went down over time.** It counted rows in the jobs table, and retention
  hard-deletes finished runs, so the lifetime figure silently dropped as old jobs aged out. It is
  now recorded at creation in a counter row, which also removes a full-table aggregate from every
  Overview load.
- **The Scripts page filters stopped filtering** after the dropdown rework, and the card
  animation died after using the folder Back button.
- **VBScript runs were missing `RR_SCRIPTS_DIR`.** The bundled `run_script.vbs` reads it and it
  was never set, so anything relying on it failed.

## Changed

- **Every dropdown is now the platform's own.** All 42 `<select>` elements are enhanced in place,
  with the native element still the source of truth, giving consistent styling, keyboard support,
  type-ahead and a search box on long lists.
- **The Audit page filters by what is actually in the log.** The entity dropdown listed types
  that mostly never appear. Action and Actor are now built from the log's own distinct values
  with counts, ordered by frequency.
- **The Users card counts activity, not roles.** It read `1 administrator, 1 platform manager,
  1 viewer`, which is three clauses of text on a card meant to be read at a glance. It now shows
  how many accounts signed in over the same 14 days the trend chart uses, computed once so the
  two can never disagree.
- Motion pass: navigation sections and all three folder trees fold instead of snapping,
  modals, toasts and rows animate out as well as in, controls have a press state again, and
  confirm dialogs gained a close button.
- The login page has a new animated background.

## Removed

- **The "redirect the session onto the console" attempt, added in 1.4.4.** `tscon /dest:console`
  fails with error 5023 on single-session Windows 10 and 11, because Remote Desktop moved to an
  indirect display driver in 1903. It could never succeed on these devices and only delayed the
  fallback that does work, by up to the tscon timeout, in the middle of a run. Black-screenshot
  handling now goes straight to the reconnect path.
- **The remaining `prefers-reduced-motion` gates**, following the same reasoning as 1.4.4: VDI
  images report `reduce` for every user, so honouring it disables motion for the whole fleet.
