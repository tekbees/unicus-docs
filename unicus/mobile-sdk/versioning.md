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
* **Spanish and English.** `UnicusSdkConfig.language` (device language by
  default) for the camera screens, the flow screens, the exit dialog and
  `resultMessage`; result messages addressed to the user.
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
| Language option `UnicusSdkConfig.language` (default: device language). A Spanish device now gets the camera screens in Spanish without overrides, and an English device gets `resultMessage` in English. | To keep everything in Spanish, set the language to Spanish. If you passed the complete Spanish text table only to translate the screens, replace it with the option. Texts and languages: [Android](android/texts-and-languages.md), [iOS](ios/texts-and-languages.md), [Flutter](flutter/texts-and-languages.md). |
| `resultMessage` / `messageFor` speak to the user (for example `4001` "Se agotaron los intentos."; it said "Proceso cancelado por el usuario." by mistake). Codes and outcomes are unchanged. | If your app compared message texts, compare `resultCode` instead. |
| Camera screens look like the Web SDK 5.0: colours, animations, texts and document data labels. Without company colours the default is the Web SDK theme (blue `#2c6ba9`), no longer yellow and navy. Document capture instructions are shorter so they fit the frame. | None. Review the screenshots of your own guides. |
| Logo when the company logo is not available: iOS camera screens show no logo (as on the web); Android shows the Unicus logo in colour. | None. |
| Newer camera engine (packages from 14 October 2026). Fixes the "rotate your device" screen that never went away on the latest iPhones and iOS versions. | Update the SDK. |
| Some `resultMessage` texts now say whom to contact (`2002`, `4012`, `4013`: «Contacta a quien te envió el enlace.»); `2013` says the verification is under manual review; native exit codes `9991`–`9998` and document check codes `10001`–`10020` have their own message. Codes and outcomes are unchanged. | If your app compared message texts, compare `resultCode` instead. |
| `4000` (`INVALID_REQUEST`, outcome `ERROR`) on Android, iOS and Flutter. | Handle it as an integration error; see [Result codes](result-codes.md). |
| The exit dialog says «¿Salir de la validación?» and, on iOS too, that progress is kept and the verification can be resumed. Camera texts reworded. | None. Review the screenshots of your own guides. |
| `resultMessage` speaks to the person when the camera session ends early (for example `2061` "This verification has no attempts left.") and when they leave the camera after a `7006` ("We could not confirm the document is physical. Try again with the original document."); before it carried the technical text. The detail stays in `errorMessage`. The document retry screen asks for "your original document". Codes and outcomes are unchanged. | If your app kept `resultMessage` for diagnostics, use `errorMessage`. |
| `serverResponse.expectedMediaStatusEnumInt` replaces `digitalIdSpoofStatusEnumInt`, which is deprecated. Correction: neither name is filled in what your app receives (see the next row). A document that could not be confirmed as physical shows as result code `7006`. | Do not read either field: handle `7006`. Read the detail of the document checks on your server. Existing code still compiles, with a warning on the deprecated name. |
| Less data reaches the app. The result (`serverResponse`, `raw["serverResponse"]`) and the events (`response`, `raw`) keep only `responseType`, `success`, `resultCode`, `isCompletelyDone`, `photoIDNextStepEnumInt` and `isFrontSide` from each camera answer. Document data, match levels, the status of each check (including `expectedMediaStatusEnumInt`), age groups, session data and fields added later no longer reach the app. Flow step results keep their `validations`. API logs of capture answers keep only what the SDK reads, unless `includeSensitiveApiLogData`. | If your app read other fields, move that logic to your server: read them with the [webhook](../sdk-web-v5/webhooks.md) or [Get a transaction status](../sdk-web-v5/transaction-status.md). |
| No IP is sent. `clientIp` of the verification request is deprecated and ignored: Unicus resolves the device IP on its server. | Stop setting `clientIp`. |
| Decide on your server. The result and the events are computed on the device: they can be altered and may never arrive. | Grant access only with the `TRANSACTION_FINALIZED` webhook or `query-transaction` with your server API key. See [Confirm on your backend](results-and-resuming.md#confirm-on-your-backend). |
