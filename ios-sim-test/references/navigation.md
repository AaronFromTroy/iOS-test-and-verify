# iOS Simulator Navigation Reference

## Gesture commands via `xcrun simctl io booted`

All coordinates are in logical points (pt), not pixels. The simulator uses the same point coordinate system as the physical device.

### Tap
```bash
xcrun simctl io booted tap <x> <y>
```
Single tap at the given point. Use this for buttons, tabs, list rows, toggles.

### Swipe / Drag
```bash
xcrun simctl io booted swipe <startX> <startY> <endX> <endY> [duration]
```
- `duration` is optional, in seconds (e.g., `0.5`). Slower swipes register as scrolls; faster ones as flicks.
- Scroll down: swipe UP → `swipe 195 650 195 250`
- Scroll up: swipe DOWN → `swipe 195 250 195 650`
- Swipe left (go to next page): `swipe 350 400 50 400`
- Swipe right (go back or prev page): `swipe 50 400 350 400`

### Press (long press)
```bash
xcrun simctl io booted swipe <x> <y> <x> <y> 1.5
```
Swipe from and to the same point with a longer duration simulates a long press.

### Hardware buttons
```bash
xcrun simctl io booted button home          # Home button / swipe up to home
xcrun simctl io booted button sideButton    # Lock/sleep button
```

### Keyboard input
If a text field is focused after a tap, you can send keyboard input:
```bash
xcrun simctl io booted key return          # Press Return/Enter
```
For typing text, the most reliable approach is using `osascript`:
```bash
osascript -e 'tell application "Simulator" to activate' && \
  osascript -e 'tell application "System Events" to type "hello world"'
```

---

## Common device dimensions (logical points)

| Device | Width | Height |
|---|---|---|
| iPhone SE (3rd gen) | 375 | 667 |
| iPhone 16 | 390 | 844 |
| iPhone 16 Plus | 430 | 932 |
| iPhone 16 Pro | 393 | 852 |
| iPhone 16 Pro Max | 430 | 932 |
| iPad (10th gen) | 820 | 1180 |

---

## Common tap zones

| Zone | Approximate coordinates (iPhone 16) |
|---|---|
| Status bar area | y ≈ 20–60 |
| Navigation bar (back button) | x ≈ 30, y ≈ 60 |
| Navigation bar (title center) | x ≈ 195, y ≈ 60 |
| Navigation bar (right button) | x ≈ 360, y ≈ 60 |
| Screen center | 195, 422 |
| Bottom tab bar item 1 | x ≈ 40, y ≈ 820 |
| Bottom tab bar item 2 | x ≈ 130, y ≈ 820 |
| Bottom tab bar item 3 (center) | x ≈ 195, y ≈ 820 |
| Bottom tab bar item 4 | x ≈ 260, y ≈ 820 |
| Bottom tab bar item 5 | x ≈ 350, y ≈ 820 |
| Safe area bottom edge | y ≈ 800 |
| Keyboard (return key) | x ≈ 330, y ≈ 720 (portrait) |

---

## Reading what's on screen

After every gesture, take a new screenshot and read it:

```bash
xcrun simctl io booted screenshot /tmp/sim-after-tap.png
# Then use Read tool on /tmp/sim-after-tap.png
```

If the screenshot looks identical to the one before, the tap may have missed or the animation hasn't completed. Wait 0.5–1s and try again.

---

## Checking app logs in real time

```bash
# Stream all logs from the app (replace with your bundle ID)
xcrun simctl spawn booted log stream --predicate 'subsystem == "com.example.MyApp"'

# Show recent logs (last 2 minutes)
xcrun simctl spawn booted log show --last 2m 2>/dev/null | grep -v "^$" | tail -50

# Check for crashes
xcrun simctl spawn booted log show --last 5m | grep -iE "(crash|fault|exception|killed)" | tail -20
```

---

## Getting UI element information (Accessibility Inspector)

If you need to find where specific UI elements are, use the Accessibility Inspector approach:

```bash
# Dump the full accessibility tree (requires idb or similar MCP)
# Without idb, rely on screenshots + visual judgment for coordinates
```

If an `idb` MCP is connected, it provides `idb_ui_describe` which returns element positions without guessing.

---

## Working with multiple simulators

```bash
# List all booted simulators with their UDIDs
xcrun simctl list devices | grep Booted

# Target a specific UDID instead of "booted"
xcrun simctl io <UDID> screenshot /tmp/shot.png
xcrun simctl io <UDID> tap 195 422
```
