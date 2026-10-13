---
description: >-
  La opción de idioma del SDK Android de Unicus (español, inglés o el idioma
  del dispositivo), qué cubre y cómo cambiar algunos textos con las llaves
  Unicus_*.
---

# Textos e idiomas

El SDK incluye todos sus textos en **español e inglés**. Una opción,
`UnicusSdkConfig.language`, elige el idioma de todo lo que muestra el SDK:

| `UnicusLanguage` | Idioma |
| --- | --- |
| `AUTO` (por defecto) | El idioma del dispositivo: español en un dispositivo en español, inglés en cualquier otro. |
| `SPANISH` | Siempre español. |
| `ENGLISH` | Siempre inglés. |

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

La opción aplica a todas las pantallas de la verificación:

| Pantallas | Los textos vienen de | Idioma |
| --- | --- | --- |
| **Pantallas de cámara** (liveness, captura del documento, reintentos, permiso de cámara, progreso de carga) | Los textos incluidos en el SDK, reemplazados llave por llave por tus `verificationTextOverrides`. | `language`. Tus reemplazos se muestran tal como los entregas. |
| **Confirmación de datos del documento** (etiquetas de los datos leídos del documento) | Las etiquetas incluidas en el SDK, o tu `verificationOcrLocalization`. | `language`. |
| **Pantallas del flujo** (consentimiento, información, formulario, firma, OTP, firma de documentos) en modo WebView | El flujo configurado en el portal administrativo (títulos, descripciones, consentimiento, acuerdo, etiquetas de formularios en español e inglés) más la app de flujo de Unicus. | `language`. |
| **Elementos nativos del SDK** ("Cerrar" en la barra superior, diálogo de salida, error de carga) | Recursos Android del SDK. | `language`. |
| **`result.resultMessage`** | El catálogo de resultados de Unicus. | `language`. |

El diálogo de salida pregunta «¿Salir de la validación?» / "Leave the
verification?" e informa al usuario que su avance queda guardado y que puede
retomar la validación más tarde (consulta
[Resultados y reanudación](../results-and-resuming.md)).

Con [Pasos de UI personalizados](custom-ui-steps.md) tu app muestra las
pantallas del flujo: usa `context.language` (`es` o `en`, resuelto desde la
misma opción) y los textos localizados del paso (`step.title`, `step.config`).

{% hint style="info" %}
Antes de esta versión las pantallas de cámara salían en inglés salvo que tu app
entregara una tabla en español, y `resultMessage` siempre venía en español. Con
el valor por defecto `AUTO`, un dispositivo en español ahora ve las pantallas
de cámara en español y un dispositivo en inglés recibe `resultMessage` en
inglés. Si tu app entregaba la tabla completa en español solo para traducir las
pantallas, reemplázala por `language = UnicusLanguage.SPANISH`.
{% endhint %}

## Cambiar algunos textos de cámara

Para cambiar la redacción de algunos textos, entrega un mapa de llaves
`Unicus_*` en `verificationTextOverrides`. Tus textos ganan sobre los
incluidos; las llaves que no entregues conservan el texto incluido en el idioma
elegido. Las constantes están en `UnicusVerificationTextKey`.

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

* Un reemplazo tiene un solo texto, que se muestra en cualquier idioma. Con
  `AUTO`, fija el idioma o arma el mapa según el idioma del dispositivo (por
  ejemplo desde tus propios recursos con `getString(R.string.…)`).
* Los saltos de línea HTML (`<br/>`) se convierten en saltos de línea; las demás
  etiquetas HTML se eliminan.
* El proyecto `example/` del paquete trae las tablas completas en inglés y
  español (`SampleUnicusTexts.kt`) como referencia para tu propia redacción.
* `verificationOcrLocalization` reemplaza las etiquetas de la pantalla de
  confirmación de los datos del documento. Pide el formato a Tekbees si
  necesitas cambiarlas.

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

## Mensajes de resultado

`result.resultMessage` es el mensaje de Unicus para el código en el idioma
elegido. Le habla al usuario y le dice qué hacer, por ejemplo `4001` «Se
agotaron los intentos.» / "No attempts left." o `1005` «Se perdió la conexión.
Revisa tu internet e inténtalo de nuevo.» / "The connection was lost. Check
your internet and try again.". Un motivo escrito por Unicus para un paso
rechazado se entrega tal cual.

`UnicusResultCode.messageFor(code)` devuelve el mensaje en español de un código
y `UnicusResultCode.messageFor(code, UnicusLanguage.ENGLISH)` el mensaje en
inglés. Decide qué hacer según `outcome` y `resultCode`
([Códigos de resultado](../result-codes.md)), nunca comparando textos: la
redacción puede cambiar entre versiones.

Siguiente: [Lista de salida a producción](release-checklist.md).
