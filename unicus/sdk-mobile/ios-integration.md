---
description: >-
  Integrate Unicus identity verification in native iOS applications (Swift)
  using only the Unicus SDK.
---

# IOS INTEGRATION

The Unicus iOS SDK lets your native iOS application run identity verification.
Your app integrates **Unicus** only. The SDK creates the Unicus transaction,
fetches the session configuration, applies the company branding, presents the
native verification screens, processes the encrypted biometric data through
Unicus, and returns the final result.

{% hint style="info" %}
The Unicus iOS SDK includes everything required for identity verification:
camera capture, liveness, document scanning, encryption, and the Unicus API
calls. Do not add other biometric or document-capture libraries for this flow.
Your app only imports `UnicusSDK`.
{% endhint %}

## What Unicus will provide

Before starting the integration, request the following values from your Unicus
administrator or Tekbees support team.

| Value | Description | Example |
| --- | --- | --- |
| `baseUrl` | Unicus API environment URL. | `https://alpha.idunicus.com:8080` |
| `apiKey` | Customer token generated for your company. | `<UNICUS_CUSTOMER_TOKEN>` |

The Unicus internal keys (the Tekbees session device id and the Unicus
verification engine key) are embedded inside the Unicus iOS SDK. The customer
app must not request, store, or pass those internal keys.

The customer app uses `apiKey` only for its Unicus Customer Token. Transaction
creation uses that customer token through `X-Customer-ID`. The active-country
lookup uses the same customer token through `X-Device-ID`, because that endpoint
uses it to resolve the customer's enabled countries. Session calls such as
`/get-restart-session` and `/sdk-execution-keys` use the Tekbees session device
id embedded in the SDK through `X-Device-ID`.

{% hint style="warning" %}
Use the values for the correct environment. Sandbox, staging, and production
credentials are different. Do not commit production credentials in public
repositories.
{% endhint %}

## Requirements

| Platform | Requirement |
| --- | --- |
| iOS | iOS `15.0` or newer |
| Xcode | Xcode `16` or newer |
| Language | Swift `5.9` or newer (completion handlers and `async/await` are both supported) |
| Devices | Physical iPhone with camera for full validation |
| Permissions | Camera is required. Location is optional and the SDK continues if the user denies it. |

## 1. Review the customer package

Tekbees provides a customer package named like this:

{% code overflow="wrap" %}
```text
unicus_sdk_ios_0.1.0_customer_package.zip
```
{% endcode %}

When unzipped, it contains:

{% code overflow="wrap" %}
```text
unicus_sdk_ios_0.1.0_customer_package/
  README.md
  TECHNICAL_INTEGRATION.md
  sdk/
    Frameworks/
      UnicusSDK.xcframework
      <Unicus verification engine>.xcframework
      <Unicus verification engine>ForDevelopment.xcframework
    UnicusSDK.podspec
    Package.swift
  example/
    UnicusSDKExample.xcodeproj
    UnicusSDKExample/
      Config/Unicus.xcconfig
      ContentView.swift
      ExampleViewModel.swift
      SampleUnicusTexts.swift
```
{% endcode %}

| Path | Purpose |
| --- | --- |
| `README.md` | Quick start for the package. |
| `TECHNICAL_INTEGRATION.md` | Technical reference for the customer's development team. |
| `sdk/Frameworks/UnicusSDK.xcframework` | Unicus SDK public module (device and simulator slices). |
| `sdk/Frameworks/*.xcframework` | Unicus verification engine binaries required by the SDK. The framework whose name ends with `ForDevelopment` is used only by Debug builds on physical devices. |
| `sdk/UnicusSDK.podspec` | CocoaPods integration file. |
| `sdk/Package.swift` | Swift Package Manager integration file. |
| `example/` | Runnable Xcode project (SwiftUI) already configured to use the included binaries. |
| `example/UnicusSDKExample/SampleUnicusTexts.swift` | Editable language/text maps using public `Unicus_` keys. |

The package contains the closed Unicus SDK with all its internal components,
resources, and a runnable example. Customer applications integrate only
Unicus.

