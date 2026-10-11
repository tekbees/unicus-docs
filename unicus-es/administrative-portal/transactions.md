---
description: >-
  Consulta enrolamientos, verificaciones y pruebas de vida, su detalle y su
  constancia en PDF; decide los posibles duplicados y exporta reportes.
---

# Transacciones y reportes

## Los listados

**Operación → Enrolamientos**, **Verificaciones** y **Pruebas de vida** listan
las transacciones de tu empresa, las más recientes primero. Los enrolamientos
se actualizan en vivo.

* Busca por número de documento o por nombre y apellido.
* Filtra por estado: exitoso, procesando, fallido, en revisión, sospechoso,
  eliminado o expirado.
* Elige el rango de fechas (hasta 90 días) y las columnas que quieres ver.
* **Ver pruebas** aparece si tu empresa tiene credenciales de prueba: muestra
  solo las transacciones de prueba (ver
  [Customer Token](../sdk-web-v5/customer-token.md)).

## El detalle de una transacción

Abre una transacción para ver, en pestañas:

| Pestaña | Qué muestra |
| --- | --- |
| **Resumen** | Resultado, estado, persona, documento, fechas y canal. |
| **Biometría** | Prueba de vida y comparación del rostro (con el documento o con el enrolamiento), frente al nivel mínimo de tu empresa. |
| **Documento** | El documento capturado, los datos leídos y sus validaciones. Si un paso de captura fue rechazado, el motivo. |
| **Flujo** | Cada paso del flujo con su resultado, en orden. |
| **Ubicación** | Dónde se hizo la verificación, de forma aproximada. |
| **Evidencia técnica** | Datos técnicos de la captura, para soporte. |

### Datos personales

Las fotos (rostro, documento, evidencias) y los datos personales leídos del
documento solo los ve quien tiene el permiso **Ver datos personales**. Sin él,
esos datos aparecen ocultos (`***`) y la ubicación se muestra redondeada; el
nombre, el documento y los resultados siguen visibles.

Cada consulta completa de datos personales queda registrada en
[Auditoría](users-roles-and-approvals.md).

### Constancia en PDF

**Descargar PDF** genera la constancia de la transacción (de enrolamiento, de
verificación de identidad o de prueba de vida). Unicus guarda la huella
(SHA-256) de cada constancia que genera, con quién y cuándo. Si no tienes el
permiso de datos personales, la constancia los omite y lo indica.

Para comprobar que una constancia que alguien te entregó es auténtica, usa
**Herramientas → Verificar constancia** y suelta el PDF: el archivo no sale de
tu navegador, solo su huella. El resultado es **Constancia auténtica**,
**Documento modificado** o **Constancia no registrada**.

## Cola de revisión

Cuando un rostro nuevo coincide con otros enrolamientos de tu empresa, el
enrolamiento queda **en revisión** y aparece en **Operación → Cola de
revisión** (el menú muestra cuántos hay pendientes).

Para cada caso decides:

* **Enrolamiento válido:** es una persona distinta y el enrolamiento sigue.
* **Es la misma persona: suplantación:** el enrolamiento se rechaza.

Cada decisión exige una observación y no se puede deshacer desde el portal. Tu
backend recibe el resultado por el [webhook](../sdk-web-v5/webhooks.md). Para
decidir se necesita el permiso de editar enrolamientos.

## Reportes

**Operación → Reportes** busca las transacciones de un rango de hasta un mes y
las descarga en CSV: TID, resultado, estado, documento, nombre, número de
pruebas de vida y fecha.

{% hint style="info" %}
**Descargar** exporta la página de resultados que estás viendo. Para tu
conciliación o tus informes automáticos, consulta los resultados desde tu
backend con el [webhook](../sdk-web-v5/webhooks.md) o
[query-transaction](../sdk-web-v5/transaction-status.md).
{% endhint %}

## Validación de correo

**Herramientas → Validación de correo** lista las validaciones de correo de tu
empresa con su estado y, en el detalle, la información del dominio y del
proveedor de correo.

## Eliminar una transacción

El portal no elimina transacciones. Para eliminar un enrolamiento (y su rostro
de la búsqueda de duplicados) o una verificación, tu backend usa
`delete-transaction` (ver [API de servidor](../sdk-web-v5/server-api.md)).
