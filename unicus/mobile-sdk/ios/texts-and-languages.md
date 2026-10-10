---
description: >-
  Change the wording and language of the iOS verification screens with the
  Unicus_* text keys, and how the flow screens choose their language.
---

# Texts and languages

A verification on iOS shows two kinds of screens, and their texts come from
different places:

| Screens | Where the texts come from | Language |
| --- | --- | --- |
| **Camera screens** (face, document, retries, permission, upload progress) | The SDK's built-in English texts, replaced by your `Unicus_*` overrides. | The one you pass. Without overrides: English. |
| **Flow screens** in `.webView` mode (consent, info, form, signature, OTP, `sign_document`, result and error screens) | The flow in the administrative portal (step titles, descriptions, consent text, form labels) plus Unicus fixed texts. | Spanish when the device language is Spanish, English otherwise. |
| **Your screens** in `.custom` mode | Your app. Step texts from the portal arrive in `step.title` / `step.description`. | `context.language` (`es` or `en`), same rule as above. |

The flow texts are configured per flow in the portal; see
[Flows and UI steps](../flows-and-ui-steps.md) and the web
[Texts and languages](../../sdk-web-v5/texts-and-languages.md) page, which
describes the same portal fields.

## Override the camera screen texts

Pass a dictionary of `Unicus_*` keys in `verificationTextOverrides`. Keys you do
not pass keep the English default.

{% code overflow="wrap" %}
```swift
let spanish = Locale.preferredLanguages.first?.hasPrefix("es") == true

var config = UnicusSdkConfig(apiKey: "<CUSTOMER_TOKEN>", environment: .dev)
config.verificationTextOverrides = spanish ? [
    UnicusVerificationTextKey.actionImReady: "ESTOY LISTO",
    UnicusVerificationTextKey.presessionFrameYourFace: "Ubica tu rostro<br/>en el óvalo",
    "Unicus_retry_header": "Intentemos de nuevo"
] : [:]
UnicusSdk.shared.configure(config)
```
{% endcode %}

* Use the constants of `UnicusVerificationTextKey` or the key strings directly.
* `<br/>` becomes a line break; other HTML tags are removed.
* To follow the device language, choose the dictionary from
  `Locale.preferredLanguages` as above, so the camera screens match the flow
  screens.
* The texts apply from the next `start`.

{% hint style="info" %}
The example project in the package contains complete English and Spanish
dictionaries (`example/UnicusSDKExample/SampleUnicusTexts.swift`). Copy the one
you need into your app and edit the wording.
{% endhint %}

## Key groups

| Prefix | Screens |
| --- | --- |
| `Unicus_action_*` | Buttons: OK, I'm ready, try again, take / accept / retake photo, confirm. |
| `Unicus_instructions_*`, `Unicus_presession_*` | Instructions before the face capture and positioning hints. |
| `Unicus_feedback_*`, `Unicus_accessibility_*` | Live guidance while framing the face, and its VoiceOver texts. |
| `Unicus_camera_permission_*`, `Unicus_initializing_camera`, `Unicus_camera_feed_*` | Camera permission and camera start. |
| `Unicus_idscan_*` | Document capture: type selection, front and back, review, data confirmation, NFC. |
| `Unicus_result_*` | Upload progress and result messages of face and document. |
| `Unicus_retry_*` | Retry screen after a capture that was not good enough. |

Frequently changed keys:

| Key | Default text |
| --- | --- |
| `Unicus_action_im_ready` | I'M READY |
| `Unicus_action_try_again` | TRY AGAIN |
| `Unicus_instructions_header_ready_1` | Get Ready For |
| `Unicus_presession_frame_your_face` | Frame Your Face In The Oval |
| `Unicus_idscan_capture_id_front_instruction_message` | Scan Front of ID |
| `Unicus_idscan_capture_id_back_instruction_message` | Scan Back of ID |
| `Unicus_retry_header` | Let's Try That Again |
| `Unicus_result_session_abort_message` | Session Ended<br>Please Try Again |

`verificationOcrLocalization` changes the labels of the document data
confirmation screen; leave it `nil` unless Tekbees provides a dictionary.
