---
description: >-
  Which texts the Unicus Android SDK shows, in which language, and how to
  change them with the Unicus_* text keys.
---

# Texts and languages

A verification shows three kinds of screens. Each one takes its wording from a
different place:

| Screens | Wording comes from | Language |
| --- | --- | --- |
| **Flow screens** (consent, info, form, signature, OTP, document signing) in WebView mode | The flow configured in the administrative portal (titles, descriptions, consent, agreement, form labels in Spanish and English) plus the Unicus flow app. | Device language: Spanish when the device is in Spanish, English otherwise. |
| **Camera screens** (liveness, document capture, data confirmation) | The SDK defaults, replaced by your `verificationTextOverrides`. | English by default. Pass your own texts for any other language. |
| **SDK native elements** (top bar "Close", "Leave the verification?" dialog, load error) | Android resources of the SDK. | Spanish or English, following your app's locale. |

With [Custom UI steps](custom-ui-steps.md) your app renders the flow screens:
use `context.language` (`es` or `en`) and the localized texts of the step
(`step.title`, `step.config`).

## Camera screen texts

Pass a map of `Unicus_*` keys in `verificationTextOverrides`. Keys you do not
pass keep their default. The constants are in `UnicusVerificationTextKey`.

{% code overflow="wrap" %}
```kotlin
val spanish = mapOf(
    UnicusVerificationTextKey.ACTION_IM_READY to "ESTOY LISTO",
    UnicusVerificationTextKey.ACTION_TRY_AGAIN to "INTENTAR DE NUEVO",
    UnicusVerificationTextKey.INSTRUCTIONS_HEADER_READY_1 to "Prepárate para tu<br/>video selfie",
    UnicusVerificationTextKey.FEEDBACK_CENTER_FACE to "Centra tu rostro",
    "Unicus_idscan_capture_id_front_instruction_message" to "Muestra el frente de tu documento"
)

UnicusSdk.shared.configure(
    UnicusSdkConfig(apiKey = token, environment = UnicusEnvironment.DEV)
        .copy(verificationTextOverrides = spanish)
)
```
{% endcode %}

* HTML line breaks (`<br/>`) become line breaks; other HTML tags are removed.
* The `example/` project of the package ships complete English and Spanish
  catalogs (`SampleUnicusTexts.kt`). Copy the one you need as a starting point.
* To follow the device language, build the map at runtime from your own
  resources (`getString(R.string.…)`) and call `configure` with it.
* The confirmation screen of the data read from the document has its own
  dictionary, `verificationOcrLocalization`. Ask Tekbees for the format if you
  need to change it.

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

## Messages for your own screens

`result.resultMessage` is the Unicus message for the code, in Spanish. Use it
in logs; write your own user-facing texts per `outcome` and `resultCode`
([Result codes](../result-codes.md)) in the languages of your app.

Next: [Release checklist](release-checklist.md).
