---
description: >-
  Pantallas que el usuario puede ver cuando algo falla en Web SDK 5.0, qué
  significa cada una y cómo diagnosticar problemas de integración.
---

# Errores y solución de problemas

## Pantallas que ve el usuario

La aplicación web de Unicus muestra una pantalla de error con título, en el idioma del usuario, con un
código cuando aplica. La página del cliente recibe el mismo caso a través de
`OnUnicus:error` (pantallas de error; le sigue `OnUnicus:exit` cuando el usuario las
cierra) o de `OnUnicus:finished` (pantallas de resultado). Las pantallas de error que lo permiten muestran
un botón **Reintentar**; en el iframe también muestran **Cerrar**.

| El usuario ve | Código | Causa | Qué hacer |
| --- | --- | --- | --- |
| El enlace no es válido | — | La URL no tenía transacción ni token de traspaso al celular (hand-off). | Abre el flujo desde el botón o desde un enlace generado por Unicus. |
| La sesión expiró o ya fue usada | `2051` (o `401`) | La transacción expiró, ya se completó o se reabrió desde un enlace antiguo. | Crea una nueva transacción (cierra y vuelve a hacer clic en el botón). |
| El enlace expiró o ya fue usado | `2051` | El enlace de traspaso de un solo uso se abrió dos veces o después de expirar. | Genera un nuevo QR/SMS/WhatsApp desde el computador. |
| No pudimos continuar en este momento. Por favor intenta de nuevo | `2054` | Unicus no pudo procesar la solicitud en ese momento. El enlace y la transacción siguen siendo válidos. | El usuario presiona **Reintentar** en la misma pantalla. No se necesita una nueva transacción, QR ni mensaje. |
| Esta transacción no tiene un flujo de verificación configurado | `2002` | No hay flujo para esta empresa y tipo de transacción. Por lo general se detecta antes: el propio botón muestra "Verificación no configurada". | Asigna un flujo en el portal. |
| Esta verificación tiene una configuración que no reconocemos | — | El flujo contiene algo que esta versión de la aplicación web no puede ejecutar. | Revisa el flujo en el portal; contacta a soporte. |
| Este paso no pudo continuar | `2052` | El servidor rechazó un paso (por ejemplo, fuera de orden o no incluido en el flujo); se muestra el motivo. Sin botón Reintentar. | Por lo general el flujo cambió mientras la transacción estaba abierta: inicia una nueva transacción. |
| Demasiados intentos | `429` | Demasiadas solicitudes desde el mismo dispositivo en poco tiempo. | Espera un momento y presiona **Reintentar**. |
| Tu sesión no corresponde a esta verificación | `403` | La sesión pertenece a otra transacción. | Vuelve a abrir desde un enlace nuevo. |
| No pudimos conectarnos | — | Sin conexión, o una respuesta inesperada de Unicus. | Presiona **Reintentar**. |
| No pudimos iniciar la cámara segura | `9997`, `9004` | No se pudieron cargar los componentes de la cámara o el servicio biométrico no respondió. | Presiona **Reintentar**; revisa el navegador y la red. |
| Se denegó el permiso de cámara (pantalla de resultado) | `9996` | El usuario rechazó el acceso a la cámara. El flujo termina con una falla: `OnUnicus:finished` con `9996`. | Pide al usuario que permita la cámara en la configuración del navegador y que empiece de nuevo. |
| Gira tu celular a vertical para continuar (pantalla de cámara) | — | El celular se giró durante la captura. | Vuelve a ponerlo en vertical; la captura continúa. |
| No pudimos verificar tu identidad (pantalla de resultado) | código del paso | Un paso obligatorio falló, o el usuario canceló la cámara (`2041`). | Muestra al usuario una opción de reintento; el código indica el motivo. |
| En revisión manual | `2013` | Revisión manual pendiente. | Espera el webhook. |

## Diagnóstico de problemas de integración

