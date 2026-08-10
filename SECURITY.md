# Security Policy — Writ Desktop

This is the **authoritative security-claims document** for the open-source Writ
desktop app (the single `writ-agent` / `writ-agentd` Rust process + Tauri shell).
Marketing copy, the README, and the in-app UI **must not exceed** the bounded
claims stated here. It implements **PRODUCTION_READINESS.md PROD-21** and is the
single source of truth referenced by **PROD-24** for the bounded-claims
reconciliation.

> **Scope of this file.** This covers the *local-first desktop application* —
> the `writ-agent` crate built on `--features local`
> as the `writ-agentd` daemon, plus the Tauri shell. The hosted **Writ Cloud**
> service has its own separate security documentation and its own disclosure
> intake. Org-level SOC 2 attaches to Writ Cloud, **not** to the single-user
> offline desktop.

---

## Reporting a vulnerability

**Please report security issues privately. Do not open a public GitHub issue for
a vulnerability.**

Use **either** channel:

1. **Email:** `security@usewrit.app` — PGP key available on request. Encrypt
   anything sensitive (proof-of-concept, exploit details).
2. **GitHub private advisory:** open a draft advisory via *Security → Report a
   vulnerability* on the `writ-desktop` repository. This keeps the report private
   until a coordinated fix ships.

Please include: affected version (see the supported-versions table below), OS +
architecture, reproduction steps or a proof-of-concept, and the impact you
believe it has. **Never include real credentials, secrets, or extracted data** in
a report — redact them.

### Response targets (coordinated disclosure)

We follow a coordinated-disclosure model. Indicative targets (best-effort for an
OSS project, not a contractual SLA):

| Stage | Target |
|---|---|
| Acknowledge receipt | within **3 business days** |
| Initial triage + severity assessment | within **10 business days** |
| Fix or mitigation plan communicated | within **30 days** for High/Critical |
| Public disclosure / advisory | **coordinated** with the reporter, typically after a fix ships or **90 days**, whichever comes first |

If we cannot meet a target we will say so and keep you updated. We will credit
reporters in the advisory and the CHANGELOG **Security** section unless you ask to
remain anonymous.

### Safe harbor

We will **not** pursue or support legal action against good-faith security
research that:

- respects the disclosure process above and gives us reasonable time to fix
  before public disclosure;
- targets **your own installation** of the app (the desktop app is local-first;
  do not attack other people's machines or the hosted Writ Cloud
  infrastructure of other tenants);
- does not exfiltrate data beyond the minimum needed to demonstrate the issue,
  does not degrade service for others, and does not violate privacy or applicable
  law.

Activity consistent with this policy is considered authorized. If in doubt about
whether an action is in scope, ask first at `security@usewrit.app`.

### Bounty

There is **no paid bug-bounty program at launch.** Intake is **coordinated
disclosure via private advisory / email with safe harbor** only. We credit
reporters publicly (with consent). A paid bounty may be introduced later; this
section is the authoritative statement until then.

---

## Supported versions

Security fixes are backported to the versions below. We are a fast-moving
single-binary OSS app, so the supported window is intentionally narrow:

| Version line | Supported | Notes |
|---|---|---|
| Latest minor (e.g. `1.x` newest) | ✅ Yes | All security fixes land here first. |
| Previous minor | ✅ Yes (security only) | Critical/High fixes backported until the next minor ships. |
| Older minors | ❌ No | Upgrade to a supported line. |
| Pre-1.0 / beta builds | ❌ No | Dogfood/beta builds receive no backports. |

The app version is single-sourced from the `writ-agent` crate's `Cargo.toml`
(`writ-agent` crate, currently `1.0.0`). The bundled **Chromium** revision is
versioned on a **separate axis** and updated
independently; a Chromium security update may ship as a patch release without an
app feature change. Check `CHANGELOG.md` for the per-release Security section.

---

## What is protected — and what is NOT (bounded scope)

This section is the load-bearing honesty boundary. It mirrors
**SECURITY_AND_ENTITLEMENTS_SPEC.md §1.6** and **DESKTOP_APP_PLAN.md §0** exactly.
**No other document or marketing surface may claim more than this.**

### At-rest encryption — what it IS

All local data is encrypted at rest using three layers
(SECURITY_AND_ENTITLEMENTS_SPEC.md §1.1):

- **Layer A — whole database.** Everything in `writ.db` (and its `-wal`/`-shm`)
  — workflow steps, runs, extracted data, monitor content, automations, **all
  columns** — is encrypted with **SQLCipher (AES-256)**, transparent per-page.
