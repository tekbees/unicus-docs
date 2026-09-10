---
description: >-
  Integrate Unicus identity verification in native Flutter applications for
  Android and iOS without calling the native provider directly.
---

# FLUTTER INTEGRATION

The Unicus Flutter SDK lets your Flutter application run native identity
verification on Android and iOS. Your app integrates **Unicus** only. The SDK
creates the Unicus transaction, fetches the session configuration, applies the
company branding, opens the native verification screens, processes the
encrypted biometric data through Unicus, and returns the final result.

{% hint style="info" %}
Do not integrate the native verification provider directly in your Flutter app.
Do not add provider dependencies, do not edit provider native code, and do not
call provider APIs from your application. The provider is already wrapped inside
the Unicus Flutter SDK.
{% endhint %}

## What Unicus will provide

Before starting the integration, request the following values from your Unicus
administrator or Tekbees support team.

| Value | Description | Example |
| --- | --- | --- |
| `baseUrl` | Unicus API environment URL. | `https://alpha.idunicus.com:8080` |
| `apiKey` | Customer token generated for your company. | `<UNICUS_CUSTOMER_TOKEN>` |

The native provider device key and the Tekbees Unicus session device id are
embedded inside the Unicus Flutter SDK. The customer app must not request,
store, or pass those internal keys.

The customer app uses `apiKey` only for its Unicus Customer Token. The SDK uses
Tekbees internal keys for session and provider operations.

{% hint style="warning" %}
Use the values for the correct environment. Sandbox, staging, and production
credentials are different. Do not commit production credentials in public
repositories.
{% endhint %}

## Requirements

| Platform | Requirement |
| --- | --- |
| Flutter | `3.19.0` or newer |
| Dart | `3.3.0` or newer |
| Android | `minSdkVersion 21` or newer |
| iOS | iOS `15.0` or newer |
| Devices | Physical Android or iOS device with camera for full validation |

## 1. Add the dependency

Tekbees will provide the approved SDK delivery option for your project.

For closed external delivery, use the binary SDK package provided by Tekbees.
That package includes the public Flutter wrapper plus the Android and iOS native
binaries. Do not add the native provider dependency directly and do not request
Tekbees internal keys.

For internal Tekbees pilots or source-based private pilots, the SDK can also be
used from the private repository:

{% code overflow="wrap" %}
```yaml
dependencies:
  flutter:
    sdk: flutter

  unicus_sdk_flutter:
    git:
      url: git@bitbucket.org:tekbees/unicus_sdk_flutter.git
      ref: v0.1.0
```
{% endcode %}

Then install the dependency:

{% code overflow="wrap" %}
```bash
flutter pub get
```
{% endcode %}

If Tekbees provides a local pilot package instead of a Git dependency, use:

{% code overflow="wrap" %}
```yaml
dependencies:
  unicus_sdk_flutter:
    path: ../unicus_sdk_flutter
```
{% endcode %}

## 2. Configure Android

Open `android/app/src/main/AndroidManifest.xml` and add the required
permissions above the `<application>` tag.

{% code overflow="wrap" %}
```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <uses-permission android:name="android.permission.CAMERA" />
    <uses-permission android:name="android.permission.INTERNET" />

    <application>
        ...
    </application>
</manifest>
```
{% endcode %}

Confirm your app uses Android embedding v2. Most current Flutter apps already
include this metadata inside `<application>`:

{% code overflow="wrap" %}
```xml
<meta-data
    android:name="flutterEmbedding"
    android:value="2" />
```
{% endcode %}

No additional native provider dependency is required in Android.

## 3. Configure iOS

Open `ios/Runner/Info.plist` and add a camera usage description.

{% code overflow="wrap" %}
```xml
<key>NSCameraUsageDescription</key>
<string>Camera access is required to verify your identity.</string>
```
{% endcode %}

The Unicus Flutter SDK uses CocoaPods for iOS integration. If your Flutter
project has Swift Package Manager enabled globally, disable it in `pubspec.yaml`.

{% code overflow="wrap" %}
```yaml
flutter:
  config:
    enable-swift-package-manager: false
```
{% endcode %}

Then install pods:

{% code overflow="wrap" %}
```bash
cd ios
pod install
cd ..
```
{% endcode %}

No provider import is required in `AppDelegate`, `SceneDelegate`, or any iOS
native file.

## 4. Configure the SDK

