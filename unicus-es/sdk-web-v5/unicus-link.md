---
description: >-
  Envía al usuario un enlace de verificación en lugar de integrar el botón:
  créalo en el portal administrativo o desde tu backend.
---

# Unicus Link

Unicus Link crea un enlace de verificación que le envías al usuario por correo,
SMS, WhatsApp o cualquier otro canal. El usuario lo abre y hace la misma
verificación de Web SDK 5.0 que con el botón: el flujo asignado en el portal,
los colores y el logo de tu empresa, y el traspaso al celular cuando abre el
enlace en un computador.

Úsalo cuando el usuario no pasa por tu sitio web ni por tu aplicación web. Para
verificar al usuario dentro de tu página, usa el [botón](quick-start.md).

{% hint style="info" %}
No combines Unicus Link y el botón para el mismo usuario y el mismo proceso. El
botón crea y abre su propia transacción; un enlace crea otra.
{% endhint %}

## Opción 1: desde el portal administrativo

1. Inicia sesión en el [portal administrativo de Unicus](https://app.idunicus.com/).
2. Abre **Unicus Link** y selecciona **Generate Link**.
3. Ingresa los datos del documento del usuario.
4. Envía el enlace generado al usuario por el canal que prefieras.

<figure><img src="../.gitbook/assets/image (21).png" alt="Opción Generate Link en el portal administrativo" width="375"><figcaption></figcaption></figure>

## Opción 2: desde tu backend

<mark style="color:green;">`POST`</mark> `<unicus-server-api-url>/init-api-transaction`

Llámalo solo desde tu servidor: la petición lleva tu Customer Token.

### Encabezados

| Nombre | Obligatorio | Descripción |
| --- | --- | --- |
| `X-Customer-ID` | Sí | [Customer Token](customer-token.md) del ambiente de destino. |
| `Content-Type` | Sí | `application/json` |

### Cuerpo

| Nombre | Obligatorio | Ejemplo | Descripción |
| --- | --- | --- | --- |
| `document` | Sí | `123456789` | Número de documento del usuario. |
| `documentType` | Sí | `ID` | Tipo de documento: `ID`, `FD`, `PP` o `DL`. |

### Ejemplo

{% code overflow="wrap" %}
```bash
curl --request POST '<unicus-server-api-url>/init-api-transaction' \
  --header 'X-Customer-ID: <CUSTOMER_TOKEN>' \
  --header 'Content-Type: application/json' \
  --data '{ "document": "123456789", "documentType": "ID" }'
```
{% endcode %}

### Respuesta

{% code overflow="wrap" %}
```json
{
  "success": true,
  "resultCode": 0,
  "resultMessage": "A Transaction Enrollment was created.",
  "url": "https://id.idunicus.com/?token=<TID>&process=enrollment"
}
```
{% endcode %}

Envía el valor de `url` al usuario. El `tid` que contiene es el identificador
de la transacción: guárdalo para relacionar el webhook y consultar el
resultado.

## Lo que ve el usuario

* **En un celular o tableta,** toda la verificación ocurre en el navegador.
* **En un computador,** los pasos anteriores al primer paso con cámara ocurren
  ahí, y el usuario continúa en el celular por código QR, WhatsApp o SMS (ver
  [Flujos y traspaso al celular](flows-and-handoff.md)).
* La barra de direcciones se limpia apenas se lee el enlace: el identificador
  de la transacción no queda en el historial del navegador.
* Si el usuario abre el enlace después de que la transacción terminó o expiró,
  ve "La sesión expiró o ya fue utilizada".

## Cómo obtener el resultado

Un enlace no tiene una página tuya alrededor, así que no hay eventos
`OnUnicus:*` en el navegador. Tu backend recibe el resultado por el
[webhook](webhooks.md) (`TRANSACTION_FINALIZED`) o consultando
[el estado de la transacción](transaction-status.md) con el `tid`.

## Buenas prácticas

* El enlace identifica una transacción: envíalo solo a la persona que debe
  verificarse, y no lo publiques ni lo reutilices para otra persona.
* Si el tipo de transacción no tiene un flujo asignado, el enlace no se puede
  usar (código de resultado `2002`): asigna el flujo en el portal primero.
* Si el usuario perdió el enlace o expiró, crea uno nuevo.
