# Public installers, private source

The development repository is private. Public installer/runtime assets live in `cloudarok/ragnarok-releases`. Those downloads contain executable application code; they do not include the development repository, Git history, credential files, or source ZIP.

Native installer CI builds on each supported OS, installs and launches twice, and exercises the bundled runtime. Publish only assets from a successful native run for the exact release source. Keep the source ZIP on the private release. Public publication uses an explicit filename allowlist and freshly computed checksums.

## Signing

Mac browser downloads are paused after discovering invalid inherited Electron signatures in alpha.17. Local Mac builds now use explicit ad-hoc signing (`identity: "-"`), with hardened runtime disabled for that local-only mode, and fail if deep/strict signature verification fails. This is not publisher signing or notarization. Windows builds without publisher credentials remain engineering previews. No signing credentials are bundled or committed. To produce a publisher-signed release, dispatch Desktop installers with `signed=true` and configure repository secrets:

- macOS: `MAC_CSC_LINK`, `MAC_CSC_KEY_PASSWORD`, `APPLE_ID`, `APPLE_APP_SPECIFIC_PASSWORD`, `APPLE_TEAM_ID`.
- Windows: `WIN_CSC_LINK`, `WIN_CSC_KEY_PASSWORD`.

The packaging command requires these credentials in signed mode and enables electron-builder's signing/notarization; missing signing fails the build. Both the completed Mac bundle and its copy installed from the final DMG must pass deep/strict verification with the Ragnarok identifier. Signed mode additionally requires the expected Developer ID team, a valid stapled notarization ticket, and Gatekeeper acceptance. Reports distinguish local signature integrity from browser distribution readiness. Do not publish Mac download buttons until `macSecurity.browserDistributionReady` is true for both Mac architectures and a browser-downloaded clean-machine launch has been checked. Signing credentials are not currently configured.

## Updates and rollback

Tools → Check for updates explicitly reads public GitHub release metadata. It selects a newer supported installer with an exact repository URL and SHA-256 digest, then opens release notes. It never downloads or executes an installer automatically. Stable versions rank above alpha versions. An update server error leaves the installed app unchanged.

Before an upgrade, finish or stop active runs and close every Ragnarok window. Back up the entire runtime state directory (including SQLite WAL/SHM if present) and encrypted desktop settings while the application is stopped. Install the new version and confirm the runtime and models connect. Keep the previous installer and its checksums.

To roll back, close the new app, install the previous app, and restore the matching pre-upgrade backup. Do not open newer databases with an older app without an explicit migration compatibility guarantee. OS-encrypted keys remain tied to the original OS account. Reconnect credentials on a new computer.

Automatic binary replacement, signed update manifests, production rollout rings and clean-machine publisher trust validation remain future release requirements.
