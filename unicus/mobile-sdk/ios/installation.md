---
description: >-
  Add the Unicus iOS SDK to your Xcode project: package contents, Xcode,
  CocoaPods or Swift Package Manager, permissions and the Debug build phase.
---

# Installation

## Package contents

Tekbees delivers a ZIP named like `unicus_sdk_ios_<version>_customer_package.zip`:

{% code overflow="wrap" %}
```text
unicus_sdk_ios_<version>_customer_package/
  README.md                  Package quick start.
  QUICKSTART.md              One page: dependency, Info.plist, code, outcomes.
  ERRORS.md                  Every UnicusSdkError code.
  TECHNICAL_INTEGRATION.md   Integration reference.
  sdk/
    Frameworks/
      UnicusSDK.xcframework                           Unicus SDK (device + simulator).
      <Unicus verification engine>.xcframework        Verification engine used by the SDK.
      <Unicus verification engine>ForDevelopment.xcframework   Only for Debug builds on a device.
    UnicusSDK.podspec        CocoaPods integration.
    Package.swift            Swift Package Manager integration.
    swap_facetec_development_framework.sh   Build phase script for Debug device builds.
  example/                   Runnable Xcode project that uses the binaries.
```
{% endcode %}

Your app embeds two frameworks: `UnicusSDK.xcframework` and the verification
engine framework. The `ForDevelopment` framework is never embedded directly (see
[Debug builds on a physical iPhone](#debug-builds-on-a-physical-iphone)).

{% hint style="info" %}
Hosted Swift Package Manager and CocoaPods repositories are **coming soon**.
Until then, integrate from the package folder as shown below. Moving to the
hosted repository will only change the dependency line.
{% endhint %}

## 1. Copy the SDK into your repository

{% code overflow="wrap" %}
```text
YourApp/
  Vendor/
    Unicus/            ← contents of the package's sdk/ folder
      Frameworks/
      UnicusSDK.podspec
      Package.swift
      swap_facetec_development_framework.sh
```
{% endcode %}

## 2. Add the dependency (choose one)

{% tabs %}
{% tab title="Xcode (manual)" %}
1. Select your app target → *General* → *Frameworks, Libraries, and Embedded
   Content*.
2. Add `Vendor/Unicus/Frameworks/UnicusSDK.xcframework` and the verification
   engine framework (the one **without** `ForDevelopment` in its name).
3. Set both to **Embed & Sign**.

Embedding the `ForDevelopment` framework as well breaks the build.
{% endtab %}

{% tab title="CocoaPods" %}
{% code overflow="wrap" %}
```ruby
platform :ios, '15.0'

target 'YourApp' do
  use_frameworks!
  pod 'UnicusSDK', :path => 'Vendor/Unicus'
end
```
{% endcode %}

Then run `pod install` and open the `.xcworkspace`.
{% endtab %}

{% tab title="Swift Package Manager" %}
1. *File* → *Add Package Dependencies…* → *Add Local…*
2. Choose the `Vendor/Unicus` folder (it contains `Package.swift`).
3. Link the `UnicusSDK` product to your app target.

The product brings both binary frameworks.
{% endtab %}
{% endtabs %}

## 3. Info.plist

| Key | Required | Purpose |
| --- | --- | --- |
| `NSCameraUsageDescription` | Yes | Face and document capture. Without it iOS terminates the app when the camera opens. |
| `NSLocationWhenInUseUsageDescription` | No | Location as transaction metadata. Without the key the SDK skips location and continues. |

{% code overflow="wrap" %}
```xml
<key>NSCameraUsageDescription</key>
<string>We use the camera to verify your identity.</string>
<key>NSLocationWhenInUseUsageDescription</key>
<string>We record where the verification starts.</string>
```
{% endcode %}

If your `Info.plist` declares `WKAppBoundDomains`, add the flow app domain of
your environment (DEV: `dev-id.idunicus.com`); otherwise `start` fails with
`webview_domain_not_allowed`. Apps without that key need nothing.

No changes are needed in `AppDelegate`, `SceneDelegate` or any other native file.

## Debug builds on a physical iPhone

The verification engine ships as a production binary and a development binary
(the framework whose name ends in `ForDevelopment`). **Debug builds installed on
a physical iPhone** must use the development binary. The script included in the
package swaps it in after the embed step; it does nothing for Release builds, the
simulator, or builds without code signing.

1. Select your app target → *Build Phases* → **+** → *New Run Script Phase*.
2. Move it **after** *Embed Frameworks* (with CocoaPods: after
   *[CP] Embed Pods Frameworks*).
3. Shell `/bin/bash`; uncheck *Based on dependency analysis*.
4. Script (use the exact file name of the `ForDevelopment` framework in your
   `Frameworks` folder):

{% code overflow="wrap" %}
```bash
bash "${SRCROOT}/Vendor/Unicus/swap_facetec_development_framework.sh" \
     "${SRCROOT}/Vendor/Unicus/Frameworks/<Unicus verification engine>ForDevelopment.xcframework"
```
{% endcode %}

{% hint style="warning" %}
If your project enables *User Script Sandboxing* (`ENABLE_USER_SCRIPT_SANDBOXING`),
the phase cannot modify the app bundle. Set it to `No` for the app target, as
the included example does.
{% endhint %}

Release, TestFlight and App Store builds keep the production binary.

## Run the included example

Before touching your app, run the example to validate the token, the device and
the permissions:

1. Open `example/UnicusSDKExample.xcodeproj`.
2. Fill `UNICUS_API_KEY` in `example/UnicusSDKExample/Config/Unicus.xcconfig`
   with your DEV Customer Token.
3. Select your team in *Signing & Capabilities* and run on a physical iPhone.

The *Inicio rápido* button runs the minimal integration (`QuickStart.swift`). The
rest of the example lets you try every option and shows the sanitized API logs.

## Minimum versions

| Item | Minimum |
| --- | --- |
| iOS deployment target | 15.0 |
| Xcode | 16 |
| Swift | 5.9 (Swift 6 language mode compiles; see [Custom UI steps](custom-ui-steps.md#threading) for provider code) |
| Architectures | arm64 devices; arm64 and x86\_64 simulators (build only) |

Nothing else is required: no extra permissions for the flow screens (they use
the system WebKit) and no keys to request. Code obfuscation or app shielding
tools: see [Compatibility and security](../compatibility-and-security.md).

## Next step

[Configuration](configuration.md).
