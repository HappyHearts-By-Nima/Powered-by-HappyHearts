# Releasing new versions of Happy Hearts VPN

This document is the single source of truth for how release assets are named and
how the in-app update metadata stays in sync. Read it before every release so
the update check never drifts again.

## 1. Fixed release-asset naming convention (mandatory, from v1.3 onward)

Upload **`HappyHearts.apk`** to **every** GitHub release, forever. The name is
**version-independent** — it never changes between releases.

```
/assets
  └── HappyHearts.apk          ← REQUIRED, same name on EVERY release
  └── HappyHearts-v1.3.apk     ← optional, human-readable versioned copy
```

Why:

- `api.github.com/repos/<owner>/<repo>/releases/latest` is rate-limited to
  **60 requests/hour per source IP** for unauthenticated calls. Iranian
  mobile/ISP networks sit behind shared/CGNAT IPs, so this limit is frequently
  hit by unrelated users sharing an IP. The app now has a rate-limit-free
  fallback that only works if releases always carry the fixed name.
- GitHub serves a permanent redirect for
  `.../releases/latest/download/HappyHearts.apk` to the **newest** release's
  asset with that exact name. No API call, no rate limit.
- Keeping the name identical lets the app keep a stable, fixed download URL on
  every release.

Do **not** rename or remove the `HappyHearts.apk` asset on future releases, and
do not change how previously-published v1.0.1 / v1.1 / v1.2 assets are named —
only the releases from the next one onward follow this convention.

## 2. version.json — the rate-limit-free metadata fallback

`version.json` at the repo root is a tiny metadata file the app reads via
`raw.githubusercontent.com` (a CDN — **not** the rate-limited REST API) whenever
the primary `api.github.com` call is rate-limited, blocked, or throws. It must
never drift from the actual latest release.

Current schema:

```json
{
  "tag_name": "v1.3",
  "apk_name": "HappyHearts.apk",
  "apk_size": 70923706,
  "notes": "Human-readable release notes (markdown)"
}
```

The app uses `tag_name` for the version comparison, `apk_name` to build the
per-tag download URL, and `apk_size` as the strict integrity target.

### Keeping version.json in sync — automated (recommended)

The workflow `.github/workflows/update-version-json.yml` fires on every
`release` published event, fetches the release's `.apk` asset (preferring the
fixed `HappyHearts.apk`), rewrites `version.json`, and commits it back to
`main`. No manual step needed once GitHub Actions is enabled for the repo.

### Keeping version.json in sync — manual (if Actions is unavailable)

If you publish a release without Actions, update `version.json` by hand and
commit it **as part of the same release**, otherwise the fallback check will be
stale:

1. Publish the release, choosing the fixed name `HappyHearts.apk` (plus the
   optional versioned copy).
2. Copy the release body into the `notes` field and set `tag_name` to the exact
   tag (e.g. `v1.3`). If you don't know the byte size, the app's strict primary
   check simply degrades to the soft mirror path — getting it right is better.
3. Commit `version.json` to `main` and push.

## 3. Checklist for every release

- [ ] Tag the new version (e.g. `v1.3`) and title the release clearly.
- [ ] Upload **`HappyHearts.apk`** (fixed name — required).
- [ ] *(Optional)* also upload `HappyHearts-v<version>.apk` for display.
- [ ] Publish.
- [ ] Confirm `.../releases/latest/download/HappyHearts.apk` returns HTTP 200
      (it 404s today until the first fixed-name release is published).
- [ ] Confirm `version.json` was updated (automatically, or by hand).