# Changelog

All notable changes to **Writ Desktop** (the open-source `writ-agent` /
`writ-agentd` Rust app + Tauri shell) are documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

> **Versioning notes.**
> - The app version is **single-sourced** from
>   the `writ-agent` crate's `Cargo.toml` (`writ-agent` `package.version`) and
>   surfaced via `env!("CARGO_PKG_VERSION")`. CI fails a tag whose version does
>   not match `vX.Y.Z` and fails a tag with no matching entry in this file.
> - The bundled **Chromium / patchright** revision is versioned on a **separate
>   axis** and updated
>   independently of the app major; a Chromium security bump can ship as an app
>   patch release.
> - Every release entry MUST include a **Security** subsection (RELEASE_CHECKLIST.md).
> - Supported versions and the bounded security-claims scope live in
>   [`SECURITY.md`](SECURITY.md).

---

## [Unreleased]

First open-source desktop release of Writ — a **local-first** automation app: a
single Rust process (`writ-agentd`, `--features local`) hosting an in-process
browser engine behind a loopback API, wrapped in a Tauri shell. It records,
runs, and exposes web-action workflows entirely on the user's own machine; the
hosted Writ Cloud link is strictly opt-in. Nothing here makes the local app
require cloud.

### Added

- **Saved crawls: a crawl is now callable like a workflow, with freshness.** A
  `crawl_jobs` row is one RUN — its settings live on the row and its id dies with that
  run — so a crawl had no stable handle to expose as an API, and "re-crawl with the same
  settings" meant refilling a form. New `crawl_definitions` (desktop migration
  `0024_crawl_definitions.sql`) owns the configuration under a stable slug, and
  `crawl_jobs.definition_id` (`ON DELETE SET NULL`, so deleting a config never destroys
  the data its runs collected) makes runs its history.

  That history is what makes **`max_age`** answerable: a freshness *contract*, not a
  cache flag — "data collected within this many seconds is acceptable, otherwise go get
  it again". A hit returns the collected rows inline with nothing crawled; a miss
  re-crawls the saved settings. Only a `completed` run with at least one fetched page is
  reusable, so a crawl that converged having fetched nothing (the shape a host that 403s
  every page produces) can never pin an empty answer behind a long window. A cold call
  answers `202` with a crawl id and status URL, because a whole-site crawl outlives an
  HTTP request; `wait: true` blocks instead and a `504` still carries the collectable
  handle.

  Surfaced everywhere the crawl already was: local REST
  (`/v1/crawl/definitions[/{ref}][/run|/data]`, proxied to the fleet on a linked
  managed build), the stdio/HTTP MCP server (`writ_saved_crawls`,
  `writ_run_saved_crawl`, `writ_saved_crawl_data`, plus `save_as` on
  `writ_crawl_site`), all four SDKs, and a *Call this crawl* panel on the crawl page
  that saves the settings and mints a scoped key. `save_as` escalates to the `admin`
  capability because it creates reusable callable configuration — without that the MCP
  surface would have been strictly more permissive than REST for the same action.
- **`max_age` on workflow runs.** The same freshness contract now applies to running a
  saved workflow (non-streaming only — replaying stored data down a live event stream
  would misrepresent what is happening). Cache identity includes the run INPUTS, so a
  parameterised workflow can never serve one caller's answer to another; only successful
  runs are stored. Available on `writ_run_workflow`, on every generated per-workflow MCP
  tool, and in all four SDKs.
- **Official client SDKs (`sdks/`).** Hand-written, test-covered clients for the
  loopback agent API in four languages — TypeScript (`@usewrit/agent-sdk`,
  zero-dep, Node ≥ 18), Python (`writ-agent`, sync + async on httpx), Go
  (`github.com/usewrit/writ/sdks/go`, stdlib-only), and Rust (`writ-client`) —
  plus a canonical OpenAPI 3.1 spec (`sdks/openapi/writ-agent.yaml`, 73 paths /
  102 operations, lint-clean) and an openapi-generator pipeline for Java/C#/
  Ruby/PHP (`sdks/codegen/`). All four SDKs share one contract
  (`sdks/DESIGN.md`): `runtime.json`/profile discovery with liveness probing,
  uniform `Page`/error models over the daemon's mixed envelopes, typed SSE run
  events, and a `run_and_wait` helper with SSE→polling fallback. Verified by
  per-language mock-server suites (~180 tests) and a live smoke against a real
  `writ-agentd`.
- **Local-first engine + loopback API.** In-process `LocalEngine` runs recorded
  workflows; a `127.0.0.1`-bound server exposes REST, an OpenAI-compatible
  Chat/Responses surface (SSE), and an MCP server (tools auto-derived from
  recorded steps) for Claude Desktop / CLI clients.
