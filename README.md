# FitShala Channel-Attribution Redirect Page

A single self-contained static page (`index.html`) that counts one click in PostHog and
forwards the visitor to the correct app store. Zero framework, zero build step, zero
tracking beyond one analytics ping. No device fingerprinting.

---

## What it does (on load, invisible — no visible UI)

1. Reads the channel from its own `?c=` query param.
2. Fires exactly ONE PostHog event `channel_link_opened` with:
   - `channel` — bucketed enum (see below), always non-null.
   - `target_store` — `android` or `ios` (from `navigator.userAgent`).
   - `utm_source` / `utm_medium` / `utm_campaign` — only if already present on the URL (omitted otherwise).
   - `$set_once: { first_channel: <channel> }` — sets the person's first channel; first write wins.
3. Redirects (`window.location.replace`) to the Play Store (Android / unknown UA) or App Store (iOS).
   The redirect is delayed ~300ms so the analytics event reliably lands before navigation.

### Channel bucket logic (closed enum)

Valid buckets: `reddit | gym_qr | producthunt | whatsapp | other`

| URL | `channel` value |
|-----|-----------------|
| `?c=reddit` (any valid bucket) | that bucket, e.g. `reddit` |
| `?c=foo` (present but unknown) | `other` |
| no `c` param at all | `direct` |

---

## Short-URL scheme

Point each acquisition channel at the same page with a different `?c=`:

| Channel | Link |
|---------|------|
| Reddit | `https://<user>.github.io/<repo>/?c=reddit` |
| Gym QR poster | `https://<user>.github.io/<repo>/?c=gym_qr` |
| Product Hunt | `https://<user>.github.io/<repo>/?c=producthunt` |
| WhatsApp | `https://<user>.github.io/<repo>/?c=whatsapp` |

Wrap these in a short-link service (e.g. a custom domain or bit.ly) for posters/QRs if desired.
Any unknown `?c=` value is safely bucketed to `other`; a bare link with no `?c=` counts as `direct`.

The Android redirect appends `&referrer=utm_source%3D<channel>` to the Play Store URL so the
Play Install Referrer API can recover the channel inside the installed app. iOS has no
equivalent, so the App Store URL carries no referrer (expected).

---

## TODO before deploy (founder)

Edit the two placeholders near the top of the `<script>` in `index.html`:

1. **`PLAY_PACKAGE`** — currently `'com.fitshala.app'`. Confirm the final Android `applicationId`.
2. **`APP_STORE_URL`** — currently `'https://apps.apple.com/app/idXXXXXXXXX'`. Replace with the
   real App Store listing URL once the iOS app is published.

The PostHog API key embedded in the page is the **public** project key (safe to embed by design).

---

## Deployment (GitHub Pages)

1. Put `index.html` at the root of the repo/branch you serve from (or in `/docs`, or in this
   `redirect_site/` folder if you configure Pages to serve from it).
2. In the GitHub repo: **Settings → Pages → Build and deployment → Deploy from a branch**, pick
   the branch and folder that contains `index.html`.
3. GitHub serves it at `https://<user>.github.io/<repo>/`. Test each channel link above.

No build, no dependencies — the page is fully inlined and self-contained.
