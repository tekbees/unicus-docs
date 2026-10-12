---
description: >-
  Envía al usuario un enlace de verificación en lugar de integrar el botón:
  desde el portal administrativo por WhatsApp o SMS, o desde tu backend.
---

# Unicus Link

Unicus Link le envía a la persona un enlace de verificación. La persona lo
abre y hace la misma verificación de Web SDK 5.0 que con el botón: el flujo
asignado en el portal, los colores y el logo de tu empresa, y el traspaso al
celular cuando abre el enlace en un computador.

Úsalo cuando la persona no pasa por tu sitio web ni por tu aplicación web. Para
verificarla dentro de tu página, usa el [botón](quick-start.md).

{% hint style="info" %}
No combines Unicus Link y el botón para la misma persona y el mismo proceso.
El botón crea y abre su propia transacción; un enlace crea otra.
{% endhint %}

## Opción 1: desde el portal administrativo

En el [portal administrativo](https://app.idunicus.com/), abre
**Herramientas → Unicus Link**:

1. **¿Por dónde lo enviamos?** Elige WhatsApp o SMS. Solo aparecen los canales
   habilitados para tu empresa; si no ves ninguno, pídelo a
   [Soporte](support.md).
2. **Celular de la persona.** El número, con su indicativo de país. Solo se
   envía a números de los países activos de tu empresa.
3. **Documento de la persona.** El tipo (cédula, cédula de extranjería,
   pasaporte o licencia de conducción) y el número.
4. Selecciona **Enviar**.

Antes de enviarlo, el portal muestra **cómo lo recibe la persona**. El mensaje
de WhatsApp usa una plantilla aprobada por WhatsApp y no se puede
personalizar.

* El enlace tiene una vigencia corta y se usa una sola vez.
* La transacción aparece en el portal con las demás, y tu backend recibe el
  resultado por el [webhook](webhooks.md).
* Para enviar el enlace por otro canal (correo, tu propio chat) o elegir una
  variante del flujo, créalo desde tu backend (opción 2).

Para usar Unicus Link en el portal se necesita el permiso de Unicus Link en el
rol (ver
[Usuarios, roles y aprobaciones](../administrative-portal/users-roles-and-approvals.md)).

## Opción 2: desde tu backend

<mark style="color:green;">`POST`</mark> `<unicus-server-api-url>/init-api-transaction`

Llámalo solo desde tu servidor, con la [API key](server-api.md) de tu empresa.
Unicus devuelve el enlace y tú lo envías por el canal que prefieras.

### Encabezados

| Nombre | Obligatorio | Descripción |
| --- | --- | --- |
| `Authorization` | Sí | `Bearer <API_KEY>` del ambiente de destino. |
| `Content-Type` | Sí | `application/json` |

### Cuerpo

| Nombre | Obligatorio | Ejemplo | Descripción |
| --- | --- | --- | --- |
| `document` | Sí | `123456789` | Número de documento de la persona. |
| `documentType` | Sí | `ID` | Tipo de documento: `ID` (cédula), `FD` (cédula de extranjería), `PP` (pasaporte) o `DL` (licencia de conducción). |
| `flowId` | No | `credito-con-firma` | Identificador de una variante del flujo, en lugar del flujo por defecto. |

Consulta todos los campos (incluido cómo pedir un enlace nuevo para una
transacción abierta) en
[API de servidor y API keys](server-api.md#init-api-transaction).

### Ejemplo

{% code overflow="wrap" %}
```bash
curl --request POST '<unicus-server-api-url>/init-api-transaction' \
  --header 'Authorization: Bearer <API_KEY>' \
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
  "resultMessage": "Enrollment Transaction was created.",
  "tid": "<TID>",
  "url": "<enlace a la web app de Unicus>"
}
```
{% endcode %}

Envía el valor de `url` a la persona y guarda el `tid` junto a tu usuario: el
webhook y la consulta del resultado se refieren a él.

## Lo que ve la persona

* **En un celular o tableta,** toda la verificación ocurre en el navegador.
* **En un computador,** los pasos anteriores al primer paso con cámara ocurren
  ahí, y la persona continúa en el celular por código QR, WhatsApp o SMS (ver
  [Flujos y traspaso al celular](flows-and-handoff.md)).
* La barra de direcciones se limpia apenas se lee el enlace: el identificador
  de la transacción no queda en el historial del navegador.
* Si abre el enlace después de que la transacción terminó o expiró, ve "Este
  enlace ya no está activo".

## Cómo obtener el resultado

Un enlace no tiene una página tuya alrededor, así que no hay eventos
`OnUnicus:*` en el navegador. Tu backend recibe el resultado por el
[webhook](webhooks.md) (`TRANSACTION_FINALIZED`) o consultando
[el estado de la transacción](transaction-status.md) con el `tid`. En el
portal, la transacción aparece en
[Transacciones](../administrative-portal/transactions.md).

## Buenas prácticas

* El enlace identifica una transacción: envíalo solo a la persona que debe
  verificarse, y no lo publiques ni lo reutilices para otra persona.
* Si el tipo de transacción no tiene un flujo asignado, el enlace no se puede
  crear (código de resultado `2002`): asigna el flujo en el portal primero.
* Si la persona perdió el enlace o expiró, pide un enlace nuevo para la misma
  transacción (`reissueLink`) o crea uno nuevo.
