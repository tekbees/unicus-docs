---
description: >-
  Crea y publica los flujos de verificación de tu empresa, y elige cuál corre
  para cada tipo de transacción.
---

# Flujos de verificación

Un **flujo** es la secuencia de pasos que hace la persona: consentimiento,
instrucciones, prueba de vida, documento, comparación facial, código OTP,
formulario, firma… Los flujos se componen en **Plataforma → Flujos**, y el
botón, Unicus Link y los SDK móviles usan el flujo que asignes, sin cambiar tu
código. Para saber cómo se ve cada paso para la persona, consulta
[Flujos y traspaso al celular](../sdk-web-v5/flows-and-handoff.md).

## Casos de uso, flujo por defecto y variantes

Los flujos se agrupan por **caso de uso**: enrolamiento, verificación, prueba
de vida y estimación de edad.

* Cada caso de uso tiene un **flujo por defecto**: es el que corre cuando tu
  integración no pide uno específico.
* Puedes tener **variantes** (otro producto, una prueba A/B). Tu integración
  las pide por su identificador con `data-flow-id` en el botón o `flowId` en
  `init-api-transaction`.
* Sin flujo por defecto, ese tipo de transacción no se puede crear (código
  `2002`, "Validación no configurada").

Si tu empresa aún no tiene flujos, usa **Crear flujos por defecto**.

## Crear un flujo

1. **Nuevo flujo:** nombre interno, **identificador** (el que usará tu
   integración; no se puede cambiar después), caso de uso y una plantilla o
   en blanco.
2. Arrastra los pasos desde la paleta. El editor muestra qué pasos ocurren en
   el navegador y cuáles en el celular.
3. Selecciona un paso para editar sus textos (en español e inglés) y su
   configuración.
4. El editor valida el flujo mientras trabajas y te indica qué falta corregir.
   El borrador se guarda solo.

### Pasos disponibles

| Grupo | Pasos |
| --- | --- |
| Pantallas | Consentimiento (siempre al inicio, antes de cualquier paso con cámara), Información |
| Captura biométrica | Prueba de vida, Documento, Cotejo facial |
| Análisis del rostro | Verificación de edad |
| Complementos | Firma electrónica (firma manuscrita), Código OTP (SMS, WhatsApp o correo), Formulario, Firma con Unicus Sign (el paso "Firma de documento": firma un documento de Unicus Sign prellenado con los datos del flujo) |

Algunos pasos dependen de los servicios de tu plan (por ejemplo, el código OTP
o la firma con Unicus Sign); los que no tienes aparecen deshabilitados. La
firma con Unicus Sign necesita además que conectes tu cuenta de Unicus Sign
(ver [Integración](integration.md#conexiones)). Además
de los pasos, un flujo puede incluir **verificaciones** del lado del servidor
(por ejemplo, consultas en listas) si tu plan las incluye.

## Publicar

**Publicar versión** convierte el borrador en una versión nueva, con notas
opcionales. Una versión publicada no cambia: para modificarla, edita el
borrador y publica otra.

* Las transacciones nuevas usan la versión publicada; las que ya estaban en
  curso terminan con la que empezaron.
* **Versiones** muestra el historial: compara versiones o restaura una como
  borrador.

## Asignar flujos

**Asignación de flujos** define qué flujo corre en cada caso de uso:

* **Siempre la última publicada:** cada publicación aplica de inmediato.
* **Una versión fija:** la asignación no cambia hasta que la actualices. Útil
  para probar una versión nueva sin afectar a tus usuarios.

## Buenas prácticas

* Prueba cada versión nueva con [credenciales de prueba](../sdk-web-v5/customer-token.md)
  antes de asignarla a producción.
* Mantén los textos claros y cortos: se muestran en pantalla y en la lista de
  progreso (ver [Textos e idiomas](../sdk-web-v5/texts-and-languages.md)).
* Desde el listado puedes **duplicar** un flujo para crear una variante, o
  **exportarlo en JSON** como respaldo.
