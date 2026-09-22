# MyCarWidget

A CarPlay-dashboard widget app in the spirit of *myCar: Car Widgets & Dashboard*.

## Why there is no CarPlay target

Apple grants CarPlay **app** entitlements only for a fixed set of categories
(navigation, audio, communication, EV charging, parking, quick food ordering,
parked video). A personalization/dashboard app fits none of them and will not be
granted one.

What this project uses instead: since **iOS 26**, ordinary WidgetKit widgets
appear on the CarPlay dashboard. Per Apple's CarPlay Developer Guide, you enable
that simply by supporting the **`.systemSmall`** family -- no CarPlay entitlement
required. Caveat: a widget can only tap-to-launch its app in CarPlay if that app
is a CarPlay app, so ours is glance-only in the car. Fine for a themed clock.

## Layout

```
project.yml              XcodeGen spec -- source of truth for targets
bootstrap.sh             installs xcodegen if needed, generates the .xcodeproj
Sources/App/             app target only (config UI)
Sources/Shared/          compiled into BOTH targets
Sources/Widget/          widget extension only
Resources/*.entitlements App Group, one per target
```

`.xcodeproj` is gitignored on purpose -- regenerate it with `./bootstrap.sh`.

## Setup

```sh
./bootstrap.sh
open MyCarWidget.xcodeproj
```

Then in Xcode: select both targets > Signing & Capabilities > pick your Team.
Automatic signing will register the App Group on first build.

Identifiers to change if you want your own (three places, keep them in sync):
- `project.yml` -- `PRODUCT_BUNDLE_IDENTIFIER` (x2), `bundleIdPrefix`
- `Resources/*.entitlements` -- the group string
- `Sources/Shared/ConfigStore.swift` -- `AppGroup.id`

## Testing on the phone

1. Build and run to a device on iOS 26+.
2. Open the app, pick a theme, set the label.
3. Long-press home screen > add the "Car Dashboard" widget. Confirm it renders
   and that changing the theme in the app updates it.

## Testing in the car

1. Connect the iPhone to CarPlay at least once so the car appears in Settings.
2. On the phone: **Settings > General > CarPlay > [your car] > Widgets**.
3. Add "Car Dashboard" and reorder as needed.

The CarPlay Simulator is unreliable for widgets -- trust the real head unit.

## In-app purchase

A single **non-consumable** unlock: `com.nine3one2.mycarwidget.unlock`.
Launch price $4.99, rising to $9.99 after launch. Not a subscription -- the app
delivers its value at setup and has no recurring server cost, and "no
subscription" is a real differentiator in this category.

Testing locally, no App Store Connect needed:
1. Xcode > Product > Scheme > Edit Scheme > Run > Options
2. **StoreKit Configuration** > `MyCarWidget.storekit`

Then purchases run against the local config. Reset test purchases with
Debug > StoreKit > Manage Transactions while the app is running.

Before shipping, create the same product ID in App Store Connect and enroll in
the **Apple Small Business Program** (15% instead of 30% under $1M/yr).

## Photo background

**One photo is free.** It's the reason most people install an app like this,
and gating it puts a locked door in front of the moment that sells the product.
The paid tier sells themes instead. Multi-photo / slideshow / GIF are the
natural premium hooks once built -- do not advertise them until they exist.

The image is written to the App Group **container as a file**,
not to UserDefaults -- that store loads whole into memory and widget extensions
run in a tight budget.

Originals are downscaled through ImageIO's thumbnail path
(`CGImageSourceCreateThumbnailAtIndex`, max 1200px) before being written, so a
12MP photo is never fully decoded. `UIImage(data:)` + resize would spike ~48MB
and risk the extension being jetsammed.

The widget provider loads the image only when the photo is both enabled and
unlocked, and the view draws a bottom-weighted scrim so the clock stays readable
over a bright image.

## Slideshow

Premium. Free tier keeps one photo; the unlock raises the limit to 20 and turns
on rotation.

The timeline emits one entry per slide at 15-minute spacing with
`.atEnd`, so rotation resumes rather than restarting. Entries carry a
**filename, not a UIImage** -- a rotating timeline holds many entries at once,
and decoded images in each would put every slide in memory simultaneously. The
view decodes the single image it needs through a bounded `NSCache`
(`countLimit = 2`).

iOS controls widget refresh budget, so 15 minutes is a target, not a guarantee.

Photos saved by the earlier single-photo build are migrated into `Photos/` on
first read, so updating never loses a user's background.

## Maintenance widget

A second widget in the bundle: countdowns to oil change, registration,
inspection and so on. Free tier tracks 2 reminders, the unlock raises it to 12.

This is the app's answer to App Review guideline 4.2 -- it does something a user
cannot trivially do with built-in features, unlike theming, which they mostly
can. It's also deliberately *not* weather: CarPlay already ships an Apple
weather widget, so duplicating it would add nothing.

The timeline emits one entry per upcoming local midnight for a week, and the
countdown is derived from each entry's own date. That way "3 days" becomes
"2 days" exactly at midnight rather than whenever iOS next refreshes.

