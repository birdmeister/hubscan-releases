# HubScan — quick start

HubScan is a **read-only** HubSpot health audit. It runs entirely on your own
machine, makes only read calls to HubSpot, and writes its reports to a local
folder — nothing about your portal leaves your computer.

You need two things: the **HubScan binary** (below) and a **license key** from
HubScan (a Better Call Birdman trade name).

## 1. Install the binary

Download the build for your OS:

- **macOS (Apple Silicon):** `hubscan`
- **Windows (64-bit):** `hubscan.exe`

These early builds are **not yet code-signed**, so your OS will warn the first
time you run them. That is expected:

- **macOS:** the first launch is blocked with *"Apple could not verify … is free
  of malware"* (only **Move to Trash / Done** — there is no "Open" button on
  recent macOS). This is expected for an unsigned app. Clear the quarantine flag
  once in **Terminal**, then run it:
  ```bash
  cd ~/Downloads                                 # wherever you saved it
  xattr -d com.apple.quarantine hubscan-macos-arm64
  chmod +x hubscan-macos-arm64
  ./hubscan-macos-arm64 --version
  ```
  (If `xattr` says *"No such xattr: com.apple.quarantine"*, the file simply was
  never quarantined — e.g. downloaded with `curl` rather than a browser. That's
  fine; skip that line and carry on.)
  Alternatively, after the block: **System Settings → Privacy & Security →**
  scroll down to *"hubscan-macos-arm64 was blocked…"* → **Open Anyway**.
- **Windows:** SmartScreen shows "Windows protected your PC". Click **More
  info → Run anyway**. Some antivirus may also flag it; this is a known
  false-positive for unsigned bundled apps and goes away once we sign releases.

Confirm it runs (use the filename you downloaded, e.g. `hubscan-macos-arm64` or
`hubscan-windows-x64.exe`):

```bash
./hubscan-macos-arm64 --version
```

## 2. Add your license key

Save the `license.key` you were given to `~/.hubscan/license.key`:

```bash
mkdir -p ~/.hubscan && cp license.key ~/.hubscan/license.key
./hubscan license          # shows partner, expiry, days left
```

One key works for every consultant at your firm. To share it from a single
place, point each machine at a synced/network copy with
`HUBSCAN_LICENSE_FILE=/path/to/license.key`.

## 3. Connect a HubSpot portal

In the client's HubSpot, create a **Service Key** under **Settings →
Integrations → Service Keys → Create service key**. (Not *Private Apps*: that is
a different menu and a different credential, even though the token looks the
same.) On the **Scopes** tab, search each name below and tick it. All 19 are
read-only:

```
crm.objects.contacts.read      crm.schemas.custom.read
crm.objects.companies.read     crm.objects.custom.read
crm.objects.deals.read         settings.users.read
tickets                        settings.users.teams.read
crm.objects.owners.read        automation.sequences.read
crm.objects.users.read         automation
crm.objects.forecasts.read     account-info.security.read
crm.objects.leads.read         scheduler.meetings.meeting-link.read
crm.objects.quotes.read        content
communication_preferences.read
```

Two things to know before you go looking for more:

- `tickets` is the whole scope name; `automation` is needed *on top of*
  `automation.sequences.read`, and `settings.users.teams.read` *on top of*
  `settings.users.read`. Those pairs look redundant; they are not.
- **`crm.objects.{tasks,notes,calls}.read` have no toggle anywhere.** HubSpot
  grants those reads implicitly. Nothing to tick, nothing to ask support for.

Copy the access token into `HUBSPOT_TOKEN`, then check the portal is reachable
and every scope landed:

```bash
export HUBSPOT_TOKEN=<your-service-key>
./hubscan probe --portal=clientname
```

`probe` names any scope that is genuinely missing: tick it, click **Update
changes** in HubSpot (the token itself stays valid), and re-run until it passes.

[`SERVICE_KEY.md`](https://github.com/birdmeister/hubscan-releases/blob/main/SERVICE_KEY.md),
also alongside this file in your download, explains what each scope is read for,
what a client loses by withholding one, and what the three kinds of `403` mean. Worth
a read the first time a portal misbehaves, because a `403` that does *not* say
`MISSING_SCOPES` is usually a stale key that needs reissuing, not a scope
problem.

## 4. Run the audit

```bash
./hubscan scan --portal=clientname
```

Reports (Markdown, HTML, CSV, plus an API-call trace) are written to
`output/clientname/`. Use `--output=<dir>` to change the location and
`--help` to see all options.

## Updates

HubScan ships frequent improvements. When a newer version is available the tool
prints a one-line notice — you choose whether and when to download it; nothing
updates automatically.

## Uninstalling & cleaning up

HubScan has no installer — it's one file plus a small settings folder. To remove
it completely:

1. **Revoke the client's access in HubSpot.** Deleting files on your machine does
   **not** disable the token — the Service Key keeps working until you delete it
   in the portal: **Settings → Integrations → Service Keys → delete the HubScan
   key**. Do this for every client portal you connected.
2. **Delete the settings folder** (it holds your saved tokens and licence key):
   - macOS / Linux: `rm -rf ~/.hubscan`
   - Windows: delete the `.hubscan` folder in your user folder
     (`%USERPROFILE%\.hubscan`)
3. **Delete the report files** you generated — they contain client data — and the
   `hubscan` binary itself.

To disconnect just **one** client without uninstalling, do step 1 for that portal
and remove its entry from your settings folder.

## Licence

Use of HubScan is governed by the End-User Licence Agreement (`EULA.md`). In
short: read-only, runs locally, your data never reaches us; the licence covers
all your consultants until your key's expiry date.

## Support

Questions or a license/renewal: **info@hubscan.nl**.
