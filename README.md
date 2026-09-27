# site/ — the three pages the App Store listing points at

`ios/fastlane/metadata/en-US/{support,privacy,marketing}_url.txt` are not decoration:
App Review opens them by hand, a support URL is mandatory, and a broken privacy
policy URL is one of the top rejection causes. This directory is the source of those
pages.

**Why it exists at all.** The kit carried `https://mobpipe.github.io/motif` in all
three URL fields for months. Every gate was green — the fields were present,
non-blank and in-limit — and all three answered 404, because no GitHub account named
`mobpipe` ever existed. A URL is just a string to any check that only reads files.
`mobpipe/distribution/ios_listing/reachability.py` now opens them, and the release
gate refuses a kit whose links do not answer 200.

## Where it is published

Public repo **[AndreyGordeev/motif](https://github.com/AndreyGordeev/motif)**, served
by GitHub Pages from `main` at the repository root:

| Page | URL |
|---|---|
| Marketing | https://andreygordeev.github.io/motif/ |
| Privacy policy | https://andreygordeev.github.io/motif/privacy/ |
| Support | https://andreygordeev.github.io/motif/support/ |

Pages cannot serve from this repository: `mobpipe` is private.

## Updating a page

**The privacy page is generated, never edited here.** Its text is
`docs/store/privacy-policy.md` (English and Russian); render it with

```
python -m mobpipe.distribution.store_compliance.policy_html
```

and commit `privacy/index.html` together with the source. The push gate
(`python -m mobpipe.distribution.store_compliance`) refuses a page that is not exactly
the rendering of the source, scans every file of this directory for claims Layer I made
false (bundled packs, no server, purchase checked only on the device), and refuses a
store kit whose `privacy_url.txt` points anywhere but the policy URL above.

Marketing and support are edited here by hand. Then mirror the directory into the
`motif` checkout and push — the site repo carries no build step, the files are served as
they are (a `.nojekyll` marker keeps Pages from reprocessing them):

```
git clone https://github.com/AndreyGordeev/motif.git
# copy site/* over it (README.md included), keeping .nojekyll
git commit -am "…" && git push
```

Both release paths (`scripts/asc-release-when-ready.mjs`, `scripts/release-android.ps1`)
then run

```
python -m mobpipe.distribution.store_compliance.published
```

which fetches the served privacy page and its stylesheet and refuses the release if they
are not byte-for-byte the committed ones: an unpushed mirror kept the August policy —
the one that put every pack in the app binary and denied any server of ours — live
after the packs moved to download-after-purchase. The listing gate still checks that all
three URLs answer 200:

```
MOBPIPE_BUNDLE_ROOT=<exported bundle> python -m mobpipe.distribution.ios_listing
```

## Keeping them true

The privacy page states what the app does: no account, no ads, no analytics; a bought
pack is fetched from the developer's delivery Worker with the signed store purchase,
which is checked and neither stored nor logged; and one optional coordinate goes to
Apple WeatherKit (iOS) or WeatherAPI.com (Android). That text has to agree with
`ios/fastlane/metadata/privacy_nutrition_label.json` and
`android/fastlane/metadata/android/data_safety.json`, which
`mobpipe.distribution.store_compliance` proves against the shipped code. If the app ever
calls another endpoint, sends another field, adds analytics or keeps anything a request
carries, the policy source changes with those files — before the build ships, not after.
