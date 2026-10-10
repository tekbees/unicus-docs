---
description: >-
  Minimum versions, devices and WebView requirements of the Unicus mobile SDKs,
  what enterprise environments must allow, the data the SDK handles and its
  security model. Read before going live.
---

# Compatibility and security

## Minimum versions

| | Android | iOS | Flutter |
| --- | --- | --- | --- |
| Operating system | Android 5.0 (`minSdk 21`) | iOS 15.0 | Android and iOS as in the native columns |
| Build | `compileSdk 34` or newer, Android Gradle Plugin 8.x or 9.x, Java 17 | Xcode 16 or newer | Flutter 3.19 or newer, Dart 3.3 or newer |
| Language | Kotlin (Kotlin Gradle Plugin 2.0 or newer) or Java only | Swift 5.9; Objective-C through a facade | Dart; on Android, Kotlin Gradle Plugin 2.0 or newer (your Flutter version may require more) |
| Dependencies | AndroidX. The SDK brings `appcompat`, `core-ktx`, `kotlinx-coroutines-android`, `androidx.webkit` and `androidx.browser`. | System frameworks only | No `webview_flutter` or `http` dependency |
| Package manager | Maven repository in the package | XCFrameworks (Embed & Sign) or CocoaPods | `path:` dependency; CocoaPods on iOS (Swift Package Manager is not supported yet) |

Installation per platform: [Android](android/installation.md),
[iOS](ios/installation.md), [Flutter](flutter/installation.md).

## Devices

| Requirement | Why |
| --- | --- |
| Physical device with a camera | Liveness and document capture. Emulators and simulators build and run the app, but cannot complete a verification. |
| Camera permission | Asked by the SDK the first time the camera opens. If the user denies it, the result is `9996`. |
| Location permission (optional) | Recorded with the transaction when your app declares it and the user allows it. If not, the verification continues without it. |
| Stable connection | Uploads are small but must complete. The SDK retries network failures automatically and never repeats a request that may have reached Unicus when that could duplicate a transaction or an OTP code. |

## WebView requirements

Only for flows with UI steps in WebView mode (the default). See
[Flows and UI steps](flows-and-ui-steps.md#ui-step-modes).

| Platform | Requirement |
| --- | --- |
| Android | Android System WebView (Chromium) **90 or newer**, enabled, with web message listeners. The SDK checks it before showing anything and fails with `webview_unavailable` instead of a blank page. |
| iOS | The system WebKit (always present on iOS 15+). If your app enables App-Bound Domains, see below. |
| Flutter | The WebView lives in the native SDKs; there is no version conflict with `webview_flutter` in your app. |

## Enterprise environments

### Network allow-lists (proxy, firewall, MDM, per-app VPN)

Allow both domains of your environment:

| Environment | Unicus API | Flow screens |
| --- | --- | --- |
| DEV | `alpha.idunicus.com` (port 8080) | `dev-id.idunicus.com` |
| STAGING | Published when the environment is enabled. | Published when the environment is enabled. |
| PRODUCTION | Published with the production release. | Published with the production release. |

The flow screens load from a fixed address with no data in the URL. They work
behind a corporate proxy. Managed apps (Intune App SDK, BlackBerry Dynamics,
AppConfig) must allow both domains in their per-app VPN or tunnel.

### iOS App-Bound Domains

If your `Info.plist` declares `WKAppBoundDomains`, add the flow screens domain
of your environment. Otherwise `start` stops before showing anything with
`webview_domain_not_allowed`.

### RASP and app shielding

Tools such as Promon, Appdome, DexGuard or iXGuard can block or flag WebViews
and script injection. The SDK:

* adds no scripts to the page and does not use `addJavascriptInterface`;
* talks to the flow screens through the platform message channel
  (`WebViewCompat.addWebMessageListener` on Android, `WKScriptMessageHandler`
  on iOS), restricted to the main frame and the flow screens origin.

If your tool requires an allow-list, list the SDK screens:

| Platform | Components |
| --- | --- |
| Android | `com.tekbees.unicus.sdk.internal.UnicusVerificationActivity`, `com.tekbees.unicus.sdk.internal.UnicusFlowWebActivity`, `com.tekbees.unicus.sdk.internal.UnicusPermissionActivity` |
| iOS | `UnicusFlowWebViewController` of `UnicusSDK.framework` |

### Certificate pinning and apps without WebViews

WebViews do not apply your app's certificate pins (the Android WebView ignores
`network_security_config` pins). If your policy requires pinning on every
screen, or forbids WebViews with remote content, use **Custom** mode (Android,
iOS) or **Disabled** mode for biometric-only flows. Flutter apps that need
Custom mode must integrate the native SDKs.

