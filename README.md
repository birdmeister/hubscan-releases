# HubScan releases

Public distribution point for **HubScan** binaries, documentation and the in-app
update manifest. Source code lives in a private repository; only built artifacts,
docs and the signed update manifest are published here.

- **[`SERVICE_KEY.md`](SERVICE_KEY.md)** -- creating the read-only HubSpot
  Service Key HubScan runs on: which scopes to tick, what each one is read for,
  and what a `403` actually means. Read this first when connecting a portal.
- **[`QUICKSTART.md`](QUICKSTART.md)** -- install the binary, add your licence
  key, connect a portal, run an audit.
- **`manifest.signed`** -- the version manifest HubScan's update check reads,
  with a `stable` and (when a pre-release is out) `beta` channel. It is
  Ed25519-signed, and every build verifies that signature against a key compiled
  into it before acting on anything the manifest says: `hubscan update` replaces
  the running binary, so what the manifest claims decides what code runs on your
  machine. A manifest that does not verify is refused and says so. From 0.16.0
  onward this is the only manifest published -- the earlier unsigned
  `manifest.json` is gone, so builds older than 0.16.0 no longer see update
  notices and should be replaced once from the releases below.
- **Releases** -- downloadable macOS (arm64) and Windows (x64) binaries per
  version. Each release also attaches the two documents above and the licence
  terms, so one download carries everything.

All of this is populated automatically by the release pipeline on each version
tag, this page included. The documents always describe the newest release; older
versions keep their own copies attached to their release.

HubScan is a Better Call Birdman trade name (KvK 84948396). Support:
info@hubscan.nl
