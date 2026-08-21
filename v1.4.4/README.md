# v1.4.4 - 2026-08-21

Desktop-session, diagnostics and dashboard release on top of 1.4.2. Same brand, so no migration
is needed. **Both the server AND the agent changed this cycle - apply both patches.**

> ### Read this before upgrading
>
> **1. Your devices' RDP Width / Height / Port now actually take effect.** The server has pushed
> those settings since 1.4 and the agent never read them, so every device silently ran
> **1920x1080 on port 3389** regardless of what the device page said. Check any device configured
> otherwise: after this patch its session resolution changes, and so does the size of any
> screenshot taken on it.
>
> **2. Dashboard preferences reset once.** Theme, sidebar state, Scripts tile/list view and editor
> font are now stored per user instead of per browser. Everyone re-picks them once, then they
> persist per account.
>
> **3. Loading spinners and status dots animate again on Cloud PC / VDI.** 1.4.2 added
> `prefers-reduced-motion` support; that has been removed. See *Reverted* below.

## Packages

| File | Size | Description |
|---|---|---|
| `ScriptWizard-1.4.4-Setup.zip` | ~60 MB | **Fresh server install.** Includes the offline Python cp312 wheelhouse, no internet required. Run `setup\install.ps1`. A detailed `setup\INSTALL.html` walkthrough is bundled. |
| `ScriptWizard-1.4.4-patch.zip` | ~0.6 MB | **Same-brand server patch** for an install that is already Script Wizard. In-place code overlay only. Run `setup\update.ps1` (add `-Deps` if dependencies changed). |
| `ScriptWizard-Agent-1.4.4.zip` | ~38 MB | **Full agent installer** for a runner device. Bundles its own offline wheelhouse and the FreeRDP client. Run `install.ps1`. |
| `ScriptWizard-Agent-1.4.4-patch.zip` | ~3.8 MB | **Agent code + FreeRDP patch** for an already-installed agent. Stops the service, overlays new source, restarts. Run `update-agent.ps1`. |

> **Upgrading from 1.4.2?** Apply the server patch with `setup\update.ps1`, then the agent patch
> with `update-agent.ps1` on each device. **Both changed.**
>
> **Schema:** no changes this release. The server restarts as part of `update.ps1`, which is
> required (new response headers are set at startup).
>
> There is no 1.4.3 - the version was skipped.

Password: shared separately.

## Fixed

### Desktop runs (agent-side - needs the agent patch)

- **Screenshots taken during a desktop run came back black.** A job was launched into whatever
  RunAs session already existed, and `find_user_session` matched a session whether it was active
  **or disconnected**. A disconnected session is still logged in and still runs processes, but
  Windows tears down its display surface when the last RDP client detaches - so `pyautogui`,
  `PIL.ImageGrab` and `mss` all captured solid black. It only worked while someone happened to be
  watching the session, because their client supplied the surface.

  A disconnected session is now **reconnected** for the duration of the run, which restores the
  framebuffer. An active session that somebody is watching is left completely untouched.
- **A session locked by `post_run_action=lock` could not be automated or captured** (the lock
  screen lives on the Winlogon secure desktop). Such a session is now reconnected, which resumes
  it unlocked. Lock detection was also made reliable - it now probes for `LogonUI.exe` in the
  session rather than trusting a `WTSINFOEX` flag whose meaning was inverted on older Windows.
- **A run could lose its display surface part-way through.** Opening `mstsc` to a device takes the
  session over and boots the agent's client; closing that window then left the session
  disconnected with nothing holding it, so captures went black for the rest of the run. A
  watchdog now re-attaches when that happens. It never creates a session, and never touches one
  that is already active.
- **The device's RDP Width / Height / Port were ignored.** They are now used for both a job's
  session and the connection pre-warm, so a pre-warmed session no longer has to be resized by the
  first job (which moved every window on it).

  Every one of these paths **degrades rather than fails**: if a reconnect is impossible - no
  credential, no FreeRDP client, another user signed in on a single-session box - the script still
  runs exactly as it did before, with a warning in the agent log that captures may be black.

### Diagnostics (server-side)

- **A crashed script reported only a number.** A process killed by Windows writes no traceback, so
  a failed run showed `Process exited with code 3221226356` and an empty log. Fatal exit codes are
  now decoded - heap corruption, access violation, missing DLL, DLL entry point not found, stack
  overflow, and others - with the likely cause and a note that it was an OS kill rather than a
  script error.
- **A crashing script's own output was discarded.** Python ran with block-buffered stdout, so
  everything a script printed before being killed was lost and the run looked like it never
  started. Scripts now run unbuffered (`-u`), which also means a long-running script's output
  **streams to the live log** instead of appearing in 8 KB chunks. `faulthandler` is armed so a
  native fault in a compiled extension names the Python line that called it.

### Dashboard

- **A second user on the same browser inherited the first user's session state - including the
  Terms & Conditions acceptance flag**, which meant they could skip the acceptance gate entirely
  with no consent record written for them. All browser-stored state is now namespaced per account.
  Preferences are *not* deleted on sign-out, so returning to your own account restores your setup.
- **Searching for a Script ID returned nothing.** Every script card shows a "Script ID" chip, but
  the search only looked at name and description. It now matches the Script ID (slug) and the
  internal id as well. Team scoping is unchanged and still applied.
- **Schedules' table was missing its card frame**, unlike every other list page.
- List tables were crowding their left and right borders; the first and last columns now line up
  with content elsewhere in the app.
- Dashboard assets now send an explicit `Cache-Control`, so a shipped fix can no longer be masked
  by a stale cached page.

## Changed

- **Device pages reworked.** The device name no longer repeats as its own subtitle; the badge
  beside it now reports the **connection** (Connected / Disconnected / Revoked), distinct from the
  Overview's Status row which reports **activity** (Idle / Busy). Save moved to the page header,
  with Revoke and Delete in an overflow menu. Configuration is split into collapsible
  **Deployment settings**, **Device credentials** and **Installed packages** sections, each
  collapsed by default and showing a summary of what is inside while closed.
- **Devices list** gains a **Username** column (the device's RunAs account) and a **Refresh**
  button, drops the Host column that duplicated the device name on every row, and no longer shows
  the enrollment token. Connected devices pulse so a held-open agent connection is visible at a
  glance.

## Reverted

- **`prefers-reduced-motion` suppression, added in 1.4.2, has been removed.** Cloud PC and VDI
  images ship with Windows animation effects switched off, so browsers on them report `reduce` for
  **every** user even though nobody asked for it. The result was that every loading spinner and
  status pulse froze, and a running job looked hung. Because the rule was `!important` on the
  universal selector, it also silently killed animations declared in any individual page.

  Nearly all motion in this dashboard is status rather than decoration, so suppressing it makes
  the UI report the wrong thing. Users who genuinely set the OS preference will now see these
  animations; that is a deliberate trade-off.

  A follow-on fix: the status pulses themselves were using `ease-out`, which finishes the ring's
  expansion in the first fraction of each cycle and leaves a long flat tail - so the In Progress
  "live" dot, and the Connected / Busy / connecting device dots, read as static even when they
  were animating. They now use default easing and pulse visibly.