## 2. Run the included example

Before changing your own app, run the included example to confirm the
environment, device, permissions, and customer token.

1. Open `example/UnicusSDKExample.xcodeproj` in Xcode.
2. Edit `example/UnicusSDKExample/Config/Unicus.xcconfig` with the values
   provided by Tekbees:

{% code overflow="wrap" %}
```properties
UNICUS_BASE_URL = https:/$()/alpha.idunicus.com:8080
UNICUS_API_KEY = <UNICUS_CUSTOMER_TOKEN>
```
{% endcode %}

{% hint style="info" %}
In `xcconfig` files, `//` starts a comment. Write the URL as
`https:/$()/host` so the double slash is preserved.
{% endhint %}

3. Select your development team in *Signing & Capabilities*.
4. Run the `UnicusSDKExample` scheme on a physical iPhone.

The example screen asks only for:

1. Document id.
2. Document type: `ID`, `FD`, `PP`, or `DL`.

The example may also show the iOS location permission prompt. If the user does
not share location, the transaction continues without location data. If your
Unicus account has more than one active country, the example shows a country
selector after the transaction id is created. If there is only one active
country, the SDK selects it automatically and continues.

The SDK creates the transaction, reads the customer session configuration,
applies the theme and text, presents the native verification experience,
processes the encrypted requests, and returns the result. The example also
shows the sanitized Unicus API responses received during the flow.

For a compile-only check on the simulator:

{% code overflow="wrap" %}
```bash
cd unicus_sdk_ios_0.1.0_customer_package/example
xcodebuild -project UnicusSDKExample.xcodeproj -scheme UnicusSDKExample \
  -sdk iphonesimulator -destination 'generic/platform=iOS Simulator' \
  CODE_SIGNING_ALLOWED=NO build
```
{% endcode %}

Use a physical iPhone with camera for full end-to-end verification. The
simulator cannot run the native verification screens.

## 3. Add the dependency

For your own app, copy the `sdk` folder from the customer package into your
application repository, for example:

{% code overflow="wrap" %}
```text
your_ios_app/
  Vendor/
    Unicus/
      Frameworks/
      UnicusSDK.podspec
      Package.swift
```
{% endcode %}

Choose one integration method.

### Option A: Xcode (manual frameworks)

1. Drag every `.xcframework` inside `Vendor/Unicus/Frameworks` into your app
   target under *General > Frameworks, Libraries, and Embedded Content*.
2. Set each one to **Embed & Sign**.

### Option B: CocoaPods

{% code overflow="wrap" %}
```ruby
platform :ios, '15.0'

target 'YourApp' do
  use_frameworks!
  pod 'UnicusSDK', :path => 'Vendor/Unicus'
end
```
{% endcode %}

Then install pods:

{% code overflow="wrap" %}
```bash
pod install
```
{% endcode %}

### Option C: Swift Package Manager

In Xcode select *File > Add Package Dependencies... > Add Local...*, choose the
`Vendor/Unicus` folder, and link the `UnicusSDK` product to your app target.

All the frameworks in `sdk/Frameworks` are part of the Unicus SDK and must be
embedded together. Do not request Tekbees internal keys.

## 4. Configure iOS

Open your app `Info.plist` and add camera and location usage descriptions.

{% code overflow="wrap" %}
```xml
<key>NSCameraUsageDescription</key>
<string>Camera access is required to verify your identity.</string>
<key>NSLocationWhenInUseUsageDescription</key>
<string>Location access helps complete the identity verification context.</string>
```
{% endcode %}

If location permission is denied, disabled, or not configured, the Unicus SDK
continues the transaction without location data.

No changes are required in `AppDelegate`, `SceneDelegate`, or any other native
file. The SDK presents the verification screens on top of the view controller
you pass, or on top of the top-most view controller when none is provided.

### Debug builds on physical devices

The Unicus verification engine is delivered as a production binary and a
development binary (the framework whose name ends with `ForDevelopment`). Debug
builds installed on a physical device must use the development binary. The
included example does this automatically with a *Run Script* build phase. Copy
that phase to your own target when you run Debug builds on devices:

