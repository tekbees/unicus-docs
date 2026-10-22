---
description: >-
  Dónde encontrar en el portal el ID de la compañía, crear API keys, configurar
  y firmar el webhook, y revisar sus entregas.
---

# Integración

**Configuración → Compañía → Integración** tiene lo que necesita tu equipo
técnico para conectarse a Unicus. La sección incluye una guía, **Cómo
integrarte con Unicus**, con el orden recomendado.

## ID de la compañía

El **ID de la compañía** es el [Customer Token](../sdk-web-v5/customer-token.md):
el valor de `customerid` en el botón y en los SDK móviles. Es público, puede ir
en tu página.

El portal muestra solo sus primeros caracteres. **Copiar ID** copia el ID
completo, y solo lo puede hacer el administrador de la compañía; a los demás
usuarios el portal les indica "Solo el administrador de la compañía puede
copiar el token". Cada copia queda registrada en
[Auditoría](users-roles-and-approvals.md#auditoria).

**Generar un ID nuevo** (solo el administrador) invalida el anterior de
inmediato: el botón y las apps que lo usan dejan de funcionar hasta que
publiques el nuevo. Hazlo solo si lo necesitas, y coordina el cambio con tu
equipo.

## API keys

Las API keys autorizan a tu **backend** a llamar los endpoints de servidor
(`init-api-transaction`, `query-transaction`, `query-id`,
`delete-transaction`). Ver [API de servidor y API keys](../sdk-web-v5/server-api.md).

* **Crear clave:** un nombre que identifique dónde la usarás y su vencimiento
  (nunca, 30, 90, 180 o 365 días). La key se muestra **una sola vez**: guárdala
  en el gestor de secretos de tu backend.
* **Revocar:** la key deja de funcionar de inmediato. Para rotarla, crea una
  nueva, despliégala y luego revoca la anterior.
* Las keys que crea tu empresa son de **producción**. Las credenciales de
  prueba las entrega Tekbees y aparecen marcadas como gestionadas por Tekbees.

Solo los administradores de la empresa ven y gestionan las API keys.

## Webhook

En **Resultados → Webhook** ingresa la URL `https://` de tu endpoint y
actívalo. Unicus le envía el resultado de cada transacción (ver
[Webhooks](../sdk-web-v5/webhooks.md)).

**Firma de webhooks** (administradores): **Generar secreto** crea el secreto
con el que se firma cada entrega. Se muestra **una sola vez**. También puedes
**rotarlo** o **desactivarlo**; los cambios aplican en menos de un minuto.

## Registro de entregas

**Plataforma → Webhooks** (administradores) muestra cada entrega de tu webhook
en los últimos días: entregadas, fallidas y en cola, con su respuesta.

* Busca una entrega por el TID de la transacción.
* **Reintentar** vuelve a enviar una entrega fallida, o **Reintentar
  fallidos** todas a la vez. Las entregas fallidas se pueden reenviar durante
  7 días.

## Conexiones

* **Unicus Sign:** para usar el paso de firma con Unicus Sign, conecta la API
  key de tu cuenta de Unicus Sign. Se guarda cifrada y puedes cambiarla o
  desconectarla.
