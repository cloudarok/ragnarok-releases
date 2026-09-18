# Install Ragnarok

Desktop installers include Electron, Node and the compiled local interface. You do not need a separate Node installation for the desktop app. Install [Git](https://git-scm.com/downloads) for repository inspection, worktrees and implementation missions. Connect your own model provider for live inference; the offline demo needs no API key.

The engineering preview is distributed through the [GitHub release](https://github.com/cloudarok/ragnarok-releases/releases/tag/v0.4.0-alpha.17). Installer downloads are public. The development repository and source archive remain private. Public installers contain the bundled application code.

## Choose the right installer

| System | Package | Requirement |
| --- | --- | --- |
| Mac with M-series chip | `Ragnarok-0.4.0-alpha.17-mac-arm64.dmg` | macOS 13 or newer |
| Mac with Intel processor | `Ragnarok-0.4.0-alpha.17-mac-x64.dmg` | macOS 13 or newer |
| Windows PC | `Ragnarok-0.4.0-alpha.17-windows-x64-setup.exe` | Windows 10/11, x64 |
| Ubuntu / Debian | `Ragnarok-0.4.0-alpha.17-linux-x64.deb` | x64 graphical desktop; native runner validation uses Ubuntu 22.04 |
| Other compatible Linux desktops | `Ragnarok-0.4.0-alpha.17-linux-x64.AppImage` | x64, FUSE 2, Electron's shared libraries and permitted user namespaces |

Windows ARM, Linux ARM, 32-bit systems and every Linux distribution are not certified by this release. The macOS 13 minimum follows the bundled [Electron 44 platform requirements](https://www.electronjs.org/blog/electron-44-0).

## macOS

**Mac downloads are paused.** The alpha.17 DMGs have invalid bundle signatures, and macOS may describe the app as damaged. The file checksum is valid; downloading it again does not repair the signature. Do not bypass this error. The website will resume Mac downloads after Developer ID signing, stapled Apple notarization and Gatekeeper checks pass. Windows and Linux packages are unaffected by this Mac signing defect.

Use Apple menu → About This Mac to identify the chip. Open the matching DMG, drag Ragnarok into Applications, eject the disk image, and open Ragnarok from Applications.

Local Mac builds now receive fresh ad-hoc signatures and must pass deep, strict signature verification. Ad-hoc signing verifies bundle integrity but does not establish Apple-approved publisher identity or notarization. These builds are for local development, not normal browser distribution. Keep Gatekeeper enabled. Public Mac releases require the publisher's Apple Developer credentials.

## Windows

Run the setup `.exe`. The guided per-user installer lets you choose a destination and creates Start menu and desktop shortcuts. It does not require a machine-wide install. Restart Ragnarok after installing Git for Windows so it receives the updated PATH.

The preview does not have a trusted publisher signature. SmartScreen or an organization's policy may block it. Verify provenance and checksums; ask your administrator if required. Do not disable Windows Defender or SmartScreen globally. Signing and reputation are release requirements before promising a warning-free install.

Uninstall through Settings → Apps. Uninstalling preserves the application profile and mission history; export or remove those separately if desired.

## Linux

On Ubuntu/Debian, open the DEB with your package installer, or from its download directory run:

```sh
sudo apt install ./Ragnarok-0.4.0-alpha.17-linux-x64.deb
```

Open Ragnarok from the application menu. The package manager installs declared desktop-library dependencies.

For AppImage, enable “Allow executing file as program” in file Properties, or:

```sh
chmod +x Ragnarok-0.4.0-alpha.17-linux-x64.AppImage
./Ragnarok-0.4.0-alpha.17-linux-x64.AppImage
```

AppImage may require your distribution's FUSE 2 compatibility package. Some distributions restrict unprivileged user namespaces; prefer the DEB on supported Ubuntu/Debian systems in that case. Do not use `--no-sandbox` as an installation workaround. The desktop compositor controls transparent overlay behavior; Wayland and unusual window managers need separate visual validation.

## Integrity and release validation

Release assets include SHA-256 lists and installation-test reports. On macOS use `shasum -a 256 <file>`; on Linux use `sha256sum <file>`; on Windows use PowerShell `Get-FileHash <file> -Algorithm SHA256`. Compare with the matching release checksum. Checksums detect corruption but do not replace a trusted code signature.

The native installer workflow builds on Apple silicon, Intel Mac, Windows x64 and Linux x64 runners. It installs/copies the produced package, launches the installed executable twice with an isolated profile, and checks rendering, the bundled runtime/SQLite, Git and native shell commands, worktrees, file tools, change evidence, authentication, approvals and an offline mission. DEB installation is tested directly; the AppImage also launches in its self-extracting mode. FUSE-mounted launch still depends on the host distribution. These tests do not simulate browser quarantine, SmartScreen reputation or every Linux compositor.

## Source, CLI and local Web UI

The source ZIP includes the compiled local interface. Install Node.js 22.13+ and Git, extract the archive, then in its folder run:

```sh
node server.mjs
```

Open the localhost URL it prints. The CLI is `node bin/rag.mjs`; `run --demo` exits 2 because a demo does not certify an implementation. On a Git clone, first run `npm ci` and `npm run build:command`. Windows live commands use Command Prompt; macOS/Linux use the POSIX shell. Configure commands appropriate to your platform. Model subscriptions and Git must be configured separately from installing Ragnarok.

## Build installers

On the corresponding native OS, run `npm ci`, `npm run build:command`, `npm --prefix desktop ci`, then `node desktop/package.mjs <darwin|win32|linux> <arm64|x64>`. The build rejects unsupported targets and cross-OS packaging. Output is in `outputs/installers/<platform>-<architecture>/`. Run `node desktop/verify-install.mjs` with that directory to verify the installation in a disposable profile.

To distribute a verified build, run **Desktop installers** on `main`, then run **Publish verified desktop release** with its successful run ID. Publication checks the source commit, all four installation reports and the original installer checksums before creating a prerelease. Linux filenames are normalized to `x64` for downloads without changing their contents. The source ZIP is built from the exact installer source commit. Repository visibility is preserved. Enable the website release entry only after its assets have been uploaded successfully.
