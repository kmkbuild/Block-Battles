# 28_BUILD_CONFIGURATION_PLATFORM_AND_RELEASE.md
## Block Battles — Build Configuration, Platform, and Release

**Governing documents:** `18_TECHNICAL_ARCHITECTURE.md`, `19_UNITY_PROJECT_STRUCTURE_AND_SCENE_ARCHITECTURE.md`, `26_PERFORMANCE_MEMORY_AND_MOBILE_OPTIMIZATION.md`
**Document status:** Layer C — Deployment Authority

**Inheritance confirmation:** This document outlines the physical pipeline to turn the C# scripts and Unity assets into a shippable product. It strictly adheres to the Android-first, solo-developer paradigm, ensuring that builds are automated, safe, and compliant with modern store requirements.

---

## 1. Platform Strategy

- **Absolute MVP:** Android-first (Google Play Store).
- **MVP+ / Future:** iOS (Apple App Store).
- **Why:** Android allows for rapid, iterative "Internal App Sharing" deployment without the strict provisioning profiles and hardware requirements (macOS) imposed by iOS during the early prototyping phases.

---

## 2. Unity Build Configuration (Global)

- **Scripting Backend:** IL2CPP (Required for 64-bit ARM64 support).
- **Api Compatibility Level:** .NET Standard 2.1.
- **C++ Compiler Configuration:** Release (for QA and Prod builds).
- **Managed Stripping Level:** Low to Medium. (High stripping can aggressively remove classes injected via reflection or JSON serialization. Must be tested thoroughly if increased).

---

## 3. Development Build

Used for daily local iteration.
- **Development Build:** Checked.
- **Script Debugging:** Checked.
- **Autoconnect Profiler:** Checked.
- **Pre-processor Define:** `DEBUG`, `ENABLE_CHEAT_CONSOLE`
- **Output:** Local `.apk` directly to USB-connected device.

---

## 4. QA Build

Used for Playtesting and performance verification.
- **Development Build:** Unchecked. (Critical to measure actual CPU performance).
- **Script Debugging:** Unchecked.
- **Pre-processor Define:** `ENABLE_CHEAT_CONSOLE` (The debug UI overlay remains active).
- **Output:** `.aab` (Android App Bundle) uploaded to Google Play Internal Testing.

---

## 5. Release Build (Production)

The shipping configuration.
- **Development Build:** Unchecked.
- **Pre-processor Define:** `RELEASE` (All cheat consoles and debug logging stripped out).
- **Output:** `.aab` only.

---

## 6. Debug Symbols

- **Android:** Unity must be configured to output `symbols.zip` during the IL2CPP build process. This file MUST be uploaded to Google Play Console alongside the `.aab` to de-obfuscate crash stack traces.
- **iOS:** dSYM generation enabled and routed to crash reporting service.

---

## 7. Versioning

- **App Version (String):** Semantic versioning (e.g., `1.0.0`). 
  - Major: Core gameplay overhauls.
  - Minor: Content updates (New Enemies, Relics).
  - Patch: Bug fixes.
- **Build Number / Version Code (Integer):** Increments linearly by `+1` on every single CI/CD build or manual export. (e.g., Version `1.0.0 (15)`).

---

## 8. Application Identity

- **Package Identifier:** `com.[companyname].[gamename]` (Lower case, no special characters).
- **App Name:** `Block Battles`
- **Icon:** Must utilize Android Adaptive Icons (Foreground + Background separation) to ensure it renders correctly on circles, squares, and squircle device masks.
- **Splash Screen:** Unity Splash Screen customized with the game's background color (`09_VISUAL_DESIGN_SYSTEM`) and a high-res studio logo.

---

## 9. Android Configuration

*(Note: Exact API levels shift yearly. Verify against current Google Play requirements at the time of build).*

- **Min SDK:** API 26 (Android 8.0) - Ensures >95% market coverage while allowing modern features.
- **Target SDK:** API 34+ (Always match the latest Google Play requirement).
- **Architecture:** ARM64 ONLY. (Google Play no longer accepts 32-bit ARMv7 apps).
- **Permissions:** **Minimal.** 
  - Required: `INTERNET`, `ACCESS_NETWORK_STATE` (for Ads/Analytics).
  - Banned: Location, Contacts, Camera, Storage (use scoped storage/persistent data path instead).
- **Orientation:** Portrait only.
- **Graphics API:** Vulkan (Primary) + OpenGLES3 (Fallback).

---

## 10. iOS Considerations (MVP+)

