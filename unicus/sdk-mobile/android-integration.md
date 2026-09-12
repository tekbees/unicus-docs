---
description: >-
  Integrate Unicus identity verification in native Android applications
  (Kotlin or Java) using only the Unicus SDK.
---

# ANDROID INTEGRATION

The Unicus Android SDK lets your native Android application run identity
verification. Your app integrates **Unicus** only. The SDK creates the Unicus
transaction, fetches the session configuration, applies the company branding,
opens the native verification screens, processes the encrypted biometric data
through Unicus, and returns the final result.

{% hint style="info" %}
The Unicus Android SDK includes everything required for identity verification:
camera capture, liveness, document scanning, encryption, and the Unicus API
calls. Do not add other biometric or document-capture libraries for this flow,
and do not forward activity results or permission results: the Unicus SDK
handles them internally.
{% endhint %}

## What Unicus will provide

Before starting the integration, request the following values from your Unicus
administrator or Tekbees support team.

| Value | Description | Example |
| --- | --- | --- |
| `baseUrl` | Unicus API environment URL. | `https://alpha.idunicus.com:8080` |
| `apiKey` | Customer token generated for your company. | `<UNICUS_CUSTOMER_TOKEN>` |

The Unicus internal keys (the Tekbees session device id and the Unicus
verification engine key) are embedded inside the Unicus Android SDK. The customer
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
| Android | `minSdk 21` or newer, `compileSdk 36` recommended |
| Build tools | Android Gradle Plugin `8.x` or newer, Gradle `8.x` or newer, Java `17` |
| Language | Kotlin `2.x` (a Java-friendly callback API is also provided) |
| Libraries | AndroidX. The SDK brings `appcompat`, `core-ktx`, and `kotlinx-coroutines-android` transitively. |
| Devices | Physical Android device with camera for full validation |
| Permissions | Camera is required. Location is optional and the SDK continues if the user denies it. |

## 1. Review the customer package

Tekbees provides a customer package named like this:

{% code overflow="wrap" %}
```text
unicus_sdk_android_0.1.0_customer_package.zip
```
{% endcode %}

When unzipped, it contains:

{% code overflow="wrap" %}
```text
unicus_sdk_android_0.1.0_customer_package/
  README.md
  TECHNICAL_INTEGRATION.md
  sdk/
    repo/
      com/tekbees/unicus/unicus-sdk/0.1.0/
      ...
  example/
    settings.gradle.kts
    app/
      build.gradle.kts
      unicus.properties.example
      src/main/kotlin/com/tekbees/unicus/example/
        MainActivity.kt
        UnicusExampleScreen.kt
        SampleUnicusTexts.kt
```
{% endcode %}

| Path | Purpose |
| --- | --- |
| `README.md` | Quick start for the package. |
| `TECHNICAL_INTEGRATION.md` | Technical reference for the customer's development team. |
| `sdk/repo/` | Local Maven repository with the closed Unicus SDK (`com.tekbees.unicus:unicus-sdk`) and its internal dependencies. |
| `example/` | Runnable Android Studio project (Jetpack Compose) already configured to use the included SDK. |
| `example/app/src/main/kotlin/.../SampleUnicusTexts.kt` | Editable language/text maps using public `Unicus_` keys. |

The package contains the closed Unicus SDK with all its internal components,
resources, and a runnable example. Customer applications integrate only
Unicus.

## 2. Run the included example

Before changing your own app, run the included example to confirm the
environment, device, permissions, and customer token.

{% code overflow="wrap" %}
```bash
cd unicus_sdk_android_0.1.0_customer_package/example
cp app/unicus.properties.example app/unicus.properties
```
{% endcode %}

Edit `app/unicus.properties` with the values provided by Tekbees:

{% code overflow="wrap" %}
```properties
UNICUS_BASE_URL=<UNICUS_BASE_URL>
UNICUS_API_KEY=<UNICUS_CUSTOMER_TOKEN>
```
{% endcode %}

Then open the `example` folder in Android Studio and run the `app`
configuration on a physical device, or use the command line:

{% code overflow="wrap" %}
```bash
./gradlew :app:installDebug
```
{% endcode %}

The example screen asks only for:

1. Document id.
2. Document type: `ID`, `FD`, `PP`, or `DL`.

The example may also show the Android location permission prompt. If the user
does not share location, the transaction continues without location data. If
your Unicus account has more than one active country, the example shows a
country selector after the transaction id is created. If there is only one
active country, the SDK selects it automatically and continues.

The SDK creates the transaction, reads the customer session configuration,
applies the theme and text, opens the native verification experience, processes
the encrypted requests, and returns the result. The example also shows the
sanitized Unicus API responses received during the flow.

For a compile-only check:

{% code overflow="wrap" %}
```bash
./gradlew :app:assembleDebug
```
{% endcode %}

Use a physical Android device with camera for full end-to-end verification.

## 3. Add the dependency

For your own app, copy `sdk/repo` from the customer package into your
application repository, for example:

{% code overflow="wrap" %}
```text
your_android_app/
  vendor/
    unicus/
      repo/
```
{% endcode %}

Register the local Maven repository in `settings.gradle.kts`:

{% code overflow="wrap" %}
```kotlin
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
        maven {
            url = uri(rootDir.resolve("vendor/unicus/repo"))
        }
    }
}
```
{% endcode %}

Then add the dependency to your application module (`app/build.gradle.kts`):

{% code overflow="wrap" %}
```kotlin
dependencies {
    implementation("com.tekbees.unicus:unicus-sdk:0.1.0")
}
```
{% endcode %}

The Unicus SDK requires Java 17 compatibility in the application module:

{% code overflow="wrap" %}
```kotlin
android {
    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }
}
```
{% endcode %}

Sync the project. The internal dependencies of the Unicus SDK are resolved
automatically from the same local repository; do not declare them yourself and
do not request Tekbees internal keys.

## 4. Configure Android

The Unicus SDK manifest already declares the required permissions and the
internal activities it needs. They are merged automatically into your app:

{% code overflow="wrap" %}
```xml
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
```
{% endcode %}

If your application must not request location for this flow, remove the
location permissions in your `AndroidManifest.xml` and the SDK continues
without location data:

{% code overflow="wrap" %}
```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" tools:node="remove" />
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" tools:node="remove" />
</manifest>
```
{% endcode %}

No `Activity` changes are required. The SDK opens the verification from its
own transparent host activity, so your app does not override
`onActivityResult` or `onRequestPermissionsResult`.

If your release build uses R8/ProGuard, no extra rules are needed: the SDK
ships consumer rules that keep the Unicus public API and its internal
components.

## 5. Configure the SDK

Configure the shared `UnicusSdk` instance in the part of your app that owns the
verification flow.

{% code overflow="wrap" %}
```kotlin
import com.tekbees.unicus.sdk.UnicusSdk
import com.tekbees.unicus.sdk.UnicusSdkConfig

fun configureUnicus() {
    UnicusSdk.shared.configure(
        UnicusSdkConfig(
            baseUrl = "<UNICUS_BASE_URL>",
            apiKey = "<UNICUS_CUSTOMER_TOKEN>",
            collectLocationOnStart = true
        )
    )
}
```
{% endcode %}

Java:

{% code overflow="wrap" %}
```java
UnicusSdk.getShared().configure(
    new UnicusSdkConfig.Builder("<UNICUS_BASE_URL>", "<UNICUS_CUSTOMER_TOKEN>")
        .collectLocationOnStart(true)
        .build()
);
```
{% endcode %}

Call `configureUnicus()` before starting the first verification. A common place
is after the user reaches the screen where identity verification can begin.
`collectLocationOnStart` is optional and defaults to `true`. Set it to `false`
only when your app must not request location for this flow.

## 6. Start a verification

Send the document type and the user's document number to `start`, together with
the `Activity` that is currently on screen. All callbacks are delivered on the
main thread.

