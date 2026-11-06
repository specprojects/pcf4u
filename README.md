# PCF4U – Android app (React Native + Expo)

This is the members' mobile app for **pcf4u.com**, the new platform of the Persecuted Christian Forum (PCF). It is built from the PCF4U Project Specification. The logo, brand colours and contact details come from persecutedchristianforum.com and the spec.

> **Sample mode.** The pcf4u.com server (Laravel API, spec section 8) does not exist yet, so the app runs on built-in **sample data** and every screen works offline. Once the API is live, set `API_BASE_URL` in `src/api/config.ts` and the app switches to real data. No screen code needs to change. Sample content is labelled in the app.

## Features (mapped to the spec)

| Spec | In the app |
|---|---|
| §2 Home page | Hero with names in English / Telugu / Hindi and Matthew 5:10; Join Us / Donate / Emergency Help buttons; always-visible toll-free strip; 8 quick-access tiles; live statistics; mission; 4 services; events; latest news; FAQ; newsletter sign-up |
| §3 Registration | Mobile **OTP login** (SMS or WhatsApp, max 3 requests per 10 minutes). Then a **4-step registration** with every field from the spec table: relation, gender, marital status, date of birth with auto age, date of marriage, caste, mother tongue, education, profession, church, denomination, the **State → District → Mandal → Constituency → Village** chain, address and PIN, about me, passport-cropped profile photo (≤ 2 MB), family photo, consent and terms |
| §3 Status flow | Submitted → Under review → Approved, with the Member ID (PCF-0001) shown on approval |
| §4 Documents | ID card, certificate and authorised letter, priced individually or as a combo; pay → ready. **On-screen ID card**, front and back at CR80 proportions, with photo, Telugu/Hindi names, toll-free number and a QR code linking to `pcf4u.com/verify/<id>` |
| §5 Directory | 12 categories, A–Z bar, search, verified/featured badges; listing page with call, WhatsApp, directions, share and report |
| §6.1 Jobs | Search, job details, one-tap apply with your PCF profile |
| §6.2 Shop | PCF store and vendor products, prayer-linked products, cart, cash on delivery where allowed, checkout through the payment gateway |
| §6.3 Prayer | All 5 types (video, phone, in-person, site visit, mass prayer), date, slot, language, address for site visits, offering |
| §6.4 Classifieds | Browse by category, post a free ad with photos (goes to moderation), report |
| §6.5 News & e-paper | News by category, article sharing; e-paper archive with page flip, pinch zoom and "share page on WhatsApp" |
| §7 Emergency | One-tap toll-free call / WhatsApp; **incident report** (location, police station, date, description, photos) → case number; **case tracker** (New → Assigned → Legal aid → Closed) |
| Events & gallery | Events with a photo gallery per event and a zoomable full-screen viewer |
| Donate | ₹ amounts; payment on PCF's gateway page, so no card or UPI details pass through the app |
| Language | English / తెలుగు / हिन्दी switch in the drawer (menu, home page, common buttons) |

**Not in this app (per the spec):**
- Admin panel, coordinator approvals and editor tools stay on the website.
- The "partner app like Zomato" for vendors and pastors is a later phase.
- Aadhaar upload is deliberately left out of the app.

## Structure

```
App.tsx                         fonts, splash, providers (language, session, cart)
app.json                        name, package com.pcf4u.app, icon, splash, permissions
src/api/       config.ts (contacts, API URL, PLACEHOLDERS) · client.ts (HTTP + secure token)
               index.ts (every API call; sample fallbacks) · mockData.ts (SAMPLE content) · types.ts
src/components ui · DrawerContent · FormFields · FormStatus · LocationFields · PhotoPicker
               IdCard · GalleryGrid · ZoomableImage
src/screens    27 screens (Home, Emergency, CaseTracker, About, News, Article, EPaper, EPaperReader,
               Events, EventDetail, Directory, Listing, Jobs, JobDetail, Shop, Cart, Prayer,
               Classifieds, ClassifiedDetail, PostAd, Donate, Contact, Account, Login, Register,
               IdCard, PhotoViewer)
src/navigation AppNavigator (drawer + stack + deep links) · types
src/i18n       English / Telugu / Hindi strings
src/theme      colours (logo orange #EC5A2C, yellow #FCC200), fonts, spacing
```

## Connecting the real server

Set `API_BASE_URL = 'https://pcf4u.com/api/v1'` in `src/api/config.ts`. The endpoints the app expects are listed beside each function in `src/api/index.ts`: `/auth/otp`, `/auth/verify`, `/me`, `/members`, `/me/documents`, `/directory`, `/jobs`, `/products`, `/orders`, `/prayer-bookings`, `/classifieds`, `/incidents`, `/news`, `/epaper`, `/events`, `/locations`, `/stats` and others.

Auth uses Laravel Sanctum bearer tokens, stored in the Android Keystore through expo-secure-store.

## Privacy & security
- Personal data is never logged.
- The only thing stored on the phone is the login token, and it is encrypted.
- Payments open on the gateway's own page.
- Phone numbers of classified posters stay hidden.
- Location permission is blocked; maps use Google Maps links instead.
- The only runtime permission is the camera, used for profile and incident photos.

## Build the APK and AAB

### Option A: GitHub (no Android Studio needed)
1. Push this folder to a GitHub repository.
2. Open **Actions → Build Android APK and AAB → Run workflow**.
3. After about 15 minutes, download **pcf4u-apk** (`app-release.apk`, to install on phones) and **pcf4u-aab** (`app-release.aab`, for Google Play).

### Option B: on a computer
You need Node 22, JDK 17 and Android Studio.
```bash
npm install
npx expo prebuild --platform android
cd android
./gradlew assembleRelease bundleRelease
# APK → android/app/build/outputs/apk/release/app-release.apk
# AAB → android/app/build/outputs/bundle/release/app-release.aab
```

### Signing for Google Play
Create an upload key once:
```bash
keytool -genkeypair -v -keystore pcf4u-upload.keystore -alias pcf4u -keyalg RSA -keysize 2048 -validity 10000
```
Then add these GitHub secrets:
- `ANDROID_KEYSTORE_BASE64` – the output of `base64 -w0 pcf4u-upload.keystore`
- `ANDROID_KEYSTORE_PASSWORD`
- `ANDROID_KEY_ALIAS`
- `ANDROID_KEY_PASSWORD`

Back the keystore up: every future update must be signed with the same key. Without these secrets, the build is signed with a test key. That's fine for installing on phones, but Google Play will reject it.

**Develop:** `npm start`, then `npm run android` with an emulator or a connected phone.

## Placeholders to confirm with PCF

| Item | Now |
|---|---|
| All lists (news, directory, jobs, products, classifieds, events, e-paper, statistics) | Sample data, until the API is connected |
| Email | `info@pcf4u.com`, **assumed** |
| Document prices | ₹100 ID card / ₹150 certificate / ₹200 letter / ₹350 combo, **assumed** (an open question in the spec) |
| Denominations | 9 sample names; replace with the current site's 9 options |
| Location master | A few Telangana / AP districts as samples; the real list comes from `/locations` |
| Service descriptions, FAQ answers, history, leadership | Placeholder text (the spec asks PCF for real content) |
| Telugu / Hindi translations | First drafts; need review by a native speaker |
| Payment, donation and verify pages | URLs on pcf4u.com that the web team must create |
| Logo | Rebuilt from the 675 px website logo; supply the original artwork for sharper icons |
# pcf4u