- **Tray live-activity popover.** Left-clicking the tray icon (macOS menu bar /
  Windows taskbar; the "Live activity…" tray-menu item on Linux, whose trays are
  menu-only) opens a compact always-on-top panel anchored to the icon: live
  runs/sessions with second-ticking elapsed and inline Stop, one-click Quick run
  for the most recent workflows (optimistic live row), the monitors' next-check
  countdown, recent results, engine start, and quit — all without opening the
  app. Rows deep-link into the main window (`open_main_window` IPC →
  `tray:navigate`), and the tray menu's "Quick run…" now opens the command
  palette. The popover is a second `tray` webview window on `/tray` (same
  capability set as `main`), auto-dismissing on focus loss / Esc / outside click.
- **Watch-and-act local surface.** Monitors (with a safety interval floor —
  HTML 60s / JS 300s — and change-only persistence), automations, personas,
  secrets, and file assets, all local.
- **At-rest encryption of all local data** (three layers, see `SECURITY.md`):
  whole-DB **SQLCipher (AES-256)** over `writ.db`; field-level
  **XChaCha20-Poly1305** (`WF1`) on credential/secret blobs; **age STREAM** for
  `files/`/`artifacts/`/`logs/`. Root key in the OS keyring (`writ-vault/master`)
  with a hardened `0600` `vault.key` fallback (DPAPI-wrapped on Windows); every
  layer key HKDF-derived.
- **Optional cloud link ("appears as an agent").** OAuth 2.0 Device
  Authorization Grant pairing (`wto_`/`wtr_`), a metadata-only `local_catalog`
  advertisement of `cloud_callable` workflows, and cloud-initiated
  `run_local_workflow` dispatch with a normal `task_result` reply — the recipe
  and credentials never leave the device. Loopback `/v1/cloud/*` lifecycle REST +
  public `/api/v1/local-workflows/*` cloud REST. See
  [`CLOUD_LINK_CONTRACT.md`](CLOUD_LINK_CONTRACT.md) (`cloud-link/1`).
- **Reflection-only entitlements.** Ed25519-JWS plan/feature/limit document,
  verified against a `kid`-pinned compiled-in key map, with a 1h refresh boundary
  and a 7-day offline grace; used for UI/upsell only and **never** to gate
  enforcement or billing (all re-resolved server-side per call).
- **Supply-chain + attribution scaffolding.** `cargo-deny` (advisories /
  licenses / bans / sources) + `cargo audit` gates, a CycloneDX SBOM generator
  (`scripts/desktop/gen-sbom.sh`), the SQLCipher link gate, and
  `cargo-about`-generated [`THIRD_PARTY_NOTICES.md`](THIRD_PARTY_NOTICES.md).
- **Documentation.** `SECURITY.md` (coordinated disclosure + bounded scope),
  `CLOUD_LINK_CONTRACT.md` (versioned wire contract), `ENTERPRISE_DEPLOYMENT.md`
  (air-gapped / managed-policy / MDM / offline updates), `SUPPLY_CHAIN.md`,
  `docs/desktop/AUTO_UPDATE.md`, and `RELEASE_CHECKLIST.md`.

#### AI concierge — API builder

- **Sign in without clicking (`login_post`).** When the concierge builds a
  callable API for a site, it can now replay the sign-in as a single request
  instead of the recorded form. It reads the login `POST` from the captured
  network trace — credentials the agent holds are revealed as `{{placeholders}}`
  in the request body (JSON *and* percent-encoded form bodies) so the sign-in is
  seen to be reconstructable — and emits a `login_post` step. That step
  **replaces** the recorded fill/click login block, so the workflow becomes
  `navigate → login_post → api_call`: the in-page `fetch` establishes the session
  cookie and every later `api_call` reuses it. The `login_post` step type replays
  through the same authenticated `fetch` path as `api_call`. Opportunistic and
  safe — it falls back to the DOM login (which always stays as the fallback) when
  the sign-in body carries a token the agent does not hold (CSRF / nonce /
  authenticity token), on SSO/redirect logins, or after two rejected attempts.
- **GET `api_call` steps are callable functions.** A read-only (`GET`) `api_call`
  recorded in a workflow is now auto-detected on the workflow's **Connect** tab as
  an exposable function — parity with `extract` / `evaluate` steps — backed by a
  step group spanning the auth prelude through the call, so every invocation
  returns live, authenticated data. `POST`/mutating calls are not surfaced as data
  getters.
- **Leaner recorded API workflows.** End-of-discovery cleanup drops a `navigate`
  that only preceded `api_call` / `login_post` steps (those fetch their URL
  directly, so no positioning navigate is needed — typically the `navigate`
  between a login and its data calls). The pass is origin-safe (keeps a `navigate`
  whose data call would otherwise become cross-origin) and **never removes the
  entry navigate**, which establishes the site origin and cookie context.
