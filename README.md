# Quake Ports — SideStore sources

Two SideStore/AltStore sources that bundle the three Quake ports and update
themselves when a new release ships.

## Add to SideStore

- **iPhone & iPad:** `https://raw.githubusercontent.com/rebelancap/quake-ports/main/apps-ios.json`
- **Apple Vision Pro:** `https://raw.githubusercontent.com/rebelancap/quake-ports/main/apps-visionos.json`

(In SideStore: *Sources → + → paste the URL*.)

## How it works

`generate.py` reads the **latest GitHub release** of each app repo in `config.json`,
picks one iOS IPA (a `.ipa` whose name does **not** contain `vision`/`xros`) and one
visionOS IPA (name **does** contain `vision`/`xros`), reads each IPA's
`CFBundleShortVersionString`, and writes `apps-ios.json` + `apps-visionos.json`.

The `.github/workflows/build-sources.yml` Action runs it every 3 hours (and on demand),
committing the refreshed JSON. SideStore polls the raw URLs, so a new app release
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