Create one `UnicusSdkFlutter` instance in the part of your app that owns the
verification flow.

{% code overflow="wrap" %}
```dart
import 'package:unicus_sdk_flutter/unicus_sdk_flutter.dart';

final UnicusSdkFlutter unicus = UnicusSdkFlutter();

Future<void> configureUnicus() async {
  await unicus.configure(
    const UnicusSdkConfig(
      baseUrl: '<UNICUS_BASE_URL>',
      apiKey: '<UNICUS_CUSTOMER_TOKEN>',
    ),
  );
}
```
{% endcode %}

Call `configureUnicus()` before starting the first verification. A common place
is after the user reaches the screen where identity verification can begin.

## 5. Start a verification

Send the document type and the user's document number to `start`.

{% code overflow="wrap" %}
```dart
Future<void> startUnicusVerification() async {
  final UnicusVerificationResult result = await unicus.start(
    const UnicusVerificationRequest.enrollmentVerify(
      document: UnicusDocument(
        type: UnicusDocumentType.id,
        externalDatabaseRefId: '123456789',
      ),
    ),
  );

  if (result.success) {
    // The identity verification was successful.
  } else {
    // Show a retry, rejection, or support path according to your business flow.
  }
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
manually for the standard Flutter integration. It should also not ask the user
or the application developer to select a process value.

The standard sequence is:

1. Your app calls `unicus.start(...)`.
2. The SDK calls `/start-mobile-transaction` and receives a new transaction id
   `tid`.
3. The SDK calls `/get-restart-session` using that `tid`.
4. The SDK applies the company colors, logo, and Unicus verification text.
5. The SDK opens the native verification screen.
6. The SDK sends the encrypted verification data to Unicus.
7. Your app receives one `UnicusVerificationResult`.

## Document types

Use the enum provided by the SDK.

| Dart value | API value | Description |
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

For the standard Flutter integration, the application should only provide
document type and document id.

## Read the result

`start` returns a `UnicusVerificationResult`.

| Field | Description |
| --- | --- |
| `success` | `true` when the verification completed successfully. |
| `outcome` | Normalized result category: success, warning, failed, canceled, error, or unknown. |
| `tid` | Unicus transaction id created by the SDK. |
| `resultCode` | Unicus result code, when available. |
| `resultMessage` | Human-readable result message, when available. |
| `status` | Native session status. |

Recommended result handling:

{% code overflow="wrap" %}
```dart
switch (result.outcome) {
  case UnicusVerificationOutcome.success:
    // Continue with the verified user.
    break;
  case UnicusVerificationOutcome.warning:
    // Continue or route to manual review according to your business rules.
    break;
  case UnicusVerificationOutcome.canceled:
    // Allow the user to retry.
    break;
  case UnicusVerificationOutcome.failed:
  case UnicusVerificationOutcome.error:
    // Show the configured failure flow.
    break;
  case UnicusVerificationOutcome.unknown:
    // Show a support or retry path.
    break;
}
```
{% endcode %}

## Listen to progress events

You can subscribe to SDK events before starting verification.

{% code overflow="wrap" %}
```dart
final subscription = unicus.events.listen((event) {
  debugPrint('Unicus event: ${event.name}');
  debugPrint('Transaction id: ${event.tid}');
  debugPrint('Message: ${event.message}');
});
```
{% endcode %}

Cancel the subscription when the screen is disposed:

{% code overflow="wrap" %}
```dart
await subscription.cancel();
```
{% endcode %}

## Customize verification text

The SDK includes default English text based on the current Unicus web SDK
configuration. If your application needs different language or wording, provide
text overrides when configuring Unicus.

The example app includes complete English and Spanish Unicus text maps based on
the current web SDK language configuration in
`example/lib/sample_unicus_texts.dart`. Use that file as the starting point for
your own language file. The example start screen intentionally asks only for
document id and document type; text is changed in code, not through the demo UI.

{% code overflow="wrap" %}
```dart
await unicus.configure(
  const UnicusSdkConfig(
    baseUrl: '<UNICUS_BASE_URL>',
    apiKey: '<UNICUS_CUSTOMER_TOKEN>',
    verificationTextOverrides: sampleUnicusTextOverrides,
  ),
);
```
{% endcode %}

Unicus merges your overrides with the default text and sends the final text map
to the native Android and iOS verification screens before the session opens.
Provider-specific native text keys are adapted internally by the SDK, and
`<br/>` line breaks are converted to native line breaks.

Common text keys:

| Dart key | Screen text |
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
```dart
await unicus.configure(
  const UnicusSdkConfig(
    baseUrl: '<UNICUS_BASE_URL>',
    apiKey: '<UNICUS_CUSTOMER_TOKEN>',
    enableApiLogging: true,
  ),
);

