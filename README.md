# Quake Ports — SideStore sources

Two SideStore/AltStore sources that bundle the four Quake ports and update
themselves when a new release ships.

## Add the SideStore source

| Device | Source | Source URL |
| --- | --- | --- |
| iPhone / iPad | Quake ports | `https://raw.githubusercontent.com/rebelancap/quake-ports/main/apps-ios.json` |
| iPhone / iPad | All ports | `https://raw.githubusercontent.com/rebelancap/all-ports/main/apps-ios.json` |
| Apple Vision Pro | Quake ports | `https://raw.githubusercontent.com/rebelancap/quake-ports/main/apps-visionos.json` |
| Apple Vision Pro | All ports | `https://raw.githubusercontent.com/rebelancap/all-ports/main/apps-visionos.json` |

Every app here is in both sources — add either one (Quake ports carries just the Quake
family; [All ports](https://github.com/rebelancap/all-ports) carries every rebelancap port).

In [SideStore](https://sidestore.io) / [AltStore](https://altstore.io): *Sources → **+** → paste the URL*.

On **Apple Vision Pro**, first install SideStore onto the headset with
[iloader](https://github.com/rebelancap/iloader/releases#release-visionos) (SideStore/AltStore can't be
installed on visionOS the usual way — iloader is what gets SideStore there). Then add the source in
SideStore exactly as above.

## How it works

`generate.py` reads the **latest GitHub release** of each app repo in `config.json`,
picks one iOS IPA (a `.ipa` whose name does **not** contain `vision`/`xros`) and one
visionOS IPA (name **does** contain `vision`/`xros`), reads each IPA's
`CFBundleShortVersionString`, and writes `apps-ios.json` + `apps-visionos.json`.

The `.github/workflows/build-sources.yml` Action runs it the moment a port publishes a
release (each port repo dispatches `app-released` via `rebelancap/all-ports`), plus a
10-minute cron as a safety net and on demand, committing the refreshed JSON. SideStore polls the raw URLs, so a new app release
propagates to users with no manual step.

## Setup (one time)

1. Push this folder as a **public** repo named `quake-ports` (or edit the raw URLs above).
2. In `config.json`, set each app's `repo`, `bundleIdentifier`, `iconURL`, and text.
3. Put icon PNGs (1024²) at `assets/<app>.png` and `assets/source-icon.png`.
4. Actions → *Build SideStore sources* → **Run workflow** once to generate the JSON.

## Requirements on the app repos

- Each release must attach exactly **one iOS IPA and one visionOS IPA**, named so the
  platform is detectable — e.g. `vkQuake-1.0.0-iOS.ipa` and `vkQuake-1.0.0-visionOS.ipa`.
- The IPA's **`CFBundleShortVersionString` must equal the version you want SideStore to
  show** (it compares that string to decide "is there an update"). Keeping it equal to
  the release tag is the simplest convention.

## Instant updates (optional)

The 3-hour schedule is usually fine. For an immediate refresh when an app repo publishes,
add this step to that repo's release workflow (needs a PAT with `repo` scope stored as a
secret, e.g. `SOURCE_DISPATCH_TOKEN`):

```yaml
- name: Refresh SideStore source
  run: |
    curl -s -X POST \
      -H "Authorization: Bearer ${{ secrets.SOURCE_DISPATCH_TOKEN }}" \
      -H "Accept: application/vnd.github+json" \
      https://api.github.com/repos/rebelancap/quake-ports/dispatches \
      -d '{"event_type":"app-released"}'
```

## Test locally

```sh
python3 generate.py          # writes apps-ios.json + apps-visionos.json
```
