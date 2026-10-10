---
description: >-
  How the Flutter SDK shows the non-biometric steps of a flow: the secure web
  view (default), the Disabled mode, and why the Custom mode is native-only.
---

# UI steps

A Unicus flow mixes **biometric steps** (liveness, document, face match), which
always run in the SDK's native camera screens, **server steps** (age check),
which have no screen, and **UI steps**: consent, information, form, signature,
OTP and `sign_document`. `uiStepMode` decides how the UI steps run. The
concepts are in [Flows and UI steps](../flows-and-ui-steps.md).

| `uiStepMode` | UI steps | Flows that can run |
| --- | --- | --- |
| `UnicusUiStepMode.webView` (default) | Shown by the Unicus flow screens in a secure web view owned by the SDK. | All. |
| `UnicusUiStepMode.disabled` | Not allowed. | Only flows with biometric and server steps. |
| Custom (your own screens) | Rendered by your app. | **Not available from Flutter.** Native Android and iOS SDKs only. |

## WebView (default)

Nothing to code: the SDK opens the flow screens when a UI step comes and
returns to the camera or to your app when it ends. The screens use your
company branding and the flow texts from the portal.

* The web view is native (Android `WebView`, iOS `WKWebView`) and belongs to
  the SDK: your app does not need `webview_flutter`, and there is no version
  conflict with it.
* It loads only the Unicus flow screens of your environment; the session
  travels through a private channel, never in the URL.
* `secureScreens: true` blocks screenshots on Android and covers the screens
  in the iOS app switcher.
* Android needs an up-to-date *Android System WebView* (Chromium 90 or newer),
  otherwise `webview_unavailable`. On iOS, apps with `WKAppBoundDomains` must
  list the flow screens domain, otherwise `webview_domain_not_allowed`. See
  [Compatibility and security](../compatibility-and-security.md).

## Disabled

For apps that must not show any web content:

{% code overflow="wrap" %}
```dart
await unicus.configure(const UnicusSdkConfig(
  apiKey: '<CUSTOMER_TOKEN>',
  environment: UnicusEnvironment.dev,
  uiStepMode: UnicusUiStepMode.disabled,
));
```
{% endcode %}

Before showing anything, the SDK checks that every pending step can run. If
the flow has any UI step, `start` throws `flow_not_supported` (9020) and the
transaction is closed. Assign a camera-only flow to your company in the portal,
and collect consent in your own app if you need it.

## Custom mode is native-only

The native SDKs offer a Custom mode in which the app draws consent, form,
signature, OTP and `sign_document` with its own screens while the SDK makes
the calls to Unicus. **It is not exposed to Dart.** If your Flutter app cannot
use the web view and needs UI steps, integrate the native SDKs directly (for
example in an add-to-app module or a platform-specific screen):

* [Android Custom UI steps](../android/custom-ui-steps.md)
* [iOS Custom UI steps](../ios/custom-ui-steps.md)

The `stepProgress` event (`sign_document` progress) comes from that mode, so
Flutter apps normally do not receive it. In WebView mode a `sign_document`
step is reported with `stepCompleted` / `stepFailed`, and the result codes
4011–4014 are the same on every platform.
