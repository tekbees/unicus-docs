---
description: >-
  The language option of the Unicus Android SDK (Spanish, English or the device
  language), what it covers, and how to change some texts with the Unicus_*
  keys.
---

# Texts and languages

The SDK ships every text in **Spanish and English**. One option,
`UnicusSdkConfig.language`, picks the language of everything the SDK shows:

| `UnicusLanguage` | Language |
| --- | --- |
| `AUTO` (default) | The device language: Spanish on a Spanish device, English on any other. |
| `SPANISH` | Always Spanish. |
| `ENGLISH` | Always English. |

{% tabs %}
{% tab title="Kotlin" %}
{% code overflow="wrap" %}
```kotlin
UnicusSdk.shared.configure(
    UnicusSdkConfig(apiKey = "<CUSTOMER_TOKEN>", environment = UnicusEnvironment.DEV)
        .copy(language = UnicusLanguage.SPANISH)
)
```
{% endcode %}
{% endtab %}

{% tab title="Java" %}
{% code overflow="wrap" %}
```java
UnicusSdk.getShared().configure(
        new UnicusSdkConfig.Builder("<CUSTOMER_TOKEN>", UnicusEnvironment.DEV)
                .language(UnicusLanguage.SPANISH)
                .build());
```
{% endcode %}
{% endtab %}
{% endtabs %}

The option applies to every screen of the verification:

| Screens | Wording comes from | Language |
| --- | --- | --- |
| **Camera screens** (liveness, document capture, retries, camera permission, upload progress) | The SDK's built-in texts, replaced key by key by your `verificationTextOverrides`. | `language`. Your overrides are shown as you pass them. |
| **Document data confirmation** (labels of the data read from the document) | The SDK's built-in labels, or your `verificationOcrLocalization`. | `language`. |
| **Flow screens** (consent, info, form, signature, OTP, document signing) in WebView mode | The flow configured in the administrative portal (titles, descriptions, consent, agreement, form labels in Spanish and English) plus the Unicus flow app. | `language`. |
| **SDK native elements** (top bar "Close", exit dialog, load error) | Android resources of the SDK. | `language`. |
| **`result.resultMessage`** | The Unicus result catalog. | `language`. |

The exit dialog asks "¿Salir de la validación?" / "Leave the verification?"
and tells the user that their progress is saved and they can resume the
verification later (see [Results and resuming](../results-and-resuming.md)).

With [Custom UI steps](custom-ui-steps.md) your app renders the flow screens:
use `context.language` (`es` or `en`, resolved from the same option) and the
localized texts of the step (`step.title`, `step.config`).

{% hint style="info" %}
Before this version the camera screens were in English unless your app passed
a Spanish table, and `resultMessage` was always in Spanish. With the default
`AUTO`, a Spanish device now gets Spanish camera screens and an English device
gets `resultMessage` in English. If your app passed the complete Spanish table
only to translate the screens, replace it with
`language = UnicusLanguage.SPANISH`.
{% endhint %}

## Change some camera texts

To change the wording of some texts, pass a map of `Unicus_*` keys in
`verificationTextOverrides`. Your texts win over the built-in ones; keys you do
not pass keep the built-in text in the selected language. The constants are in
`UnicusVerificationTextKey`.

{% code overflow="wrap" %}
```kotlin
UnicusSdk.shared.configure(
    UnicusSdkConfig(apiKey = token, environment = UnicusEnvironment.DEV)
        .copy(
            language = UnicusLanguage.SPANISH,
            verificationTextOverrides = mapOf(
                UnicusVerificationTextKey.ACTION_IM_READY to "COMENZAR",
                UnicusVerificationTextKey.INSTRUCTIONS_HEADER_READY_1 to "Prepárate para tu<br/>video selfie",
                "Unicus_idscan_capture_id_front_instruction_message" to "Muestra el frente de tu documento"
            )
        )
)
```
{% endcode %}

* An override has a single text, shown whatever the language. With `AUTO`,
  either fix the language or build the map for the language of the device
  (for example from your own resources with `getString(R.string.…)`).
* HTML line breaks (`<br/>`) become line breaks; other HTML tags are removed.
* The `example/` project of the package ships the complete English and Spanish
  tables (`SampleUnicusTexts.kt`) as a reference for custom wording.
* `verificationOcrLocalization` replaces the labels of the document data
  confirmation screen. Ask Tekbees for the format if you need to change them.

## Main key groups

The SDK accepts about 175 keys. These groups cover what most users see:

| Group | Prefix | Examples |
| --- | --- | --- |
| Buttons | `Unicus_action_` | `Unicus_action_im_ready`, `Unicus_action_try_again`, `Unicus_action_take_photo`, `Unicus_action_accept_photo`, `Unicus_action_retake_photo`, `Unicus_action_confirm`, `Unicus_action_continue`, `Unicus_action_ok` |
| Instructions before the selfie | `Unicus_instructions_` | `Unicus_instructions_header_ready_1`, `Unicus_instructions_message_ready_1` |
| Guidance during the selfie | `Unicus_feedback_`, `Unicus_presession_` | `Unicus_feedback_center_face`, `Unicus_feedback_move_phone_closer`, `Unicus_feedback_hold_steady`, `Unicus_presession_frame_your_face`, `Unicus_presession_remove_dark_glasses` |
| Document capture | `Unicus_idscan_` | `Unicus_idscan_type_selection_header`, `Unicus_idscan_capture_id_front_instruction_message`, `Unicus_idscan_capture_id_back_instruction_message`, `Unicus_idscan_review_id_front_instruction_message` |
| Results and uploads | `Unicus_result_` | `Unicus_result_facescan_upload_message`, `Unicus_result_idscan_unsuccess_message`, `Unicus_result_idscan_success_front_side_message` |
| Retry screen | `Unicus_retry_` | `Unicus_retry_header`, `Unicus_retry_subheader_message`, `Unicus_retry_instruction_message_1` |
| Camera permission | `Unicus_camera_permission_` | `Unicus_camera_permission_header`, `Unicus_camera_permission_message_enroll`, `Unicus_camera_permission_launch_settings` |
| Accessibility | `Unicus_accessibility_` | `Unicus_accessibility_cancel_button`, `Unicus_accessibility_tap_guidance` |
| NFC reading | `Unicus_idscan_nfc_`, `Unicus_action_scan_nfc` | `Unicus_idscan_nfc_status_ready_message`, `Unicus_action_skip_nfc` |

The complete list is the `UnicusVerificationTextKey` object (every constant is
one key) and the sample catalogs of the example project.

## Flow screen texts

Titles, descriptions, consent text, signature agreement, form labels and the
other step texts are written per flow in the administrative portal, in Spanish
and English. Changing them needs no app release. See
[Flows and UI steps](../flows-and-ui-steps.md).

## Result messages

`result.resultMessage` is the Unicus message for the code in the selected
language. It speaks to the user and says what to do next, for example `4001`
"Se agotaron los intentos." / "No attempts left." or `1005` "Se perdió la
conexión. Revisa tu internet e inténtalo de nuevo." / "The connection was lost.
Check your internet and try again.". A reason written by Unicus for a rejected
step is passed as it is.

`UnicusResultCode.messageFor(code)` returns the Spanish message of a code and
`UnicusResultCode.messageFor(code, UnicusLanguage.ENGLISH)` the English one.
Decide what to do from `outcome` and `resultCode`
([Result codes](../result-codes.md)), never by comparing message texts: the
wording can change between versions.

Next: [Release checklist](release-checklist.md).
