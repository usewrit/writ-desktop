# Troubleshooting

Before anything else: **Settings → About** gives you the app version and a button to
open the log directory. Both are the first two things any bug report needs.

## "Engine stopped" in the bottom-left corner

The app is a thin window over a background engine (`writ-agentd`) that does the actual
browser work. The pill shows red when the window cannot reach it on `127.0.0.1`.

1. Click the **▶** next to the pill to restart the engine.
2. If it will not start, check the logs (**Settings → About → Open logs**) for the
   `writ-agentd` lines at the end.
3. A firewall or endpoint-security product that blocks **loopback** connections will
   break this. The engine listens only on `127.0.0.1` and never on a public interface —
   allow the app to talk to itself.
4. If a previous run left a stale process, quit Writ fully and relaunch.

## "Network error — check your connection"

This almost always means the same thing as the red pill: the window cannot reach the
local engine. It is not about your internet connection. Follow the steps above.

## Recording opens a blank browser, or will not start

Writ needs a Chromium runtime. **Settings → Engine & Data** shows which one it resolved
and where it lives.

- If it reports none, install it from that screen (about 150 MB).
- If a download failed part-way, install it again — a partial download is discarded
  rather than used.
- On Linux, the app needs a working WebKitGTK for its own window; a missing system
  library shows up as the app failing to launch at all rather than a blank browser.

## A recorded workflow used to work and now fails

Sites change. A step that pointed at a button that has moved or been renamed will fail
on replay. Open the workflow, find the failing step, and re-record just that step —
the rest of the workflow is preserved.

If the site added a login wall or 2FA, attach a **persona** so the run can sign in.

## Updates aren't installing

See [`AUTO_UPDATE.md`](./AUTO_UPDATE.md). The common causes, in order:

1. Updates are set to **Disabled** in Settings, or a machine policy disabled them.
2. The machine cannot reach the update host.
3. The update was fetched but **refused verification** — the app deliberately fails
   closed here and keeps running the version you have. If this persists, report it: a
   manifest that will not verify is worth looking at.

## The app will not open on macOS

If Gatekeeper reports that it cannot check the app for malicious software, the download
is likely incomplete. Re-download and verify the hash against `SHA256SUMS` rather than
bypassing the check. Releases are signed with an Apple Developer ID and notarized.

## Everything is broken and I want a clean slate

Quit the app, then move (do not delete, at first) your data directory:

```bash
mv ~/.writ ~/.writ.broken          # macOS / Linux
```

```powershell
Move-Item $env:USERPROFILE\.writ $env:USERPROFILE\.writ.broken   # Windows
```

Relaunch. You get a fresh install, and your old workflows and credentials are still in
the `.broken` directory if you need to recover something. **Deleting that directory
permanently destroys your workflows and stored credentials** — it is not recoverable
from anywhere else unless you had linked a cloud account.

## Filing a good bug report

[Open an issue](https://github.com/usewrit/writ-desktop/issues/new/choose) with:

- the app version (**Settings → About**) and your OS + version
- what you did, what you expected, what happened
- the relevant tail of the logs

**Please do not paste logs without reading them first** — they can contain URLs you
automate and page content. Redact anything you would not post publicly. If the problem
is a security vulnerability, do not open an issue at all: follow
[`SECURITY.md`](../SECURITY.md).
