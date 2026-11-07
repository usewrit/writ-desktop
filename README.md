<div align="center">
  <img src="./assets/banner.svg" alt="Writ Desktop — record a web task once, replay it forever, on your own machine" width="100%">

  <br/>

  <p align="center">
    <a href="https://github.com/usewrit/writ-desktop/releases/latest"><img src="https://img.shields.io/github/v/release/usewrit/writ-desktop?style=flat-square&label=download&color=FF4A24" alt="Latest release"></a>
    <a href="./LICENSE"><img src="https://img.shields.io/badge/license-proprietary-6E6E76?style=flat-square" alt="License: proprietary"></a>
    <img src="https://img.shields.io/badge/macOS-11%2B-000000?style=flat-square&logo=apple&logoColor=white" alt="macOS 11+">
    <img src="https://img.shields.io/badge/Windows-10%2B-0078D4?style=flat-square&logo=windows&logoColor=white" alt="Windows 10+">
    <img src="https://img.shields.io/badge/Linux-deb%20%C2%B7%20AppImage-FCC624?style=flat-square&logo=linux&logoColor=black" alt="Linux">
    <img src="https://img.shields.io/badge/MCP-ready-FF4A24?style=flat-square" alt="MCP ready">
    <img src="https://img.shields.io/badge/local--first-yes-16a34a?style=flat-square" alt="Local-first">
  </p>

  <h3 align="center">Record a web task once. Replay it forever — on your own machine.</h3>
  <p align="center">Turn any website into a workflow, a REST API, or an MCP tool your AI assistant can call.</p>

  <p align="center">
    <a href="#install"><b>Install</b></a> ·
    <a href="#what-it-does"><b>What it does</b></a> ·
    <a href="#connect-your-ai-mcp"><b>Connect your AI</b></a> ·
    <a href="./docs/TROUBLESHOOTING.md"><b>Troubleshooting</b></a> ·
    <a href="./SECURITY.md"><b>Security</b></a> ·
    <a href="https://github.com/usewrit/writ-desktop/issues"><b>Report a bug</b></a>
  </p>
</div>

---

