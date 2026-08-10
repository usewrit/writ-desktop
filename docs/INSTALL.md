# Installing Writ Desktop

Download the installer for your platform from the
[latest release](https://github.com/usewrit/writ-desktop/releases/latest).

## macOS

Requires **macOS 11 Big Sur or newer**. There are two builds — take the `aarch64` one on
Apple silicon (M1 and later) and the `x64` one on an Intel Mac. If you pick the wrong
one it simply will not launch.

1. Open the `.dmg` and drag **Writ** to Applications.
2. Launch it from Applications.

The build is signed with an Apple Developer ID and notarized by Apple, so Gatekeeper
should let it open normally. If you see "Writ can't be opened because Apple cannot check
it for malicious software", you almost certainly have a partial download — re-download
and check the hash (below) rather than working around Gatekeeper on a binary you can't
verify.

## Windows

Requires **Windows 10 or newer** (x64).

Run the `.msi` (or `.exe`) and follow the installer. The default install is per-machine
and needs administrator rights.

**This build is not yet Authenticode-signed.** SmartScreen will show *"Windows protected
your PC"* and name the publisher as unknown; choose **More info → Run anyway** to
continue. Because Windows cannot vouch for the file for you, verify it against
`SHA256SUMS` (below) before running it — on this platform that check is doing real work,
not ceremony.

## Linux

Requires **glibc 2.35 or newer** (Ubuntu 22.04 and equivalents) with a working
WebKitGTK. Builds are published for **x86_64** and **arm64** — match your machine
(`uname -m`).

```bash
# Debian / Ubuntu
sudo dpkg -i writ_*.deb && sudo apt-get -f install

# Anything else
chmod +x Writ_*.AppImage && ./Writ_*.AppImage
```

Linux packages are not OS-signed on any distribution; `SHA256SUMS` plus the signed
update manifest is the integrity story here.

## Verifying your download

Every release includes `SHA256SUMS`. Download it alongside the installer and:

```bash
shasum -a 256 -c SHA256SUMS --ignore-missing     # macOS / Linux
```

```powershell
Get-FileHash .\Writ_x64_en-US.msi -Algorithm SHA256   # Windows — compare to SHA256SUMS
```

The hashes are computed from the **signed** artifacts, so they match exactly what you
downloaded.

## First launch

A short setup wizard runs once. It covers:

1. **Language** — English, French, or Spanish.
2. **Browser runtime.** Writ drives a real Chromium to record and replay. If you already
   have Chrome or Chromium installed, it is detected and used. Otherwise the app downloads
   a private copy of **open-source Chromium** (**about 150-330 MB**, depending on platform)
   into its own directory — it is used only by Writ and never touches the browser you use
   every day. This is the only download the app makes on first run.

   The download is pinned to a specific Chromium revision and checked against a SHA-256
   built into the application before it is ever launched; a binary that does not match is
   refused. Writ does not redistribute a browser — you obtain it from its publisher.
3. **Launch at login** — optional; keeps scheduled monitors and API calls working
   without the window open.
4. **Connect your AI** — optional one-click MCP setup for Claude Desktop, Claude Code,
   ChatGPT, or Codex.

You can re-run the wizard any time from **Settings → General → Re-run setup**.

### Installing without an internet connection

The app itself works offline; only the one-time browser download needs the network. If the
machine cannot reach the internet, either install Chromium/Chrome through your usual
channel (Writ detects it) or copy the runtime directory across from a machine that can. The
path is shown in **Settings → Engine & Data**.

On **arm64 Linux** there is no published open-source Chromium build to download, so install
it with your package manager (`sudo apt install chromium`) — Writ will detect it.

## Where things are installed

| | macOS | Windows | Linux |
|---|---|---|---|
| Application | `/Applications/Writ.app` | `C:\Program Files\Writ` | `/usr/bin/writ` (deb) |
| Your data | `~/.writ` | `%USERPROFILE%\.writ` | `~/.writ` |

Everything the app creates — your encrypted database, downloaded browser runtime, logs —
lives under the data directory. Deleting it resets the app to a fresh install.

## Uninstalling

Remove the application, then delete `~/.writ` if you also want your workflows and
credentials gone. Uninstalling the app alone leaves your data in place so a reinstall
picks up where you left off.

If you enabled "launch at login", turn it off in **Settings → General** *before*
uninstalling, or remove the login item manually afterwards.
