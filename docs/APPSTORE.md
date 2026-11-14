# Pace on the App Store (Mac App Store + iOS App Store)

Two App Store apps, built by `bash update.sh --appstore` (→ `scripts/appstore.sh`; `--mac` / `--ios` for one,
`--doctor` to list what is missing by name). Both sell through **StoreKit 2**; Razorpay, activation codes, the
self-updater, Pour, donate and GitHub links are hidden (`IS_APPSTORE`). The website build and the Developer ID Mac app
are unchanged and install beside them.

| | Mac App Store | iOS / iPadOS |
|---|---|---|
| Bundle id | `network.co.ipconfig.pace.store` | `network.co.ipconfig.pace` |
| SKU | `pace-mac-store` | `pace-ios` |
| Built from | Tauri, Cargo feature `appstore`, `VITE_APPSTORE=1 vite build --mode app` → `dist-appstore/` | `ios/project.yml` (XcodeGen), `dist-appstore/` bundled as `www/`, `PACE_APPSTORE=YES` |
| StoreKit | `mac-store/Bridge.swift` → `libpacestore.dylib` (dlopen'd by `src-tauri/src/storekit.rs`) | `ios/Pace/Store.swift` via `Bridge.swift` |
| Package | `release/appstore/Pace_mac-app-store.pkg` | `release/appstore/Pace.ipa` |
| Minimum OS | macOS 12 | iOS 15 |

The ids, prices and texts live in **`appstore.json`**, made from `src/engine/shop.js` PRODUCTS by
`node scripts/appstore-storekit.mjs` (which also writes `ios/Pace.storekit`, the local StoreKit test file Xcode's
Run uses). `appstore.sh` refuses to build while either is stale.

## 1. Once, on developer.apple.com

1. **Identifiers → +** App IDs: `network.co.ipconfig.pace.store` (macOS) and `network.co.ipconfig.pace` (iOS). Tick
   **In-App Purchase** (on by default). Nothing else.
2. **Certificates → +**: *Apple Distribution* and *Mac Installer Distribution* (CSR from Keychain Access on the Mac
   that builds). Never "Always Trust". Names go in `.env` as `APPLE_STORE_SIGN_IDENTITY` /
   `APPLE_STORE_INSTALLER_IDENTITY` (optional — the first such identity is used).
3. **Profiles → +** *Mac App Store Connect* for `network.co.ipconfig.pace.store`, ticking the Apple Distribution
   certificate above → path in `APPLE_STORE_PROVISION_PROFILE`. (`appstore.sh` checks the profile lists that
   certificate by SHA-1, and clears quarantine with `xattr -cr`.) iOS profiles are made automatically.
4. **App Store Connect → Users and Access → Integrations → Keys**: a key with role *App Manager* →
   `APPLE_API_KEY_ID`, `APPLE_API_ISSUER`, `APPLE_API_KEY_PATH` (uploads, iOS automatic signing, `asc-builds.mjs`).
5. Install **Transporter** from the Mac App Store and **XcodeGen** (`pour install xcodegen`); iOS needs Xcode itself.

## 2. The app records (App Store Connect → Apps → +)

- **macOS app**: name *Pace — Typing Test*, primary language English (U.K.), bundle id `network.co.ipconfig.pace.store`,
  SKU `pace-mac-store`. **iOS app**: same name (or *Pace Typing Test* if taken), bundle id `network.co.ipconfig.pace`,
  SKU `pace-ios`. Their Apple ID numbers may go in `APPLE_STORE_APP_ID` / `APPLE_IOS_APP_ID`.
- Category **Education**, secondary **Games** (Word). Age rating: no objectionable content → 4+.
- Price: **Free** (with in-app purchases). Availability: all countries (India is the base storefront for prices).
- Support URL `https://ipconfig.co.network/pace`, privacy policy `https://ipconfig.co.network/privacy`,
  marketing URL `https://ipconfig.co.network/pace`. Copyright *2026 Jagadeesh Kumar S*.

## 3. In-app purchases (each app: Monetization → In-App Purchases / Subscriptions)

Make them in **both** app records with the same product ids (a product id is unique per app).

**Subscription group "Pace Pro"** (Subscriptions → +): reference name *Pace Pro*; display name *Pace Pro*.
Inside it: `network.co.ipconfig.pace.pro.year` (level 1 — upgrading to it is an upgrade) and
`network.co.ipconfig.pace.pro.month` (level 2), durations 1 year / 1 month, Family Sharing off.

**In-App Purchases → +** for the rest: *Consumable* for hearts, coins and the two packs; *Non-Consumable* for lifetime;
*Non-Renewing Subscription* for the week and one-year passes.

| Product id | Type | Reference name | Price (INR) | Display name | Description |
|---|---|---|---|---|---|
| `network.co.ipconfig.pace.pro.week` | non-renewing | Pace pro-week | ₹10 | Pace Pro One week | Every Pro feature for a week, no renewal. |
| `network.co.ipconfig.pace.pro.month` | auto-renewable | Pace pro-month | ₹20 | Pace Pro Monthly | Every Pro feature, renews each month. |
| `network.co.ipconfig.pace.pro.year` | auto-renewable | Pace pro-year | ₹229 | Pace Pro Yearly | Every Pro feature, renews each year. |
| `network.co.ipconfig.pace.pro.year.once` | non-renewing | Pace pro-year-once | ₹229 | Pace Pro One year | Every Pro feature for a year, no renewal. |
| `network.co.ipconfig.pace.pro.life` | non-consumable | Pace pro-life | ₹999 | Pace Pro Lifetime | Every Pace Pro feature, for good. |
| `network.co.ipconfig.pace.hearts.10` | consumable | Pace hearts-10 | ₹10 | 10 hearts | 10 hearts for the typing games. |
| `network.co.ipconfig.pace.hearts.50` | consumable | Pace hearts-50 | ₹49 | 50 hearts | 50 hearts for the typing games. |
| `network.co.ipconfig.pace.hearts.150` | consumable | Pace hearts-150 | ₹99 | 150 hearts | 150 hearts for the typing games. |
| `network.co.ipconfig.pace.hearts.500` | consumable | Pace hearts-500 | ₹249 | 500 hearts | 500 hearts for the typing games. |
| `network.co.ipconfig.pace.coins.100` | consumable | Pace coins-100 | ₹10 | 100 Pace Coins | 100 coins for themes and extras. |
| `network.co.ipconfig.pace.coins.600` | consumable | Pace coins-600 | ₹49 | 600 Pace Coins | 600 coins for themes and extras. |
| `network.co.ipconfig.pace.coins.1500` | consumable | Pace coins-1500 | ₹99 | 1,500 Pace Coins | 1,500 coins for themes and extras. |
| `network.co.ipconfig.pace.coins.4000` | consumable | Pace coins-4000 | ₹199 | 4,000 Pace Coins | 4,000 coins for themes and extras. |
| `network.co.ipconfig.pace.coins.10000` | consumable | Pace coins-10000 | ₹399 | 10,000 Pace Coins | 10,000 coins for themes and extras. |
| `network.co.ipconfig.pace.starter` | consumable | Pace starter | ₹49 | Starter pack | 500 coins and 20 hearts. |
| `network.co.ipconfig.pace.champion` | consumable | Pace champion | ₹149 | Champion pack | 3,000 coins and 100 hearts. |

Prices: pick that rupee price point under *Price Schedule* with India as the base country; Apple fills the other
storefronts. (`priceInr` is the nearest Apple price point at or above the website price, `shopPaise`.) Each product
needs a **review screenshot** (the Store page with that item; 640×920 or larger) and **review notes** ("Credited
to the Pace wallet / unlocks Pace Pro"). Submit the products **with the first app version** (they ride along).

**App Store Server Notifications** (App Information → App Store Server Notifications), both apps, Production and
Sandbox, **Version 2**: `https://ipconfig.co.network/pace/api/apple/notifications`.

## 4. How purchases work (what to tell App Review)

- No account is needed to buy. Pro (subscription, pass, lifetime) is decided on the device from StoreKit's current
  entitlements. Coins and hearts are stored in the Pace wallet on Pace's server, so a consumable bought while signed
  out is **kept on the device** (Keychain on iOS, the app's sandbox folder on Mac) and credited the moment the person
  signs in; nothing is lost and nothing is charged twice (credited once per transaction id).
- Every transaction is verified on the server (JWS chain to Apple Root CA G3, bundle id, environment, expiry,
  revocation) before it is credited; the app finishes a transaction only after the server says ok.
- **Restore Purchases** is on the Store page (and Settings → Account); it calls `AppStore.sync()`.
- Refunds (`REFUND` / `REVOKE` notifications) take back the unspent coins / hearts; Pro ends with the entitlement.

**Review notes** (paste into *App Review Information → Notes*, both apps):

> Pace is a typing speed test with lessons and typing games. Everything is usable without an account. In-app
> purchases: Pace Pro (monthly / yearly auto-renewing subscriptions in the group "Pace Pro", one-week and one-year
> passes that do not renew, and a lifetime unlock) and consumable coin / heart packs used in the typing games.
> No sign-in is needed to buy: Pro unlocks at once from StoreKit; coins and hearts bought while signed out are kept
> on the device and added to the wallet after the optional free sign-in. Restore Purchases: Store page, top right.
> There are no outside payment links in this build. Sign-in is optional; a demo account is not needed, but you may use
> the one below to see the wallet.

Fill *Sign-in required: No*; optionally add a demo account (email + password from the accounts dashboard).

## 5. App Privacy (both apps)

*Data used to track you*: **none**. *Data linked to you* (only when the person signs in), all for **App Functionality**,
not for tracking or advertising: **Contact Info → Email Address**, **Identifiers → User ID**, **Purchases → Purchase
History**. *Data not linked to you*: **Diagnostics → Crash Data** (anonymous crash notes). Typing results stay on the
device unless the person posts a score to the leaderboard (then **User Content → Other**, linked, App Functionality).
The iOS app ships the same answers in `ios/Pace/PrivacyInfo.xcprivacy` (no required-reason APIs).

## 6. Export compliance

Pace uses only the system's HTTPS. Both builds carry `ITSAppUsesNonExemptEncryption = NO` (Info.plist), so App
Store Connect asks nothing. If asked: "uses encryption exempt under Category 5 Part 2 — standard HTTPS only".

## 7. Screenshots

`python3 scripts/appstore-shots.py` → `release/appstore/screenshots/`:

| Set | Size | Where |
|---|---|---|
| `mac/` | 2880×1800 | macOS app → Mac screenshots (up to 10) |
| `iphone/` | 1320×2868 | iOS app → 6.9-inch iPhone (smaller iPhones scale from it) |
| `ipad/` | 2064×2752 | iOS app → 13-inch iPad |

They come from `docs/screenshots/`; the phone footer (Sponsor / Get the app) is cut off. Retake them from the App
Store build before the first submission so no screen shows a feature that build lacks.

## 8. Build, upload, submit

```
node scripts/appstore-storekit.mjs          # after any PRODUCTS change, commit appstore.json + ios/Pace.storekit
bash update.sh --appstore --doctor          # names anything missing
bash update.sh --appstore                   # both; or --mac / --ios; --no-upload keeps the files for Transporter
node scripts/asc-builds.mjs                 # processing state of the uploaded builds
```

The build number (`CFBundleVersion`) is the UTC minute, `yymmddHHMM`, new on every upload; the version is
package.json's. When the build is *VALID*: TestFlight (internal testers, sandbox purchases) → the version page →
**+ Build**, add the in-app purchases under *In-App Purchases and Subscriptions*, **Add for Review → Submit**.

Local testing without App Store Connect: open `ios/Pace.xcodeproj` (after `xcodegen`), Run — the scheme uses
`ios/Pace.storekit`. (JWS from Xcode's local StoreKit are signed by Xcode, not Apple, and environment "Xcode": the
Worker refuses them by design; use a TestFlight / sandbox build to test end to end.)

## 9. What the Mac App Store build cannot do (App Sandbox)

Entitlements (`mac-store/entitlements.appstore.plist`): `app-sandbox`, `network.client`,
`files.user-selected.read-write`. That means:

- **No self-updater** (the App Store updates it) and no update check; no Pour, donate, GitHub or download links.
- **No `pace` command-line install** into `/usr/local/bin` (shell install) — hidden.
- **No Razorpay or activation codes**; Pro comes only from StoreKit (an existing website subscriber signs in and
  keeps their account-side Pro through the accounts API as usual).
- Files: only ones the person opens or saves through a panel (Open With / drag-in text, saving a result card).
  Finder services work for text passed to Pace.
- Notifications through `osascript` may be refused inside the sandbox; the in-app notice still shows.
- The single-instance socket must live in the app's container (not `/tmp`).
- Network calls run through `/usr/bin/curl`, which inherits the sandbox and its `network.client` right.
- Owner features: the owner serial check reads the hardware serial (allowed); the Owner Panel window works.

The iOS app is a WKWebView with the bundled page; it has no updater, no outside payment links and the same
StoreKit flow.