Items live in UserDefaults rather than files -- unlike photos, this is a handful
of short strings and dates, so there's no memory cost to the widget.

## Startup sounds

No app can run code when CarPlay connects. Every "startup sound" app works
through a Shortcuts automation (CarPlay → Connects → Run Immediately) calling an
app-provided action. Ours is `PlayCarSoundIntent`, an `AudioPlaybackIntent`
exposed via `AppShortcutsProvider`, so it appears in Shortcuts with no setup.
The app walks the user through the one-time automation.

- Four built-in sounds (`Resources/Sounds`, synthesized, royalty-free). Free.
- Import your own via Files. Premium. Falls back to a built-in if refunded.
- Separate connect / disconnect selections.
- Plays at most 8 s and ducks other audio rather than stopping it.
- Requires the `audio` background mode (`Resources/App-Info.plist`, merged with
  the generated plist); without it iOS refuses to start playback from the
  background.

## Car cutouts

`VNGenerateForegroundInstanceMaskRequest` (on-device Vision, iOS 17+) lifts the
subject into a transparent PNG saved *alongside* the original, never replacing
it. The widget draws a cutout on top of the theme gradient instead of
full-bleed. Premium. **Does not run in the Simulator**; test on a device.

## Countdown and Quick Note widgets

- Countdown: trips, birthdays, anniversaries. Yearly events roll over using
  next month/day match, so a birthday entered years ago still counts right.
  1 free, 12 premium. Midnight-aligned timeline like maintenance.
- Quick Note: one line of text (parking spot, gate code), stored on
  `DashboardConfig.note`. Free. Timeline policy `.never`; edits reload it.

## Lock screen

Maintenance, Countdown and Quick Note support `accessoryCircular` /
`accessoryRectangular` / `accessoryInline` (Note: no circular). The clock widget
skips lock screen: the lock screen already shows the time. StandBy uses
`systemSmall`, which every widget already supports.

## Widget designer (Widgetsmith-style)

Users create named `WidgetDesign`s: content (clock, photo, countdown,
maintenance, note) + `DesignStyle` (theme, font, custom colors). The
**My Designs** widget is an `AppIntentConfiguration`; long-press → Edit Widget
lists designs via `DesignEntity` / `DesignQuery`, so the same widget can be
added several times showing different designs, including in CarPlay.

- **Timed widgets**: `TimedOverride` ranges (may cross midnight) switch to
  another design.
- **Smart rules**: `RuleCondition`s from on-device data only (overdue or
  due-soon reminder, countdown today, weekend). Rules beat timed ranges;
  targets' own overrides are not followed, so designs can't loop.
- The provider emits entries at every change point (schedule edges, midnights,
  15-min photo rotation) so switches happen on time without waking the app.
  Rotating designs use a 3 h timeline; others 24 h. Policy `.atEnd`.
- Free: 2 designs, all fonts, free themes. Premium: unlimited designs, custom
  colors, timed widgets, smart rules. Without the entitlement the resolver
  and style fall back silently, so a refund never breaks a placed widget.

Note: `containerBackground(for: .widget)` only draws inside a real widget.
In-app previews pass `inlineBackground: true`.

## CarPlay rendering rules (learned on the head unit)

- CarPlay shows widgets "in full color and with the background removed"
  (Apple). Anything inside `containerBackground` disappears in the car.
- Photos are therefore **content**, not background (`photoLayer` in
  `DashboardView` / `DesignView`), per Apple's own photo-widget guidance.
- Theme gradients go through `themedWidgetBackground`: container background
  where the system shows one, drawn behind the content where it doesn't
  (`showsWidgetContainerBackground == false`: CarPlay, StandBy).
- Don't use `containerBackgroundRemovable(false)`: it drops the widget from
  contexts that need a removable background.
- All widgets use `contentMarginsDisabled()` and pad themselves (16pt) so
  full-bleed content reaches the edges.
- WidgetKit silently drops images over a per-size pixel budget; small (the
  CarPlay size) gets a 640px copy via `PhotoStore.widgetImage`.

## Downgrades never delete

A refund or lapsed purchase changes what's shown, never what's saved. Stores
expose `visible(isPremiumUnlocked:)` for widgets; over-limit photos, reminders,
countdowns and designs stay on disk, show locked in the app, and return on
unlock. A widget set to a locked design falls back to the first one.

## Roadmap

- [x] Photo widget -- one photo, free tier
- [x] Multi-photo / slideshow rotation (premium hook)
- [x] Startup sounds via Shortcuts action
- [x] On-device car cutouts
- [x] Countdown + Quick Note widgets
- [x] Lock screen widget sizes
- [x] Widget designer, timed widgets, smart rules, fonts & colors
- [ ] Animated GIF support
- [x] StoreKit 2 paywall, gating extras only -- keep the free tier real
- [x] Maintenance countdown widget
- [x] App icon (Resources/Assets.xcassets, single 1024 master)
- [ ] App Store prep: screenshots, privacy labels, listing copy
