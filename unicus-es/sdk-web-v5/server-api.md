---
description: >-
  Los endpoints que llama tu backend (crear un enlace, consultar, buscar por
  documento, eliminar) y la API key que exige cada uno.
---

# API de servidor y API keys

Tu **backend** puede llamar directamente cuatro endpoints de Unicus. Todos
exigen la **API key** de tu empresa:

| Endpoint | Qué hace |
| --- | --- |
| `init-api-transaction` | Crea una transacción y devuelve un enlace de verificación para enviarle a la persona. |
| `query-transaction` | Devuelve el estado y el resultado de una transacción por su `tid`. Consulta [Consultar el estado de una transacción](transaction-status.md). |
| `query-id` | Devuelve el enrolamiento de una persona por su documento. |
| `delete-transaction` | Elimina un enrolamiento (y su rostro de la búsqueda 1:N) o una verificación. |

{% hint style="danger" %}
**El Customer Token no abre estos endpoints.** El Customer Token es público:
va en tu página como `customerid` del botón. Una solicitud que envía solo
`X-Customer-ID` / `X-Device-ID` recibe `401`. Usa la API key, y solo desde tu
servidor: nunca la pongas en una página, en una app móvil ni en un repositorio.
{% endhint %}

El botón (`<unicus-btn>`) y los SDK móviles siguen funcionando con el Customer
Token: no usan estos endpoints.

## 1. Crea una API key

