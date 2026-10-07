---
description: >-
  Consulta el estado y el resultado de una transacción de Unicus desde tu
  backend con el endpoint query-transaction.
---

# Consultar el estado de una transacción

Usa `query-transaction` desde tu **backend** para leer el estado y el resultado
de una transacción por su id (`tid`). El webhook final es el canal principal
para los resultados; usa este endpoint como respaldo y para conciliar (consulta
[Uso recomendado](#uso-recomendado)).

{% hint style="warning" %}
Llama a este endpoint solo desde tu servidor. Nunca expongas tu API key ni tu
Customer Token en un navegador o en una app móvil.
{% endhint %}

## Solicitud

<mark style="color:green;">`POST`</mark> `<unicus-server-api-url>/query-transaction`

### Headers

| Nombre | Obligatorio | Descripción |
| --- | --- | --- |
| `Authorization` | Sí, si tu empresa exige una API key | `Bearer <API_KEY>`: la API key de la empresa generada en el portal administrativo. Recomendado para toda integración. |
| `X-Customer-ID` | Solo sin `Authorization` | Customer Token del mismo ambiente donde se creó la transacción (credencial heredada). Se rechaza con `401` una vez que tu empresa exige una API key. |
| `Content-Type` | Recomendado | `application/json` |

Con una API key válida se ignora el header del Customer Token: la key
identifica a tu empresa.

### Cuerpo

| Nombre | Tipo | Obligatorio | Descripción |
| --- | --- | --- | --- |
| `tid` | string | Sí | Id de la transacción que devolvió Unicus cuando se creó la transacción. |

### Ejemplo

{% code overflow="wrap" %}
```bash
curl --request POST '<unicus-server-api-url>/query-transaction' \
  --header 'Authorization: Bearer <API_KEY>' \
  --header 'Content-Type: application/json' \
  --data '{ "tid": "<TID>" }'
```
{% endcode %}

Solo puedes consultar transacciones de tu propia empresa. Un `tid` de otra
empresa responde como no encontrado.

## Respuestas

Las respuestas de negocio siempre llegan con HTTP `200`. Lee `success` y
`resultMessage`:

| Situación | `success` | `resultCode` | `resultMessage` |
| --- | --- | --- | --- |
| Transacción encontrada (final, o nunca abierta) | `true` | `0` | `""` |
| Transacción creada o todavía en proceso | `false` | `-1` | `TRANSACTION IN PROCESS` |
| Transacción expirada tras 20 minutos de inactividad | `false` | `-1` | `TRANSACTION EXPIRED` |
| `tid` desconocido, o un `tid` de otra empresa | `false` | `-1` | `TRANSACTION NOT FOUND` / `Transaction not found` |
| El Customer Token no pertenece a una empresa activa | `false` | `-1` | `COMPANY NOT FOUND` |

{% hint style="info" %}
El `resultCode` de primer nivel solo es `0` o `-1`. El resultado de la
verificación está en `transactionResult` y `transactionStatusId`; consulta
[Códigos de resultado](result-codes.md).
{% endhint %}

### Transacción no disponible

{% code overflow="wrap" %}
```json
{
  "success": false,
  "wasProcessed": true,
  "error": false,
  "didError": false,
  "path": "query-transaction",
  "resultCode": -1,
  "resultMessage": "TRANSACTION IN PROCESS",
  "elapsedPerformanceTime": 0,
  "additionalSessionData": {},
  "tid": "<TID>"
}
```
{% endcode %}

### Transacción encontrada

Los campos sin valor se omiten. Los grupos presentes dependen del tipo de
transacción (enrolamiento, verificación, prueba de vida) y de los pasos que se
ejecutaron.

{% code overflow="wrap" expandable="true" %}
```json
{
  "success": true,
  "wasProcessed": true,
  "error": false,
  "didError": false,
  "path": "query-transaction",
  "resultCode": 0,
  "resultMessage": "",
  "elapsedPerformanceTime": 0,
  "tid": "<TID>",
  "transactionType": "<TRANSACTION_TYPE>",
  "transactionResult": "<TRANSACTION_RESULT>",
  "transactionStatus": "<STATUS_NAME>",
  "transactionStatusId": 2000,
  "transactionDate": "<ISO_DATE>",
  "transactionUpdate": "<ISO_DATE>",
  "companyTin": 0,
  "companyName": "<COMPANY_NAME>",
  "document": "<ID_NUMBER>",
  "documentType": "<ID_TYPE>",
  "name": "<NAME>",
  "lastName": "<LAST_NAME>",
  "country": "<COUNTRY_CODE>",
  "state": "<STATE>",
  "statusUser": 0,
  "userTid": "<ENROLLMENT_TID>",
  "comments": "<REVIEW_COMMENTS>",
  "liveness": { },
  "livenessStatus": 1,
  "matchLevel": { },
  "matchLevelStatus": 1,
  "ocr": { },
  "ocrStatus": 1,
  "infer": { },
  "inferStatus": -1,
  "validNumberCheck": { },
  "validNumberCheckStatus": 1,
  "government": { },
  "governmentStatus": 1,
  "documentValidation": { },
  "documentValidationStatus": 1,
  "feature": { },
  "matchIdFeature": { },
  "location": { },
  "searchDuplicated": [ ],
  "flowSteps": [
    {
      "stepId": "<STEP_ID>",
      "type": "otp",
      "outcome": "completed",
      "resultCode": 2000,
      "recordedAt": "<ISO_DATE>",
      "data": { "channel": "<CHANNEL>", "destination": "<DESTINATION>", "verifiedAt": "<ISO_DATE>" }
    }
  ],
  "faceImageList": [
    { "folder": "<FOLDER>", "filename": "<FILE_NAME>", "url": "<PRESIGNED_URL>" }
  ],
  "frontDocumentUrl": "<PRESIGNED_URL>",
  "backDocumentUrl": "<PRESIGNED_URL>",
  "frontDocumentWithoutSegmentUrl": "<PRESIGNED_URL>",
  "backDocumentWithoutSegmentUrl": "<PRESIGNED_URL>",
  "hash": "<SHA256>"
}
```
{% endcode %}

#### Transacción

| Campo | Descripción |
| --- | --- |
| `tid` | Id de la transacción. |
| `transactionType` | Nombre del tipo de transacción. |
| `transactionResult` | Nombre del resultado general de la transacción. |
| `transactionStatus` / `transactionStatusId` | Nombre y número del último código registrado en la transacción (consulta [Códigos de resultado](result-codes.md)). `transactionStatusId` se devuelve para los enrolamientos. |
| `transactionDate` / `transactionUpdate` | Fechas de creación y de última actualización. |
| `companyTin` / `companyName` | Tu empresa. |
| `comments` | Comentarios de la revisión manual, si los hay. |
| `location` | Ubicación reportada por el dispositivo, si la hay. |

#### Persona y documento

{% hint style="warning" %}
Datos personales. Almacénalos y procésalos de acuerdo con tus obligaciones de
protección de datos.
{% endhint %}

| Campo | Descripción |
| --- | --- |
| `document` / `documentType` | Número y tipo de documento. |
| `name` / `lastName` | Nombre de la persona. |
| `country` / `state` | País y estado del documento. |
| `statusUser` | Estado de la persona en Unicus. |
| `userTid` | Verificaciones: `tid` del enrolamiento de la persona. |
| `ocr` | Datos leídos del documento. |
| `searchDuplicated` | Posibles duplicados encontrados en la búsqueda 1:N de tu empresa, con sus datos e imágenes. |

#### Validaciones y biometría

Cada validación tiene un objeto con su detalle y un estado:

| Campo | Valores de estado |
| --- | --- |
| `liveness` / `livenessStatus` | `-1` no ejecutada, `0` pendiente, `1` aprobada, `2` fallida. |
| `matchLevel` / `matchLevelStatus` | Mismos valores. |
| `ocr` / `ocrStatus` | Mismos valores. |
| `infer` / `inferStatus` | Mismos valores. |
| `validNumberCheck` / `validNumberCheckStatus` | Validación del número de documento. Mismos valores. |
| `government` / `governmentStatus` | Validación contra el registro oficial. Mismos valores. |
| `documentValidation` / `documentValidationStatus` | Validaciones de autenticidad del documento. Mismos valores. |

`feature` (prueba de vida o enrolamiento) y `matchIdFeature` (coincidencia con
el documento) contienen el resultado técnico del motor biométrico, incluidos los
niveles de coincidencia y el grupo de edad (consulta
[Códigos de resultado](result-codes.md)).

#### Pasos del flujo

`flowSteps` está presente en las transacciones que ejecutaron un flujo del
portal: cada paso con un resultado registrado, en el orden del flujo.

| Campo | Descripción |
| --- | --- |
| `stepId` / `type` | Id y tipo del paso (`liveness`, `document`, `face_match`, `signature`, `otp`, `form`, `consent`, `info`, `age_check`). |
| `outcome` | `completed`, `failed` o `skipped` (paso opcional que falló). |
| `resultCode` | `2000` o el código de la falla. |
| `recordedAt` | Cuándo se registró el resultado (UTC). |
| `data` | Lo que entregó el usuario: los `values` del formulario; `channel`, `destination` y `verifiedAt` del OTP; hash de la firma, texto del acuerdo y un `imageUrl` válido por 24 horas; texto y fechas del consentimiento; `validations` del documento; edad mínima y edad certificada de `age_check`. Datos personales. |

#### Imágenes

{% hint style="warning" %}
Las imágenes son datos personales (y biométricos). Descárgalas solo si las
necesitas y almacénalas de forma segura.
{% endhint %}

| Campo | Presente |
| --- | --- |
| `faceImageList` | Imágenes del rostro capturadas durante la transacción. Cada elemento tiene una `url`, o los bytes de la imagen en `image` cuando no hay URL disponible. |
| `frontDocumentUrl`, `backDocumentUrl` | Enrolamientos con documento: frente y reverso recortados. |
| `frontDocumentWithoutSegmentUrl`, `backDocumentWithoutSegmentUrl` | Enrolamientos con documento: imágenes completas del frente y del reverso. |
| `photoIDTamperingEvidenceFrontImageUrl`, `photoIDTamperingEvidenceBackImageUrl` | Enrolamientos cuyo último código es `7006` o `2041`: evidencia de adulteración. |

Las URL de las imágenes son prefirmadas y expiran después de unos minutos (5 por
defecto): vuelve a consultar para obtener URL nuevas. Cuando una imagen no tiene
URL, su contenido en base64 llega en el campo sin el sufijo `Url`
(`frontDocument`, `backDocument`, `frontDocumentWithoutSegment`,
`backDocumentWithoutSegment`, `photoIDTamperingEvidenceFrontImage`,
`photoIDTamperingEvidenceBackImage`).

#### Otros campos

| Campo | Descripción |
| --- | --- |
| `hash` | SHA-256 calculado por Unicus sobre el contenido de la respuesta. |
| `success`, `wasProcessed`, `error`, `didError`, `path`, `resultCode`, `resultMessage`, `elapsedPerformanceTime`, `additionalSessionData` | Envoltorio técnico común a las respuestas de Unicus. |

## Errores HTTP

| Estado | Cuándo | Cuerpo |
| --- | --- | --- |
| `400` | El cuerpo no es un JSON válido o no es un objeto. | `application/problem+json` con `title` `Structure error in request [invalid-json]`. |
| `401` | API key inválida, revocada o expirada, o tu empresa exige una API key y la solicitud usó `X-Customer-ID`. | `{"status":401,"title":"Unauthorized","detail":"..."}` y `WWW-Authenticate: Bearer`. |
| `429` | Se superó el límite de solicitudes para tu credencial (600 solicitudes por minuto por defecto). | `{"status":429,"title":"Too many requests"}` y `Retry-After` en segundos. |
| `500` | Error inesperado. | `application/problem+json`. Reintenta más tarde. |

## Uso recomendado

1. **Primero el webhook.** Unicus envía `TRANSACTION_FINALIZED` una vez por
   transacción cuando su resultado es final. Basa tu decisión en él (consulta
   [Webhooks](webhooks.md)).
2. **La consulta como respaldo y conciliación.** Llama a `query-transaction`
   cuando un webhook no llegó (por ejemplo, tu endpoint estuvo caído más tiempo
   que el de los reintentos), para obtener imágenes que el webhook no incluye o
   en un proceso periódico de conciliación.
3. **No consultes de forma agresiva.** Mientras la respuesta sea
   `TRANSACTION IN PROCESS`, el usuario puede estar todavía reintentando: una
   transacción se vuelve final tras su último paso, un límite de intentos o 20
   minutos de inactividad (24 horas si el enlace nunca se abre). Si necesitas
   consultar periódicamente, usa intervalos de un minuto o más y respeta
   `Retry-After` en los `429`.
4. **Trata `2013` como pendiente.** Una transacción en revisión manual se decide
   más tarde (`TRANSACTION_REVIEW_RESOLVED`).
