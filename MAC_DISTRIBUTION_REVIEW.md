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

No Gatekeeper policy or quarantine attribute is disabled or removed as part of this repair. Previously downloaded files remain unchanged, preserving their checksum/provenance. No new Mac browser-download release can be certified until the publisher connects Apple Developer credentials and the signed workflow and download validation pass.