En el [portal administrativo](https://app.idunicus.com/), entra a la
configuración de tu compañía → **API keys** y crea una (requiere el rol ADMIN
de la compañía).

* La key tiene la forma `unk_<8 caracteres>_<32 caracteres>`.
* El portal la muestra **una sola vez**. Guárdala en el gestor de secretos o
  en las variables de entorno de tu backend.
* Cada ambiente (sandbox, producción) tiene sus propias keys.
* Para rotarla: crea una nueva, despliégala en tu backend y luego revoca la
  anterior. Una key revocada o vencida responde `401` de inmediato.

## 2. Envíala

Envía la key en el header `Authorization`, con `Content-Type:
application/json`:

{% code overflow="wrap" %}
```http
POST /query-transaction HTTP/1.1
Host: <unicus-server-api-url>
Authorization: Bearer unk_Ab12Cd34_0123456789abcdefghijABCDEFGHIJ
Content-Type: application/json

{ "tid": "<TID>" }
```
{% endcode %}

La key identifica a tu empresa: solo accedes a las transacciones de tu
empresa, y se ignora cualquier header `X-Customer-ID` / `X-Device-ID` que
envíes.

## `init-api-transaction`

Crea una transacción para una persona y devuelve un enlace de un solo uso a la
web app de Unicus. Úsalo cuando la persona no está en tu sitio web (por
ejemplo, para enviarle el enlace por correo o chat). Unicus enrola a la
persona si todavía no la conoce y la verifica si ya la conoce, con el flujo
asignado en el portal.

| Campo | Tipo | Obligatorio | Descripción |
| --- | --- | --- | --- |
| `document` | string | Sí* | Número de documento. |
| `documentType` | string | Sí* | Código del tipo de documento, como en el `clientid` del botón: `ID`, `FD`, `PP`, `DL`. |
| `flowId` | string | No | Slug de un flujo publicado de tu empresa para usarlo en lugar del asignado al tipo de transacción (`2002` si no es compatible). |
| `customParameter` | string | No | Solo para transacciones de estimación de edad, que se crean sin documento. |

\* Sin `document` se crea una transacción de estimación de edad.

{% code overflow="wrap" %}
```bash
curl --request POST '<unicus-server-api-url>/init-api-transaction' \
  --header 'Authorization: Bearer <API_KEY>' \
  --header 'Content-Type: application/json' \
  --data '{ "document": "123456789", "documentType": "ID" }'
```
{% endcode %}

```json
{
  "success": true,
  "resultCode": 0,
  "resultMessage": "Enrollment Transaction was created.",
  "tid": "<TID>",
  "url": "<enlace a la web app de Unicus>"
}
```

* Guarda el `tid` junto a tu usuario: el [webhook](webhooks.md) y
  `query-transaction` se refieren a él.
* La `url` lleva un token de un solo uso: cuando la persona la abre, el mismo
  enlace no vuelve a abrir. Si hace falta, envía uno nuevo con otra llamada.
* `resultMessage` indica si se creó un enrolamiento o una verificación.
* Si falla, `success` es `false` y `resultCode` / `resultMessage` dicen por qué
  (por ejemplo `2002`: no hay flujo asignado). Consulta
  [Códigos de resultado](result-codes.md).

## `query-transaction`

Estado y resultado de una transacción por su `tid`. Solicitud, campos de la
respuesta y uso recomendado: [Consultar el estado de una transacción](transaction-status.md).

## `query-id`

El enrolamiento de una persona, buscado por su documento en lugar de por `tid`.

| Campo | Tipo | Obligatorio | Descripción |
| --- | --- | --- | --- |
| `externalDatabaseRefID` | string | Sí | Número de documento. |
| `documentType` | string | Sí | Código del tipo de documento (`ID`, `FD`, `PP`, `DL`). |

{% code overflow="wrap" %}
```bash
curl --request POST '<unicus-server-api-url>/query-id' \
  --header 'Authorization: Bearer <API_KEY>' \
  --header 'Content-Type: application/json' \
  --data '{ "externalDatabaseRefID": "123456789", "documentType": "ID" }'
```
{% endcode %}

* **Enrolada** en tu empresa: `200` con el mismo cuerpo que devuelve
  `query-transaction` para su transacción de enrolamiento exitosa (datos de la
  persona, documento, imágenes).
* **No enrolada**, o el enrolamiento no fue exitoso: HTTP `404`.

{% hint style="warning" %}
La respuesta trae los datos de identidad y las imágenes de la persona.
Llámalo solo con una razón legítima de tu proceso y no registres el cuerpo en
tus logs.
{% endhint %}

## `delete-transaction`

Elimina una transacción de tu empresa: en un enrolamiento, la persona, su
resultado y su rostro en la búsqueda 1:N; en una verificación, su resultado.
La eliminación debe estar habilitada para tu empresa: solicítalo a Tekbees.

| Campo | Tipo | Obligatorio | Descripción |
| --- | --- | --- | --- |
| `tid` | string | Sí | Id de la transacción. |

{% code overflow="wrap" %}
```bash
curl --request POST '<unicus-server-api-url>/delete-transaction' \
  --header 'Authorization: Bearer <API_KEY>' \
  --header 'Content-Type: application/json' \
  --data '{ "tid": "<TID>" }'
```
{% endcode %}

* Eliminada: `success: true`, `resultCode` `2053`. Si la transacción seguía
  abierta, tu webhook recibe `TRANSACTION_FINALIZED` con `outcome` `DELETED`.
* No eliminada: `success: false` con el motivo en `resultMessage` (`tid`
  desconocido, eliminación no habilitada para tu empresa o un error temporal:
  reintenta más tarde).

## Errores

| HTTP | Cuándo | Qué hacer |
| --- | --- | --- |
| `401` | Sin API key, una key con formato incorrecto, revocada o vencida, o se envió solo el Customer Token. Viene con `WWW-Authenticate: Bearer`. | Revisa el header `Authorization` y el estado de la key en el portal. |
| `429` | Más de 600 solicitudes por minuto con tu key (valor por defecto). | Espera los segundos de `Retry-After`. |
| `400` | El cuerpo no es un JSON válido. | Envía un objeto JSON con los campos de arriba. |
| `5xx` | Falla temporal. | Reintenta con espera progresiva. |

Las respuestas de negocio (transacción no encontrada, no enrolada en
`query-transaction`, flujo no configurado) llegan con HTTP `200` y
`success: false`, excepto `query-id`, que responde `404` cuando la persona no
está enrolada.

## Lista de verificación

- [ ] API key creada por ambiente y guardada en tu gestor de secretos.
- [ ] Cada llamada de servidor envía `Authorization: Bearer <API_KEY>`.
- [ ] Ninguna API key en código de frontend, apps móviles, logs ni repositorios.
- [ ] Alertas de `401` en tu monitoreo (una key revocada o vencida).
- [ ] Un procedimiento de rotación: key nueva → despliegue → revocar la anterior.
