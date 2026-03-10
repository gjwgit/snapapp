# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

`snapapp` is a minimal Flutter counter app used as a **test harness** for building Flutter apps as Linux Snap packages via GitHub Actions. The primary goal is validating that `flutter_secure_storage` (and other native dependencies) can be successfully packaged into a snap using the Canonical build workflow.

## Commands

```bash
# Install dependencies
flutter pub get

# Run the app (Linux desktop)
flutter run -d linux

# Build release binary
flutter build linux --release

# Run tests
flutter test

# Run a single test file
flutter test test/widget_test.dart

# Build snap locally (requires snapcraft installed)
snapcraft

# Clean build artifacts
flutter clean
```

## Architecture

- `lib/main.dart` — The entire app: a standard Flutter counter demo. The app logic is intentionally trivial; the repo's value is in its build infrastructure.
- `snap/snapcraft.yaml` — Snap packaging config (base: core24). Uses the `flutter` snapcraft plugin targeting `lib/main.dart`. Includes build/stage packages needed for `flutter_secure_storage` (`libsecret-1-dev`, `libjsoncpp-dev`, `libssl-dev`, `lld`, `llvm`).
- `.github/workflows/installers.yaml` — CI workflow that triggers on every push. Uses `canonical/action-build@v1` to build the snap (no separate Flutter install step needed — snapcraft handles it). Uploads the resulting `.snap` as a GitHub Actions artifact.

## Key dependency notes

- `flutter_secure_storage: any` — This native dependency is the main reason the snap build is non-trivial. It requires `libsecret-1-dev`/`libjsoncpp-dev` at build time and `libsecret-1-0`/`libjsoncpp-dev` at runtime.
- `snap/snapcraft.yaml.~1~` — Backup of an older `bookoflife` snap config (core22 base). Useful as a reference for alternative package configurations (audio, gstreamer, etc.).
- The snap plugs include `password-manager-service` and `secret-service` for `flutter_secure_storage` support.

## Workflow

The CI skips a standalone Flutter setup step — `canonical/action-build@v1` invokes snapcraft directly, which downloads Flutter internally. The commented-out steps in the workflow show an alternative approach (manual Flutter install + `flutter build linux`) that is not currently used.