unicus.apiLogs.listen((entry) {
  debugPrint(entry.toPrettyJson());
});
```
{% endcode %}

By default, logs are sanitized. Encrypted biometric blobs, OCR payloads,
document data, and session tokens are redacted or truncated.

## Complete button example

{% code overflow="wrap" expandable="true" %}
```dart
import 'package:flutter/material.dart';
import 'package:unicus_sdk_flutter/unicus_sdk_flutter.dart';

class UnicusVerificationButton extends StatefulWidget {
  const UnicusVerificationButton({super.key});

  @override
  State<UnicusVerificationButton> createState() =>
      _UnicusVerificationButtonState();
}

class _UnicusVerificationButtonState extends State<UnicusVerificationButton> {
  final UnicusSdkFlutter unicus = UnicusSdkFlutter();
  bool running = false;

  @override
  void initState() {
    super.initState();
    configureSdk();
  }

  Future<void> configureSdk() async {
    await unicus.configure(
      const UnicusSdkConfig(
        baseUrl: '<UNICUS_BASE_URL>',
        apiKey: '<UNICUS_CUSTOMER_TOKEN>',
      ),
    );
  }

  Future<void> startVerification() async {
    setState(() => running = true);

    try {
      final result = await unicus.start(
        const UnicusVerificationRequest.enrollmentVerify(
          document: UnicusDocument(
            type: UnicusDocumentType.id,
            externalDatabaseRefId: '123456789',
          ),
        ),
      );

      if (!mounted) {
        return;
      }

      final message = result.success
          ? 'Identity verified successfully'
          : result.resultMessage ?? 'Identity could not be verified';

      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text(message)),
      );
    } finally {
      if (mounted) {
        setState(() => running = false);
      }
    }
  }

  @override
  Widget build(BuildContext context) {
    return FilledButton(
      onPressed: running ? null : startVerification,
      child: Text(running ? 'Verifying...' : 'Verify identity'),
    );
  }
}
```
{% endcode %}

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

iOS can use the remote logo URL returned by Unicus. Android requires the logo to
be a native drawable resource, so Android applies colors automatically. If your
Android integration requires a logo inside the native verification screen,
coordinate the drawable resource name with Tekbees support.

## Testing checklist

Use a physical device for the final validation.

1. Run `flutter pub get`.
2. Run the app on Android and accept camera permission.
3. Run the app on a signed physical iPhone and accept camera permission.
4. Test one valid document with document type `ID`, `FD`, `PP`, or `DL`.
5. Confirm the native verification screen opens.
6. Confirm the company colors and logo are displayed as expected.
7. Confirm your app receives a `UnicusVerificationResult`.
8. Confirm the transaction appears in the Unicus administrative portal.

Useful commands:

{% code overflow="wrap" %}
```bash
flutter devices
flutter run -d <DEVICE_ID>
```
{% endcode %}

For iOS physical devices, do not use `--no-codesign`. Use a signed build from
Flutter or Xcode.

## Troubleshooting

| Problem | What to check |
| --- | --- |
| The SDK says it is not configured | Confirm `unicus.configure(...)` runs before `unicus.start(...)`. |
| Camera permission is denied | Confirm Android `CAMERA` permission or iOS `NSCameraUsageDescription`. |
| No transaction id is returned | Confirm `baseUrl`, `apiKey`, document type, and document number. |
| iOS build fails after adding the package | Confirm CocoaPods is installed and Swift Package Manager is disabled for this app. |
| Native verification does not open | Confirm you are running on a supported physical device with camera access. |
| Company colors do not appear | Confirm `/get-restart-session` returns `windowColor`, `buttonColor`, and `textColor`. |
| Android logo does not appear | Confirm whether a drawable resource name was coordinated with Tekbees support. |

## Support

When contacting support, include:

1. Environment URL.
2. App platform and version.
3. Device model and OS version.
4. Unicus transaction id `tid`, if it was created.
5. Result code and result message, if available.
6. A short description of the step where the issue happened.
