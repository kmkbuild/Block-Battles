# 28_BUILD_CONFIGURATION_PLATFORM_AND_RELEASE.md
## Block Battles — Build Configuration, Platform, and Release

**Governing documents:** `18_TECHNICAL_ARCHITECTURE.md`, `19_FLUTTER_FLAME_PROJECT_STRUCTURE_AND_APP_GAME_ARCHITECTURE.md`, `26_PERFORMANCE_MEMORY_AND_MOBILE_OPTIMIZATION.md`
**Document status:** Layer C — Deployment Authority

**Inheritance confirmation:** This document outlines the physical pipeline to turn the Dart code and assets into a shippable product using the Flutter toolchain. It strictly adheres to the Android-first, solo-developer paradigm, ensuring that builds are automated, safe, and compliant with modern store requirements.

---

## 1. Platform Strategy

- **Absolute MVP:** Android-first (Google Play Store).
- **MVP+ / Future:** iOS (Apple App Store).
- **Why:** Android allows for rapid, iterative "Internal App Sharing" deployment without the strict provisioning profiles and hardware requirements (macOS) imposed by iOS during the early prototyping phases.

---

## 2. Prerequisites & Verification

Before configuring any builds, the host environment must be strictly validated:
- Flutter SDK (stable channel) installed and path configured.
- Android Studio installed with the required Android SDK and NDK.
- Java Development Kit (JDK) installed.
- **Verification:** Run `flutter doctor` and resolve any missing toolchains. Use `flutter devices` to confirm testing targets are detected.

---

## 3. Global Build Configuration (pubspec.yaml)

- **Version Management:** Handled strictly in `pubspec.yaml` (e.g., `version: 1.0.0+1`).
- **Assets:** Ensure all Flame images, audio, and custom fonts are properly declared in the `assets:` and `fonts:` sections of `pubspec.yaml`.
- **Dart AOT:** Dart automatically compiles to AOT (Ahead-of-Time) native machine code during release builds (`flutter build --release`), ensuring maximum performance.

---

## 4. Development Build

Used for daily local iteration.
- **Command:** `flutter run`
- **Behavior:** Connects to a physical device or emulator, enabling Hot Reload and Hot Restart.
- **Debug Features:** Flutter DevTools, debug overlays, full error logging.
- **Pre-processor/Environment:** Dart environment variables (e.g., `--dart-define=ENABLE_CHEAT_CONSOLE=true`).
- **Output:** Deploys temporary debug build directly to the connected device.

---

## 5. QA Build

Used for Playtesting and performance verification on devices.
- **Command:** `flutter build apk --debug` or `flutter build apk --profile`
- **Behavior:** `--profile` builds strip out debugging tools but keep enough metadata for Flutter DevTools performance profiling.
- **Pre-processor/Environment:** `--dart-define=ENABLE_CHEAT_CONSOLE=true` (The debug UI overlay remains active).
- **Output:** `.apk` file for side-loading onto QA devices.

---

## 6. Release Build (Production)

The shipping configuration.
- **Command:** `flutter build appbundle --release --obfuscate --split-debug-info=./debug-info`
- **Behavior:** Dart compiles to highly optimized AOT code. `arm64` is targeted by default. All debug symbols, asserts, and DevTools are stripped.
- **Pre-processor/Environment:** No debug flags.
- **Output:** `.aab` (Android App Bundle) only, ready for Google Play upload.

---

## 7. Versioning

- **App Version (String):** Semantic versioning in `pubspec.yaml` (e.g., `1.0.0`). 
  - Major: Core gameplay overhauls.
  - Minor: Content updates (New Enemies, Relics).
  - Patch: Bug fixes.
- **Build Number / Version Code (Integer):** The `+1` part of `1.0.0+1`. Increments linearly on every single CI/CD build or manual export.

---

## 8. Application Identity

Configured within Android project files (`android/app/build.gradle` and `AndroidManifest.xml`):
- **Package Identifier / applicationId:** `com.[companyname].[gamename]` (Lower case, no special characters).
- **App Name:** `Block Battles`
- **Icon:** Configured using the `flutter_launcher_icons` package to generate Android Adaptive Icons (Foreground + Background separation) for all device masks.
- **Splash Screen:** Handled via the `flutter_native_splash` package, matching the game's background color (`09_VISUAL_DESIGN_SYSTEM`) and displaying a high-res studio logo until the first Flutter frame is drawn.

---

## 9. Android Configuration (build.gradle)

*(Note: Exact API levels shift yearly. Verify against current Google Play requirements at the time of build).*

- **Min SDK:** `minSdkVersion`: Set based on Flutter minimum requirements and targeted device coverage. Verify current Flutter Android SDK requirements at project initialization.
- **Target SDK / Compile SDK:** `targetSdkVersion` / `compileSdkVersion`: Use the latest stable Android SDK level supported by the selected Flutter/Android toolchain. Verify current Google Play requirements before release.
- **Permissions (AndroidManifest.xml):** **Minimal.** 
  - Required: `INTERNET`, `ACCESS_NETWORK_STATE` (for Ads/Analytics).
  - Banned: Location, Contacts, Camera. Storage access relies on scoped app directories (via `path_provider`).
- **Orientation:** Portrait only (locked in `SystemChrome` via Flutter).

---

## 10. iOS Considerations (MVP+)

