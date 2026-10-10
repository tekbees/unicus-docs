---
description: >-
  Every option of UnicusSdkConfig on iOS: environments, UI step mode, resume,
  location, logging, texts and branding.
---

# Configuration

Configure the shared instance once, before the first verification (for example
at app start-up). Calling `configure` again replaces the configuration for the
next verification.

{% code overflow="wrap" %}
```swift
import UnicusSDK

UnicusSdk.shared.configure(UnicusSdkConfig(apiKey: "<CUSTOMER_TOKEN>", environment: .dev))
```
{% endcode %}

That line is enough for most apps. The environment selects the Unicus API, the
flow app and the keys embedded in the SDK; your app never handles URLs or
internal keys.

## Changing options

The simple initializer also accepts the most common options:

{% code overflow="wrap" %}
```swift
let config = UnicusSdkConfig(
    apiKey: "<CUSTOMER_TOKEN>",
    environment: .dev,
    uiStepMode: .webView,           // .webView (default) | .custom(provider) | .disabled
    resumeOpenTransactions: true,
    collectLocationOnStart: false,
    enableApiLogging: false
)
UnicusSdk.shared.configure(config)
```
{% endcode %}

Every other option is a `var` you set before calling `configure`:

{% code overflow="wrap" %}
```swift
var config = UnicusSdkConfig(apiKey: "<CUSTOMER_TOKEN>", environment: .dev)
config.secureScreens = true
config.verificationTextOverrides = [UnicusVerificationTextKey.actionImReady: "ESTOY LISTO"]
UnicusSdk.shared.configure(config)
```
{% endcode %}

## Options

| Option | Default | When to change it |
| --- | --- | --- |
| `apiKey` | — (required) | Your company's [Customer Token](../../sdk-web-v5/customer-token.md) for the environment. The only credential you provide. |
| `environment` | — (required) | `.dev` today. See [Environments](#environments). |
| `uiStepMode` | `.webView` | How flow screens (consent, info, form, signature, OTP, `sign_document`) are shown. `.custom(provider)` to render them with your UI ([Custom UI steps](custom-ui-steps.md)); `.disabled` for camera-only flows. See [Flows and UI steps](../flows-and-ui-steps.md). |
| `resumeOpenTransactions` | `true` | `false` to always create a new transaction instead of continuing the open one of the same document. See [Results and resuming](../results-and-resuming.md). |
| `collectLocationOnStart` | `true` | `false` when your app must not request location. Can be overridden per request. |
| `secureScreens` | `false` | `true` to cover the flow screens in the app switcher snapshot. |
| `prependConsent` | `false` | `true` to show a Unicus consent screen first when the flow has no consent step. Mobile apps usually collect consent themselves. |
| `verificationTextOverrides` | `[:]` | Wording or language of the camera screens. See [Texts and languages](texts-and-languages.md). |
| `verificationOcrLocalization` | `nil` | Labels of the document data confirmation screen. Leave `nil` unless Tekbees gives you a map. |
| `requestTimeout` | `120` s | Timeout of each Unicus API call. Lower only if your UX needs it. |
| `additionalHeaders` | `[:]` | Extra headers on every Unicus API call (for example a corporate proxy requirement). |
| `enableApiLogging` | `false` | `true` in development to receive sanitized API logs in `apiLogHandler`. See [Results and events](results-and-events.md#api-logs). |
| `includeSensitiveApiLogData` | `false` | Keeps tokens and encrypted payloads in the logs. Local debugging only; never in production. |
| `apiLogStringLimit` | `1200` | Longer strings are truncated in the logs. |
| `flowAppUrl` | `nil` | Not needed with `environment`. Only for an environment the SDK does not know (https). |

The advanced initializer `UnicusSdkConfig(baseUrl:apiKey:…)` exists for API
origins that are not a known environment. Prefer `environment`: with an unknown
origin the session calls fail with `missing_session_device_id`.

## Environments

| `UnicusEnvironment` | Status |
| --- | --- |
| `.dev` | Available. |
| `.staging` | Pending. |
| `.production` | Pending. Tekbees will announce it. |

With a pending environment every call (`start`, `prepareEnrollmentVerify`,
`getCompanyCountries`) fails at once with `environment_not_available`;
`error.details` lists what is missing. You can check it in advance:

{% code overflow="wrap" %}
```swift
if !UnicusEnvironment.production.isAvailable {
    print(UnicusEnvironment.production.missingValues)
}
```
{% endcode %}

Each environment has its own Customer Token. Keep the token and the environment
together in your build configuration (for example one `.xcconfig` per scheme)
so a DEV token never reaches a production build.

## Logging and events

Two handlers on the shared instance, both called on the main queue:

{% code overflow="wrap" %}
```swift
UnicusSdk.shared.eventHandler = { event in print(event.name, event.message ?? "") }
UnicusSdk.shared.apiLogHandler = { entry in print(entry.summary) }   // needs enableApiLogging
```
{% endcode %}

See [Results and events](results-and-events.md).

## Branding

Your app does not configure colours or logo. The SDK reads the company branding
(colours and logo) configured in the administrative portal for the transaction
and applies it to the camera screens and the flow screens. When a colour is
missing, the SDK chooses a readable default; when the logo cannot be loaded, it
shows the Unicus logo.

## Other members

| Member | Purpose |
| --- | --- |
| `UnicusSdk.shared.configuration` | Current configuration, `nil` before `configure`. |
| `UnicusSdk.shared.version` | SDK version (send it with support requests). |
| `UnicusSdk.shared.isSessionActive` | `true` while a verification is being prepared or shown. |
| `UnicusSdk.shared.clearResumeData()` | Forgets stored resume keys (call it on logout). |
