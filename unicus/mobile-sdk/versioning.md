---
description: >-
  Versions of the Unicus mobile SDKs, release notes and compatibility policy.
---

# Versioning

{% hint style="warning" %}
**Coming soon.** The Unicus mobile SDKs are not yet available in production.
Today only the **DEV** environment is available; Tekbees will announce the
production release. Until then, ask Tekbees for the SDK package and a Customer
Token for DEV to prepare your integration.
{% endhint %}

## Versions

The Android, iOS and Flutter SDKs share one version number and one behaviour:
the same flows, result codes and error codes. The Flutter SDK of a version
contains the native SDKs of that same version.

| SDK | Current version | Where to read it |
| --- | --- | --- |
| Android | `0.1.0` | `UnicusSdk.shared.version`; the dependency `com.tekbees.unicus:unicus-sdk:<version>` |
| iOS | `0.1.0` | `UnicusSdk.shared.version` |
| Flutter | `0.1.0` | `version` in the plugin's `pubspec.yaml` |

Include the version when you contact [Support](support.md).

## Distribution

Today Tekbees delivers each SDK as a **package** (ZIP) with the SDK, a quick
start, the error list and an example app. Hosted Maven, Swift Package Manager
and CocoaPods repositories are coming soon: with them, updating will be a
version change in your build file.

## Compatibility policy

* Versions follow `MAJOR.MINOR.PATCH`. Patch and minor versions are
  compatible: fixes, new optional configuration, new events and new fields.
* Breaking changes (a removed option, a new case in an enum your code switches
  on) come in a new major version and are announced in advance. Before the
  production release, Tekbees may still make small breaking changes; they are
  listed below.
* **Flows evolve without an SDK update.** New UI step types run in WebView mode
  with the version you have. New camera step types or a newer flow format need
  an SDK update: until then the SDK answers `flow_not_supported` before
  showing anything, never in the middle of a flow.
* If the flow screens require a newer communication protocol than your SDK
  version supports, the SDK answers `bridge_protocol_mismatch`: update the
  SDK.
* New environments (STAGING, PRODUCTION) come with a new SDK version. An older
  version answers `environment_not_available`.

## 0.1.0 (pre-release, DEV environment)

First version of the native Android and iOS SDKs and of the Flutter SDK built
on them.

* **Simple setup.** `UnicusSdkConfig(apiKey, environment)`: the environment
  selects the API, the flow screens and the embedded keys. Environments not
  yet published answer `environment_not_available`.
* **Modular flows.** The flow assigned in the portal runs on the device:
  camera steps in native screens, UI steps in WebView mode (default), in your
  own screens with Custom mode (Android, iOS) or disabled. Flow checks before
  anything is shown (`flow_not_supported`, `9020`). Progress events per
  segment and per step.
* **Document signature** (`sign_document`) in WebView and Custom modes, with
  codes `4011`–`4014`.
* **Leaving is not cancelling.** Leaving a screen returns `2003` `RESUMABLE`
  with the `tid`; only a cancel inside the camera or `cancelActiveSession`
  cancels (`2041`). `cancelActiveSession` ends the flow wherever it is.
* **Automatic resume.** `resumeOpenTransactions` (default on): the next
  `start` with the same document continues the open transaction;
  `clearResumeData()` on logout.
* **Results.** `rejectionReason` and `rejectionDetail` for `2052`,
  `isRetryable`, `isResumable`, `resumed`, per-step results.
* **Country selection** for companies with several active countries.
* **Security.** Locked-down WebView, sanitized API logs, encrypted resume key,
  obfuscated SDK, optional `secureScreens`.
* **Objective-C** facade on iOS; Java-friendly API and Kotlin coroutines on
  Android.

### Changes during the pre-release

If you received an earlier package of `0.1.0`, review these changes:

| Change | What to do |
| --- | --- |
| New outcome `RESUMABLE` for `2003` (was `WARNING`). | Add the case to exhaustive `when` / `switch` statements over the outcome. |
| Android and Flutter on Android: the SDK no longer adds the location permissions. | Declare `ACCESS_FINE_LOCATION` / `ACCESS_COARSE_LOCATION` in your app if you want location. |
| Flutter: `rejectionReason` and `rejectionDetail` come from the native SDKs. | None. |
| Flutter on Android: Kotlin Gradle Plugin 2.0 or newer (was 2.3). | None; older Kotlin versions are now accepted. |
| Starting with a base URL is an advanced option. | Prefer `UnicusSdkConfig(apiKey, environment)`. |