{% code overflow="wrap" expandable="true" %}
```bash
set -euo pipefail
if [[ "${CONFIGURATION}" != "Debug" || "${PLATFORM_NAME}" != "iphoneos" ]]; then
  exit 0
fi
if [[ "${CODE_SIGNING_ALLOWED:-YES}" == "NO" || -z "${EXPANDED_CODE_SIGN_IDENTITY:-}" ]]; then
  exit 0
fi
DEVELOPMENT_FRAMEWORK="${SRCROOT}/Vendor/Unicus/Frameworks/$(ls "${SRCROOT}/Vendor/Unicus/Frameworks" | grep ForDevelopment)"
EMBEDDED_FRAMEWORK=$(find "${TARGET_BUILD_DIR}/${FRAMEWORKS_FOLDER_PATH}" -maxdepth 1 -name "*.framework" -not -name "UnicusSDK.framework" | head -1)
if [[ -d "${DEVELOPMENT_FRAMEWORK}" && -d "${EMBEDDED_FRAMEWORK}" && -f "${EMBEDDED_FRAMEWORK}/swap-development-framework.sh" ]]; then
  sh "${EMBEDDED_FRAMEWORK}/swap-development-framework.sh" "${DEVELOPMENT_FRAMEWORK}"
fi
```
{% endcode %}

Add the phase after *Embed Frameworks*. Release and App Store builds keep the
production binary.

## 5. Configure the SDK

Configure the shared `UnicusSdk` instance in the part of your app that owns the
verification flow.

{% code overflow="wrap" %}
```swift
import UnicusSDK

func configureUnicus() {
    UnicusSdk.shared.configure(
        UnicusSdkConfig(
            baseUrl: "<UNICUS_BASE_URL>",
            apiKey: "<UNICUS_CUSTOMER_TOKEN>",
            collectLocationOnStart: true
        )
    )
}
```
{% endcode %}

Call `configureUnicus()` before starting the first verification. A common place
is after the user reaches the screen where identity verification can begin.
`collectLocationOnStart` is optional and defaults to `true`. Set it to `false`
only when your app must not request location for this flow.

## 6. Start a verification

Send the document type and the user's document number to `start`. Pass the view
controller that should present the verification (optional). Completions are
delivered on the main queue.

{% code overflow="wrap" %}
```swift
import UnicusSDK

func startUnicusVerification(from viewController: UIViewController) {
    UnicusSdk.shared.start(
        .enrollmentVerify(
            document: UnicusDocument(
                type: .id,
                externalDatabaseRefId: "123456789"
            )
        ),
        from: viewController
    ) { result in
        switch result {
        case .success(let verification):
            if verification.success {
                // The identity verification was successful.
            } else {
                // Show a retry, rejection, or support path according to your business flow.
            }
        case .failure(let error):
            // The verification could not start (configuration, network, or session error).
            print(error.code, error.message)
        }
    }
}
```
{% endcode %}

With `async/await`:

{% code overflow="wrap" %}
```swift
do {
    let result = try await UnicusSdk.shared.start(request, from: viewController)
    // Handle UnicusVerificationResult here.
} catch let error as UnicusSdkError {
    // error.code and error.message describe the failure.
}
```
{% endcode %}

When `start` is called, the SDK internally creates the transaction using the
standard Unicus mobile process:

{% code overflow="wrap" %}
```json
{
  "documentType": "ID",
  "externalDatabaseRefID": "123456789",
  "process": "ENROLLMENT-VERIFY"
}
```
{% endcode %}

Your app should not call `/start-mobile-transaction` or `/get-restart-session`
manually for the standard iOS integration. It should also not ask the user or
the application developer to select a process value.

The standard sequence is:

1. Your app calls `UnicusSdk.shared.start(...)`.
2. The SDK calls `/start-mobile-transaction` and receives a new transaction id
   `tid`.
3. The SDK requests location from the platform when enabled. If the user denies
   permission or the device cannot provide location, the SDK continues.
4. The SDK calls `/company-countries`, reads the active countries for the
   customer, and selects the only active country automatically when applicable.
