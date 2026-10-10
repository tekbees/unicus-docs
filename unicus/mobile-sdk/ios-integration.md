---
description: >-
  Integrate Unicus identity verification in a native iOS app (Swift or
  Objective-C) with the Unicus iOS SDK.
---

# iOS integration

The Unicus iOS SDK runs a complete identity verification inside your native iOS
app. Your app configures the SDK with its Customer Token, calls `start` with the
person's document and receives one result. The SDK creates the transaction,
applies your company branding, runs the flow assigned in the administrative
portal (camera screens, flow screens such as consent, form or OTP) and talks to
Unicus. Your app only imports `UnicusSDK`: do not add other biometric or
document-capture libraries.

{% hint style="warning" %}
**Coming soon.** The mobile SDKs are not yet available in production. Only the
DEV environment is available today; Tekbees will announce the production
release. Ask Tekbees for a DEV Customer Token to prepare your integration.
{% endhint %}

{% code overflow="wrap" %}
```swift
import UnicusSDK

UnicusSdk.shared.configure(UnicusSdkConfig(apiKey: "<CUSTOMER_TOKEN>", environment: .dev))

UnicusSdk.shared.start(
    .enrollmentVerify(document: UnicusDocument(type: .id, externalDatabaseRefId: "123456789")),
    from: self
) { result in
    // Result<UnicusVerificationResult, UnicusSdkError>
}
```
{% endcode %}

## Pages in this section

| Page | What you find |
| --- | --- |
| [Quick start](ios/quick-start.md) | Dependency, `Info.plist`, ten lines of code and the outcomes. |
| [Installation](ios/installation.md) | Package contents, Xcode / CocoaPods / Swift Package Manager, permissions, the Debug-on-device build phase. |
| [Configuration](ios/configuration.md) | Every `UnicusSdkConfig` option, environments, logging, branding. |
| [Start a verification](ios/start-a-verification.md) | Document, country selection, existing transaction, location, cancelling, threading. |
| [Results and events](ios/results-and-events.md) | `UnicusVerificationResult`, outcomes, errors, progress events, API logs. |
| [Custom UI steps](ios/custom-ui-steps.md) | Render the flow screens with your own UI (`.custom` provider), including `sign_document`. |
| [Texts and languages](ios/texts-and-languages.md) | `Unicus_*` text keys and the language of each screen. |
| [Objective-C](ios/objective-c.md) | The `UnicusSdkObjC` facade. |
| [Release checklist](ios/release-checklist.md) | What to check before going live. |

Shared by every mobile SDK: [Overview](overview.md),
[Flows and UI steps](flows-and-ui-steps.md),
[Results and resuming](results-and-resuming.md),
[Result codes](result-codes.md),
[Errors and troubleshooting](errors-and-troubleshooting.md),
[Compatibility and security](compatibility-and-security.md),
[Versioning](versioning.md) and [Support](support.md).

## Requirements

| Item | Requirement |
| --- | --- |
| iOS | 15.0 or newer |
| Xcode | 16 or newer |
| Swift | 5.9 or newer (completion handlers and `async/await`). Objective-C through a facade. |
| Device | Physical iPhone with camera for real verifications. The simulator builds and runs the app but cannot run the camera screens. |
| Permissions | Camera (required). Location (optional; the verification continues without it). |
| Network | HTTPS access to the Unicus API and flow app domains of your environment. See [Compatibility and security](compatibility-and-security.md). |
| Credentials | Your company's Customer Token for the environment. Nothing else: the SDK embeds every other key. |
| Distribution | Package delivered by Tekbees (ZIP). Hosted Swift Package Manager and CocoaPods repositories: coming soon. |
