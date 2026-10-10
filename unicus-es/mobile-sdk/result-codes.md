---
description: >-
  Todos los códigos de resultado que puede devolver un SDK móvil de Unicus en
  el resultado de una verificación: significado, outcome que reporta el SDK y
  qué hacer.
---

# Códigos de resultado

El resultado de `start` trae un `resultCode` y un `outcome`. Los códigos son los
mismos de [Web SDK 5.0](../sdk-web-v5/result-codes.md), del webhook final y del
estado de la transacción, más algunos códigos que produce el propio SDK.

El outcome de las tablas es el que reporta el SDK. Los rechazos de la persona
(el rostro no coincide, no cumple la edad, el documento es rechazado) son
`FAILED`; `ERROR` queda para las fallas técnicas.

## Flujo y transacción

| Código | Significado | Outcome | Qué hacer |
| --- | --- | --- | --- |
| `2000` | Todos los pasos obligatorios pasaron. Algunos flujos antiguos reportan `0` o `200`. | `SUCCESS` | Confírmalo en tu backend. |
| `2003` | La transacción sigue abierta: el usuario salió, o hay pasos pendientes. | `RESUMABLE` | Ofrece continuar. Consulta [Retomar](results-and-resuming.md#retomar). |
| `2013` | Posible identidad duplicada, en revisión manual. | `WARNING` | Trátalo como pendiente; espera `TRANSACTION_REVIEW_RESOLVED`. |
| `2002` | Tu empresa no tiene un flujo asignado, o no hay un flujo válido para la transacción. | `FAILED` | Asigna y publica un flujo en el portal. |
| `2011` | La persona ya está enrolada. | `FAILED` | Regla de negocio de tu empresa. |
| `2012` | La persona no está enrolada (el flujo compara contra un enrolamiento). | `FAILED` | Enrola primero a la persona. |
| `2041` | Cancelada por el usuario dentro de la cámara, o por tu app con `cancelActiveSession`. | `CANCELED` | Permite que el usuario empiece de nuevo. |
| `2051` | Transacción no encontrada, ya final o expirada. | `CANCELED` | Inicia una nueva verificación. |
| `2052` | Unicus rechazó un paso. `rejectionReason` trae el motivo (ver abajo). Al crear la transacción: usuario bloqueado. | `FAILED` | Consulta [Paso rechazado](#paso-rechazado-2052). |
| `2054` | Falla temporal de Unicus. No se procesó nada. | `ERROR` | Reintenta (`isRetryable`). |
| `2061` | Se agotaron los intentos de una cara del documento. | `CANCELED` | Inicia una nueva verificación. |
| `4001` | El paso OTP se quedó sin intentos. | `CANCELED` | Inicia una nueva verificación. |
| `9020` | El flujo asignado no puede correr en esta app (consulta [Reglas de flujo en móvil](flows-and-ui-steps.md)). Normalmente se reporta como el error `flow_not_supported`. | `ERROR` | Corrige el flujo o el `uiStepMode`. |

## Firma de documentos (`sign_document`)

| Código | Significado | Outcome | Qué hacer |
| --- | --- | --- | --- |
| `4011` | El firmante rechazó el documento. | `FAILED` | Mensaje de negocio. |
| `4012` | No hay un correo válido del firmante en los datos que usa el paso. | `ERROR` | Corrige la configuración del flujo o los datos recolectados antes del paso. |
| `4013` | Unicus Sign no está habilitado para tu empresa. | `ERROR` | Contacta a Tekbees. |
| `4014` | El servicio de firma no respondió. | `ERROR` | Reintenta (`isRetryable`). |

## Verificación de edad

| Código | Significado | Outcome |
| --- | --- | --- |
| `9010` | No se certifica que el rostro tenga al menos la edad mínima del flujo. | `FAILED` |
| `9011` | No se pudo estimar la edad a partir del rostro. | `FAILED` |
| `10020` | La persona no se verificó por una restricción de edad. | `FAILED` |

## Cámara y documento

Se reportan cuando terminan la transacción, y en los resultados de los pasos.
Mientras no se alcancen los límites de intentos, el usuario puede reintentar
dentro del flujo.

| Código | Significado | Outcome |
| --- | --- | --- |
| `1001` | El número leído del documento no coincide con el documento de la transacción. | `FAILED` |
| `5003` | No se pudo confirmar la prueba de vida. | `FAILED` |
| `5001`, `5002`, `5004`, `5005` | No se pudo guardar o procesar la selfie. | `FAILED` |
| `6003` | Un servicio no respondió a tiempo, o la transacción expiró. | `FAILED` |
| `6004` | Tipo de documento no soportado, o su país o tipo no está habilitado. | `FAILED` |
| `6009` | El clasificador rechazó el documento. | `FAILED` |
| `7001`, `7002` | Frente / reverso de baja calidad o ilegible. | `FAILED` |
| `7003`, `7004`, `7005` | No se pudo leer el documento. | `FAILED` |
| `7006` | El documento no pasó las verificaciones de autenticidad. | `FAILED` |
| `7007` | Documento marcado como fraude en una revisión de duplicado. | `FAILED` |
| `8001`, `8002` | No encontrado o no verificado en el registro oficial. | `FAILED` |
| `8003`–`8006` | Tiempo agotado, error o registro oficial no disponible. | `FAILED` |
| `9001` | El rostro no coincide con la foto del documento. | `FAILED` |
| `9002` | El rostro no coincide con el rostro enrolado. | `FAILED` |
| `9006` | No se encontró la foto del documento. | `FAILED` |
| `10001`, `10002` | El clasificador no pudo determinar el documento o lo encontró inválido. | `FAILED` |
| `9004`, `9005`, `9007` | Error del motor biométrico o del procesamiento. | `ERROR` |
| `10010`, `10011` | Error del clasificador de documentos. | `ERROR` |

Descripciones en detalle: [Códigos de resultado de Web SDK 5.0](../sdk-web-v5/result-codes.md).

## Paso rechazado (`2052`)

`rejectionReason` y `rejectionDetail` vienen del motivo legible por máquina que
envía Unicus (`REASON` o `REASON:detail`), nunca de un texto traducido.

| `rejectionReason` | Significado |
| --- | --- |
| `STEP_OUT_OF_ORDER` | Se envió un paso antes de los obligatorios anteriores. |
| `STEP_NOT_IN_FLOW`, `STEP_TYPE_MISMATCH` | El paso no está en el flujo de la transacción, o tiene otro tipo. Por lo general el flujo cambió mientras la transacción estaba abierta. |
| `FLOW_ALREADY_FAILED` | Un paso obligatorio ya falló. |
| `FORM_INVALID` | Formulario rechazado; el detalle lista los campos. |
| `SIGNATURE_INVALID` | Firma ausente o no válida. |
| `CONSENT_INVALID`, `CONSENT_NOT_RECORDED` | El consentimiento no se registró correctamente. |
| `OTP_INVALID_CODE`, `OTP_EXPIRED`, `OTP_RESEND_TOO_SOON`, `OTP_SEND_LIMIT`, `OTP_SEND_FAILED` | Problemas del OTP; límites como en la [web](../sdk-web-v5/result-codes.md). |
| `SIGN_EMAIL_MISSING`, `SIGN_NOT_CONNECTED`, `SIGN_DECLINED`, `STEP_NOT_AVAILABLE` | Problemas de la firma del documento. |

En modo WebView las pantallas del flujo se los muestran al usuario y le permiten
corregir el paso; tu app solo los recibe cuando el flujo termina. En modo Custom
tus pantallas los reciben en cada envío.

## Códigos del dispositivo

Los produce el SDK cuando la sesión de cámara o una pantalla del flujo termina
antes de tiempo. Nunca llegan al webhook ni al estado de la transacción.

| Código | Significado | Outcome |
| --- | --- | --- |
| `9991` | La sesión de cámara se interrumpió (por ejemplo, un error de red). | `ERROR` |
| `9994` | Bloqueo tras demasiados intentos fallidos en la sesión de cámara. | `ERROR` |
| `9995` | Error de cámara. | `ERROR` |
| `9996` | Permiso de cámara denegado. Pide al usuario que permita la cámara en la configuración del sistema. | `ERROR` |
| `9997` | Error inesperado del componente de cámara. | `ERROR` |
| `9009` | Error interno del SDK, o un error fatal de una pantalla del flujo (`raw["errorKey"]` lo nombra). | `ERROR` |
| `500` | Error genérico cuando no se devolvió ningún código. | `FAILED` |

Cuando el usuario cancela el escaneo del rostro o del documento, el SDK reporta
`2041`.

## Mensajes

`UnicusResultCode.messageFor(code)` devuelve un mensaje corto en español para
cada código, el mismo catálogo que usa Unicus. Usa tus propios textos si
necesitas otro idioma o redacción.
