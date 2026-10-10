---
description: >-
  Every error code of the Unicus mobile SDKs, what it means and what to do,
  plus the most common integration problems and how to fix them.
---

# Errors and troubleshooting

## Errors versus results

The SDK raises an error only when the verification **could not run**:
`UnicusSdkException` on Android and Flutter, `UnicusSdkError` on iOS (in
Objective-C, an `NSError` with domain `com.tekbees.unicus.sdk` and the code in
`userInfo["UnicusErrorCode"]`). A verification that ran always ends in a
result, also when the person was not verified (see
[Results and resuming](results-and-resuming.md)).

| Field | Description |
| --- | --- |
| `code` | Stable error code (table below). Route on it. |
| `message` | Text for developers. Do not show it to the user. |
| `resultCode` | Unicus result code, when Unicus answered with one (for example `2002` in `transaction_refused`). |
| HTTP status | `httpStatus` on Android and iOS, `statusCode` on Flutter: the HTTP status of the Unicus answer (401, 403, 429...). |
| Details | The Unicus answer or the original cause, for support. |

## Error codes

Codes marked with a platform exist only there; the others exist on every
platform.

### Configuration

| `code` | What happened | What to do |
| --- | --- | --- |
| `environment_not_available` | `configure` with an environment this SDK version does not have yet (STAGING, PRODUCTION). The message says what is missing. | Use `DEV`, or update the SDK when Tekbees publishes the environment. |
| `not_configured` | An operation ran before `configure`. | Call `configure` once when your app starts. |
| `invalid_configuration` | iOS, Flutter: no environment and no base URL, empty base URL, or an unknown environment name. | Use the configuration with `apiKey` and `environment`. |
| `missing_api_key` | `apiKey` is empty. | Set the Customer Token of your company. |
| `missing_session_device_id` | A custom base URL of an environment the SDK has no keys for. | Use `environment` instead of a base URL. |
| `missing_embedded_sdk_device_key` | The SDK build has no biometric engine key. | Defective build: contact Tekbees. |
| `invalid_endpoint` | iOS: the API URL could not be built from the base URL. | Use `environment`. |

### Starting the verification