### Screenshots and app switcher

Set `secureScreens` to `true` to protect the flow screens: `FLAG_SECURE` on
Android (no screenshots or screen recording) and a privacy cover in the iOS app
switcher. Default: `false`.

## Security model

* **One credential.** Your app passes only the Customer Token (`apiKey`). The
  SDK embeds the other identifiers it needs; there is no option to pass them.
  Keep the Customer Token of each environment out of public repositories.
* **Server authority.** Step order, progress and results are decided by
  Unicus. The device never decides that a step passed.
* **Encrypted capture.** Face scans and document images are encrypted on the
  device by the biometric engine and sent only to Unicus. They never reach your
  app, the result or the events.
* **Session bound to one transaction.** The SDK keeps the transaction session
  in memory and passes it to the flow screens through a private channel, never
  in a URL, storage or logs.
* **Locked-down WebView.** JavaScript only for the flow screens origin;
  navigation to any other origin blocked; no file access, mixed content or
  pop-ups; storage and cookies of the flow wiped when the screen closes. The SDK
  does not enable WebView debugging.
* **External links.** Privacy notices and documents to sign open only over
  `https`, in Custom Tabs (Android) or a Safari view (iOS).
* **Encrypted resume key.** The key that resumes an open transaction is stored
  encrypted with the Android Keystore (Android 6.0 or newer; nothing is stored
  below) or in the iOS Keychain (this device only, after first unlock). It is
  linked to a hash of your Customer Token and the document, expires after 24
  hours and is never logged. See
  [Resuming](results-and-resuming.md#automatic-resume-recommended).
* **Obfuscation.** The internal code of the SDK is obfuscated; the embedded
  identifiers are masked in the binary. On Android the SDK ships its own R8 /
  ProGuard rules: your app needs no extra rules.

## Data handled

| Data | Source | Where it goes |
| --- | --- | --- |
| Customer Token | Your app configuration | Unicus, to create the transaction. |
| Document type and number | Your app (`start`) | Unicus. |
| Face scan and document images | Camera | Encrypted, to Unicus only. |
| Data read from the document | Unicus | Shown to the user for confirmation; not in the result. |
| Location (optional) | Device | Unicus, as transaction metadata. |
| Consent, form values, OTP phone or e-mail, signature | User, in the flow steps | Unicus. |
| SDK version and platform | SDK | Unicus, in the user agent. |
| Resume key | Unicus | Encrypted on the device; deleted on a final result. |

The result in your app carries the `tid`, codes and step outcomes only. The
personal data and images are available to your backend through
[Get a transaction status](../sdk-web-v5/transaction-status.md), and are
processed by Unicus under the data processing agreement of your company.

## Logs

API logging is off by default. When you turn it on (`enableApiLogging`), the
logs are sanitized: encrypted biometric data, document data, session tokens and
the personal data of the flow steps (codes, phone numbers, e-mails, form
values, signatures, document number, location) are redacted and long values
truncated. The resume key is always redacted.

`includeSensitiveApiLogData` shows the personal data. Use it only on your own
test devices, never in a production build.

## Privacy

* Tell your users why you verify their identity. Add a `consent` step to your
  flow, or set `prependConsent` (see [Consent](flows-and-ui-steps.md#consent)).
* **Android:** declaring the location permissions means reporting location
  (collected, not shared) in the *Data safety* form of Google Play. The SDK
  itself does not add location permissions.
* **iOS:** the SDK ships a privacy manifest. Provide clear texts for
  `NSCameraUsageDescription` and, if you collect location,
  `NSLocationWhenInUseUsageDescription`, and reflect the data above in your App
  Store privacy details.
* Call `clearResumeData()` when a user logs out, so another person on the same
  device never continues their transaction.