{% code overflow="wrap" %}
```kotlin
import com.tekbees.unicus.sdk.UnicusCallback
import com.tekbees.unicus.sdk.model.UnicusDocument
import com.tekbees.unicus.sdk.model.UnicusDocumentType
import com.tekbees.unicus.sdk.model.UnicusSdkException
import com.tekbees.unicus.sdk.model.UnicusVerificationRequest
import com.tekbees.unicus.sdk.model.UnicusVerificationResult

fun startUnicusVerification(activity: Activity) {
    UnicusSdk.shared.start(
        activity,
        UnicusVerificationRequest.enrollmentVerify(
            document = UnicusDocument(
                type = UnicusDocumentType.ID,
                externalDatabaseRefId = "123456789"
            )
        ),
        object : UnicusCallback<UnicusVerificationResult> {
            override fun onSuccess(value: UnicusVerificationResult) {
                if (value.success) {
                    // The identity verification was successful.
                } else {
                    // Show a retry, rejection, or support path according to your business flow.
                }
            }

            override fun onError(error: UnicusSdkException) {
                // The verification could not start (configuration, network, or session error).
            }
        }
    )
}
```
{% endcode %}

With Kotlin coroutines, every operation has a `suspend` variant that throws
`UnicusSdkException` on failure:

{% code overflow="wrap" %}
```kotlin
import com.tekbees.unicus.sdk.start

lifecycleScope.launch {
    try {
        val result = UnicusSdk.shared.start(activity, request)
        // Handle UnicusVerificationResult here.
    } catch (error: UnicusSdkException) {
        // error.code and error.message describe the failure.
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
manually for the standard Android integration. It should also not ask the user
or the application developer to select a process value.

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
7. The SDK opens the native verification screen.
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
```kotlin
import com.tekbees.unicus.sdk.prepareEnrollmentVerify
import com.tekbees.unicus.sdk.start

suspend fun startWithCountrySelection(activity: Activity) {
    val prepared = UnicusSdk.shared.prepareEnrollmentVerify(
        activity = activity,
        document = UnicusDocument(
            type = UnicusDocumentType.ID,
            externalDatabaseRefId = "123456789"
        )
    )

    val selectedCountry: String? = if (prepared.requiresCountrySelection) {
        showYourCountryPicker(prepared.countries)
    } else {
        prepared.defaultCountry?.code
    }

    val result = UnicusSdk.shared.start(
        activity,
        prepared.toRequest(country = selectedCountry)
    )

    // Handle UnicusVerificationResult here.
}
```
{% endcode %}

`prepareEnrollmentVerify(...)` creates the Unicus transaction id, optionally
collects location, and returns the active countries. It does not open the native
verification screen. Call `start(...)` with `prepared.toRequest(...)` after your
app has selected the country. The country value should be an ISO 3166-1 alpha-2
code such as `CO`, `US`, or `MX`.

## Document types

Use the enum provided by the SDK.

| Kotlin value | API value | Description |
| --- | --- | --- |
| `UnicusDocumentType.ID` | `ID` | National ID document |
| `UnicusDocumentType.FOREIGN_DOCUMENT` | `FD` | Foreign document |
| `UnicusDocumentType.PASSPORT` | `PP` | Passport |
| `UnicusDocumentType.DRIVER_LICENSE` | `DL` | Driver license |

## Native flow behavior

The default native flow is handled internally by the SDK. No flow field is
required in the customer application.

Unicus checks the current session state:

| Session state | Behavior |
| --- | --- |
| User is already enrolled | The SDK starts face authentication. |
| User is not enrolled | The SDK starts face and document enrollment. |

For the standard Android integration, the application should only provide
document type and document id.

## Read the result

`start` delivers a `UnicusVerificationResult`.

| Field | Description |
| --- | --- |
| `success` | `true` when the verification completed successfully. |
| `outcome` | Normalized result category: `SUCCESS`, `WARNING`, `FAILED`, `CANCELED`, `ERROR`, or `UNKNOWN`. |
| `tid` | Unicus transaction id created by the SDK. |
| `resultCode` | Unicus transaction result code, when available. If Unicus has finalized the transaction, this value takes precedence over the native screen status. |
| `resultMessage` | Human-readable result message, when available. |
| `status` | Native screen status for technical diagnostics. |
| `sessionError` | `true` only for a technical interruption, cancellation, or native/session error. Business outcomes should be handled through `outcome` and `resultCode`. |

Recommended result handling:

{% code overflow="wrap" %}
```kotlin
when (result.outcome) {
    UnicusVerificationOutcome.SUCCESS -> {
        // Continue with the verified user.
    }
    UnicusVerificationOutcome.WARNING -> {
        // Continue or route to manual review according to your business rules.
    }
    UnicusVerificationOutcome.CANCELED -> {
        // Allow the user to retry.
    }
    UnicusVerificationOutcome.FAILED,
    UnicusVerificationOutcome.ERROR -> {
        // Show the configured failure flow.
    }
    UnicusVerificationOutcome.UNKNOWN -> {
        // Show a support or retry path.
    }
}
```
{% endcode %}

`UnicusResultCode.messageFor(result.resultCode)` returns the standard Unicus
message for a result code when your app needs a default text.

## Listen to progress events

You can register an event listener before starting verification. Events are
delivered on the main thread.

{% code overflow="wrap" %}
```kotlin
UnicusSdk.shared.setEventListener { event ->
    Log.d("Unicus", "Unicus event: ${event.name}")
    Log.d("Unicus", "Transaction id: ${event.tid}")
    Log.d("Unicus", "Message: ${event.message}")
}
```
{% endcode %}

Remove the listener when the screen is destroyed:

{% code overflow="wrap" %}
```kotlin
UnicusSdk.shared.setEventListener(null)
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
`example/app/src/main/kotlin/com/tekbees/unicus/example/SampleUnicusTexts.kt`.
Use that file as the starting point for your own language file. The example
start screen intentionally asks only for document id and document type; text is
changed in code, not through the demo UI.

