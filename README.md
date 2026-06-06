# HubScan releases

Public distribution point for **HubScan** binaries and the in-app update
manifest. Source code lives in a private repository; only built artifacts and
`manifest.json` are published here.

- **`manifest.json`** — version manifest read by HubScan's update check, with a
  `stable` and (when a pre-release is out) `beta` channel.
- **Releases** — downloadable macOS (arm64) and Windows (x64) binaries per version.

These are populated automatically by the release pipeline on each version tag.

HubScan is a Better Call Birdman trade name (KvK 84948396). Support: martin@emlx.nl