- **Min iOS Version:** iOS 12.0.
- **Architecture:** ARM64.
- **Graphics API:** Metal.
- **Capabilities:** In-App Purchase.
- **Privacy Manifests:** Apple strictly requires a `PrivacyInfo.xcprivacy` file detailing data collection (especially if using Ad SDKs).

---

## 11. Signing

- **Android Keystore:** A custom `.keystore` must be generated. 
- **CRITICAL:** The keystore password, alias, and alias password must be backed up securely. If the keystore is lost, you can never update the app on Google Play again without a complex manual reset process.
- **App Signing by Google Play:** Enabled. (Unity builds sign with the upload key; Google Play signs the final APK with the production key).

---

## 12. Environment Configuration

Use pre-processor directives to isolate features safely.
- `#if UNITY_EDITOR`: Code that manipulates asset files, custom inspectors.
- `#if DEVELOPMENT_BUILD`: Developer console logic.
- `#if UNITY_ANDROID`, `#if UNITY_IOS`: Platform-specific IAP/Ad network initializations.

---

## 13. Third-Party SDK Configuration

- **Rule:** Never tightly couple game logic to a 3rd party SDK.
- **Implementation:** Use interfaces (`IAnalyticsService`, `IAdsService`). 
- **Example:** `AnalyticsService` implements `IAnalyticsService` wrapping Unity Analytics or Firebase. If Firebase is removed later, the core game logic does not need to change.

---

## 14. Ads / Monetization Build Separation

- **Test Ads:** The `QA Build` and `Dev Build` MUST be configured to use Test Ad Unit IDs provided by the Ad Network (e.g., AdMob Test IDs). 
- **Risk:** Viewing live production ads on your own developer device or during automated QA testing will result in a permanent ban from the Ad Network for "fraudulent impressions."

---

## 15. Privacy / Consent Requirements

- **GDPR (Europe):** A Consent Management Platform (CMP) popup must trigger on boot for EU users before initializing Ad SDKs.
- **CCPA (California):** "Do Not Sell My Info" tracking logic.
- **ATT (iOS):** App Tracking Transparency popup must be configured in Xcode.
- **COPPA (USA):** The game must be flagged appropriately in Google Play depending on the target age demographic. (Puzzle games often appeal to children; Ad networks must be configured to serve non-personalized ads to minors).

---

## 16. Internal Testing

- **Distribution:** Use Google Play Console's "Internal Testing" track. 
- **Why:** It completely bypasses Google's lengthy review process, allowing testers to download the update instantly via a whitelist email link. It also accurately tests Google Play Billing (IAP) with mock credit cards.

---

## 17. Release Candidate (RC) Process

1. **Code Freeze:** No new features merged to `main`.
2. **QA Pass:** Execute the Test Scenarios from `27_TESTING_QA_DEBUGGING_AND_AUTOMATION.md`.
3. **RC Build:** Compile the `Release Build` (.aab).
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

- **Service:** Unity Cloud Diagnostics or Firebase Crashlytics.
- **Monitoring:** Set up alerting so the developer receives an email immediately if a crash spikes above a 1% session threshold on a new release.
- **Logs:** Ensure `Debug.LogError` calls are forwarded to the crash reporter as non-fatal exceptions to catch silent logic failures.

---

## 20. Rollback Strategy

Google Play does not allow "rolling back" to an older version number. 
- **Staged Rollouts:** Always release production updates via Staged Rollout (e.g., 10% -> 50% -> 100% of users).
- **Halt:** If crash monitoring detects an S0/S1 spike at 10% rollout, **Halt** the release immediately.
- **Fix:** Compile a hotfix with an incremented version code (e.g., `1.0.1 (16)`) and push it to replace the halted build.

---

## 21. Final Release Checklist

Do not press "Publish to Production" until this checklist is cleared:

- [ ] `DEVELOPMENT_BUILD` is unchecked.
- [ ] `DEBUG` and `ENABLE_CHEAT_CONSOLE` directives are stripped.
- [ ] Keystore alias and password are correct.
- [ ] `symbols.zip` is uploaded for crash de-obfuscation.
- [ ] Ad Network IDs are swapped from "Test" to "Production".
- [ ] IAP items match the Google Play Console product IDs exactly.
- [ ] Automated tests (Unit, Leak, GC Allocation) passed at 100%.
- [ ] Staged Rollout is set to an initial 10% (never 100% on day one).

---

**End of `28_BUILD_CONFIGURATION_PLATFORM_AND_RELEASE.md`.**
