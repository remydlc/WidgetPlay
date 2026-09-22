# Device test plan

Run in this order: each stage depends on the one before, so a failure points at
one layer instead of five. Tick as you go; note anything odd next to the item.

## 0. Before you leave the house (iPhone, iOS 26+)

- [ ] `./bootstrap.sh`, clean build (Shift-Cmd-K), run on the phone
- [ ] Signing & Capabilities: both targets show the App Group with no warning
- [ ] Scheme > Run > Options > StoreKit Configuration = `MyCarWidget.storekit`

## 1. App basics

- [ ] Pick a theme, type a greeting: preview updates
- [ ] Add one photo (free): preview shows it
- [ ] Buy the unlock (local StoreKit, nothing charged): paywall closes, locks disappear
- [ ] Add 3 more photos; slideshow toggle turns on
- [ ] Long-press a photo > Remove Background: a cutout appears **next to** the original
- [ ] Toggle to the cutout photo first (delete others temporarily): car floats on the theme

## 2. Home Screen / Lock Screen

- [ ] Add all four widgets to the Home Screen: Car Dashboard, Car Maintenance, Countdown, Quick Note
- [ ] Change the theme in the app: all four follow
- [ ] Lock Screen > customize > add Maintenance, Countdown and Note (circular / rectangular / inline)
- [ ] Put the phone on a charger sideways: StandBy shows the widgets

## 3. Data widgets

- [ ] Maintenance: add an oil change due tomorrow: shows "1 day", amber
- [ ] Add one due yesterday: shows "Overdue", red, sorted first
- [ ] Countdown: add a birthday with a date in a past year + "Repeats every year": counts to the next one
- [ ] Quick Note: type a parking spot: widget updates

## 4. Startup sound (at home first)

- [ ] Startup Sound > Preview each built-in (volume up)
- [ ] Import a custom sound from Files; preview it
- [ ] Build the Shortcuts automation from the in-app steps
- [ ] In Shortcuts, run the "Play Car Sound" action manually: sound plays with the app closed

## 5. In the car

- [ ] Plug in / connect CarPlay: **startup sound plays**, music ducks and comes back
- [ ] Settings > General > CarPlay > (car) > Widgets: add Car Dashboard, Maintenance, Countdown, Note
- [ ] Dashboard widget readable in **daylight** with a bright photo (the scrim test)
- [ ] Cutout photo looks right on the car screen
- [ ] Leave it 30+ minutes with slideshow on: photo has changed
- [ ] Disconnect: disconnect sound plays (if you set one)

## 6. Refund path

- [ ] Xcode > Debug > StoreKit > Manage Transactions > refund the unlock
- [ ] App: premium theme resets, photos trim to one, extra reminders/countdowns trimmed
- [ ] Custom startup sound falls back to a built-in (not silence)

## If something fails

Note the step number and paste the Xcode console output. Most likely suspects:
- Widget blank on device but fine in Simulator: App Group provisioning (step 0)
- Sound silent from the automation but fine in Preview: background audio mode
  missing from the built Info.plist (regenerate with bootstrap)
- Cutout error: expected in Simulator only
