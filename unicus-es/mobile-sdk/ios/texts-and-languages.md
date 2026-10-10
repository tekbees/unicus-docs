---
description: >-
  Cambia los textos y el idioma de las pantallas de verificación en iOS con las
  claves Unicus_*, y cómo eligen su idioma las pantallas del flujo.
---

# Textos e idiomas

Una verificación en iOS muestra dos tipos de pantallas, y sus textos vienen de
lugares distintos:

| Pantallas | De dónde vienen los textos | Idioma |
| --- | --- | --- |
| **Pantallas de cámara** (rostro, documento, reintentos, permiso, progreso de carga) | Los textos en inglés incluidos en el SDK, reemplazados por tus claves `Unicus_*`. | El que tú entregues. Sin reemplazos: inglés. |
| **Pantallas del flujo** en modo `.webView` (consentimiento, información, formulario, firma, OTP, `sign_document`, pantallas de resultado y de error) | El flujo en el portal administrativo (títulos y descripciones de los pasos, texto de consentimiento, etiquetas del formulario) más los textos fijos de Unicus. | Español cuando el idioma del dispositivo es español; inglés en los demás casos. |
| **Tus pantallas** en modo `.custom` | Tu app. Los textos de los pasos del portal llegan en `step.title` / `step.description`. | `context.language` (`es` o `en`), con la misma regla. |

Los textos del flujo se configuran por flujo en el portal; consulta
[Flujos y pasos de UI](../flows-and-ui-steps.md) y la página web
[Textos e idiomas](../../sdk-web-v5/texts-and-languages.md), que describe los
mismos campos del portal.

## Reemplaza los textos de las pantallas de cámara

Entrega un diccionario de claves `Unicus_*` en `verificationTextOverrides`. Las
claves que no entregues conservan el texto en inglés por defecto.

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

* Usa las constantes de `UnicusVerificationTextKey` o las claves como texto.
* `<br/>` se convierte en salto de línea; las demás etiquetas HTML se eliminan.
* Para seguir el idioma del dispositivo, elige el diccionario según
  `Locale.preferredLanguages` como arriba, así las pantallas de cámara coinciden
  con las del flujo.
* Los textos aplican desde el siguiente `start`.

{% hint style="info" %}
El proyecto de ejemplo del paquete trae diccionarios completos en inglés y en
español (`example/UnicusSDKExample/SampleUnicusTexts.swift`). Copia el que
necesites en tu app y ajusta los textos.
{% endhint %}

## Grupos de claves

| Prefijo | Pantallas |
| --- | --- |
| `Unicus_action_*` | Botones: OK, estoy listo, intentar de nuevo, tomar / aceptar / repetir foto, confirmar. |
| `Unicus_instructions_*`, `Unicus_presession_*` | Instrucciones antes de la captura del rostro e indicaciones de posición. |
| `Unicus_feedback_*`, `Unicus_accessibility_*` | Guía en vivo mientras se encuadra el rostro, y sus textos de VoiceOver. |
| `Unicus_camera_permission_*`, `Unicus_initializing_camera`, `Unicus_camera_feed_*` | Permiso de cámara e inicio de la cámara. |
| `Unicus_idscan_*` | Captura del documento: selección de tipo, frente y reverso, revisión, confirmación de datos, NFC. |
| `Unicus_result_*` | Progreso de carga y mensajes de resultado del rostro y del documento. |
| `Unicus_retry_*` | Pantalla de reintento después de una captura que no fue suficiente. |

Claves que se cambian con frecuencia:

| Clave | Texto por defecto |
| --- | --- |
| `Unicus_action_im_ready` | I'M READY |
| `Unicus_action_try_again` | TRY AGAIN |
| `Unicus_instructions_header_ready_1` | Get Ready For |
| `Unicus_presession_frame_your_face` | Frame Your Face In The Oval |
| `Unicus_idscan_capture_id_front_instruction_message` | Scan Front of ID |
| `Unicus_idscan_capture_id_back_instruction_message` | Scan Back of ID |
| `Unicus_retry_header` | Let's Try That Again |
| `Unicus_result_session_abort_message` | Session Ended<br>Please Try Again |

`verificationOcrLocalization` cambia las etiquetas de la pantalla de
confirmación de datos del documento; déjala en `nil` salvo que Tekbees te
entregue un diccionario.
