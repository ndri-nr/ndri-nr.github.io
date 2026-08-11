# ndri-nr.github.io

This repo exists for **one file**: `app-ads.txt`.

## Why it has to be its own repo

AdMob verifies an app by taking the domain from the app's Play Store listing and
crawling `https://<domain>/app-ads.txt` — the **root of the domain**. The path in
the listing URL is ignored.

The publisher site lives in the `artivy` repo, which is a GitHub Pages *project*
site served at `ndri-nr.github.io/artivy/`. A project site can never answer a
request for the domain root, so `app-ads.txt` placed there would never be found:

```
https://ndri-nr.github.io/artivy/app-ads.txt   -> exists, never crawled
https://ndri-nr.github.io/app-ads.txt          -> what AdMob actually fetches
```

Only a repo named exactly `<user>.github.io` serves that root. Hence this repo.

## Setup, once

1. Create a **public** GitHub repo named exactly `ndri-nr.github.io`.
2. Push this directory to it on `main`.
3. Settings → Pages → Source: `main`, folder `/` (root).
4. Confirm it is actually served before touching AdMob:

   ```bash
   curl -s https://ndri-nr.github.io/app-ads.txt
   ```

   That must print the `google.com, pub-...` line. If it prints HTML, Pages is
   not serving yet — wait and retry rather than assuming the file is wrong.
5. In AdMob, use "check for updates". The crawl is not instant; Google says "a
   few moments" and in practice it can take hours.

## Also required, and easy to miss

The **Website** field in the Play Console store listing must be filled in and its
domain must be this one (`Store listing → Contact details → Website`). If it is
empty, AdMob has no domain to crawl and the verification failure looks identical
to a malformed file.

## app-ads.txt

One line, no leading spaces, LF line ending:

```
google.com, pub-8668013395284480, DIRECT, f08c47fec0942fa0
```

`pub-8668013395284480` is the Artivy AdMob publisher ID.

## Adding AdMob for the other apps

**Nothing here changes.** `app-ads.txt` belongs to the *domain*, not to an app:
the line authorises that publisher ID to sell inventory for **every** app whose
store listing points at this domain. Kata·Word and StackO! are on the same AdMob
account, so they verify against this same single line.

The only per-app step is on the store side: each listing's **Website** field must
carry this domain. Miss it on one app and only that app fails verification, with
the same misleading "app-ads.txt problem" message.

A new line is needed only when a genuinely different seller appears:

- a second AdMob account, i.e. a different `pub-` ID
- a mediation partner selling the inventory, which is a `RESELLER` line rather
  than `DIRECT`, with that partner's own ID and certification authority ID

Adding a line never invalidates the others — the file is a list of authorised
sellers, and crawlers read every line.

## Deliberately not here

- **No custom domain.** Buying one and pointing it at the `artivy` repo would
  move that site to the domain root and change its URL structure from
  `/artivy/pawdoku/...` to `/pawdoku/...`. Those URLs are hard-coded into
  shipped apps (`pawdoku/lib/screens/home_screen.dart`,
  `wordle/lib/models/game_config.dart`, `stacko/scripts/menu_ui.gd`) and PawDoku
  is already live on Play, so a broken privacy-policy link is a compliance
  problem, not a cosmetic one. If a domain is ever added, attach it to **this**
  repo instead and verify the old links still resolve before relying on it.
- No build step, no dependencies. GitHub Pages serves the files as they are.
