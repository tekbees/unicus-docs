---
description: >-
  La opción de idioma del SDK iOS de Unicus (español, inglés o el idioma del
  dispositivo), qué cubre y cómo cambiar algunos textos con las claves
  Unicus_*.
---

# Textos e idiomas

El SDK incluye todos sus textos en **español e inglés**. Una opción,
`UnicusSdkConfig.language`, elige el idioma de todo lo que muestra el SDK:

| `UnicusLanguage` | Idioma |
| --- | --- |
| `.auto` (por defecto) | El idioma del dispositivo: español cuando el primer idioma preferido del dispositivo es español, inglés en otro caso. |
| `.spanish` | Siempre español. |
| `.english` | Siempre inglés. |

{% code overflow="wrap" %}
```swift
var config = UnicusSdkConfig(apiKey: "<CUSTOMER_TOKEN>", environment: .dev)
config.language = .spanish
UnicusSdk.shared.configure(config)

// o en el inicializador:
// UnicusSdkConfig(apiKey: "<CUSTOMER_TOKEN>", environment: .dev, language: .spanish)
```
{% endcode %}

La opción aplica a todas las pantallas de la verificación:

| Pantallas | De dónde vienen los textos | Idioma |
| --- | --- | --- |
| **Pantallas de cámara** (rostro, documento, reintentos, permiso, progreso de carga) | Los textos incluidos en el SDK, reemplazados clave por clave por tus claves `Unicus_*`. | `language`. Tus reemplazos se muestran tal como los entregas. |
| **Confirmación de datos del documento** (etiquetas de los datos leídos del documento) | Las etiquetas incluidas en el SDK, o tu `verificationOcrLocalization`. | `language`. |
| **Pantallas del flujo** en modo `.webView` (consentimiento, información, formulario, firma, OTP, `sign_document`, pantallas de resultado y de error) | El flujo en el portal administrativo (títulos y descripciones de los pasos, texto de consentimiento, etiquetas del formulario) más los textos fijos de Unicus. | `language`. |
| **Diálogo de salida y error de carga** de las pantallas del flujo | El SDK. | `language`. |
| **Tus pantallas** en modo `.custom` | Tu app. Los textos de los pasos del portal llegan en `step.title` / `step.description`. | `context.language` (`es` o `en`), resuelto desde la misma opción. |
| **`result.resultMessage`** | El catálogo de resultados de Unicus. | `language`. |

El diálogo de salida pregunta «¿Salir de la validación?» / "Leave the
verification?" e informa al usuario que su avance queda guardado y que puede
retomar la validación más tarde (consulta
[Resultados y reanudación](../results-and-resuming.md)).

Los textos del flujo se configuran por flujo en el portal; consulta
[Flujos y pasos de UI](../flows-and-ui-steps.md) y la página web
[Textos e idiomas](../../sdk-web-v5/texts-and-languages.md), que describe los
mismos campos del portal.

{% hint style="info" %}
Antes de esta versión las pantallas de cámara salían en inglés salvo que tu app
entregara reemplazos en español, y `resultMessage` siempre venía en español.
Con el valor por defecto `.auto`, un dispositivo en español ahora ve las
pantallas de cámara en español y un dispositivo en inglés recibe
`resultMessage` en inglés. Si tu app entregaba el diccionario completo en
español solo para traducir las pantallas, reemplázalo por
`config.language = .spanish`.
{% endhint %}

## Cambiar algunos textos de cámara

Para cambiar la redacción de algunos textos, entrega un diccionario de claves
`Unicus_*` en `verificationTextOverrides`. Tus textos ganan sobre los
incluidos; las claves que no entregues conservan el texto incluido en el idioma
elegido.

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

* Un reemplazo tiene un solo texto, que se muestra en cualquier idioma. Con
  `.auto`, fija el idioma o elige el diccionario según
  `Locale.preferredLanguages`.
* Usa las constantes de `UnicusVerificationTextKey` o las claves como texto.
* `<br/>` se convierte en salto de línea; las demás etiquetas HTML se eliminan.
* Los textos aplican desde el siguiente `start`.

{% hint style="info" %}
El proyecto de ejemplo del paquete trae los diccionarios completos en inglés y
en español (`example/UnicusSDKExample/SampleUnicusTexts.swift`) como referencia
para tu propia redacción.
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

| Clave | Texto incluido en inglés | Texto incluido en español |
| --- | --- | --- |
| `Unicus_action_im_ready` | I'M READY | ESTOY LISTO |
| `Unicus_action_try_again` | TRY AGAIN | INTENTAR OTRA VEZ |
| `Unicus_instructions_header_ready_1` | Get Ready For | Prepárate para |
| `Unicus_presession_frame_your_face` | Frame Your Face In The Oval | Ubica tu rostro en el óvalo. |
| `Unicus_initializing_camera` | Opening the camera | Abriendo la cámara |
| `Unicus_idscan_capture_id_front_instruction_message` | Place the front of the document in the frame | Ubica el frente del documento en el recuadro |
| `Unicus_idscan_capture_id_back_instruction_message` | Place the back of the document in the frame | Ubica el reverso del documento en el recuadro |
| `Unicus_retry_header` | Let's Try That Again | Intentemos de nuevo |
| `Unicus_result_session_abort_message` | Session Ended<br>Please Try Again | Sesión finalizada<br>Inténtalo de nuevo |

`verificationOcrLocalization` reemplaza las etiquetas incluidas de la pantalla
de confirmación de datos del documento; déjala en `nil` salvo que Tekbees te
entregue un diccionario.

## Mensajes de resultado

`result.resultMessage` es el mensaje de Unicus para el código en el idioma
elegido. Le habla al usuario y le dice qué hacer, por ejemplo `4001` «Se
agotaron los intentos.» / "No attempts left.". Un motivo escrito por Unicus para
un paso rechazado se entrega tal cual.

`UnicusResultCode.messageFor(code)` devuelve el mensaje en español de un código
y `UnicusResultCode.messageFor(code, language: .english)` el mensaje en inglés.
Decide qué hacer según `outcome` y `resultCode`
([Códigos de resultado](../result-codes.md)), nunca comparando textos.
