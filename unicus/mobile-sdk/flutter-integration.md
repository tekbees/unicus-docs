---
description: >-
  Integrate Unicus identity verification in a Flutter app for Android and iOS:
  what the Flutter SDK does, the pages of this section and the requirements.
---

# Flutter integration

{% hint style="warning" %}
**Coming soon.** The Unicus mobile SDKs are not yet available in production.
Only the **DEV** environment is available today; `staging` and `production`
answer `environment_not_available`. Tekbees will announce the production
release.
{% endhint %}

The Unicus Flutter SDK (`unicus_sdk_flutter`) is a thin Dart layer over the
native Unicus SDKs for Android and iOS. Your app calls `configure` once and
`start` with the user's document; the SDK creates (or resumes) the transaction,
runs the flow your company assigned in the administrative portal (camera steps
in native screens, consent, form, signature and OTP steps in the SDK's secure
web view) and returns one `UnicusVerificationResult`. You do not edit
`MainActivity` or `AppDelegate`, call the Unicus API or add other biometric
packages. The only secret your app passes is the
[Customer Token](../sdk-web-v5/customer-token.md), as `apiKey`.

{% code overflow="wrap" %}
```dart
final unicus = UnicusSdkFlutter();
await unicus.configure(
  const UnicusSdkConfig(apiKey: '<CUSTOMER_TOKEN>', environment: UnicusEnvironment.dev),
);
final result = await unicus.start(
  const UnicusVerificationRequest.enrollmentVerify(
    document: UnicusDocument(type: UnicusDocumentType.id, externalDatabaseRefId: '123456789'),
  ),
);
```
{% endcode %}

## Pages in this section

| Page | What you will find |
| --- | --- |
| [Quick start](flutter/quick-start.md) | Install, configure, start and handle the result in a few lines. |
| [Installation](flutter/installation.md) | Package contents, `pubspec.yaml`, Android Gradle setup, iOS Podfile and `Info.plist`, Debug builds on a physical iPhone. |
| [Configuration](flutter/configuration.md) | Every `UnicusSdkConfig` option, environments, logs, branding. |
| [Start a verification](flutter/start-a-verification.md) | Document types, country selection, existing transactions, location, cancelling, threading rules. |
| [Results and events](flutter/results-and-events.md) | `UnicusVerificationResult`, outcomes, the `events` and `apiLogs` streams. |
| [UI steps](flutter/ui-steps.md) | WebView (default) and Disabled modes; why Custom is native-only. |
| [Texts and languages](flutter/texts-and-languages.md) | `Unicus_*` text keys, overrides and languages of the flow screens. |
| [Release checklist](flutter/release-checklist.md) | What to test and check before going live. |

Shared by the three mobile SDKs: [Overview](overview.md),
[Flows and UI steps](flows-and-ui-steps.md),
[Results and resuming](results-and-resuming.md),
[Result codes](result-codes.md),
[Errors and troubleshooting](errors-and-troubleshooting.md),
[Compatibility and security](compatibility-and-security.md),
[Versioning](versioning.md) and [Support](support.md).

## Requirements

| | Minimum |
| --- | --- |
| Flutter / Dart | Flutter 3.19, Dart 3.3 |
| Android | `minSdk` 21, `compileSdk` 34, Android Gradle Plugin 8.x or 9.x, Kotlin Gradle Plugin 2.0, JDK 17. Your Flutter version can require more: Flutter 3.47 requires AGP 8.11.1 and Kotlin Gradle Plugin 2.2.20. |
| Android device | Android System WebView based on Chromium 90 or newer (for flows with UI steps), a camera. |
| iOS | iOS 15.0, CocoaPods (Swift Package Manager not supported in this release). |
| Permissions | Camera (Android: added by the plugin; iOS: `NSCameraUsageDescription`). Location is optional. |
| Credentials | Customer Token of your company. No URL or internal key to configure: the SDK carries them per environment. |

Distribution today is a package delivered by Tekbees (ZIP). Hosted
repositories (pub, Maven, CocoaPods, Swift Package Manager) are coming soon.

{% hint style="info" %}
The Flutter SDK runs the same native engine as the Android and iOS SDKs, so
flows, result codes and errors are identical on the three. Only the *Custom*
UI step mode (your own screens for consent, forms, signature and OTP) is not
available from Dart; apps that need it integrate the
[Android](android-integration.md) or [iOS](ios-integration.md) SDK directly.
{% endhint %}