5. The SDK calls `/get-restart-session` using the `tid` and selected country
   when one is available.
6. The SDK applies the company colors, logo, and Unicus verification text.
7. The SDK presents the native verification screen.
8. The SDK sends the encrypted verification data to Unicus.
9. Your app receives one `UnicusVerificationResult`.

Only one verification can run at a time. Calling `start` while another
verification is active fails with the error code `session_active`.

## Country selection

For most integrations, calling `start(...)` is enough. If your Unicus account
has only one active country, the SDK selects it automatically. If the account
has several active countries and your app must let the user choose, use
`prepareEnrollmentVerify(...)` first.

{% code overflow="wrap" %}
```swift
func startWithCountrySelection(from viewController: UIViewController) async throws {
    let prepared = try await UnicusSdk.shared.prepareEnrollmentVerify(
        document: UnicusDocument(type: .id, externalDatabaseRefId: "123456789")
    )

    let selectedCountry: String? = prepared.requiresCountrySelection
        ? await showYourCountryPicker(prepared.countries)
        : prepared.defaultCountry?.code

    let result = try await UnicusSdk.shared.start(
        prepared.toRequest(country: selectedCountry),
        from: viewController
    )

    // Handle UnicusVerificationResult here.
}
```
{% endcode %}

`prepareEnrollmentVerify(...)` creates the Unicus transaction id, optionally
collects location, and returns the active countries. It does not present the
native verification screen. Call `start(...)` with `prepared.toRequest(...)`
after your app has selected the country. The country value should be an
ISO 3166-1 alpha-2 code such as `CO`, `US`, or `MX`.

## Document types

Use the enum provided by the SDK.

| Swift value | API value | Description |
| --- | --- | --- |
| `UnicusDocumentType.id` | `ID` | National ID document |
| `UnicusDocumentType.foreignDocument` | `FD` | Foreign document |
| `UnicusDocumentType.passport` | `PP` | Passport |
| `UnicusDocumentType.driverLicense` | `DL` | Driver license |

## Native flow behavior

The default native flow is handled internally by the SDK. No flow field is
required in the customer application.

Unicus checks the current session state:

| Session state | Behavior |
| --- | --- |
| User is already enrolled | The SDK starts face authentication. |
| User is not enrolled | The SDK starts face and document enrollment. |

For the standard iOS integration, the application should only provide document
type and document id.

## Read the result

`start` delivers a `UnicusVerificationResult`.

| Field | Description |
| --- | --- |
| `success` | `true` when the verification completed successfully. |
| `outcome` | Normalized result category: `.success`, `.warning`, `.failed`, `.canceled`, `.error`, or `.unknown`. |
| `tid` | Unicus transaction id created by the SDK. |
| `resultCode` | Unicus transaction result code, when available. If Unicus has finalized the transaction, this value takes precedence over the native screen status. |
| `resultMessage` | Human-readable result message, when available. |
| `status` | Native screen status for technical diagnostics. |
| `sessionError` | `true` only for a technical interruption, cancellation, or native/session error. Business outcomes should be handled through `outcome` and `resultCode`. |

Recommended result handling:

{% code overflow="wrap" %}
```swift
switch result.outcome {
case .success:
    // Continue with the verified user.
    break
case .warning:
    // Continue or route to manual review according to your business rules.
    break
case .canceled:
    // Allow the user to retry.
    break
case .failed, .error:
    // Show the configured failure flow.
    break
case .unknown:
    // Show a support or retry path.
    break
}
```
{% endcode %}

`UnicusResultCode.messageFor(result.resultCode)` returns the standard Unicus
message for a result code when your app needs a default text.

## Listen to progress events

You can register an event handler before starting verification. Events are
delivered on the main queue.

{% code overflow="wrap" %}
```swift
UnicusSdk.shared.eventHandler = { event in
    print("Unicus event: \(event.name)")
    print("Transaction id: \(event.tid ?? "-")")
    print("Message: \(event.message ?? "-")")
}
```
{% endcode %}

Remove the handler when the screen is dismissed:

{% code overflow="wrap" %}
```swift
UnicusSdk.shared.eventHandler = nil
```
{% endcode %}

Common event names: `locationCollected`, `locationSkipped`, `sessionPrepared`,
`processRequest`, `livenessProcessed`, `enrollmentProcessed`,
`authenticationComplete`, `idScanProcessed`, `idScanBackRequired`,
`idScanUserConfirmation`, `idScanComplete`, `completed`, `error`, and
`nativeExit`.

## Customize verification text

The SDK includes default English text based on the current Unicus web SDK
configuration. If your application needs different language or wording, provide
text overrides when configuring Unicus.

The example app includes complete English and Spanish Unicus text maps based on
the current web SDK language configuration in
`example/UnicusSDKExample/SampleUnicusTexts.swift`. Use that file as the
starting point for your own language file. The example start screen
intentionally asks only for document id and document type; text is changed in
code, not through the demo UI.

{% code overflow="wrap" %}
```swift
UnicusSdk.shared.configure(
    UnicusSdkConfig(
        baseUrl: "<UNICUS_BASE_URL>",
        apiKey: "<UNICUS_CUSTOMER_TOKEN>",
        verificationTextOverrides: SampleUnicusTexts.overrides
    )
)
```
{% endcode %}

Unicus merges your overrides with the default text and applies the final text
map to the native verification screens before the session opens.
Internal native text keys are adapted by the SDK, and `<br/>` line breaks are
converted to native line breaks.

Common text keys:

| Swift key | Screen text |
| --- | --- |
| `UnicusVerificationTextKey.actionImReady` | Ready button. |
| `UnicusVerificationTextKey.actionContinue` | Continue button. |
| `UnicusVerificationTextKey.actionTryAgain` | Retry button. |
| `UnicusVerificationTextKey.feedbackCenterFace` | Face alignment feedback. |
| `UnicusVerificationTextKey.initializingCamera` | Camera initialization message. |
| `UnicusVerificationTextKey.idScanTypeSelectionHeader` | Document scan title. |
| `UnicusVerificationTextKey.resultFaceScanUploadMessage` | Face upload message. |

For advanced OCR confirmation labels, coordinate the
`verificationOcrLocalization` dictionary with Tekbees support.

## Optional API logs for testing

During sandbox testing, API logs can help your team confirm the responses
received from Unicus.

{% hint style="warning" %}
API logs are for development and QA only. Do not enable sensitive logs in
production builds.
{% endhint %}

{% code overflow="wrap" %}
```swift
UnicusSdk.shared.configure(
    UnicusSdkConfig(
        baseUrl: "<UNICUS_BASE_URL>",
        apiKey: "<UNICUS_CUSTOMER_TOKEN>",
        enableApiLogging: true
    )
)

UnicusSdk.shared.apiLogHandler = { entry in
    print(entry.toPrettyJson())
}
```
{% endcode %}

By default, logs are sanitized. Encrypted biometric blobs, OCR payloads,
document data, and session tokens are redacted or truncated.

## Complete button example

{% code overflow="wrap" expandable="true" %}
```swift
import UIKit
import UnicusSDK

final class VerificationViewController: UIViewController {

    private let verifyButton = UIButton(type: .system)

    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemBackground

        UnicusSdk.shared.configure(
            UnicusSdkConfig(
                baseUrl: "<UNICUS_BASE_URL>",
                apiKey: "<UNICUS_CUSTOMER_TOKEN>"
            )
        )

        verifyButton.setTitle("Verify identity", for: .normal)
        verifyButton.addTarget(self, action: #selector(startVerification), for: .touchUpInside)
        verifyButton.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(verifyButton)
        NSLayoutConstraint.activate([
            verifyButton.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            verifyButton.centerYAnchor.constraint(equalTo: view.centerYAnchor)
        ])
    }

    @objc private func startVerification() {
        verifyButton.isEnabled = false
        verifyButton.setTitle("Verifying...", for: .normal)

        UnicusSdk.shared.start(
            .enrollmentVerify(
                document: UnicusDocument(type: .id, externalDatabaseRefId: "123456789")
            ),
            from: self
        ) { [weak self] result in
            guard let self = self else { return }
            self.verifyButton.isEnabled = true
            self.verifyButton.setTitle("Verify identity", for: .normal)

            let message: String
            switch result {
            case .success(let verification):
                message = verification.success
                    ? "Identity verified successfully"
                    : (verification.resultMessage ?? "Identity could not be verified")
            case .failure(let error):
                message = error.message
            }

            let alert = UIAlertController(title: "Unicus", message: message, preferredStyle: .alert)
            alert.addAction(UIAlertAction(title: "OK", style: .default))
            self.present(alert, animated: true)
        }
    }
}
```
{% endcode %}

