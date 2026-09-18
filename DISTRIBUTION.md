# Public installers, private source

The development repository is private. Public installer/runtime assets live in `cloudarok/ragnarok-releases`. Those downloads contain executable application code; they do not include the development repository, Git history, credential files, or source ZIP.

Native installer CI builds on each supported OS, installs and launches twice, and exercises the bundled runtime. Publish only assets from a successful native run for the exact release source. Keep the source ZIP on the private release. Public publication uses an explicit filename allowlist and freshly computed checksums.

## Signing

Unsigned builds are explicitly labelled engineering previews. No signing credentials are bundled or committed. To produce a publisher-signed release, dispatch Desktop installers with `signed=true` and configure repository secrets:

- macOS: `MAC_CSC_LINK`, `MAC_CSC_KEY_PASSWORD`, `APPLE_ID`, `APPLE_APP_SPECIFIC_PASSWORD`, `APPLE_TEAM_ID`.
- Windows: `WIN_CSC_LINK`, `WIN_CSC_KEY_PASSWORD`.

The packaging command requires these credentials in signed mode and enables electron-builder's signing/notarization; missing signing fails the build. A signed release still needs native signature verification and an external clean-machine install check before changing download copy. This release has no publisher signing credentials and remains unsigned.

## Updates and rollback

Tools → Check for updates explicitly reads public GitHub release metadata. It selects a newer supported installer with an exact repository URL and SHA-256 digest, then opens release notes. It never downloads or executes an installer automatically. Stable versions rank above alpha versions. An update server error leaves the installed app unchanged.

Before an upgrade, finish or stop active runs and close every Ragnarok window. Back up the entire runtime state directory (including SQLite WAL/SHM if present) and encrypted desktop settings while the application is stopped. Install the new version and confirm the runtime and models connect. Keep the previous installer and its checksums.

To roll back, close the new app, install the previous app, and restore the matching pre-upgrade backup. Do not open newer databases with an older app without an explicit migration compatibility guarantee. OS-encrypted keys remain tied to the original OS account. Reconnect credentials on a new computer.

Automatic binary replacement, signed update manifests, production rollout rings and clean-machine publisher trust validation remain future release requirements.
