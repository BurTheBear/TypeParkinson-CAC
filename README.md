# TypeParkinson

<p>
  <img src="docs/assets/practice.webp" width="230" alt="Practice screen">
  <img src="docs/assets/today.webp" width="230" alt="Today screen">
  <img src="docs/assets/progress.webp" width="230" alt="Progress screen">
</p>

Built by Atharv Sharma ([@BurTheBear](https://github.com/BurTheBear)) and Pranav Atreya ([@pranav-atreya10](https://github.com/pranav-atreya10)) for the 2026 Congressional App Challenge.

TypeParkinson measures how someone touches the screen, builds a keyboard around it, lets them compare standard and personal typing, and then keeps learning as they type. Around the keyboard it adds daily tracking of hands and voice, speech for when talking is hard, medication timing, and a report to share with a care team. It's an iPhone and iPad app plus a system keyboard for supported apps.

## What it does

**The keyboard (in supported apps)**
- **Personal touch model.** The setup exercises measure where this person's touches land around a key: an average offset and a spread on each axis (`Shared/TouchModel.swift`). The keyboard keeps refining it on the device from every word the person keeps. Only those few numbers are stored, never text.
- **Smart correction.** At the end of each word, `Shared/IntentDecoder.swift` works out the most likely intended word with Bayes' rule: how likely the touches are under the touch model, times how likely the word is (`Shared/WordList.swift` ranks, with the system dictionary as a fallback). The alignment can drop a shaky double contact, supply a missed letter, or swap two letters, so "tje", "hellloo", "tomorow" and "teh" become "the", "hello", "tomorrow" and "the". A chip puts back exactly what was typed in one tap.
- **Touch handling for shaky hands.** Offset correction, a repeat filter for bounces, a brush filter for touches that lift too quickly, drift tolerance, a slower hold-to-delete, a large Undo / Delete word bar, a key pop-up, and three themes including high contrast. Touches come straight from UIKit with hardware timestamps (`Shared/MultiTouchSurface.swift`).
- **Word suggestions and saved phrases** above the keys.

**Touch pattern classifier**
`TypeParkinson/Model/PatternClassifier.swift` is a Gaussian naive Bayes classifier that runs on the device. It estimates the person's touch pattern (steady, shaky, slow and drifting, sudden jumps, or mixed) from up to nine measurements of their exercises and practice, and shows the probability of each. The pattern tunes the keyboard: how wide the touch model starts, how readily smart correction acts, and which filters are on. Its parameters are set from published descriptions of motor symptoms rather than trained on patient data. It describes touches; it doesn't diagnose anything.

**Every day**
- **Today**: the Aura (a living colour field from today's check compared with recent days), daily rings for Check, Practice and Voice, a streak, insights, and quick actions.
- **Daily check**: six taps compared with the person's usual; the keyboard can read touches differently for the day without moving any keys. A "how does typing feel" rating is saved with it.
- **Practice**: trace a spiral (distance from the line, smoothness, speed), rhythm tapping (taps in ten seconds, evenness, accuracy), typing practice (words a minute, accuracy), a **tremor meter** (ten seconds of motion-sensor data, Hann-windowed Fourier analysis: strongest shake rate in hertz and its strength), and a **voice volume** exercise (microphone loudness against a target zone, inspired by speech therapy for Parkinson's).
- **Speak**: type with the personal keyboard and have it read aloud; saved phrases; a full-screen **communication card** ("I have Parkinson's. Please give me a little extra time.").
- **Medication**: names and times, reminders with a "Took it" button that logs the dose even with the app closed.
- **Progress**: achievements, insights (week-over-week change, steadiest time of day, steadiest window after a dose, whether feelings match the numbers), the touch pattern, a chart per measurement, and a one-page PDF report or CSV export for a care team.
- **Siri and Shortcuts**: "Start my TypeParkinson check", "Speak for me", "Measure my tremor", "Trace a spiral".

Nothing in the app is a medical device, assessment or diagnosis; the screens and report say so where results appear.

## Project layout

- `TypeParkinson/` — the app.
  - `Model/` — `AppModel` (state and flow), `ActivityStore` (history), `TextComposer` (in-app typing with smart correction), `PatternClassifier`, `MotorAnalysis` (spiral, tapping, typing), `SensorAnalysis` and `SensorRecorders` (tremor and voice), `Insights` (insights and achievements), `Medication`, `Speaker`, `ReportBuilder`, `Routing` (tabs, Siri, notifications).
  - `Views/` — `Aura` and `DesignKit` (the design system), `HomeScreens` (Today), `PracticeViews`, `SensorSessions`, `SpeakView`, `ProgressViews`, `CareViews` (medication, communication card), `AccountScreens` (Settings), and the setup flow.
- `KeyboardExtension/` — the system keyboard. Letters, numbers, symbols and number pads; follows each text field's keyboard type and return key; suggestions, phrases, smart correction and on-device learning. Runs without Full Access.
- `Shared/` — compiled into both: layout, touch interpretation, the touch model, the decoder and word list, settings, themes, the suggestion bar and phrase panel, and the privacy manifest.
- `TypeParkinsonTests/` — unit tests (Swift Testing): layouts, touch interpretation, settings, profiles, smart correction, the touch model, the pattern classifier, tremor and voice analysis, spiral and tapping scores, history, insights and achievements.
- `Config/` — entitlements. The app and keyboard share settings through the App Group `group.com.pranavatreya.typeparkinson` (change it in both entitlements files and `Shared/KeyboardSettings.swift` together). The app also has Sign in with Apple and iCloud key-value storage.
- `docs/` — privacy policy and support pages, served by GitHub Pages.
- `AppStore/` — listing text, review notes, release checklist and a screenshot script. Start with [AppStore/README.md](AppStore/README.md).
- `typeparkinson-mockup/`, `TypeParkinson Website Design/` — design mockups.

## Running

Open `TypeParkinson.xcodeproj` and run the **TypeParkinson** scheme. Run the tests with ⌘U, or:

```bash
xcodebuild test -project TypeParkinson.xcodeproj -scheme TypeParkinson -destination 'platform=iOS Simulator,name=iPhone 17 Pro'
```

To use the keyboard: Settings → Apps → TypeParkinson → Keyboards → turn on TypeParkinson. It doesn't need Full Access.

The tremor meter needs a real iPhone's motion sensors, and the voice exercise asks for the microphone; both only measure while their exercise runs and keep nothing but the result.

## How it fits together

First launch: onboarding (five pages) → account (Sign in with Apple, or continue without one) → touch exercises → profile and touch pattern → proposed keyboard → check against a standard keyboard → turn on the keyboard → Today.

- **Accounts are optional.** Signing in with Apple saves the touch profile and approved keyboard to the person's own iCloud key-value store (`CloudStore`), keyed by their Apple user ID, so their other devices pick it up. There is no server. Practice history and medication stay on the device. Settings has sign out and delete account and data.
- **The keyboard only reads shared data.** The app writes approved settings and phrases to the App Group; the keyboard reads them each time it appears. Without Full Access it can't write there or use the network; what it learns while typing stays in its own container and starts again when the person redoes the exercises.

## Debug-only aids

- **Simulate tremor (demo)** — mouse clicks are too precise to show much, so simulator Debug builds add an offset, jitter and occasional extra contact to touches, and the tremor and voice exercises use sample recordings. Toggle it on the setup screen or in Settings › Developer, where you can also load three weeks of sample history.
- **`-demoScreen <name>`** launch argument opens a screen with sample data: `onboarding`, `account`, `welcome`, `exercises`, `profile`, `proposal`, `check`, `compare`, `enable`, `home`, `recalResult`, `settings` (add `-demoSignedIn YES` to sign in, and `-demoTab today|practice|speak|progress|settings` with `home`).

None of these are compiled into Release builds.


## Presentation and project website

The [presentation folder](presentation/README.md) links the preserved [v1 archive](presentation/v1/README.md) and the active [v2 brief](presentation/v2/BRIEF.md). Current progress is in [presentation/PROGRESS.md](presentation/PROGRESS.md).

The landing page is in [docs/index.html](docs/index.html). Privacy and support retain their existing URLs. These materials do not constitute a CAC application submission or App Store release. The v2 build/test baseline and remaining hardware checks are in [validation](presentation/v2/VALIDATION.md).
