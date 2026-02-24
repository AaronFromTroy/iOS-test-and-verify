---
name: ios-sim-test
description: >
  Builds an iOS app, runs it on the simulator, navigates through the UI, and visually verifies it
  works before reporting done. Use this skill whenever the user wants to run, test, check, verify,
  or preview their iOS app in the simulator — even if they just say "run it", "see how it looks",
  "test it on simulator", or "does it work?". The skill won't say it's done until it has actually
  looked at the running app and confirmed things look right.
---

# iOS Simulator Test

## What this skill does

It runs the full cycle: **build → boot simulator → install & launch → navigate & screenshot → verify visually → report**.

You do not declare success until you have taken screenshots and confirmed the app looks and behaves correctly. If something looks wrong, you say so clearly and describe what you saw.

---

## Step 1: Understand the project

First, identify the project type — this determines the build command:

```bash
# Check for framework indicators
ls package.json app.json ios/ android/ Pods/ *.xcworkspace *.xcodeproj 2>/dev/null
```

| If you see | Project type | Build approach |
|---|---|---|
| `app.json` with `expo` key | Expo | `npx expo run:ios` |
| `package.json` + `ios/` folder (no Expo) | React Native | `npx react-native run-ios` |
| `*.xcworkspace` or `*.xcodeproj` | Native iOS | `xcodebuild` (Step 2) |
| `pubspec.yaml` | Flutter | `flutter run -d <simulator-id>` |

**For React Native / Expo / Flutter projects**, skip to Step 2b. The framework handles building and launching in one command, and handles simulator targeting automatically.

**For native Xcode projects**, continue with Step 2a. You'll need to know:
- What workspace/project file to use (`.xcworkspace` preferred over `.xcodeproj`)
- What scheme and target to build

```bash
# List available schemes
xcodebuild -list -workspace MyApp.xcworkspace  # or -project MyApp.xcodeproj

# List available simulators
xcrun simctl list devices available
```

If the project uses CocoaPods, check whether `Pods/` exists. If not, run `pod install` first and use the `.xcworkspace`.

---

## Step 2a: Build for simulator (native Xcode)

Use `xcodebuild` with the `iphonesimulator` SDK. The `-destination` flag targets the simulator without needing it booted first.

```bash
xcodebuild \
  -workspace MyApp.xcworkspace \
  -scheme MyApp \
  -configuration Debug \
  -destination 'platform=iOS Simulator,name=iPhone 16,OS=latest' \
  -derivedDataPath /tmp/ios-sim-build \
  build \
  2>&1 | tee /tmp/xcodebuild.log | tail -50
```

**Interpreting the output:**
- `** BUILD SUCCEEDED **` → proceed to Step 3
- `** BUILD FAILED **` → read the log, surface the errors to the user, stop here

For large projects the log is verbose. If piping to `tee`, you can `grep` for errors:
```bash
grep -E "(error:|BUILD FAILED|warning:)" /tmp/xcodebuild.log | head -40
```

Find the built `.app` bundle:
```bash
find /tmp/ios-sim-build -name "*.app" -type d | head -5
```

---

## Step 2b: Build and launch (React Native / Expo / Flutter)

These frameworks handle the full build → install → launch cycle in one command. Skip Steps 3 and 4 — go directly to Step 5 once the app is running.

**React Native:**
```bash
# Targets a booted simulator automatically; or specify with --simulator
npx react-native run-ios
npx react-native run-ios --simulator "iPhone 16"
```

**Expo:**
```bash
npx expo run:ios
# Or for a dev build already installed:
npx expo start --ios
```

**Flutter:**
```bash
# List available simulators
flutter devices

flutter run -d <device-id>
# Example: flutter run -d "iPhone 16 (simulator)"
```

Watch the output for errors. These commands stream build logs, so you'll see failures inline. If the metro bundler (RN/Expo) or Flutter runner fails, the error is usually clear in the output.

Once the app opens in the simulator, wait ~3 seconds for the JS bundle to load before taking screenshots.

---

## Step 3: Boot the simulator

Pick a simulator that matches what you built for. A booted simulator is faster to use.

```bash
# Check what's already booted
xcrun simctl list devices | grep Booted

# Boot a specific device if none is running
xcrun simctl boot "iPhone 16"

# Open the Simulator.app so the UI is visible (helps with screenshot accuracy)
open -a Simulator
```