**Writ Desktop** is the whole of [Writ](https://usewrit.app) in an app on your computer. Browsers run
on your machine, over your own internet connection, with the sign-ins you already have. Everything
it does on your computer is free, with or without an account: builds, recordings, runs, Harvest,
monitors, automations and schedules, with no credits and no meter. Your workflows and credentials
stay in an encrypted database in your home directory, and nothing leaves the machine unless you link
a Writ account, or add your own AI key (those requests go straight from your computer to your AI
provider).

**Writ Desktop or Writ Cloud?** Writ is a cloud service first. [Writ Cloud](https://usewrit.app)
runs the same APIs on Writ's servers and reads the web at scale (a page to Markdown or HTML, a site
map, a whole-site harvest), with residential connections and CAPTCHA solving when a site needs them,
metered in credits, over REST and an MCP server. Writ Desktop is the optional, free way to run them
on your own computer: your own sign-ins, systems only your network reaches, and personal volume. One
home connection gets throttled and challenged at volume; that is what Writ Cloud is for.

> **This repository is the download and issue tracker for the app.** Writ Desktop is
> proprietary and its source is not published here — see [`LICENSE`](./LICENSE). The client
> SDKs, the MCP bridge and the agent *are* open source and live in their own repositories — see
> [Open source](#open-source) below.

## Install

Get Writ Desktop from **[usewrit.app/desktop](https://usewrit.app/desktop)**, or take the installer
for your platform from the
**[latest release](https://github.com/usewrit/writ-desktop/releases/latest)**.

| Platform | Versions | Files |
|---|---|---|
| macOS | 11 Big Sur or newer, Apple silicon and Intel | `.dmg`, signed and notarized |
| Windows | Windows 10 or newer, x64 and ARM64 | `.msi` / `.exe` — **not yet code-signed**, see below |
| Linux | glibc 2.35 or newer (Ubuntu 22.04 and equivalents), x64 and arm64 | `.deb` / `.AppImage` |

> **Windows shows a SmartScreen warning.** The Windows build is not yet
> Authenticode-signed, so Windows will say *"Windows protected your PC — unknown
> publisher"*. Choose **More info → Run anyway**. This is the honest state of the build,
> not a step you should get used to skipping: verify the download against `SHA256SUMS`
> below before you run it. Signing is coming; until it does, this page will keep saying so.

Every release ships a `SHA256SUMS` file. On Windows it is the only integrity check
available to you, so it is worth the ten seconds:

```bash
shasum -a 256 -c SHA256SUMS --ignore-missing
```

Full instructions, including what the first launch downloads and how to install
without an internet connection, are in [`docs/INSTALL.md`](./docs/INSTALL.md).

The app updates itself. Each update's manifest is signature-checked against a key
compiled into the application before anything is installed, and an update that does not
verify is refused. In **Settings → Engine & Data → Updates** you can check for updates now or pick
the **Update channel** (**Stable**, or **Beta** for new features earlier, with less testing). You can
turn automatic updates off, or point a managed fleet at an internal feed — see
[`docs/AUTO_UPDATE.md`](./docs/AUTO_UPDATE.md).

### First run

The first screen offers **Create a free account or sign in** and **Continue in local-only mode**.
Local-only mode needs no account: you record, run and schedule with your own AI key, without sync or
the marketplace. You can sign in later from **Account**. A short setup then asks for your language,
what brings you to Writ, and where your runs should happen; it installs the browser Writ uses
(**Install Chromium**), offers **Start Writ when I log in**, and lets you **Connect an AI assistant**
and **Import your logins**. You can skip any step and change it later in **Settings**.

## What it does

The app has the same doors as Writ Cloud: **Create**, **APIs**, **Monitors**, **Harvest** and
**Automations**, with Scribe on **⌘J**.

**Build an API, or record a workflow.** Ask Scribe for an API, or drive a real browser inside the
app: every click, type, and navigation lands as an editable step. Replay it whenever you like — the
same result every time, at zero AI cost, because a replay executes the recorded steps rather than
asking a model to re-figure out the page.

**Call it like an API.** Every workflow is immediately callable three ways from your own
code, all on the loopback port with a local API key:

- `POST /v1/workflows/{id}/run` — REST, with start-and-poll or wait-for-result delivery
- an OpenAI-compatible base URL per workflow, for anything that already speaks that protocol
- `POST /mcp` — the workflow appears as an MCP tool

The routes are listed on [Desktop local API](https://usewrit.app/docs/local-api), and the
[SDKs](https://usewrit.app/docs/sdks) find a running Writ Desktop by themselves.

**Watch a page.** Point a monitor at a URL or a selector and get told when it changes.
Checks run on a schedule from your own machine, while Writ Desktop is running. Slack, Discord and
Telegram alerts are free; email and SMS alerts need a linked Writ account.

**Chain it into automations.** Trigger on a change, a schedule, a webhook, or another
workflow finishing. Send the result to a connector, a file, or the next workflow.

**Harvest a site.** Point it at a domain and let it collect pages into structured data,
including PDFs and scanned documents.

**When a run needs you.** A run that meets a sign-in, a verification code or a CAPTCHA pauses and
asks for you — an alert from your system, and the **Needs you** page in the app. You finish the step
in the live browser and the run goes on. Writ Desktop never solves CAPTCHAs by itself: you solve them.

**Bring your own sign-ins.** **Settings → Import logins → Import from browser** reads the sites
you are already signed in to in Chrome, Edge or Firefox (up to 50 at a time) and saves them as
personas on this computer. It never reads your saved passwords.

**Use your own AI key.** In **Settings → AI Provider**, pick OpenAI, Anthropic (Claude), Google
Gemini, OpenRouter, Ollama (local) or a custom OpenAI-compatible server, paste the key and click
**Test connection**. With your own key, Scribe and builds are unlimited; the key is encrypted on your
computer and your provider bills you directly. Writ's own AI is available through a linked account
and is billed in credits.

## Connect your AI (MCP)

Writ Desktop runs a local MCP server, named `writ`, so an assistant can call your workflows as
tools. The app writes the config for you — open **MCP & APIs**, tab **Endpoints**, section **MCP
tools**, pick your client in **2. Pick a client**, and click **Add to {client}**. Claude Code, Claude
Desktop, Codex, Cursor, Windsurf and VS Code are one-click; anything else that speaks MCP can point at
the same local endpoint (the command under **Local command**, or `http://127.0.0.1:8131/mcp` with a
local key from **MCP & APIs → API Keys**).

Your assistant then gets your recorded workflows as callable tools, running locally,
with no key to paste and no page for the model to re-interpret. Setup for every client is on
[Connect an AI assistant](https://usewrit.app/docs/mcp).

## Reach it from other devices

Everything you do on this computer stays free on every plan. Three ways let other machines call the
APIs on it:

| Way in | What it does | Plan |
|---|---|---|
| Your Writ account | A schedule, a published endpoint or an assistant reaches your computer through Writ Cloud, over a connection the app opens outward. No port to open, no public address. Turn on **Callable from the cloud** per workflow. | Every plan |
| LAN mode | **Settings → Engine & Data → Network access → Expose on local network**: a phone, a server or another computer calls the engine directly with a local key. | Paid plans |
| Public URL | **Start public URL** gives this computer a public `https` address through Cloudflare Tunnel. It changes each time and closes when Writ quits. | Paid plans |

See [Link Desktop to Writ Cloud](https://usewrit.app/docs/uplink) and
[Writ Desktop](https://usewrit.app/docs/desktop) for the details.

## Where your data lives

| Thing | Where |
|---|---|
| Workflows, runs, results | encrypted SQLite database under `~/.writ` |
| Credentials and secrets | OS keyring where available, encrypted vault otherwise |
| The engine | a background process bound to `127.0.0.1` only (unless you turn on LAN mode) |
| Cloud | nothing, unless you link an account |

**Settings → General → This machine → Keep everything on this computer** goes further: Writ Cloud
is then contacted only to sign you in, check your plan and download your own cloud items when you
ask. **Settings → Engine & Data → Privacy & Data** makes an encrypted backup of your data, restores
it, and deletes everything.

[`SECURITY.md`](./SECURITY.md) is the authoritative statement of what is protected and —
just as importantly — what is **not** claimed. It is deliberately narrower than most
security pages. Nothing in this README should be read as exceeding it.

## Open source

Writ Desktop is proprietary. These projects around it are open source:

| Project | What it is |
|---|---|
| [`usewrit/writ-sdks`](https://github.com/usewrit/writ-sdks) | Python, TypeScript, Rust, and Go clients for Writ Desktop and Writ Cloud |
| [`writ-mcp`](https://www.npmjs.com/package/writ-mcp) | the MCP connector for Writ Cloud, on npm |
| [`usewrit/writ-agent`](https://github.com/usewrit/writ-agent) | the agent that connects a computer or server you control to your Writ account, with no interface (Writ Desktop is the full app) |
| [`usewrit/writ-plugins`](https://github.com/usewrit/writ-plugins) | the Writ plugin for Claude Code, Codex, Gemini CLI, Cursor and VS Code |

Every open-source component bundled in the app is listed with its full licence text in
[`THIRD_PARTY_NOTICES.md`](./THIRD_PARTY_NOTICES.md), which is regenerated from the
dependency graph on every release and also readable inside the app under
**Settings → About → Open-source licences**.

## Support

- **Bug or crash** → [open an issue](https://github.com/usewrit/writ-desktop/issues/new/choose).
  Include your version (**Settings → About**), OS, and what you were doing.
- **Security vulnerability** → do **not** open a public issue. Follow [`SECURITY.md`](./SECURITY.md).
- **Everything else** → support@usewrit.app

Release notes for every version are in [`CHANGELOG.md`](./CHANGELOG.md).
