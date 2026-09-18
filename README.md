# Ragnarok

Your AI development environment.

[Download Ragnarok](https://github.com/cloudarok/ragnarok-releases/releases) for macOS (Apple silicon and Intel), Windows x64, and Linux x64 (DEB or AppImage).

These are engineering previews. Current macOS and Windows installers are unsigned; macOS builds are not notarized. Review release notes and SHA-256 checksums before installing. Keep operating-system security protections enabled.

Install Git for repository work. Each desktop installer includes its runtime. Connect your own model provider in Models or try the offline demo. No paid inference credits or shared provider keys are bundled.

For the terminal workspace, install Node.js 22.13+ and choose **Tools → Install terminal command** in Ragnarok. Follow the displayed PATH instructions, keep the desktop app open, then run `rag` in your project. `rag ui` opens the shared local Web UI.

The optional Cloud runtime archive is application code for a supervised Linux worker. VM hosting and model usage require your own connected accounts. Its setup guide is included with the release.

## Install

- **macOS 13+:** Open the matching DMG, drag Ragnarok into Applications, and launch it from there.
- **Windows 10/11 x64:** Run the guided per-user setup, then launch Ragnarok from Start.
- **Linux x64:** Install the DEB with your package manager, or make the AppImage executable. AppImage requires FUSE 2 and working user namespaces. Prefer DEB if your distribution restricts AppImage sandboxing.

## Updates and rollback

Use **Tools → Check for updates** to inspect newer releases. Updates are installed manually; the app never silently replaces itself. Finish active work, close Ragnarok, and back up application state before upgrading. To roll back, reinstall the previous version and restore its matching pre-upgrade state backup; do not assume newer databases can be opened by older apps.

This repository distributes installers and runtime archives. They contain bundled application code. The development source repository and source archives remain private.
