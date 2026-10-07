---
description: >-
  El resultado final de cada transacción, enviado a tu backend: un evento
  TRANSACTION_FINALIZED firmado por transacción, con reintentos hasta que lo confirmes.
---

# Webhooks

El webhook es el **resultado autoritativo** de una verificación. Los eventos del
navegador del botón le indican a tu página qué ocurrió, pero cualquier cosa que reporte el navegador del
usuario puede ser manipulada: decide en tu backend, con el webhook
(o con [Consultar el estado de una transacción](transaction-status.md)).

Unicus envía **un evento por transacción**, cuando su resultado es final:

| Evento | Cuándo |
| --- | --- |
| `TRANSACTION_FINALIZED` | Una vez por transacción, cuando termina: aprobada, rechazada, enviada a revisión manual, expirada, cancelada o eliminada. |
| `TRANSACTION_REVIEW_RESOLVED` | Solo para una transacción que terminó en `REVIEW`, cuando Tekbees decide la revisión manual. |

{% hint style="warning" %}
**¿Vienes de Web SDK 4.x?** Los webhooks por paso (`LIVENESS_FACEMAP`,
`FRONT_DOCUMENT`, `BACK_DOCUMENT`, `MATCH_DOCUMENT`, `VERIFY_LIVENESS`) ya no
se envían. Tu endpoint recibe solo los eventos de esta página. Consulta
[Migración desde Web SDK 4.x](migration-from-v4.md).
{% endhint %}

## 1. Configura tu endpoint

