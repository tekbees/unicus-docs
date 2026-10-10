---
description: >-
  Every option of UnicusSdkConfig, the environments, logging, text overrides
  and branding of the Unicus Android SDK.
---

# Configuration

## Simplest configuration

{% tabs %}
{% tab title="Kotlin" %}
{% code overflow="wrap" %}
```kotlin
UnicusSdk.shared.configure(
    UnicusSdkConfig(apiKey = "<CUSTOMER_TOKEN>", environment = UnicusEnvironment.DEV)
)
```
{% endcode %}
{% endtab %}

{% tab title="Java" %}
{% code overflow="wrap" %}
```java
UnicusSdk.getShared().configure(new UnicusSdkConfig("<CUSTOMER_TOKEN>", UnicusEnvironment.DEV));
```
{% endcode %}
{% endtab %}
{% endtabs %}

`apiKey` is your company's
[Customer Token](../../sdk-web-v5/customer-token.md) for that environment. It
is the only credential your app handles: the SDK embeds its own keys and there
is no option to pass them.

Call `configure` once before any other operation (for example in
`Application.onCreate`). Calling it again replaces the configuration; do it
while no verification is running. Without `configure`, every operation fails
with `not_configured`.

## Changing options

{% tabs %}
{% tab title="Kotlin" %}
{% code overflow="wrap" %}
```kotlin
val config = UnicusSdkConfig(apiKey = "<CUSTOMER_TOKEN>", environment = UnicusEnvironment.DEV)
    .copy(
        secureScreens = true,
        enableApiLogging = BuildConfig.DEBUG,
        verificationTextOverrides = mySpanishTexts
    )
UnicusSdk.shared.configure(config)
```
{% endcode %}
{% endtab %}

{% tab title="Java" %}
{% code overflow="wrap" %}
```java
UnicusSdkConfig config = new UnicusSdkConfig.Builder("<CUSTOMER_TOKEN>", UnicusEnvironment.DEV)
        .secureScreens(true)
        .enableApiLogging(BuildConfig.DEBUG)
        .verificationTextOverrides(mySpanishTexts)
        .build();
UnicusSdk.getShared().configure(config);
```
{% endcode %}
{% endtab %}
{% endtabs %}

## Options

| Option | Default | When to change it |
| --- | --- | --- |
| `apiKey` | — (required) | Your Customer Token. One per environment. |
| `environment` | — (required in the simple form) | `DEV`, `STAGING` or `PRODUCTION`. See [Environments](#environments). |
| `uiStepMode` | `UnicusUiStepMode.WebView` | `Custom(provider)` to render consent, form, signature, OTP and document signing with your own screens; `Disabled` for biometric-only flows. See [Custom UI steps](custom-ui-steps.md) and [Flows and UI steps](../flows-and-ui-steps.md). |
| `secureScreens` | `false` | `true` blocks screenshots and screen recording (`FLAG_SECURE`) on the flow screens. Recommended in production. |
| `resumeOpenTransactions` | `true` | When the user leaves and you call `start` again with the same document, the SDK continues the open transaction. Set `false` to always create a new one. See [Results and resuming](../results-and-resuming.md). |
| `collectLocationOnStart` | `true` | `false` never asks for location, even if your app declares the permission. |
| `verificationTextOverrides` | empty | `Unicus_*` keys to change the wording or language of the camera screens. See [Texts and languages](texts-and-languages.md). |
| `verificationOcrLocalization` | `null` | Localization of the screen where the user confirms the data read from the document. Ask Tekbees for the format. |
| `prependConsent` | `false` | `true` adds a consent step at the start when the assigned flow has none. In WebView mode the flow screens show it; in Custom mode your provider must support `consent`. |
| `requestTimeoutMillis` | `120000` | Timeout of each call to Unicus. |
| `additionalHeaders` | empty | Extra HTTP headers for every call to Unicus (for example for a corporate proxy). |
| `enableApiLogging` | `false` | `true` emits sanitized API logs to `setApiLogListener`. Debug builds only. |
| `includeSensitiveApiLogData` | `false` | Keeps redacted fields in the logs. Never in production. |
| `apiLogStringLimit` | `1200` | Maximum length of a string in the API logs before truncation. |
| `flowAppUrl` | `null` | Only when Tekbees gives you a non-standard environment. |
| `baseUrl` | from `environment` | Advanced constructor `UnicusSdkConfig(baseUrl, apiKey, …)`. Only when Tekbees asks you to. |
| `wrapper` | `null` | Reserved for Unicus wrapper SDKs (Flutter). Leave it `null`. |

## Environments

| `UnicusEnvironment` | Status in this version |
| --- | --- |
| `DEV` | Available. |
| `STAGING` | Pending: `configure` throws `environment_not_available`. |
| `PRODUCTION` | Pending: `configure` throws `environment_not_available`. Tekbees will announce it with an SDK update. |

Each environment's addresses and keys are embedded in the SDK version, so a new
environment arrives with a new SDK version. You can check it before
configuring:

{% code overflow="wrap" %}
```kotlin
val env = UnicusEnvironment.fromWireValue(BuildConfig.UNICUS_ENV) ?: UnicusEnvironment.DEV
if (!env.isAvailable) {
    Log.w("Unicus", "Environment $env not available yet: ${env.missingValues}")
}
```
{% endcode %}

`configure` throws `UnicusSdkException` (unchecked in Java) with code
`environment_not_available` when the environment is not usable.

Keep the Customer Token out of source control: inject it per build type
(`buildConfigField`, a properties file ignored by Git, or your CI secrets).

## Logging

To see the calls the SDK makes while you integrate, turn on `enableApiLogging` and register a listener; the entries are
sanitized (biometric data, document data, tokens, form values, OTP codes,
phone, e-mail and location are redacted). See
[Results and events](results-and-events.md#api-logs).

## Branding

Colours come from the company branding in the administrative portal and are
applied automatically to the camera screens and the flow screens. Without a
text colour, the SDK picks white or near-black, whichever contrasts better with
your brand colour; without a button colour, it uses the brand colour.

On Android the logo shown by the SDK's native screens (camera screens and the
top bar of the flow screens) must be a **drawable of your app**. Remote logo
URLs are not loaded there:

{% code overflow="wrap" %}
```kotlin
UnicusSdk.shared.setAndroidLogoResourceName("my_company_logo") // res/drawable/my_company_logo.png
```
{% endcode %}

Without it, the Unicus logo is shown.

## Other members

| Member | Use |
| --- | --- |
| `UnicusSdk.shared.version` | SDK version (send it to [Support](../support.md)). |
| `UnicusSdk.shared.configuration` | Current configuration, `null` before `configure`. |
| `UnicusSdk.shared.isSessionActive` | `true` while a verification is being prepared or shown. |
| `UnicusSdk.shared.clearResumeData(context)` | Forgets the stored resume keys. Call it on logout. |

Next: [Start a verification](start-a-verification.md).