- **Layer B — field AEAD.** Credential and secret blobs are additionally sealed
  with **XChaCha20-Poly1305** (magic `WF1`, random 192-bit nonce, AAD bound to
  `table|column|row_id`) on top of Layer A — defense in depth for the
  highest-value columns only.
- **Layer C — files.** `~/.writ/files`, `artifacts/` (workflow output,
  screenshots, diffs), and `logs/` are encrypted with **age STREAM**
  (ChaCha20-Poly1305, 64 KiB chunks; per-file key derived via HKDF).

The root key (random 32 bytes) lives in the **OS keyring**
(`writ-vault/master`); every layer key is HKDF-derived from it. In-memory key
material uses `secrecy` + `zeroize`.

**Threat model this defends against:** a **stolen or lost laptop**, **another
local user or process**, **malware reading files at rest**, and **leaked backups
or copied `~/.writ` directories**.

**Bounded compliance claim:** *"AES-256 / XChaCha20-Poly1305 encryption at rest,
satisfying SOC 2 CC6.1 / GDPR Art. 32 **intent**."* This is an intent/posture
claim, not an attestation.

### What is explicitly NOT claimed

We deliberately do **not** claim any of the following, and you should treat any
copy that does as a bug to be fixed:

- ❌ **Not FIPS-validated.** RustCrypto (`chacha20poly1305`, `argon2`, etc.) and
  `age` are not FIPS 140-validated cryptographic modules.
- ❌ **Not HSM-backed.** Keys live in the OS keyring / a `0600` file fallback,
  not a hardware security module.
- ❌ **No anti-swap pinned memory.** `zeroize` is best-effort; Rust/LLVM register
  spills and OS swap can retain plaintext copies. Memory is not pinned.
- ❌ **Not full-disk-encryption equivalent.** At-rest encryption protects the
  data files; it is not a substitute for OS full-disk encryption on a *running,
  unlocked* machine.
- ❌ **No zero-knowledge once linked.** When you opt into Writ Cloud, a metered
  cloud call necessarily transmits the call inputs, the chosen result, and
  duration/status metadata. We do not claim zero-knowledge after linking.
- ❌ **No protection on a running, unlocked machine** against a privileged local
  attacker. Once unlocked, the key is in memory by design.
- ❌ **Recipe/IP is not cryptographically protected.** Workflow recipes are
  protected only by access control and the metadata-only catalog projection
  (see CLOUD_LINK_CONTRACT.md), not by cryptography against a determined operator
  of their own machine.
- ❌ **gVisor / isolated-execution tier is not operational** in the desktop and
  is **not advertised** as an entitlement (`isolated_execution = "n/a"`).
- ❌ **True single-file crypto-erase is not guaranteed.** Under a shared
  file-master key, per-file crypto-erase is impossible without a wrapped-per-file
  key column; "Delete my data" is best-effort `secure_delete` + DB row removal.
  See the data-erasure stance below.

---

## The `vault.key` file-fallback disclosure (weakest posture)

When no OS keyring is available — the canonical case is **headless Linux with no
Secret Service** — the root key falls back to a file at `~/.writ/vault.key`.

- The file is created `0600`, opened with `O_NOFOLLOW`, and its owner + mode are
  checked on every open. On Windows the fallback is **DPAPI-wrapped**.
- **This is the weakest at-rest posture.** Anyone who can read your home
  directory (another local admin, a backup that captured `~/.writ`, malware
  running as your user) obtains the root key and therefore the **entire decrypted
  database**. The keyring-backed path does not have this exposure.
- The app **logs this fallback loudly** at startup so it is never silent.
- On servers / headless deployments, gate the root behind an env-injected
  `WRIT_VAULT_WRAP` secret rather than relying on the bare `vault.key` file. The
  headless `writ-agentd` never uses the interactive app-lock passphrase (it would
  block launchd/systemd auto-restart); it uses the keyring or `WRIT_VAULT_WRAP`.

If you require strong at-rest protection on a headless host, either provide a
Secret Service keyring or set `WRIT_VAULT_WRAP`; do not rely on the bare file
fallback.

---

## Data deletion / right-to-erasure stance

The desktop is a **single-user local store**: all your data is on your own
machine, so for local data **you are your own data controller**. Cloud-retained
metadata exists only when you link, and is governed by the Writ Cloud privacy
policy.

"Delete all my data" / per-asset hard-delete performs `secure_delete` + DB row
deletion + janitor-guarded file unlink, and "reset to factory" wipes `~/.writ`
(including the keyring entry). However — per the bounded claim above — **true
single-file crypto-erase is not guaranteed** under the shared file-master key.
Erasure is honest best-effort, not a cryptographic shred guarantee. We state this
plainly rather than over-claiming GDPR "right to erasure."

---

