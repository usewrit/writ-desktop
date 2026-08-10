# Updates

Writ Desktop updates itself. This page describes what that means, what is verified
before anything is installed, and how to turn it off or point it somewhere else.

## What happens

1. The app asks the release channel whether a newer version exists.
2. If one is announced, the app fetches the release **manifest** — a small signed
   document describing the version, the artifacts, and their hashes.
3. The manifest's signature is checked against an Ed25519 public key **compiled into the
   application**. The key is not fetched, so a compromised update server cannot supply
   its own key. The signing algorithm is fixed in code; there is no negotiable algorithm
   field to downgrade.
4. Only if that check passes does the app download the installer, confirm its hash
   matches the manifest, and stage it.
5. The update applies on the next restart.

Anything that fails — an unknown signing key, a bad signature, a hash mismatch, an
expired manifest, or an attempt to move you to an *older* version — is refused, and the
app carries on running the version you already have. There is no path where an
unverified update is installed.

## Turning it off

**Settings → General → Updates.**

- **Automatic** (default) — check, verify, and stage updates in the background.
- **Disabled** — never check. You update by downloading a release yourself.

Disabling updates does not disable anything else. The app is fully functional offline.

## Managed deployments

If your organisation delivers a machine policy, it can:

- force **`update_mode`** to `disabled`, so the app never checks; or
- set **`update_mode`** to `managed` together with an **`update_feed_url`** pointing at
  an internal mirror you control, so machines fetch both the manifest and the artifacts
  from inside your network.

A machine policy is not user-overridable when present. Signature verification is
**not** relaxed for an internal feed: a mirrored manifest must still be signed by the
key compiled into the application, so mirroring is a distribution decision, not a trust
decision.

## Release channels

| Channel | Who it is for |
|---|---|
| `stable` | everyone, by default |
| `beta` | opt-in, pre-release builds |

Pre-release versions are marked as such on the releases page.

## Verifying an update by hand

If you would rather not let the app update itself, disable updates and install releases
manually. Every release carries `SHA256SUMS` computed over the signed artifacts — see
[`INSTALL.md`](./INSTALL.md#verifying-your-download).

## If an update fails

The app keeps running the version you have. Check **Settings → About** for the current
version, and the release notes in
[`CHANGELOG.md`](../CHANGELOG.md) for whether a known issue applies. If it keeps
failing, [open an issue](https://github.com/usewrit/writ-desktop/issues/new/choose) with
your version and OS — and see
[`TROUBLESHOOTING.md`](./TROUBLESHOOTING.md#updates-arent-installing).