{% code overflow="wrap" %}
```kotlin
UnicusSdk.shared.configure(
    UnicusSdkConfig(
        baseUrl = "<UNICUS_BASE_URL>",
        apiKey = "<UNICUS_CUSTOMER_TOKEN>",
        verificationTextOverrides = SampleUnicusTexts.overrides
    )
)
```
{% endcode %}

Unicus merges your overrides with the default text and applies the final text
map to the native verification screens before the session opens.
Internal native text keys are adapted by the SDK, and `<br/>` line breaks are
converted to native line breaks.

Common text keys:

| Kotlin key | Screen text |
| --- | --- |
| `UnicusVerificationTextKey.ACTION_IM_READY` | Ready button. |
| `UnicusVerificationTextKey.ACTION_CONTINUE` | Continue button. |
| `UnicusVerificationTextKey.ACTION_TRY_AGAIN` | Retry button. |
| `UnicusVerificationTextKey.FEEDBACK_CENTER_FACE` | Face alignment feedback. |
| `UnicusVerificationTextKey.INITIALIZING_CAMERA` | Camera initialization message. |
| `UnicusVerificationTextKey.ID_SCAN_TYPE_SELECTION_HEADER` | Document scan title. |
| `UnicusVerificationTextKey.RESULT_FACE_SCAN_UPLOAD_MESSAGE` | Face upload message. |

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
```kotlin
UnicusSdk.shared.configure(
    UnicusSdkConfig(
        baseUrl = "<UNICUS_BASE_URL>",
        apiKey = "<UNICUS_CUSTOMER_TOKEN>",
        enableApiLogging = true
    )
)

UnicusSdk.shared.setApiLogListener { entry ->
    Log.d("Unicus API", entry.toPrettyJson())
}
```
{% endcode %}

By default, logs are sanitized. Encrypted biometric blobs, OCR payloads,
document data, and session tokens are redacted or truncated.

## Complete button example

