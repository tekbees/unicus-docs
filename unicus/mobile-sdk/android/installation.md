---
description: >-
  Install the Unicus Android SDK: package contents, Gradle setup, permissions,
  minimum versions and R8 / ProGuard.
---

# Installation

## Package contents

Tekbees delivers the SDK as a ZIP package:

{% code overflow="wrap" %}
```text
unicus_sdk_android_<version>_customer_package/
  README.md                    Quick start of the package.
  TECHNICAL_INTEGRATION.md     Technical reference.
  sdk/repo/                    Local Maven repository:
    com/tekbees/unicus/unicus-sdk/<version>/     the Unicus SDK (AAR)
    <biometric engine>/                          its embedded biometric engine
  example/                     Android Studio project that uses the SDK binary.
```
{% endcode %}

The repository holds everything the SDK needs. Your app declares **one**
dependency, `com.tekbees.unicus:unicus-sdk`; the biometric engine and the other
libraries arrive transitively.

{% hint style="info" %}
A hosted Maven repository is coming soon. When it is published you will only
replace the local `maven { }` entry with the URL from Tekbees; the dependency
line does not change.
{% endhint %}

## Gradle setup

1. Copy `sdk/repo` into your project, for example `vendor/unicus/repo`, and
   commit it (or store it in your internal artifact repository).
2. Register the repository in `settings.gradle.kts`:

   {% code overflow="wrap" %}
   ```kotlin
   dependencyResolutionManagement {
       repositories {
           google()
           mavenCentral()
           maven { url = uri(rootDir.resolve("vendor/unicus/repo")) }
       }
   }
   ```
   {% endcode %}

   With Groovy (`settings.gradle`):

   {% code overflow="wrap" %}
   ```groovy
   dependencyResolutionManagement {
       repositories {
           google()
           mavenCentral()
           maven { url = uri("${rootDir}/vendor/unicus/repo") }
       }
   }
   ```
   {% endcode %}

3. Add the dependency to the app module:

   {% code overflow="wrap" %}
   ```kotlin
   dependencies {
       implementation("com.tekbees.unicus:unicus-sdk:0.1.0")
   }
   ```
   {% endcode %}

4. Sync Gradle and build. To check the setup, run the `example/` project of the
   package on a device (`cp app/unicus.properties.example app/unicus.properties`,
   fill in your Customer Token, then `./gradlew :app:installDebug`).

The SDK brings `kotlinx-coroutines-android`, `appcompat`, `core-ktx`,
`androidx.webkit` and `androidx.browser`. If your app pins older versions of
these libraries, Gradle resolves to the newest one.

## Minimum versions

| Item | Minimum |
| --- | --- |
| `minSdk` | 21 |
| `compileSdk` | 34 |
| Android Gradle Plugin | 8.x (9.x supported) |
| Kotlin Gradle plugin | 2.0, or no Kotlin at all (Java-only apps are supported) |
| Java toolchain | 17 |
| Android System WebView | Chromium 90, for flows with screens in WebView mode |

## Permissions

| Permission | Who declares it | Notes |
| --- | --- | --- |
| `CAMERA` | The SDK manifest (merged automatically). | Requested from the user when the camera opens. You do not handle the result. |
| `INTERNET` | The SDK manifest. | |
| `ACCESS_FINE_LOCATION` / `ACCESS_COARSE_LOCATION` | **Your app**, only if you want location. | Optional. The SDK does not declare them. |

Location is optional transaction metadata. When your app declares one of the
location permissions and `collectLocationOnStart` is `true` (default), the SDK
asks the user and sends the location with the transaction. When the user
denies it, the app does not declare it, or the device has no location, the
verification continues without it. If you declare location, report it in the
Play Console *Data safety* form (approximate / precise location, collected,
not shared).

{% code overflow="wrap" %}
```xml
<!-- AndroidManifest.xml of your app, only if you want location -->
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
```
{% endcode %}

The SDK registers its own activities (camera host, flow screens, permission
prompt). You do not add anything else to your manifest and you do not override
`onActivityResult` or `onRequestPermissionsResult`.

## R8 / ProGuard

No rules are needed. The AAR ships its consumer rules: they keep the public API,
the SDK activities and the biometric engine. Minify your release build as
usual.

If your app uses RASP or app-shielding tools that require an allowlist, see
[Compatibility and security](../compatibility-and-security.md); the activities
to allow are `com.tekbees.unicus.sdk.internal.UnicusVerificationActivity` and
`com.tekbees.unicus.sdk.internal.UnicusFlowWebActivity`.

## Updating the SDK

Replace the `vendor/unicus/repo` folder with the one from the new package and
change the version in the dependency line. Read the
[Versioning](../versioning.md) page for the changes between versions.

Next: [Configuration](configuration.md).