Wait a few seconds for the simulator to finish booting before installing:
```bash
xcrun simctl list devices | grep Booted
```

---

## Step 4: Install and launch

```bash
# Install the built app (use the .app path from Step 2)
xcrun simctl install booted /tmp/ios-sim-build/Build/Products/Debug-iphonesimulator/MyApp.app

# Launch the app (use the bundle identifier from Info.plist or build output)
xcrun simctl launch booted com.example.MyApp
```

To find the bundle identifier if you don't know it:
```bash
/usr/libexec/PlistBuddy -c "Print CFBundleIdentifier" \
  /tmp/ios-sim-build/Build/Products/Debug-iphonesimulator/MyApp.app/Info.plist
```

After launch, give the app 2–3 seconds to render its initial screen before taking screenshots.

---

## Step 5: Navigate and screenshot

This is where you see what the app actually looks like. Take screenshots liberally — they're cheap and give you ground truth.

```bash
# Take a screenshot and save it
xcrun simctl io booted screenshot /tmp/sim-screenshot-01.png

# Then read it as an image to see what's on screen
# Use the Read tool on /tmp/sim-screenshot-01.png
```

**Navigation gestures** — interact with the app to explore different screens:

```bash
# Tap at coordinates (x, y) — useful for buttons, list items
xcrun simctl io booted tap <x> <y>

# Swipe from (x1,y1) to (x2,y2)
xcrun simctl io booted swipe <x1> <y1> <x2> <y2>

# Scroll down (swipe up)
xcrun simctl io booted swipe 200 600 200 200
```

**Reading screenshot coordinates**: On a standard iPhone 16 simulator, the screen is typically 390×844 pt (logical). Common tap targets:
- Center of screen: ~195, 422
- Bottom tab bar: y ~800+
- Navigation bar back button: ~30, 60
- Large centered button: ~195, 500

After each navigation action, wait ~1s then take another screenshot to confirm the transition happened.

**Read each screenshot** using the Read tool — don't just assume the navigation worked.

See `references/navigation.md` for a fuller gesture reference.

---

## Step 6: Verify

After navigating through the key screens, evaluate:

1. **Does the app launch without crashing?**
   - Check for crash logs: `xcrun simctl spawn booted log show --last 1m | grep -i crash`

2. **Do the main screens render correctly?**
   - No blank/white screens where content should be
   - No visible layout overlaps or clipped text
   - Images loading (or appropriate placeholders)

3. **Does navigation work?**
   - Tapping navigation items moves to the right screen
   - Back/dismiss actions work

4. **Any console errors worth surfacing?**
   ```bash
   xcrun simctl spawn booted log show --last 2m --predicate 'subsystem == "com.example.MyApp"' 2>/dev/null | tail -30
   ```

---

## Step 7: Report

Write a clear summary with:
- Build result (succeeded / failed with errors listed)
- Screens you visited and what you saw in each screenshot
- Any issues found (visual bugs, crashes, broken navigation)
- Overall verdict: working / needs attention

If everything looks good: say so, and describe what you verified. Don't just say "it works" — describe what you saw.

If something is wrong: describe exactly what's broken (include the screenshot path for the user to look at too), so they can act on it.

---

## Common issues

| Problem | Likely cause | Fix |
|---|---|---|
| `xcodebuild: error: SDK 'iphonesimulator' not found` | Xcode not installed or not selected | `xcode-select --install` or `sudo xcode-select -s /Applications/Xcode.app` |
| `Unable to boot device in current state: Booted` | Simulator already booted | Fine, just proceed |
| `No such file: MyApp.app` | Wrong derived data path or build failed | Check `/tmp/xcodebuild.log` |
| App launches but crashes immediately | Runtime error | Check `xcrun simctl spawn booted log show` |
| Screenshot is blank/black | Simulator not fully booted | Wait a few more seconds and retry |
| CocoaPods dependency errors | Haven't run `pod install` | `cd <project_root> && pod install` |
| RN: `No bundle URL present` | Metro bundler not running | Run `npx react-native start` in a separate terminal first |
| Expo: `Unable to resolve module` | Missing node_modules | Run `npm install` or `yarn` then retry |
| Flutter: `No connected devices` | No simulator booted | Run `xcrun simctl boot "iPhone 16"` first |

---

## Resources

- `references/navigation.md` — Full gesture reference and coordinate tips
