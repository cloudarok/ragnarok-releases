# Mac download signature incident — 2026-09-18

The user's Apple silicon alpha.17 DMG exactly matched the published SHA-256 (`c3b9ddbcc36f9cda010a0379fecf0ad3825bebee703fd693da236423cde7efaa`). `hdiutil verify` passed. This was not a truncated download.

Both `codesign --verify --deep --strict` and Gatekeeper rejected the mounted application with **code has no resources but signature indicates they must be present**. Its identity was still `Electron`, with an ad-hoc signature and no team.

The release configured `mac.identity: null`, which skipped signing after electron-builder changed the bundle, executable name, metadata and resources. The inherited upstream signature was no longer valid. CI invoked the unquarantined executable directly and exercised application behavior; it did not verify bundle signatures or the browser-download trust path. Passing that test was insufficient evidence for Mac distribution.

## Corrections

- Explicit fresh ad-hoc signing for local development builds; no inherited signature left in place.
- Deep/strict/all-architecture verification with the `com.ragnarok.desktop` bundle identifier after signing and again on the app copied from the final DMG. macOS copying uses `ditto` to preserve bundle metadata.
- Signed release mode must also pass Developer ID/team verification, stapled-ticket validation and Gatekeeper assessment. Those checks cannot be replaced by an ad-hoc signature.
- Installation reports record signature and distribution readiness separately. The release workflow rejects Mac assets without signature-integrity evidence.
- Future app update checks exclude the defective Mac previews through alpha.17. Website Mac downloads are paused and the public release is annotated; Windows/Linux remain available.
- Empty CI certificate variables are normalized instead of being treated as certificate paths. Desktop startup unlocks the credential vault before launching the timed runtime handshake. Disposable installation tests do not access the user's Keychain; they fail on fatal errors instead of displaying an unattended modal and have a hard termination deadline. OS credential access remains a separate interactive validation requirement.

No Gatekeeper policy or quarantine attribute is disabled or removed as part of this repair. Previously downloaded files remain unchanged, preserving their checksum/provenance. No new Mac browser-download release can be certified until the publisher connects Apple Developer credentials and the signed workflow and download validation pass.

## Validation

The corrected Apple silicon DMG was installed into a disposable profile on macOS 26.6.2. Its app passed deep/strict/all-architecture signature verification and two complete installation smoke launches. Changing a bundled JavaScript resource made the new signature verifier reject it; restoring the exact bytes restored validity. The verifier also rejects the original published alpha.17 app and refuses to classify the repaired ad-hoc build as suitable for public distribution. Eight focused signing/update regressions, UI typecheck and the website build passed.

The local test profile deliberately excludes real Keychain credentials. Publisher signing, notarization and interactive credential access under a stable Developer ID identity still need their own checks. No replacement public Mac artifact has been released.

Both Apple silicon and Intel Mac jobs also passed signing integrity and installation/restart checks on [clean native CI runners](https://github.com/cloudarok/ragnarok-harness/actions/runs/35379489396), using source `a3c68b8`. These are ad-hoc local previews, not notarized public releases.
