---
description: >-
  Consulta enrolamientos, verificaciones y pruebas de vida, su detalle y su
  constancia en PDF; decide los posibles duplicados y exporta reportes.
---

# Transacciones y reportes

## Los listados

**Operación → Enrolamientos**, **Verificaciones**, **Pruebas de vida** y, si tu
empresa tiene la validación de edad, **Validación de edad** listan las
transacciones de tu empresa, las más recientes primero. Los enrolamientos se
actualizan en vivo.

Cada listado y su detalle necesitan el permiso de ver de su tipo de
transacción (por ejemplo, ver enrolamientos); cambiar o eliminar necesitan el
permiso de editar o eliminar de ese mismo tipo.

* Busca por número de documento o por nombre y apellido.
* Filtra por estado: exitoso, procesando, fallido, en revisión, sospechoso,
  eliminado o expirado.
* Elige el rango de fechas (hasta 90 días) y las columnas que quieres ver.
* **Ver pruebas** aparece si tu empresa tiene credenciales de prueba: muestra
  solo las transacciones de prueba (ver
  [Customer Token](../sdk-web-v5/customer-token.md)).

## El detalle de una transacción

Abre una transacción para ver, en pestañas (**Actualizar** vuelve a cargarla;
mientras la transacción está en curso, la página muestra **Actualizando en
vivo** y se actualiza sola):

| Pestaña | Qué muestra |
| --- | --- |
| **Resumen** | Resultado, estado, persona, documento, fechas y canal. |
| **Biometría** | Prueba de vida y comparación del rostro (con el documento o con el enrolamiento), frente al nivel mínimo de tu empresa. |
| **Documento** | El documento capturado, los datos leídos y sus validaciones. Si un paso de captura fue rechazado, el motivo. |
| **Flujo** | Cada paso del flujo con su resultado, en orden. |
| **Ubicación** | Dónde se hizo la verificación, de forma aproximada. |
| **Evidencia técnica** | Datos técnicos de la captura, para soporte. |

### Datos personales

Las fotos (rostro, documento, evidencias) y los datos personales solo los ve
completos quien tiene el permiso **Ver datos personales**. Sin él:

* siguen visibles el nombre, el tipo y número de documento, las fechas y los
  resultados;
* las fotos no se muestran;
* aparecen ocultos (`***` o enmascarados) la fecha y el lugar de nacimiento,
  el sexo, el tipo de sangre, la fecha y el lugar de expedición, y lo que la
  persona escribió en el flujo (formularios, teléfono o correo verificado con
  OTP);
* la ubicación se muestra aproximada (unos 10 km).

Cada consulta completa de datos personales queda registrada en
[Auditoría](users-roles-and-approvals.md).

### Constancia en PDF

**Descargar PDF** genera la constancia de la transacción (de enrolamiento, de
verificación de identidad o de prueba de vida). Unicus guarda la huella
(SHA-256) de cada constancia que genera, con quién y cuándo.

* Si tienes el permiso de datos personales, **Descargar PDF** abre un menú:
  **PDF completo** (con las imágenes y los datos personales) o **PDF sin datos
  personales** (para compartir con terceros).
* Sin el permiso, es un solo botón y la constancia ya sale sin ellos (y lo
  indica).

Para comprobar que una constancia que alguien te entregó es auténtica, usa
**Herramientas → Verificar constancia** y suelta el PDF: el archivo no sale de
tu navegador, solo su huella. El resultado es **Constancia registrada**,
**Documento modificado** o **Constancia no registrada**. Si la transacción
cambió después de generarse la constancia, el resultado lo advierte: "La
transacción cambió después de generarse esta constancia (por ejemplo, por una
decisión de revisión): el PDF ya no la muestra como está ahora."

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

Un enrolamiento (y su rostro de la búsqueda de duplicados) se elimina desde
su detalle con **Eliminar transacción** (se necesita el permiso de eliminar
enrolamientos). Las verificaciones solo se
eliminan desde tu backend con `delete-transaction`, que también sirve para
enrolamientos (ver [API de servidor](../sdk-web-v5/server-api.md)).