| Síntoma | Revisa |
| --- | --- |
| El botón se queda gris con "Reintentar" | `OnUnicus:error` (y la consola del navegador) tiene la causa. Lo más común: el Customer Token pertenece a otro ambiente, `clientid` no tiene número, o el dominio de la API está bloqueado por la CSP (`connect-src`). |
| El botón dice "Verificación no configurada" | Código de resultado `2002`: asigna un flujo al tipo de transacción en el portal, o elimina un `data-flow-id` incorrecto. |
| El botón está gris neutro y nunca cambia | La transacción no se está creando. Confirma que `customerid` y `transactiontype` estén definidos cuando el elemento se conecta, y que el script se cargó (`customElements.get('unicus-btn')` está definido). `button.getAttribute('state')` te indica en qué estado está. |
| No pasa nada al hacer clic | Un clic durante `loading` abre el flujo cuando la transacción está lista (hasta 30 segundos en una red lenta). Si el estado es `error`, lee `OnUnicus:error`. Los clics se ignoran en `no_flow` y con el atributo `disabled`. Verifica que ninguna capa superpuesta de tu página intercepte el clic. |
| La página queda cubierta unos segundos y el botón vuelve a "Reintentar" | No se pudo cargar la verificación (`OnUnicus:error`: *the verification could not be loaded*). Revisa la CSP `frame-src`, los bloqueadores de anuncios o de privacidad, y la red. |
| El iframe está en blanco | La CSP `frame-src` debe permitir el dominio de la aplicación web de Unicus. Busca en la consola del navegador un frame rechazado. |
| La cámara nunca se abre | El sitio debe servirse por HTTPS; el iframe necesita el permiso `camera` (lo define el botón; un encabezado `Permissions-Policy` de la página padre que deniegue `camera` lo anula). En un computador la cámara nunca se usa: el flujo hace el traspaso a un celular. |
| Los eventos nunca se disparan | Los listeners deben registrarse en el elemento presente en el DOM (o en `document`, ya que los eventos se propagan). Con React, regístralos en `useEffect` mediante un `ref`. Los nombres de los eventos distinguen mayúsculas y minúsculas: `OnUnicus:finished`. |
| La verificación se cierra sola, sin `finished` ni `exit` | Tu página eliminó o volvió a crear el elemento `<unicus-btn>` (cambio de ruta, renderizado condicional, lista con nueva key). Mantenlo montado mientras la verificación esté abierta. |
| `OnUnicus:loaded` se dispara varias veces | Es lo esperado después de un cambio de `customerid`, `clientid`, `transactiontype` o `data-flow-id`, y en el primer clic después de `finished` / `exit`. Conserva siempre el `tid` más reciente. |
| `finished` llega con `success: false` y `resultCode: 2013` | No es un error: la transacción está en revisión manual. |
| El espejo en el computador deja de actualizarse | El computador también consulta la transacción a Unicus cada pocos segundos y muestra el resultado cuando Unicus lo tiene. En cualquier caso, el resultado queda en Unicus y llega a tu webhook. |
| El logo o los colores de la empresa no aparecen | La marca se configura en el portal en el mismo ambiente que el Customer Token. Los colores deben estar en hexadecimal (`#rrggbb`). |

## Log de diagnóstico

Las pantallas de verificación no muestran detalles técnicos al usuario. Cuando un
problema requiere un análisis más detallado, Tekbees puede habilitar un log de diagnóstico en el
ambiente de pruebas (sandbox): un panel en las pantallas de verificación con un registro con marca de tiempo de los
últimos pasos del flujo. Solicítalo a través de [Soporte](support.md).

## Qué enviar a soporte

* `tid` de la transacción.
* Versión del botón: `customElements.get('unicus-btn').version`.
* Navegador y dispositivo (user agent).
* El payload de `OnUnicus:error` o de `OnUnicus:finished`.
* El log de diagnóstico, cuando Tekbees lo haya habilitado para ti.
* Fecha y hora con zona horaria.