- **Min iOS Version:** iOS 12.0 (defined in `ios/Podfile`).
- **Architecture:** ARM64.
- **Graphics API:** Metal (handled by Impeller in modern Flutter).
- **Privacy Manifests:** Apple strictly requires a `PrivacyInfo.xcprivacy` file detailing data collection.

---

## 11. Signing

- **Android Keystore:** A custom `keystore.jks` must be generated. 
- **Configuration:** Created a `key.properties` file in `android/` containing the alias and passwords. The `android/app/build.gradle` must be configured to read this file and apply the signing config to the `release` build type.
- **CRITICAL:** The keystore password, alias, and alias password must be backed up securely.
- **App Signing by Google Play:** Enabled. (Flutter builds sign with the upload key; Google Play signs the final APK with the production key).

---

## 12. Environment Configuration

Use Dart environment defines to isolate features safely (`--dart-define`).
- `const bool isDev = bool.fromEnvironment('DEV_MODE', defaultValue: false);`
- Use these constants to gate debug menus, logging, and cheat logic, ensuring the Flutter compiler physically tree-shakes out the debug code in release builds.

---

## 13. Third-Party SDK Configuration

- **Rule:** Never tightly couple game logic to a 3rd party SDK.
- **Implementation:** Use domain interfaces (`IAnalyticsService`, `IAdsService`). 
- **Example:** `AnalyticsService` implements `IAnalyticsService` wrapping a Flutter analytics package. If the package changes, the core game logic does not.

---

## 14. Ads / Monetization Build Separation

- **Test Ads:** The `QA Build` and `Dev Build` MUST be configured to use Test Ad Unit IDs provided by the Ad Network (e.g., AdMob Test IDs via `google_mobile_ads`). 
- **Risk:** Viewing live production ads on your own developer device or during automated QA testing will result in a permanent ban from the Ad Network.

---

## 15. Privacy / Consent Requirements

- **GDPR (Europe):** A Consent Management Platform (CMP) popup must trigger on boot for EU users before initializing Ad SDKs.
- **CCPA (California):** "Do Not Sell My Info" tracking logic.
- **ATT (iOS):** App Tracking Transparency popup must be configured using `app_tracking_transparency` package.
- **COPPA (USA):** The game must be flagged appropriately in Google Play.

---

## 16. Internal Testing

- **Distribution:** Use Google Play Console's "Internal Testing" track, uploading the AAB directly.
- **Why:** Bypasses Google's lengthy review process, accurately tests Google Play Billing (IAP) with mock credit cards, and validates the exact AAB format that users will receive.

---

## 17. Release Candidate (RC) Process

1. **Code Freeze:** No new features merged to `main`.
2. **QA Pass:** Execute the Test Scenarios from `27_TESTING_QA_DEBUGGING_AND_AUTOMATION.md`.
3. **RC Build:** Compile the `Release Build` (`flutter build appbundle --release`).
4. **Upload:** Push to Google Play "Closed Testing" track.
5. **Review:** Wait for Google's automated/manual review approval.

---

## 18. Store Asset Checklist

Before hitting "Publish", ensure the store listing contains:
- [ ] **Hi-Res Icon:** 512x512 PNG.
- [ ] **Feature Graphic:** 1024x500 PNG.
- [ ] **Screenshots:** Minimum 4. Must include actual gameplay (Board), the Shop, and Relic mechanics. Portrait orientation.
- [ ] **Short Description:** 80-character hook.
- [ ] **Long Description:** Formatted description of features.
- [ ] **Content Rating Questionnaire:** Completed and verified.

---

## 19. Crash Monitoring

- **Service:** Firebase Crashlytics via FlutterFire packages.
- **Monitoring:** Set up alerting for crash spikes.
- **Logs:** Ensure unhandled Flutter framework errors (`FlutterError.onError`) and async Dart errors (`PlatformDispatcher.instance.onError`) are forwarded to the crash reporter as non-fatal exceptions to catch silent logic failures.
- **Obfuscation Mapping:** Upload the debug mapping files generated by `--split-debug-info` to the Play Console and Crashlytics.

---

## 20. Rollback Strategy

Google Play does not allow "rolling back" to an older version number. 
- **Staged Rollouts:** Always release production updates via Staged Rollout (e.g., 10% -> 50% -> 100% of users).
- **Halt:** If crash monitoring detects an S0/S1 spike at 10% rollout, **Halt** the release immediately.
- **Fix:** Compile a hotfix with an incremented version code in `pubspec.yaml` (e.g., `1.0.1+16`) and push it to replace the halted build.

---

## 21. Final Release Checklist

Do not press "Publish to Production" until this checklist is cleared:

- [ ] Dart definitions for Dev/Cheat modes are disabled.
- [ ] `flutter build appbundle --release --obfuscate --split-debug-info` executed successfully.
- [ ] `key.properties` and `keystore.jks` configurations are correct and active.
- [ ] Debug symbols mapped and uploaded.
- [ ] Ad Network IDs are swapped from "Test" to "Production".
- [ ] IAP items match the Google Play Console product IDs exactly.
- [ ] `flutter test` and `flutter analyze` passed at 100%.
- [ ] Staged Rollout is set to an initial 10% (never 100% on day one).

---

**End of `28_BUILD_CONFIGURATION_PLATFORM_AND_RELEASE.md`.**