## How the security stack is verified (so claims are not just asserted)

- **Cipher-active fail-closed gate.** `local::db::assert_cipher_active` reads
  `PRAGMA cipher_version` at open and refuses to start if it is empty (which is
  what plain SQLite returns). This is **load-bearing**: if `sqlx` ever resolved a
  cipher-less `libsqlite3-sys`, `PRAGMA key` would silently no-op and `writ.db`
  would be written **plaintext while appearing to work**. The `sqlcipher-link-gate`
  CI job builds `--features local` and runs this test on every PR
  (SECURITY_AND_ENTITLEMENTS_SPEC.md §1.5, SUPPLY_CHAIN.md).
- **Supply-chain gates.** `cargo-deny` (advisories / licenses / bans / sources)
  + `cargo audit` run as required merge gates; an SBOM is generated per release.
  See `SUPPLY_CHAIN.md` and `THIRD_PARTY_NOTICES.md`.
- **Entitlement verification is reflection-only.** Desktop-held entitlements are
  Ed25519-JWS signed by the cloud and verified against a **pinned** public key,
  but they **never** gate enforcement or billing — every paid/metered capability
  is re-resolved server-side per call. See CLOUD_LINK_CONTRACT.md and
  SECURITY_AND_ENTITLEMENTS_SPEC.md Part 2.
- **Loopback hardening.** The local API binds `127.0.0.1` **by default**, behind a
  single `0600` bearer token + Origin/Host allowlist (DNS-rebinding defense). It
  rejects cloud tokens (`wto_`/`wtr_`/`wte`) — they are never local-API tokens. LAN
  exposure is an **explicit opt-in** with the same token + rebind guards still in
  force — see **Network exposure** below.

---

## Network exposure (loopback default, opt-in LAN)

The standalone daemon (`writ-agentd`) is **loopback-only by default**: it binds
`127.0.0.1` and is **never reachable from the network**. Nothing on your LAN — or
the wider internet — can reach the local API/MCP/OpenAI surface unless you take an
explicit action.

You may **opt in** to expose the API on your LAN (so other devices on the same
network can call it — e.g. an MCP client on another machine, a home-lab box) via a
UI button. This persists `network_exposed = true` under `[server]` in
`~/.writ/config.toml` (env override `WRIT_NETWORK_EXPOSE`) and, **on the next
daemon restart**, binds `0.0.0.0` (all interfaces) instead of loopback. The bind is
fixed for a running listener's lifetime — toggling the flag never silently re-binds
a live process; the app applies it by restarting the sidecar.

The security posture when exposed:

- **The bearer token is ALWAYS required**, in both modes. Loopback and LAN callers
  alike must present a valid `wlt_` UI token or a scoped `wlk_` client key; there is
  no anonymous access in either mode.
- **DNS-rebind protection stays enforced.** The Origin/Host allowlist still rejects
  foreign browser origins (e.g. `http://evil.example`). When exposed it additionally
  accepts only the daemon's **own** bind/LAN address (loopback plus this machine's
  resolved LAN IP); a different LAN IP, a wrong port, or any foreign origin is still
  rejected. A browser page on another site cannot rebind to your daemon.
- **Defensive bind guard.** The daemon **refuses to bind any non-loopback address
  unless `network_exposed` is explicitly true**, so an accidental or forced
  non-loopback bind can never expose the API silently. Binding a non-loopback
  address emits a clear `WARNING` log line.

**Recommendations when you expose on the LAN:**

- Mint and hand out a **scoped `wlk_` key** (least-privilege: `read` or `run`,
  not the full-access `wlt_` UI token) for the calling device.
- Enable the **app-lock** so secret-touching routes stay sealed (HTTP `423`) until
  you authenticate, even from a LAN caller.
- Treat exposure as trusted-LAN-only. There is **no TLS** on the local listener —
  do not port-forward it to the public internet; tunnel over SSH/WireGuard/a
  reverse proxy with TLS if you need remote access.
- `GET /v1/network/status` reflects the **currently-running** bind
  (`exposed`/`bind_host`/`port`/`lan_url`); after toggling, `needs_restart` from
  `POST /v1/network/expose` signals the pending restart.

---

## Reconciliation

Any claim in the README, website, in-app copy, or release notes that exceeds the
**"What is explicitly NOT claimed"** list above is a defect. When in doubt, this
file wins. Cross-references: SECURITY_AND_ENTITLEMENTS_SPEC.md §1.6 / §2.1,
DESKTOP_APP_PLAN.md §0 / §9, PRODUCTION_READINESS.md PROD-21 / PROD-24,
CLOUD_LINK_CONTRACT.md, SUPPLY_CHAIN.md.