| `code` | What happened | What to do |
| --- | --- | --- |
| `missing_document` | `start` without a document and without a `tid`. | Start with `enrollmentVerify` and the document. |
| `invalid_request` | iOS (Objective-C), Flutter: unknown document type, or no document and no `tid`. | Use `ID`, `FD`, `PP` or `DL`. |
| `session_active` | Another verification is running. | Wait for its result or call `cancelActiveSession`. |
| `transaction_refused` | Unicus did not create the transaction. `resultCode`: `2002` no flow assigned, `2012` person not enrolled, `2052` user blocked. | Show a business message for the `resultCode`; review your company in the portal. |
| `missing_tid` | Unicus answered without a transaction id. | Retry; contact support with the API logs if it persists. |
| `flow_not_supported` | The assigned flow cannot run in this app (`9020`): UI steps with Disabled mode, a step type your Custom provider does not support, an unsupported camera combination, a flow not published for mobile, or a flow format this SDK cannot read. The transaction is closed. | Fix the flow in the portal or the `uiStepMode`; update the SDK for newer flows. See [Flow rules](flows-and-ui-steps.md#flow-rules-for-mobile). |

### Session and network

| `code` | What happened | What to do |
| --- | --- | --- |
| `network_error` | No connection or timeout after the automatic retries. | Check the connection and start again: the open transaction resumes. |
| `rate_limited` | Too many requests (HTTP 429). | Wait a moment and start again. |
| `temporary_failure` | Temporary failure in Unicus (`2054`). Nothing was processed. | Start again. |
| `transaction_expired` | The transaction does not exist or expired (`2051`). | Start a new verification. |
| `session_expired` | The transaction session expired and could not be renewed, or a flow screen reported it (`2051`, HTTP 401). | Start again. |
| `session_mismatch` | The session belongs to another transaction (HTTP 403). | Start again; contact support if it persists. |
| `session_refresh_failed` | iOS: the flow progress could not be read again between steps. | Start again: the transaction resumes. |
| `invalid_response` | Android: Unicus returned an answer the SDK could not read. | Retry; contact support if it persists. |
| `http_<status>` | Unexpected HTTP status without another code (for example `http_500`). | Retry; contact support with the status. |

### Flow screens (WebView mode)

| `code` | What happened | What to do |
| --- | --- | --- |
| `webview_unavailable` | Android: Android System WebView missing, disabled or older than Chromium 90. Reserved on iOS. | Ask the user to update *Android System WebView* from Google Play, or use Custom mode. |
| `webview_domain_not_allowed` | iOS: your app declares `WKAppBoundDomains` without the Unicus flow app domain. | Add the domain to `Info.plist`, or use Custom mode. |
| `webview_load_failed` | The flow screens did not load after one retry. | Check the network, proxy and allow-lists for the flow app domain. |
| `bridge_protocol_mismatch` | The flow screens use a version this SDK does not support. | Update the SDK. |

### App and device

| `code` | What happened | What to do |
| --- | --- | --- |
| `no_view_controller` | iOS, Flutter on iOS: no visible view controller to present from, or the screen did not appear. | Call `start` with the app in the foreground, from a controller that is in a window. |
| `no_activity` | Flutter on Android: the plugin is not attached to an activity. | Call `start` with the app in the foreground. |
| `host_activity_lost` | Android, Flutter on Android: no live activity to show the next step. | Retry with the app in the foreground. |
| `unsupported_flow` | iOS: the requested camera mode is not in this build. | Contact Tekbees. |
| `not_initialized`, `unicus_initialize_failed` | The biometric engine could not start. | Check the camera, the date and time of the device and the network; contact support with the message. |
| `plugin_unavailable` | Flutter: the native plugin is not registered (unsupported platform, or a test without a mock). | Run on Android or iOS. |

### Custom UI steps

| `code` | What happened | What to do |
| --- | --- | --- |
| `custom_provider_failed` | Android: your provider's `present` threw an exception. | Fix the provider (see the cause). |
| `step_failed` | iOS: a step call of your Custom screen failed (network, session). | Show a retry in your screen. |
| `step_inactive` | iOS: a context call after the step ended. | Do not reuse old step contexts. |
| `sign_status_rejected` | Android: Unicus refused the signature status query (for example `STEP_OUT_OF_ORDER`). | End the step with `fail`. |
| `unsupported_operation` | Android: a signature method called on a step context that the SDK did not create. | Use the context the SDK gives you. |

### Internal

| `code` | What happened | What to do |
| --- | --- | --- |
| `unicus_start_failed`, `unicus_prepare_failed`, `unicus_countries_failed`, `unicus_error` | Unexpected error in that operation; the original cause is attached. | Contact support with the message and the API logs. |
| `cancelled` | Internal: a step was stopped by a cancellation. Your `start` receives the `2041` or `2003` result instead. | None. |
| `transaction_status_failed` | iOS, internal: not delivered to your app. | None. |

## Common problems

| Symptom | Check |
| --- | --- |
| `environment_not_available` at startup | Only `DEV` is available today. Use the same environment as your Customer Token. |
| `transaction_refused` with `2002` | Assign a flow to your company in the portal and publish it for mobile. |
| `flow_not_supported` | Open the flow in the portal: a `face_match` without an earlier `liveness`, a `document` alone in a camera group, a flow published for web only, or UI steps with `uiStepMode` Disabled (or missing in your Custom provider). |
| The transaction is not created with a token that works on the web | The Customer Token belongs to another environment. Tokens differ per environment. |
| The camera never opens | The app must run on a physical device. iOS: `NSCameraUsageDescription` must be in `Info.plist`. Result `9996`: the user denied the camera; ask them to allow it in the system settings. |
| The flow screens stay blank or fail with `webview_load_failed` | Corporate proxy, MDM, per-app VPN or firewall blocking the flow app domain. See [Compatibility and security](compatibility-and-security.md#enterprise-environments). |
| `webview_unavailable` on some Android devices | Old or disabled Android System WebView (managed fleets, kiosks). Update it, or use Custom mode. |
| The verification ends with `2003` | The user left a screen: not an error. Offer to continue; the next `start` with the same document resumes. |
| Every `start` creates a new transaction | `resumeOpenTransactions` is off, the document changed, the flow was changed in the portal, more than 20 minutes passed, or Android 5.x. |
| iOS Debug build on a physical iPhone cannot start the verification | Debug builds on a physical iPhone need the development framework swap. See [iOS installation](ios/installation.md). |
| Location is never sent | Optional. Android: declare the location permissions in your app. iOS: add `NSLocationWhenInUseUsageDescription`. The user can refuse; the verification continues. |
| The logo does not appear on Android | Android shows the logo from a drawable of your app (`setAndroidLogoResourceName`); remote logos are not shown. See [Android configuration](android/configuration.md). |

## API logs

For diagnosis, turn on `enableApiLogging` and register an API log listener.
The logs are sanitized: encrypted biometric data, document data, session tokens
and the personal data of the flow steps are redacted. Do **not** turn on
`includeSensitiveApiLogData` in production, and never send logs with it on to
anyone.

## What to send to support

See [Support](support.md).
