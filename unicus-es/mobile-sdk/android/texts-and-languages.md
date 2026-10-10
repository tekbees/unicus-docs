---
description: >-
  Qué textos muestra el SDK Android de Unicus, en qué idioma y cómo cambiarlos
  con las llaves de texto Unicus_*.
---

# Textos e idiomas

Una verificación muestra tres tipos de pantallas. Cada una toma sus textos de
un lugar distinto:

| Pantallas | Los textos vienen de | Idioma |
| --- | --- | --- |
| **Pantallas del flujo** (consentimiento, información, formulario, firma, OTP, firma de documentos) en modo WebView | El flujo configurado en el portal administrativo (títulos, descripciones, consentimiento, acuerdo, etiquetas de formularios en español e inglés) más la app de flujo de Unicus. | Idioma del dispositivo: español cuando el dispositivo está en español, inglés en otro caso. |
| **Pantallas de cámara** (liveness, captura del documento, confirmación de datos) | Los textos por defecto del SDK, reemplazados por tus `verificationTextOverrides`. | Inglés por defecto. Entrega tus propios textos para cualquier otro idioma. |
| **Elementos nativos del SDK** ("Cerrar" en la barra superior, diálogo "¿Salir de la verificación?", error de carga) | Recursos Android del SDK. | Español o inglés, según el idioma de tu app. |

Con [Pasos de UI personalizados](custom-ui-steps.md) tu app muestra las
pantallas del flujo: usa `context.language` (`es` o `en`) y los textos
localizados del paso (`step.title`, `step.config`).

## Textos de las pantallas de cámara

Entrega un mapa de llaves `Unicus_*` en `verificationTextOverrides`. Las llaves
que no entregues conservan su valor por defecto. Las constantes están en
`UnicusVerificationTextKey`.

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

* Los saltos de línea HTML (`<br/>`) se convierten en saltos de línea; las demás
  etiquetas HTML se eliminan.
* El proyecto `example/` del paquete trae catálogos completos en inglés y
  español (`SampleUnicusTexts.kt`). Copia el que necesites como punto de
  partida.
* Para seguir el idioma del dispositivo, arma el mapa en tiempo de ejecución
  desde tus propios recursos (`getString(R.string.…)`) y llama a `configure`
  con él.
* La pantalla de confirmación de los datos leídos del documento tiene su propio
  diccionario, `verificationOcrLocalization`. Pide el formato a Tekbees si
  necesitas cambiarlo.

## Principales grupos de llaves

El SDK acepta unas 175 llaves. Estos grupos cubren lo que ve la mayoría de los
usuarios:

| Grupo | Prefijo | Ejemplos |
| --- | --- | --- |
| Botones | `Unicus_action_` | `Unicus_action_im_ready`, `Unicus_action_try_again`, `Unicus_action_take_photo`, `Unicus_action_accept_photo`, `Unicus_action_retake_photo`, `Unicus_action_confirm`, `Unicus_action_continue`, `Unicus_action_ok` |
| Instrucciones antes de la selfie | `Unicus_instructions_` | `Unicus_instructions_header_ready_1`, `Unicus_instructions_message_ready_1` |
| Guía durante la selfie | `Unicus_feedback_`, `Unicus_presession_` | `Unicus_feedback_center_face`, `Unicus_feedback_move_phone_closer`, `Unicus_feedback_hold_steady`, `Unicus_presession_frame_your_face`, `Unicus_presession_remove_dark_glasses` |
| Captura del documento | `Unicus_idscan_` | `Unicus_idscan_type_selection_header`, `Unicus_idscan_capture_id_front_instruction_message`, `Unicus_idscan_capture_id_back_instruction_message`, `Unicus_idscan_review_id_front_instruction_message` |
| Resultados y cargas | `Unicus_result_` | `Unicus_result_facescan_upload_message`, `Unicus_result_idscan_unsuccess_message`, `Unicus_result_idscan_success_front_side_message` |
| Pantalla de reintento | `Unicus_retry_` | `Unicus_retry_header`, `Unicus_retry_subheader_message`, `Unicus_retry_instruction_message_1` |
| Permiso de cámara | `Unicus_camera_permission_` | `Unicus_camera_permission_header`, `Unicus_camera_permission_message_enroll`, `Unicus_camera_permission_launch_settings` |
| Accesibilidad | `Unicus_accessibility_` | `Unicus_accessibility_cancel_button`, `Unicus_accessibility_tap_guidance` |
| Lectura NFC | `Unicus_idscan_nfc_`, `Unicus_action_scan_nfc` | `Unicus_idscan_nfc_status_ready_message`, `Unicus_action_skip_nfc` |

La lista completa es el objeto `UnicusVerificationTextKey` (cada constante es
una llave) y los catálogos de ejemplo del proyecto de ejemplo.

## Textos de las pantallas del flujo

Los títulos, descripciones, texto de consentimiento, acuerdo de firma,
etiquetas de formularios y demás textos de los pasos se escriben por flujo en
el portal administrativo, en español e inglés. Cambiarlos no requiere publicar
tu app. Consulta [Flujos y pasos de UI](../flows-and-ui-steps.md).

## Mensajes para tus propias pantallas

`result.resultMessage` es el mensaje de Unicus para el código, en español.
Úsalo en logs; escribe tus propios textos para el usuario según `outcome` y
`resultCode` ([Códigos de resultado](../result-codes.md)) en los idiomas de tu
app.

Siguiente: [Lista de salida a producción](release-checklist.md).
