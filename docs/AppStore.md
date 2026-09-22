# App Store submission

Generated and length-checked by script. Limits are App Store Connect's.

## Name

**Widget Play: Car Widgets** on the App Store; the Home Screen label stays **Widget Play**
(`INFOPLIST_KEY_CFBundleDisplayName`). The ": Car Widgets" suffix puts the most
searched phrase in the name, which App Store search weights most heavily.

- Availability is only confirmed by creating the app record in App Store Connect,
  which also reserves the name. Fallback: "Widget Play for Cars".
- Write **CarPlay** as one word everywhere (Apple trademark style), and keep it
  out of the app name itself. Using it in the subtitle and description to
  describe compatibility is standard.
- Words in the name and subtitle are indexed automatically; don't repeat them
  in keywords.
- Bundle ID stays `com.nine3one2.mycarwidget`. Users never see it, and it can't
  change after the app record exists.

## Listing

| Field | Limit | Used |
|---|---|---|
| name | 30 | 24 |
| subtitle | 30 | 24 |
| promo | 170 | 150 |
| keywords | 100 | 97 |
| description | 4000 | 1742 |

**Name:** Widget Play: Car Widgets

**Subtitle:** Personalize your CarPlay

**Promotional text:** Make CarPlay your personal car display: your photos, your widgets, your startup sound. Launch price for a limited time: one purchase, no subscription.

**Keywords:** `dashboard,startup sound,maintenance,oil change,countdown,car photo,theme,lock screen,registration`

(Don't repeat words already in the name or subtitle; Apple indexes those
automatically. Commas, no spaces after them.)

**Description:**

```
Make your CarPlay screen yours, without a subscription.

YOUR CAR, YOUR PHOTOS
Put a favorite photo on your CarPlay dashboard, free. Unlock to add up to 20 and let them rotate through the day. Long-press any photo to lift your car out of its background so it floats on your theme. It's done right on your iPhone, and nothing is uploaded.

A STARTUP SOUND WHEN CARPLAY CONNECTS
Pick from built-in sounds, or import your own. A step-by-step guide sets it up once in Shortcuts, and it plays every time you get in. Add a second sound for when you park.

DESIGN YOUR OWN WIDGETS
Build as many widgets as you like: a clock, a photo, a countdown, a maintenance reminder or a note, each with its own font and colors. Schedule them to change through the day, or let smart rules switch them for you, like showing your oil change the moment it's overdue.

WIDGETS THAT ACTUALLY HELP
• Maintenance: countdowns to your next oil change, registration, inspection or tire rotation, color-coded so overdue items stand out
• Countdown: days until a road trip, birthday or anniversary, with yearly events rolling over automatically
• Quick Note: a parking spot, gate code or address, big enough to read at a glance
• Dashboard: a themed clock with your car's name

Widgets work on CarPlay, your Home Screen, the Lock Screen and in StandBy.

SIX THEMES
Midnight and Sunset are free. Unlock Mono, Carbon, Aurora and Ember.

PRIVATE BY DESIGN
No account, no ads, no tracking. Your photos, sounds and reminders stay on your iPhone.

ONE PRICE, ONCE
Everything premium is a single one-time purchase, shared with up to five family members through Family Sharing. No weekly plans, no renewals.

CarPlay widgets require iOS 26 or later and a CarPlay-compatible vehicle.
```

## Pricing

Launch at **$4.99** as an introductory price, rising to **$9.99**. The app reads
the price from StoreKit (`product.displayPrice`), so the change is made entirely
in App Store Connect: no code change, no new build. Keep dollar amounts out of
metadata (per-storefront currencies; Apple discourages prices in listing text).
"Launch price for a limited time" is only true while it lasts, so remove it
when the price changes.

## App Privacy (nutrition label)

Answer **"No, we do not collect data from this app."** → label shows
**Data Not Collected**.

Why that's accurate, from the code:
- No network calls, analytics or crash-reporting SDKs.
- Photos, cutouts, sounds, reminders, countdowns and the note are stored in the
  App Group container on device only.
- Purchases go through StoreKit; Apple processes them, not us.

`Resources/PrivacyInfo.xcprivacy` (in both targets) declares no tracking, no
collected data, and the UserDefaults required reasons `1C8F.1` (App Group) and
`CA92.1`.

**Privacy policy and support URLs are both required.** Ready-to-host pages
are in `docs/site/widgetplay/` (privacy.html, support.html, style.css,
icon.png, index.html: plain static files, light/dark, phone-friendly), published
with GitHub Pages from the public repo github.com/remydlc/WidgetPlay. Enter:

- Privacy Policy URL: `https://remydlc.github.io/WidgetPlay/privacy.html`
- Support URL: `https://remydlc.github.io/WidgetPlay/support.html`

Both pages use `nine3one2@gmail.com` as the contact address. Apple checks that the support page loads and has a
way to contact you.

```
This app's main interface is its widgets (CarPlay dashboard, Home Screen, Lock Screen). The app itself configures them.

BACKGROUND AUDIO: The app declares the audio background mode for a single purpose: its "Play Car Sound" App Intent (AudioPlaybackIntent) plays a short startup sound, capped at 8 seconds, when the user runs it from a Shortcuts automation triggered by "CarPlay Connects". iOS does not allow starting playback from the background without this mode. The app plays no other background audio.

TO TEST THE SOUND: Open the app > Startup Sound > tap Preview. To test the automation: Shortcuts > Automation > + > CarPlay > Connects > Run Immediately > add "Play Car Sound" (listed under this app).

IN-APP PURCHASE: One non-consumable unlock (no subscription). Restore Purchase is on the main screen and in the paywall.

No account or login is required. The app makes no network requests besides StoreKit.
```

## Before you submit

- [x] Name: Widget Play: Car Widgets (Home Screen: Widget Play)
- [ ] Create the app record in App Store Connect (reserves the name)
- [ ] Create IAP product `com.nine3one2.mycarwidget.unlock`, non-consumable, $4.99
- [ ] Enroll in the App Store Small Business Program (15% commission)
- [ ] Schedule the price rise: IAP → Price → add **$9.99** with a start date
      (e.g. 30 days after launch). Existing buyers keep their unlock.
- [ ] When it takes effect: remove "Launch price for a limited time" from the
      promotional text (editable anytime, no new version), and update
      `displayPrice` in `MyCarWidget.storekit` so local tests match.
- [ ] Publish `docs/site/widgetplay/` to github.com/remydlc/WidgetPlay and enable Pages
- [ ] Screenshots: 6.9" iPhone required; include one of the CarPlay dashboard
- [ ] Age rating questionnaire (expect 4+)
- [ ] Archive, upload, and test via TestFlight with your testers first
