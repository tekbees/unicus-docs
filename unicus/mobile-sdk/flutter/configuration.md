---
description: >-
  Every option of UnicusSdkConfig in the Flutter SDK: environments, UI step
  mode, resuming, location, logs, texts and branding.
---

# Configuration

Call `configure` once before any other operation, for example when your app
starts or right before the first verification. Calling it again replaces the
configuration for the next `start`.

{% code overflow="wrap" %}
```dart
final unicus = UnicusSdkFlutter();

await unicus.configure(const UnicusSdkConfig(
  apiKey: '<CUSTOMER_TOKEN>',
  environment: UnicusEnvironment.dev,
));
```
{% endcode %}

That is the whole required configuration: your
[Customer Token](../../sdk-web-v5/customer-token.md) and the environment. The
API address, the flow screens address and the internal keys come with the SDK
for each environment.

{% hint style="warning" %}
Do not hard-code the Customer Token in source control. Load it from your build
configuration or your backend, and keep one token per environment.
{% endhint %}

## Environments

| `UnicusEnvironment` | Status |
| --- | --- |
| `dev` | Available. Use it for development and tests. |
| `staging` | Pending: `configure` throws `environment_not_available`. |
| `production` | Pending: `configure` throws `environment_not_available`. Tekbees will announce it. |

`environment_not_available` is thrown by `configure`, and its message says what
is missing. When Tekbees publishes an environment you only update the SDK
package: your code keeps the same `environment` value.

## All options

| Option | Default | When to change it |
| --- | --- | --- |
| `apiKey` | required | Your Customer Token. |
| `environment` | `null` | Always set it (`UnicusEnvironment.dev` today). Required unless Tekbees gives you a `baseUrl`. |
| `baseUrl` | `''` | Advanced. Only when Tekbees asks you to use an explicit API address; it overrides the address of `environment`. |
| `flowAppUrl` | `null` | Advanced. `https` origin of the flow screens, only for an environment the SDK does not know. |
| `uiStepMode` | `UnicusUiStepMode.webView` | `UnicusUiStepMode.disabled` if your app must not show web content (camera-only flows). See [UI steps](ui-steps.md). |
| `resumeOpenTransactions` | `true` | `false` to always create a new transaction instead of continuing the open one of the same document. |
| `secureScreens` | `false` | `true` to protect the flow screens: no screenshots or screen recording on Android, covered in the iOS app switcher. Recommended for regulated apps. |
| `prependConsent` | `false` | `true` to show a consent step first when the flow has none (as the web does). Off because mobile apps usually collect consent themselves. |
| `collectLocationOnStart` | `true` | `false` to never ask for location. With `true`, location is collected only if your app declares the permission and the user grants it. |
| `requestTimeout` | 2 minutes | Timeout of each call to Unicus. Raise it only for very slow networks. |
| `additionalHeaders` | `{}` | Extra HTTP headers on every call to Unicus, for example one your corporate proxy requires. |
| `verificationTextOverrides` | `{}` | Your wording or language for the camera screens. See [Texts and languages](texts-and-languages.md). |
| `verificationOcrLocalization` | `null` | Labels of the document data confirmation screen. Coordinate the dictionary with Tekbees support. |
| `androidLogoResourceName` | `null` | Android only: name of a drawable of your app shown as the logo in the camera screens (for example `customer_logo`). |
| `enableApiLogging` | `false` | `true` in development to receive sanitized logs on `unicus.apiLogs`. |
| `includeSensitiveApiLogData` | `false` | Debug only: keeps encrypted payloads and tokens in the logs. Never in a release build. |
| `apiLogStringLimit` | `1200` | Maximum length of each string in the logs before truncation. |

## Recommended configuration

{% code overflow="wrap" %}
```dart
await unicus.configure(UnicusSdkConfig(
  apiKey: customerToken, // loaded from your configuration
  environment: UnicusEnvironment.dev,
  secureScreens: true,
  verificationTextOverrides: myUnicusTexts, // optional, see Texts and languages
  androidLogoResourceName: 'customer_logo', // optional, Android logo
));
```
{% endcode %}

## Logs

API logging is off by default. Turn it on only while developing:

{% code overflow="wrap" %}
```dart
await unicus.configure(const UnicusSdkConfig(
  apiKey: '<CUSTOMER_TOKEN>',
  environment: UnicusEnvironment.dev,
  enableApiLogging: true,
));

unicus.apiLogs.listen((entry) {
  debugPrint(entry.summary); // "/path -> 200 (350 ms)"
});
```
{% endcode %}

Request blobs, response blobs, document data, OCR results and session tokens
are replaced by `<redacted>` unless `includeSensitiveApiLogData` is `true`.
Keep both flags off in release builds. See
[Results and events](results-and-events.md#api-logs).

## Branding

Colours and logo come from your company configuration in the administrative
portal; you do not pass them in code.

* **Colours** (background, primary, buttons, text) are applied to the camera
  screens and the flow screens on both platforms.
* **Logo on iOS:** the SDK loads the logo configured in the portal (an
  `https` PNG or JPG, an inline raster image, or a simple SVG made of basic
  paths). If it cannot be decoded it falls back to the Unicus logo.
* **Logo on Android:** the camera screens can only show an image from your
  app's resources. Add your logo to `android/app/src/main/res/drawable` and
  pass its name in `androidLogoResourceName`; otherwise the Unicus logo is
  shown. The flow screens (web view) use the portal logo on both platforms.

Next: [Start a verification](start-a-verification.md).
