# iOS Test and Verify

A Claude Code plugin that builds your iOS app, runs it on the simulator, navigates the UI, takes screenshots, and visually verifies it works — before reporting done.

Works with native Xcode projects, React Native, Expo, and Flutter.

## Install

```bash
/plugins add https://github.com/AaronFromTroy/iOS-test-and-verify
```

Run this inside Claude Code. The skill becomes available globally across all your projects.

## Usage

Once installed, just talk to Claude naturally inside your iOS project:

```
"run the app"
"test it on the simulator"
"does it look right?"
"check if the login screen works"
"verify the app before I submit"
```

Claude handles the rest: **build → boot simulator → install & launch → navigate & screenshot → verify visually → report**

It won't say it's done until it has actually looked at the running app and confirmed things look right.

## Requirements

- macOS with Xcode installed
- iOS Simulator (included with Xcode)
- The relevant toolchain for your project type (CocoaPods, Node, Flutter SDK, etc.)

## Skills included

| Skill | Description |
|---|---|
| [ios-sim-test](ios-sim-test/) | Full build-run-verify cycle for iOS Simulator |