{% code overflow="wrap" expandable="true" %}
```kotlin
import android.os.Bundle
import android.widget.Button
import android.widget.Toast
import androidx.appcompat.app.AppCompatActivity
import androidx.lifecycle.lifecycleScope
import com.tekbees.unicus.sdk.UnicusSdk
import com.tekbees.unicus.sdk.UnicusSdkConfig
import com.tekbees.unicus.sdk.model.UnicusDocument
import com.tekbees.unicus.sdk.model.UnicusDocumentType
import com.tekbees.unicus.sdk.model.UnicusSdkException
import com.tekbees.unicus.sdk.model.UnicusVerificationRequest
import com.tekbees.unicus.sdk.start
import kotlinx.coroutines.launch

class VerificationActivity : AppCompatActivity() {

    private lateinit var verifyButton: Button

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_verification)
        verifyButton = findViewById(R.id.verify_button)

        UnicusSdk.shared.configure(
            UnicusSdkConfig(
                baseUrl = "<UNICUS_BASE_URL>",
                apiKey = "<UNICUS_CUSTOMER_TOKEN>"
            )
        )

        verifyButton.setOnClickListener { startVerification() }
    }

    private fun startVerification() {
        verifyButton.isEnabled = false
        verifyButton.text = "Verifying..."

        lifecycleScope.launch {
            try {
                val result = UnicusSdk.shared.start(
                    this@VerificationActivity,
                    UnicusVerificationRequest.enrollmentVerify(
                        document = UnicusDocument(
                            type = UnicusDocumentType.ID,
                            externalDatabaseRefId = "123456789"
                        )
                    )
                )
                val message = if (result.success) {
                    "Identity verified successfully"
                } else {
                    result.resultMessage ?: "Identity could not be verified"
                }
                Toast.makeText(this@VerificationActivity, message, Toast.LENGTH_LONG).show()
            } catch (error: UnicusSdkException) {
                Toast.makeText(this@VerificationActivity, error.message, Toast.LENGTH_LONG).show()
            } finally {
                verifyButton.isEnabled = true
                verifyButton.text = "Verify identity"
            }
        }
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

Android requires the logo to be a native drawable resource, so Android applies
the colors automatically and shows the Unicus logo by default. To show your own
logo inside the native verification screen, add a drawable to your app and
register its name before starting:

{% code overflow="wrap" %}
```kotlin
UnicusSdk.shared.setAndroidLogoResourceName("my_company_logo")
```
{% endcode %}

Use a PNG with a transparent background, about 720x266 pixels, placed in
`res/drawable-nodpi/`.

## Testing checklist

Use a physical device for the final validation.

1. Sync the project in Android Studio.
2. Run the app on a physical Android device and accept camera permission.
3. Test one valid document with document type `ID`, `FD`, `PP`, or `DL`.
4. Test location permission accepted and denied. Both paths should continue.
5. If the account has one active country, confirm the flow continues without
   showing a country selector.
6. If the account has several active countries, confirm your app shows a
   country selector before opening the native verification screen.
7. Confirm the native verification screen opens.
8. Confirm the company colors and logo are displayed as expected.
9. Confirm your app receives a `UnicusVerificationResult`.
10. Confirm the transaction appears in the Unicus administrative portal.

Useful commands:

{% code overflow="wrap" %}
```bash
./gradlew :app:assembleDebug
./gradlew :app:installDebug
adb logcat -s UnicusSDK
```
{% endcode %}

## Troubleshooting

| Problem | What to check |
| --- | --- |
| Error code `not_configured` | Confirm `UnicusSdk.shared.configure(...)` runs before `start(...)`. |
| Error code `session_active` | A previous verification is still running. Wait for its callback before starting again. |
| Gradle cannot resolve `com.tekbees.unicus:unicus-sdk` | Confirm the `maven { url = uri(...) }` entry points to the copied `repo` folder and that `sdk/repo` was copied completely. |
| Camera permission is denied | The native screen shows its own camera permission guidance. Confirm the `CAMERA` permission was not removed from the merged manifest. |
| Location permission is denied | This does not stop the transaction. The SDK continues without location data. |
| Country selector does not appear | Confirm the customer account has more than one active country in Unicus. For one active country, the SDK selects it automatically. |
| No transaction id is returned | Confirm `baseUrl`, `apiKey`, document type, and document number. |
| Native verification does not open | Confirm you are running on a supported physical device with camera access and that the `Activity` passed to `start` is on screen. |
| Company colors do not appear | Confirm `/get-restart-session` returns `windowColor`, `buttonColor`, and `textColor`. |
| Company logo does not appear | Android only shows drawable resources. Register one with `setAndroidLogoResourceName(...)`. |

## Support

When contacting support, include:

1. Environment URL.
2. App platform and version.
3. Device model and OS version.
4. Unicus transaction id `tid`, if it was created.
5. Result code and result message, if available.
6. A short description of the step where the issue happened.