SwiftUI apps can call the same API from a view model using `async/await`; the
included example shows this pattern.

## Appearance

The SDK automatically applies the company appearance configured in Unicus and
returned by the session endpoint.

| Field | Behavior |
| --- | --- |
| `backgroundColor` | Native verification screen background color. |
| `windowColor` | Primary native verification color. |
| `buttonColor` | Button, progress, frame, and OCR accent color. |
| `textColor` | Button and feedback text color. |
| `logo` | Company logo. |

iOS uses the remote logo URL returned by Unicus when the URL points to a raster
image such as PNG or JPG. If the logo cannot be downloaded, the SDK shows the
Unicus logo included in the package.

## Testing checklist

Use a physical device for the final validation.

1. Build the app for the simulator to confirm the frameworks are linked.
2. Run the app on a signed physical iPhone and accept camera permission.
3. Test one valid document with document type `ID`, `FD`, `PP`, or `DL`.
4. Test location permission accepted and denied. Both paths should continue.
5. If the account has one active country, confirm the flow continues without
   showing a country selector.
6. If the account has several active countries, confirm your app shows a
   country selector before presenting the native verification screen.
7. Confirm the native verification screen opens.
8. Confirm the company colors and logo are displayed as expected.
9. Confirm your app receives a `UnicusVerificationResult`.
10. Confirm the transaction appears in the Unicus administrative portal.

Useful commands:

{% code overflow="wrap" %}
```bash
xcrun xctrace list devices
xcodebuild -scheme YourApp -destination 'id=<DEVICE_ID>' build
```
{% endcode %}

For physical iPhones, always use a signed build from Xcode. Simulator builds are
compile checks only.

## Troubleshooting

| Problem | What to check |
| --- | --- |
| Error code `not_configured` | Confirm `UnicusSdk.shared.configure(...)` runs before `start(...)`. |
| Error code `session_active` | A previous verification is still running. Wait for its completion before starting again. |
| Error code `no_view_controller` | Pass the presenting view controller in `start(_:from:)` or make sure a key window with a root view controller exists. |
| `dyld: Library not loaded` at launch | Confirm every `.xcframework` from `sdk/Frameworks` is set to **Embed & Sign** in the app target. |
| Camera permission is denied | Confirm `NSCameraUsageDescription` is present in `Info.plist`. |
| Location permission is denied | This does not stop the transaction. The SDK continues without location data. |
| Country selector does not appear | Confirm the customer account has more than one active country in Unicus. For one active country, the SDK selects it automatically. |
| No transaction id is returned | Confirm `baseUrl`, `apiKey`, document type, and document number. |
| Native verification does not open on a device Debug build | Confirm the *Run Script* phase that swaps the Unicus development framework runs after *Embed Frameworks*. |
| Native verification does not open | Confirm you are running on a supported physical device with camera access. |
| Company colors do not appear | Confirm `/get-restart-session` returns `windowColor`, `buttonColor`, and `textColor`. |
| Logo does not appear | Confirm `/get-restart-session` returns `logo` as an HTTPS PNG or JPG URL. SVG URLs are not valid for the native mobile logo. |

## Support

When contacting support, include:

1. Environment URL.
2. App platform and version.
3. Device model and OS version.
4. Unicus transaction id `tid`, if it was created.
5. Result code and result message, if available.
6. A short description of the step where the issue happened.