- **AI provider model fallback.** The local AI engine now falls back to the
  selected provider's catalog default model when the stored model id is blank
  (e.g. a raw-API config write or a partial environment config), instead of
  sending an empty `"model"` to the provider.

#### Marketplace installs (cloud-linked)

- **Run modal collects an install's inputs.** A marketplace install's proxy
  workflow row carries no steps, so the Run modal previously opened empty (or not
  at all). `GET /v1/workflows/:id` now folds in the install's data-less attach
  manifest — required inputs, secret slots (picked by local vault **key name**;
  values never leave the vault), and a persona slot — and the modal collects and
  persists them as install **bindings**, so scheduled and background runs resolve
  the same picks. The marketplace **Installed** page's Run button routes through
  the same modal (each install row now carries its `local_workflow_id`) instead
  of blind-dispatching by slug. Loopback twins of the MCP elicitation:
  `GET /v1/cloud/marketplace/installs/{slug}/attach` and
  `POST /v1/cloud/marketplace/installs/{slug}/bindings`.
- **Uninstall removes everything.** Uninstalling a listing (or deleting its
  proxy workflow row) now releases the cloud install grant (best-effort when
  offline) and removes BOTH local rows — the sealed-recipe install row and the
  proxy `workflows` row — so a leftover proxy can no longer error with
  "reinstall the listing" and an orphaned sealed recipe is never left behind.
  New `POST /v1/cloud/marketplace/uninstall`; `DELETE /v1/workflows/:id` on a
  proxy routes through the same full uninstall.

#### Dragnet — distributed whole-site crawl (cloud / self-host)

- **Scribe crawls a whole site from one instruction.** Telling the AI concierge
  (**Scribe**) to "get all the data of this site" now launches **Dragnet**: the
  cloud / self-host coordinator maps the site (`robots.txt` + sitemap + link
  graph) and fans the pages out as **shards** across the connected agent fleet —
  each agent crawling a different slice. Shards ride the **existing** task
  dispatch as ordinary `scheduled` automation tasks (no new agent wire
  contract), and every page aggregates into one deduped, change-tracked workflow
  dataset that is read through the Workflow Data API and exposable as an API / MCP
  tool. Crawls can be **authenticated** — a linked persona is restored per shard,
  so pages behind a login are reachable. Coordinator-side politeness
  (`robots.txt`, per-page delay, a global concurrency cap) is enforced by the
  orchestrator. New coordinator REST: `POST`/`GET /api/crawl`,
  `GET /api/crawl/{id}`, `POST /api/crawl/{id}/cancel`; results via
  `GET /api/workflows/{id}/data`. Included in the full-stack self-host; see
  [`docs/DRAGNET.md`](docs/DRAGNET.md).

### Security

- **All local data is encrypted at rest** with a **fail-closed** cipher gate:
  the daemon asserts `PRAGMA cipher_version` is non-empty at open and refuses to
  start otherwise, so a cipher-less SQLite build cannot silently write a
  plaintext `writ.db`. The gate is enforced in CI (`sqlcipher-link-gate`).
- **Never-trust-BYO-agents boundary.** Identity, billing, metering, and
  success-truth live in the cloud (ws-gateway + backend); the desktop reflects
  only. A forked binary cannot bill the cloud, exceed a cloud-enforced limit, or
  grant itself a paid capability.
- **Entitlements are Ed25519-signed and verified against a pinned key** — never
  HS256 (which would ship the signing secret). Tamper/forge/stale/absent fails
  **closed** (under-reports), never grants.
- **Loopback hardening:** local API binds `127.0.0.1` only, behind a single
  `0600` bearer token + Origin/Host allowlist (DNS-rebinding defense); cloud
  tokens (`wto_`/`wtr_`/`wte`) are rejected by the local API.
- **Bounded claims (do not exceed `SECURITY.md`):** no FIPS-validation, no HSM,
  no anti-swap pinned memory, no full-disk-encryption equivalence, no
  zero-knowledge once linked, recipe/IP not cryptographically protected, gVisor
  isolation not operational. The no-keyring `vault.key` fallback is the weakest
  posture and is disclosed loudly.

### Known limitations

- True single-file crypto-erase is not guaranteed under the shared file-master
  key; "Delete my data" is best-effort `secure_delete` + DB row removal
  (`SECURITY.md`).
- Headless Linux with no Secret Service falls back to the `~/.writ/vault.key`
  file — gate behind `WRIT_VAULT_WRAP` on servers.
- Backend entitlement issuance (`build_entitlements_jws` +
  `WRIT_ENTITLEMENTS_SIGNING_KEY` + the `.well-known` JWKS) is the cloud-side
  addition tracked in SECURITY_AND_ENTITLEMENTS_SPEC.md §2.6; the desktop verify
  path is built and ships multi-key.

[Unreleased]: https://github.com/usewrit/writ-desktop/commits/main