En el [portal administrativo](https://app.idunicus.com/), ve a
**Compañía → Configuraciones → Webhook** e ingresa la URL de tu endpoint, por ejemplo
`https://api.example.com/unicus/webhook`.

<figure><img src="../.gitbook/assets/image (25).png" alt="Configuración del webhook en el portal administrativo" width="563"><figcaption><p>Configuración del webhook en el portal administrativo.</p></figcaption></figure>

Requisitos:

* Una URL pública `https://`. Se rechazan los hosts que resuelven a direcciones privadas, de loopback o
  link-local, y **no se siguen redirecciones**: indica la
  URL final.
* Responde cualquier `2xx` en menos de **30 segundos**. Cualquier otra respuesta (o la ausencia de respuesta) genera
  reintentos.
* Una URL por empresa y ambiente (el ambiente de pruebas (sandbox) y producción se configuran
  por separado).

## 2. Activa las firmas (recomendado)

En la misma pantalla, **Firma de webhooks → Generar secreto**. El portal muestra el
secreto (`whsec_…`) **una sola vez**: guárdalo en el gestor de secretos de tu backend, nunca en
código del navegador ni en un repositorio. En menos de un minuto todos los webhooks llegan firmados.

| Encabezado | Valor |
| --- | --- |
| `X-Unicus-Event-Id` | Id único del evento, el mismo en cada reintento (`fin_<tid>` o `rev_<tid>`). También está en `meta.event_id`. |
| `X-Unicus-Timestamp` | Tiempo Unix en segundos en que se firmó esta entrega. |
| `X-Unicus-Signature` | `v1=` + HMAC-SHA256 en hexadecimal de `timestamp + "." + raw body`, con el secreto (la cadena `whsec_…` completa, en UTF-8) como clave. |

Para verificar una solicitud:

1. Lee los **bytes del cuerpo crudo (raw body)** antes de parsear el JSON (un JSON
   re-serializado no coincide).
2. Recházala si `X-Unicus-Timestamp` está a más de **5 minutos** de tu
   reloj.
3. Calcula `HMAC-SHA256(secret, timestamp + "." + rawBody)` en hexadecimal, antepón
   `v1=` y compáralo con `X-Unicus-Signature` en **tiempo constante**.

**Rotar secreto** emite uno nuevo; el anterior deja de validar en menos de un
minuto, así que actualiza tu endpoint de inmediato. `X-Unicus-Event-Id` se envía incluso
sin secreto.

{% tabs %}
{% tab title="Node.js (Express)" %}
```js
import crypto from 'node:crypto';
import express from 'express';

const app = express();
const SECRET = process.env.UNICUS_WEBHOOK_SECRET; // whsec_...

app.post('/unicus/webhook', express.raw({ type: 'application/json' }), (req, res) => {
  const timestamp = req.get('X-Unicus-Timestamp') ?? '';
  const signature = req.get('X-Unicus-Signature') ?? '';
  const age = Math.abs(Date.now() / 1000 - Number(timestamp));
  const expected = 'v1=' + crypto.createHmac('sha256', SECRET)
    .update(`${timestamp}.`).update(req.body).digest('hex');
  const valid = age <= 300 && signature.length === expected.length
    && crypto.timingSafeEqual(Buffer.from(signature), Buffer.from(expected));
  if (!valid) return res.sendStatus(401);

  const event = JSON.parse(req.body.toString('utf8'));
  res.sendStatus(200);           // primero confirma...
  handleUnicusEvent(event);      // ...luego procesa (de forma idempotente, por meta.event_id)
});
```
{% endtab %}

{% tab title="Python (Flask)" %}
```python
import hashlib, hmac, os, time
from flask import Flask, request, abort

app = Flask(__name__)
SECRET = os.environ["UNICUS_WEBHOOK_SECRET"].encode()  # whsec_...

@app.post("/unicus/webhook")
def unicus_webhook():
    timestamp = request.headers.get("X-Unicus-Timestamp", "")
    signature = request.headers.get("X-Unicus-Signature", "")
    raw = request.get_data()  # bytes crudos, antes de parsear
    expected = "v1=" + hmac.new(SECRET, timestamp.encode() + b"." + raw, hashlib.sha256).hexdigest()
    if not timestamp.isdigit() or abs(time.time() - int(timestamp)) > 300 \
            or not hmac.compare_digest(signature, expected):
        abort(401)
    event = request.get_json()
    enqueue(event)  # procesa de forma asíncrona e idempotente por event["meta"]["event_id"]
    return "", 200
```
{% endtab %}

{% tab title="Java" %}
```java
// rawBody: los bytes del cuerpo de la solicitud exactamente como se recibieron.
static boolean isValid(String secret, String timestamp, String signature, byte[] rawBody) throws Exception {
    long ts;
    try { ts = Long.parseLong(timestamp); } catch (NumberFormatException e) { return false; }
    if (Math.abs(Instant.now().getEpochSecond() - ts) > 300) return false;
    Mac mac = Mac.getInstance("HmacSHA256");
    mac.init(new SecretKeySpec(secret.getBytes(StandardCharsets.UTF_8), "HmacSHA256"));
    mac.update((timestamp + ".").getBytes(StandardCharsets.UTF_8));
    String expected = "v1=" + HexFormat.of().formatHex(mac.doFinal(rawBody));
    return MessageDigest.isEqual(expected.getBytes(StandardCharsets.UTF_8),
            signature == null ? new byte[0] : signature.getBytes(StandardCharsets.UTF_8));
}
```
{% endtab %}

{% tab title="PHP" %}
```php
$secret    = getenv('UNICUS_WEBHOOK_SECRET'); // whsec_...
$raw       = file_get_contents('php://input');
$timestamp = $_SERVER['HTTP_X_UNICUS_TIMESTAMP'] ?? '';
$signature = $_SERVER['HTTP_X_UNICUS_SIGNATURE'] ?? '';
$expected  = 'v1=' . hash_hmac('sha256', $timestamp . '.' . $raw, $secret);

if (!ctype_digit($timestamp) || abs(time() - (int) $timestamp) > 300
    || !hash_equals($expected, $signature)) {
    http_response_code(401);
    exit;
}
$event = json_decode($raw, true);
// Guárdalo, responde 200 y procesa de forma idempotente por $event['meta']['event_id'].
http_response_code(200);
```
{% endtab %}

{% tab title="C# (.NET)" %}
```csharp
static bool IsValid(string secret, string timestamp, string signature, byte[] rawBody)
{
    if (!long.TryParse(timestamp, out var ts)) return false;
    if (Math.Abs(DateTimeOffset.UtcNow.ToUnixTimeSeconds() - ts) > 300) return false;
    using var hmac = new HMACSHA256(Encoding.UTF8.GetBytes(secret));
    var prefix = Encoding.UTF8.GetBytes(timestamp + ".");
    var hash = hmac.ComputeHash(prefix.Concat(rawBody).ToArray());
    var expected = "v1=" + Convert.ToHexString(hash).ToLowerInvariant();
    return CryptographicOperations.FixedTimeEquals(
        Encoding.UTF8.GetBytes(expected), Encoding.UTF8.GetBytes(signature ?? ""));
}
```
{% endtab %}
{% endtabs %}

## 3. Gestiona las entregas

* **Al menos una vez.** Un webhook puede llegar más de una vez (un timeout de tu
  lado, un error de red). Guarda `meta.event_id` e ignora un id que ya
  procesaste.
* **Reintentos.** Cualquier respuesta distinta de `2xx` se reintenta durante unas 22 horas: después de 10 s,
  30 s, 2 min, 10 min, 30 min, 1 h, 2 h, 4 h, 6 h y 8 h. Si tu endpoint está
  caído por más tiempo, concilia con [Consultar el estado de una transacción](transaction-status.md).
* **Responde rápido.** Persiste el evento y responde `200`; haz el trabajo lento
  (actualizar tus sistemas, notificar al usuario) después.
* **Orden.** Para una misma transacción, `TRANSACTION_FINALIZED` siempre llega antes que
  `TRANSACTION_REVIEW_RESOLVED`; entre transacciones distintas no hay orden.
* **Datos personales.** El payload contiene los datos del documento de la persona (y
  opcionalmente imágenes): registra en los logs solo el `tid` y el resultado, nunca el cuerpo.

## `TRANSACTION_FINALIZED`

{% code overflow="wrap" %}
```json
{
  "meta": {
    "event": "TRANSACTION_FINALIZED",
    "event_id": "fin_6f1c2a4e-1b7d-4c8e-9a51-2f3b4c5d6e7f",
    "version": 2,
    "occurred_at": "2026-10-05T15:04:05Z",
    "ok": true,
    "code": 200
  },
  "data": {
    "tid": "6f1c2a4e-1b7d-4c8e-9a51-2f3b4c5d6e7f",
    "type": "ENROLLMENT",
    "outcome": "APPROVED",
    "result_code": 2000,
    "result_message": "Success",
    "finalized_by": "PROCESS",
    "created_at": "2026-10-05T14:51:10Z",
    "started_at": "2026-10-05T14:52:02Z",
    "finalized_at": "2026-10-05T15:04:05Z",
    "attempts": { "face": 0, "document_front": 1, "document_back": 0, "document": 0 },
    "idNumber": "<NÚMERO DE DOCUMENTO>",
    "location": "{\"latitude\":4.7398,\"longitude\":-74.1137}",
    "document": {
      "idType": "ID",
      "idNumberOCR": "<NÚMERO DE DOCUMENTO LEÍDO DEL DOCUMENTO>",
      "idName": "<NOMBRES>",
      "idLastName": "<APELLIDOS>",
      "birthDate": "<FECHA>",
      "match_level": 6,
      "digital_id_spoof": 1,
      "face_on_document_status": 1,
      "full_id_status": 1,
      "text_on_document_status": 1,
      "unexpectedMediaEncounteredAtLeastOnce": false
    },
    "biometrics": { "ageEstimateGroup": 4, "livenessCheck": true },
    "flow": { "id": "onboarding", "version": 3 }
  }
}
```
{% endcode %}

### `meta`

| Campo | Tipo | Descripción |
| --- | --- | --- |
| `event` | string | `TRANSACTION_FINALIZED`. |
| `event_id` | string | `fin_<tid>`. Nunca cambia entre reintentos: úsalo para deduplicar. |
| `version` | int | Versión del payload, actualmente `2`. |
| `occurred_at` | string | Cuándo terminó la transacción (ISO 8601, UTC). |
| `ok` / `code` | bool / int | `true` / `200` solo cuando `outcome` es `APPROVED`; en otro caso `false` / `400`. Prefiere `data.outcome`. |

### `data`

| Campo | Tipo | Descripción |
| --- | --- | --- |
| `tid` | string | Id de la transacción: el que `OnUnicus:loaded` entregó a tu página. |
| `type` | string | `ENROLLMENT`, `VERIFY`, `LIVENESS` o `FACE_AGE_ID`. |
| `outcome` | string | `APPROVED`, `REJECTED`, `REVIEW`, `EXPIRED`, `CANCELLED` o `DELETED` (ver más abajo). |
| `result_code` | int | `2000` cuando se aprueba; en otro caso, el código que la terminó. Consulta [Códigos de resultado](result-codes.md). |
| `result_message` | string | Descripción de `result_code`. |
| `finalized_by` | string | Qué la terminó: `PROCESS` (el resultado de una captura), `FLOW` (el último paso del flujo o un paso fallido), `INACTIVITY`, `NOT_STARTED`, `CANCEL`, `DELETE`. |
| `created_at` | string | Cuándo se creó la transacción. |
| `started_at` | string | Cuándo el usuario la abrió por primera vez. Ausente si nunca se abrió. |
| `finalized_at` | string | Cuándo terminó. |
| `attempts` | object | Intentos fallidos por captura: `face`, `document_front`, `document_back`, `document` (rostro contra documento). Los errores técnicos del lado de Unicus no se cuentan. |
| `last_failure` | object | `{ code, message, step }` del último intento fallido. Ausente cuando se aprueba o cuando nada falló. |
| `idNumber` | string | Número de documento con el que se creó la transacción (`clientid`). |
| `location` | string | Texto JSON `{"latitude":…,"longitude":…}` cuando el usuario compartió su ubicación. Ausente en otro caso. |
| `document` | object | Datos y validaciones del documento, cuando se capturó un documento (ver más abajo). |
| `biometrics` | object | Resultados faciales, cuando se capturó el rostro: `ageEstimateGroup`, `livenessCheck`, `matchLevel` (verificación). |
| `flow` | object | `{ id, version }` del flujo del portal que ejecutó la transacción. Ausente en transacciones sin flujo. |
| `images` | object | Solo si Tekbees habilitó las imágenes para tu empresa: `document_front`, `document_back` (JPEG en base64). En otro caso, obtenlas con [Consultar el estado de una transacción](transaction-status.md). |

**`document`** puede contener: `idType`, `idNumberOCR` (el número leído del
documento), `idName`, `idLastName`, `birthDate`, `placeBirth`, `issue_date`,
`issue_place`, `height`, `bloodType`, `gender`, `idCountryCode`, `address1`–`address3`,
`custom_field_1`–`custom_field_5`, y las validaciones `match_level` (rostro contra
la foto del documento, `0`–`7`), `age_estimate_group`, `digital_id_spoof`,
`face_on_document_status`, `full_id_status`, `text_on_document_status` y
`unexpectedMediaEncounteredAtLeastOnce`. Los campos presentes dependen
del país y de la plantilla del documento. Compara `idNumberOCR` con `idNumber` si
necesitas asegurarte de que el documento pertenece a la persona que esperabas.

### Resultados

| `outcome` | Cuándo | Qué hacer |
| --- | --- | --- |
| `APPROVED` | Todos los pasos obligatorios se aprobaron (`result_code` `2000`). | Continúa tu proceso. |
| `REJECTED` | Una falla definitiva (por ejemplo, se alcanzó el límite de reintentos de una captura, o hubo demasiados códigos OTP incorrectos), o el usuario falló un paso y luego estuvo ausente durante 20 minutos. | No continúes. Ofrece una nueva verificación si corresponde. `last_failure` indica qué falló. |
| `REVIEW` | La verificación se completó pero Unicus la retiene para revisión manual (por ejemplo, una posible identidad duplicada, `2013`). | Trátala como pendiente. Espera `TRANSACTION_REVIEW_RESOLVED`. |
| `EXPIRED` | Se abrió y luego se abandonó durante 20 minutos sin una falla pendiente, o el enlace nunca se abrió en 24 horas (`finalized_by: NOT_STARTED`). `result_code` `6003`. | Crea una nueva transacción si el usuario regresa. |
| `CANCELLED` | El usuario canceló la verificación (`2041`). | Ofrece empezar de nuevo. |
| `DELETED` | La transacción fue eliminada. | — |

Mientras la transacción no sea final, un intento fallido **no** es final: el usuario puede
reintentar con el mismo enlace (hasta 10 intentos por captura). Una vez enviado el evento,
la transacción se cierra y su enlace deja de funcionar.

## `TRANSACTION_REVIEW_RESOLVED`

{% code overflow="wrap" %}
```json
{
  "meta": {
    "event": "TRANSACTION_REVIEW_RESOLVED",
    "event_id": "rev_6f1c2a4e-1b7d-4c8e-9a51-2f3b4c5d6e7f",
    "version": 2,
    "occurred_at": "2026-10-05T16:20:00Z",
    "ok": true,
    "code": 200
  },
  "data": {
    "tid": "6f1c2a4e-1b7d-4c8e-9a51-2f3b4c5d6e7f",
    "type": "ENROLLMENT",
    "previous_outcome": "REVIEW",
    "outcome": "APPROVED",
    "result_code": 2000,
    "result_message": "Success",
    "review_decision": "enroll",
    "resolved_at": "2026-10-05T16:20:00Z"
  }
}
```
{% endcode %}

| `review_decision` | `outcome` |
| --- | --- |
| `enroll` | `APPROVED` |
| `fraud`, `fail` | `REJECTED` |

## Pruebas de tu endpoint

* Usa el ambiente de pruebas (sandbox) con su propio Customer Token y su propia URL de webhook.
* Tu endpoint debe ser accesible desde internet. Para desarrollo local usa
  un túnel (por ejemplo ngrok o Cloudflare Tunnel) y registra su URL `https`
  en el ambiente de pruebas.
* Ejecuta una transacción hasta el final y otra que abandones: después de 20 minutos de
  inactividad recibes un evento `EXPIRED` (o `REJECTED`), que es el caso
  que la mayoría de las integraciones olvida.
* Si no llega nada, verifica en el portal que el webhook esté activo y que
  la URL responda `2xx` a un `POST` con un cuerpo JSON.
