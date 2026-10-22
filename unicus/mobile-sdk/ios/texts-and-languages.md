---
description: >-
  The language option of the Unicus iOS SDK (Spanish, English or the device
  language), what it covers, and how to change some texts with the Unicus_*
  keys.
---

# Texts and languages

The SDK ships every text in **Spanish and English**. One option,
`UnicusSdkConfig.language`, picks the language of everything the SDK shows:

| `UnicusLanguage` | Language |
| --- | --- |
| `.auto` (default) | The device language: Spanish when the first preferred language of the device is Spanish, English otherwise. |
| `.spanish` | Always Spanish. |
| `.english` | Always English. |

{% code overflow="wrap" %}
```swift
var config = UnicusSdkConfig(apiKey: "<CUSTOMER_TOKEN>", environment: .dev)
config.language = .spanish
UnicusSdk.shared.configure(config)

// or in the initializer:
// UnicusSdkConfig(apiKey: "<CUSTOMER_TOKEN>", environment: .dev, language: .spanish)
```
{% endcode %}

The option applies to every screen of the verification:

| Screens | Where the texts come from | Language |
| --- | --- | --- |
| **Camera screens** (face, document, retries, permission, upload progress) | The SDK's built-in texts, replaced key by key by your `Unicus_*` overrides. | `language`. Your overrides are shown as you pass them. |
| **Document data confirmation** (labels of the data read from the document) | The SDK's built-in labels, or your `verificationOcrLocalization`. | `language`. |
| **Flow screens** in `.webView` mode (consent, info, form, signature, OTP, `sign_document`, result and error screens) | The flow in the administrative portal (step titles, descriptions, consent text, form labels) plus Unicus fixed texts. | `language`. |
| **Exit dialog and load error** of the flow screens | The SDK. | `language`. |
| **Your screens** in `.custom` mode | Your app. Step texts from the portal arrive in `step.title` / `step.description`. | `context.language` (`es` or `en`), resolved from the same option. |
| **`result.resultMessage`** | The Unicus result catalog. | `language`. |

The exit dialog asks "¿Salir de la validación?" / "Leave the verification?"
and tells the user that their progress is saved and they can resume the
verification later (see [Results and resuming](../results-and-resuming.md)).

The flow texts are configured per flow in the portal; see
[Flows and UI steps](../flows-and-ui-steps.md) and the web
[Texts and languages](../../sdk-web-v5/texts-and-languages.md) page, which
describes the same portal fields.

{% hint style="info" %}
Before this version the camera screens were in English unless your app passed
Spanish overrides, and `resultMessage` was always in Spanish. With the default
`.auto`, a Spanish device now gets Spanish camera screens and an English device
gets `resultMessage` in English. If your app passed the complete Spanish
dictionary only to translate the screens, replace it with
`config.language = .spanish`.
{% endhint %}

## Change some camera texts

To change the wording of some texts, pass a dictionary of `Unicus_*` keys in
`verificationTextOverrides`. Your texts win over the built-in ones; keys you do
not pass keep the built-in text in the selected language.

{% code overflow="wrap" %}
```swift
var config = UnicusSdkConfig(apiKey: "<CUSTOMER_TOKEN>", environment: .dev)
config.language = .spanish
config.verificationTextOverrides = [
    UnicusVerificationTextKey.actionImReady: "COMENZAR",
    UnicusVerificationTextKey.presessionFrameYourFace: "Ubica tu rostro<br/>en el óvalo",
    "Unicus_retry_header": "Intentémoslo otra vez"
]
UnicusSdk.shared.configure(config)
```
{% endcode %}

* An override has a single text, shown whatever the language. With `.auto`,
  either fix the language or choose the dictionary from
  `Locale.preferredLanguages`.
* Use the constants of `UnicusVerificationTextKey` or the key strings directly.
* `<br/>` becomes a line break; other HTML tags are removed.
* The texts apply from the next `start`.

{% hint style="info" %}
The example project in the package contains the complete English and Spanish
dictionaries (`example/UnicusSDKExample/SampleUnicusTexts.swift`) as a
reference for custom wording.
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

| Key | Built-in English | Built-in Spanish |
| --- | --- | --- |
| `Unicus_action_im_ready` | I'M READY | ESTOY LISTO |
| `Unicus_action_try_again` | TRY AGAIN | INTENTAR OTRA VEZ |
| `Unicus_instructions_header_ready_1` | Get Ready For | Prepárate para |
| `Unicus_presession_frame_your_face` | Frame Your Face In The Oval | Ubica tu rostro en el óvalo. |
| `Unicus_initializing_camera` | Opening the camera | Abriendo la cámara |
| `Unicus_idscan_capture_id_front_instruction_message` | Front of your physical ID | Frente de tu documento físico |
| `Unicus_idscan_capture_id_back_instruction_message` | Place the back in the frame | Ubica el reverso en el recuadro |
| `Unicus_retry_header` | Let's Try That Again | Intentemos de nuevo |
| `Unicus_result_session_abort_message` | Session Ended<br>Please Try Again | Sesión finalizada<br>Inténtalo de nuevo |

`verificationOcrLocalization` replaces the built-in labels of the document data
confirmation screen; leave it `nil` unless Tekbees provides a dictionary.

## Result messages

`result.resultMessage` is the Unicus message for the code in the selected
language. It speaks to the user and says what to do next, for example `4001`
"Se agotaron los intentos." / "No attempts left.". A reason written by Unicus
for a rejected step is passed as it is.

`UnicusResultCode.messageFor(code)` returns the Spanish message of a code and
`UnicusResultCode.messageFor(code, language: .english)` the English one. Decide
what to do from `outcome` and `resultCode`
([Result codes](../result-codes.md)), never by comparing message texts.
