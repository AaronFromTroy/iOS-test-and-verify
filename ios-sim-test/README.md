# ios-sim-test

A Claude Code skill that builds your iOS app, runs it on the simulator, navigates through the UI, takes screenshots, and visually verifies it works — before reporting done.

Works with native Xcode projects, React Native, Expo, and Flutter.

## What it does

When you say things like *"run it"*, *"test it on simulator"*, *"does it look right?"*, or *"check the app"*, this skill kicks in and runs the full cycle:

**build → boot simulator → install & launch → navigate & screenshot → verify visually → report**

It won't say it's done until it has actually looked at the running app and confirmed things look right.

---

## Requirements

- macOS with Xcode installed
- iOS Simulator (included with Xcode)
- The relevant toolchain for your project type (CocoaPods, Node, Flutter SDK, etc.)

---

## How to install in Claude Code

### Option 1: Install via plugin command (recommended)

```bash
/plugins add https://github.com/AaronFromTroy/iOS-test-and-verify
```

Run this inside Claude Code. The skill becomes available globally.

### Option 2: Install from a local directory

1. Clone or download this repository somewhere on your machine:
   ```bash
   git clone https://github.com/AaronFromTroy/iOS-test-and-verify ~/skills/ios-sim-test
   ```

2. Open your Claude Code settings file (`~/.claude/settings.json`) and add:
   ```json
   {
     "skills": [
       {
         "path": "/Users/yourname/skills/ios-sim-test/ios-sim-test"
       }
     ]
   }
   ```

3. Restart Claude Code. The skill is now active.

### Option 3: Install directly in your project

If you only want the skill available in a specific project:

1. Copy or symlink this directory into your project's `.claude/skills/` folder:
   ```bash
   mkdir -p /path/to/your/project/.claude/skills
   cp -r ~/skills/ios-sim-test/ios-sim-test /path/to/your/project/.claude/skills/
   ```

2. Claude Code will pick it up automatically when you open that project.

---

## Usage

Once installed, just talk to Claude naturally inside your iOS project:

```
"run the app"
"test it on the simulator"
"does it look right?"
"check if the login screen works"
"verify the app before I submit"
```

Claude will handle the rest — build, launch, navigate, screenshot, verify, and give you a clear report of what it saw.

---

## Project structure

```
ios-sim-test/
├── SKILL.md              # Skill definition and step-by-step instructions for Claude
└── references/
    └── navigation.md     # Gesture reference and device coordinate tables
```

---

## Supported project types

| Project type | Detection | Build method |
|---|---|---|
| Native Xcode | `.xcworkspace` / `.xcodeproj` | `xcodebuild` |
| React Native | `package.json` + `ios/` folder | `npx react-native run-ios` |
| Expo | `app.json` with `expo` key | `npx expo run:ios` |
| Flutter | `pubspec.yaml` | `flutter run -d <simulator-id>` |
