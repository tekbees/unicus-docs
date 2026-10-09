---
description: >-
  Códigos de resultado que se devuelven en los eventos de Web SDK 5.0, el
  webhook final y el endpoint query-transaction: qué significa cada uno, si es
  final y cómo se relaciona con el resultado (outcome) del webhook.
---

# Códigos de resultado

El mismo número significa lo mismo en todos los lugares donde aparece:

| Dónde | Campo |
| --- | --- |
| Eventos | `OnUnicus:finished` y `OnUnicus:details` (`state.resultCode`), `OnUnicus:error` (`resultCode`). |
| Webhook final | `data.result_code` (el código que finalizó la transacción) y `data.last_failure.code` (el último intento fallido). |
| `query-transaction` | `transactionStatusId` (último código registrado en la transacción) y `flowSteps[].resultCode`. El `resultCode` de primer nivel de esa respuesta no es un código de resultado: es `0` o `-1`; consulta [Consultar el estado de una transacción](transaction-status.md). |

## Final o con reintento

Una falla **no** finaliza la transacción por sí sola. El usuario puede volver a
intentarlo con el mismo enlace hasta que ocurra una de estas situaciones:

| Límite | Resultado |
| --- | --- |
| Un código listado como **final** más abajo. | La transacción finaliza de inmediato. |
| 10 intentos fallidos de rostro (prueba de vida o verificación). | Finaliza `REJECTED` con el código de la última falla. |
| 10 intentos fallidos de coincidencia con el documento. | Finaliza `REJECTED` con el código de la última falla. |
| 10 intentos en un lado del documento (frente o reverso). | Finaliza `REJECTED` con `2061`. |
| 20 minutos sin actividad después de que el usuario abrió el enlace. | Finaliza `REJECTED` con el código de la última falla, o `EXPIRED` (`6003`) cuando lo último que ocurrió no fue una falla. |
| El enlace nunca se abrió en 24 horas. | Finaliza `EXPIRED` (`6003`). |

Los errores de Unicus o de un proveedor (`5001`, `5002`, `5004`, `5005`, `7003`,
`7004`, `7005`, `8003`–`8006`, `9004`, `9005`, `9007`, `10010`, `10011`) nunca
consumen un intento. Una vez que la transacción es final, su enlace deja de
funcionar (`2051`) y Unicus envía el webhook final.

## Resultado del webhook

`TRANSACTION_FINALIZED` incluye un `outcome` y un `result_code`:

| `outcome` | `result_code` | Cuándo |
| --- | --- | --- |
| `APPROVED` | `2000` | La última etapa fue exitosa, o se completó el último paso obligatorio del flujo. |
| `REVIEW` | `2013` | Posible identidad duplicada: queda en espera de revisión manual. A continuación llega `TRANSACTION_REVIEW_RESOLVED` con `APPROVED` (`2000`) o `REJECTED` (`2013`, o `7007` cuando se marca como fraude, `8002` cuando se rechaza). |
| `REJECTED` | Un código final de los listados abajo, o la última falla | Un código final, un límite de intentos, o 20 minutos de inactividad después de una falla. |
| `EXPIRED` | `6003` | Abierta e inactiva durante 20 minutos sin una falla pendiente (`finalized_by: INACTIVITY`), o nunca abierta en 24 horas (`finalized_by: NOT_STARTED`). |
| `CANCELLED` | `2041` | El usuario canceló la verificación. |
| `DELETED` | `2053` | La transacción se eliminó con `delete-transaction`. |

## Códigos finales

| Código | Significado | Resultado |
| --- | --- | --- |
| `2000` | Todos los pasos obligatorios fueron aprobados. Los flujos heredados pueden informar `0` o `200` en los eventos. | `APPROVED` |
| `2013` | El rostro se encontró en la búsqueda 1:N de la empresa (posible duplicado). Trátalo como pendiente, no como una falla; la decisión llega con `TRANSACTION_REVIEW_RESOLVED`. | `REVIEW` |
| `2011` | La persona ya está enrolada. | `REJECTED` |
| `2012` | La persona no está enrolada (verificación, o un flujo que compara contra un enrolamiento). También se devuelve al crear la transacción. | `REJECTED` |
| `2014` | El rostro se encontró en el grupo 1:N de fraude. | `REJECTED` |
| `2061` | Se agotaron los intentos en un lado del documento. | `REJECTED` |
| `4001` | El paso OTP se quedó sin intentos de verificación. | `REJECTED` |
| `9010` | Paso `age_check`: no se certifica que el rostro tenga al menos la edad mínima del flujo. | `REJECTED` |
| `9011` | Paso `age_check`: no fue posible estimar la edad a partir del rostro. | `REJECTED` |
| `10020` | La persona no fue verificada debido a una restricción de edad. | `REJECTED` |
| `1001`, `7001`, `7002`, `8001`–`8006`, `6009` | Una de las validaciones de documento del flujo falló al completarse la captura del documento (ver abajo). | `REJECTED` |
| `2041` | El usuario canceló la sesión de cámara. Cerrar o salir de la página no cancela. | `CANCELLED` |
| `2053` | La transacción fue eliminada. | `DELETED` |
| `6003` | La transacción expiró por inactividad o nunca se inició. También se usa para un tiempo de espera agotado de un servicio durante un paso, que admite reintento. | `EXPIRED` |

## Creación de la transacción

Los devuelve el botón en `OnUnicus:error` cuando no es posible crear la
transacción.

| Código | Significado | Solución |
| --- | --- | --- |
| `2002` | No hay un flujo válido para la empresa y el tipo de transacción, `data-flow-id` desconocido, o un flujo no compatible con el tipo de transacción. El motivo está en `resultMessage`. | Asigna o publica el flujo en el portal administrativo. |
| `2012` | El flujo compara el rostro contra un enrolamiento y la persona no está enrolada. | Enrola primero a la persona. |
| `2052` | Unicus se negó a crear la transacción. El botón lo informa como `user is currently blocked`. | Revisa a la persona en el portal o contacta a soporte. |

## Sesión y enlace

| Código | Significado | Qué hacer |
| --- | --- | --- |
| `2051` | La transacción no se encontró, ya es final o el enlace expiró. | Crea una nueva transacción. |
| `2054` | Falla temporal en Unicus. No se consumió nada: el mismo enlace y la misma transacción siguen siendo válidos. | La aplicación web ofrece **Retry** (Reintentar). No crees una nueva transacción; si persiste, contacta a soporte. |
| `2003` | Los pasos de cámara terminaron, pero el flujo tiene más pasos obligatorios (firma, OTP, formulario). Se ve en los resultados de los pasos, nunca como código final. | Nada; el flujo continúa. |

## Paso rechazado por el servidor (`2052` durante el flujo)

Mientras el flujo se ejecuta, `2052` significa que la API de Unicus rechazó un
paso. El motivo se puede leer de forma programática en `resultMessage` (`REASON`
o `REASON:detail`) y la aplicación web le muestra al usuario un mensaje
apropiado. Motivos principales:

| `resultMessage` | Significado |
| --- | --- |
| `STEP_OUT_OF_ORDER:<stepIds>` | Se envió un paso antes de los pasos obligatorios previos. El cliente retoma en el paso correcto. |
| `STEP_NOT_IN_FLOW:<stepIds>` / `STEP_TYPE_MISMATCH:<stepId>` | El paso no pertenece al flujo de la transacción o es de otro tipo (cliente desactualizado). |
| `FLOW_ALREADY_FAILED:<stepId>` | Un paso obligatorio ya falló; la transacción no puede continuar. |
| `NO_FLOW` | La transacción no tiene un flujo almacenado. |
| `FORM_INVALID:<problems>` | Formulario rechazado. Cada problema es `<KIND>:<field>`, con `KIND` igual a `REQUIRED`, `INVALID_TYPE`, `PATTERN_MISMATCH` o `UNKNOWN_FIELD`. El usuario ve los errores junto a los campos. |
| `SIGNATURE_INVALID:<what>` | Firma ausente o no válida. |
| `CONSENT_INVALID` / `CONSENT_NOT_RECORDED` | La pantalla de consentimiento no se registró correctamente. |
| `OTP_INVALID_CODE` | Código incorrecto; el usuario puede reintentar hasta `4001`. |
| `OTP_EXPIRED` | El código expiró (5 minutos); se solicita uno nuevo. |
| `OTP_RESEND_TOO_SOON:<seconds>` / `OTP_SEND_LIMIT` | Es demasiado pronto para reenviar, o no se permiten más envíos. |
| `OTP_SEND_FAILED:<channel>:<reason>` | No fue posible enviar el código por ese canal. `reason` es `NO_OTP_TEMPLATE`, `OTP_TEMPLATE_INVALID:<code>` (configuración) o `PROVIDER_ERROR` (proveedor). |

## Códigos de cámara y documento

Se informan en los resultados de los pasos de `OnUnicus:details`, como
`last_failure.code` en el webhook y como código final cuando finalizan la
transacción. Salvo que aparezcan en la sección **Códigos finales**, el
usuario puede reintentar dentro de los límites indicados arriba.

| Código | Descripción |
| --- | --- |
| `1001` | El número de documento leído del documento de identidad no coincide con el de la transacción (`clientid`), o no se pudo comparar (validación del flujo `id_number_match`). |
| `5003` | No se pudo confirmar la prueba de vida. |
| `5001` / `5002` | No se pudo guardar la selfie o la imagen de auditoría. |
| `5004` / `5005` | Error interno o de comunicación al procesar la selfie. |
| `6003` | Un servicio no respondió a tiempo (por ejemplo, el clasificador de documentos). |
| `6004` | Tipo de documento no soportado: el documento no coincidió con ninguna plantilla conocida, o su país o tipo no está habilitado. |
| `6009` | El clasificador de documentos rechazó el documento. |
| `7001` / `7002` | Frente / reverso del documento con baja calidad o ilegible (para cédulas colombianas, también un número que no está compuesto solo por dígitos o una fecha de expedición ilegible). |
| `7003` / `7004` / `7005` | Falló la lectura del documento: tiempo de espera agotado, no se pudieron leer los datos, error de comunicación. |
| `7006` | El documento no superó las verificaciones de autenticidad: no se pudo confirmar que fuera un documento físico (suplantación digital), el documento no estaba completamente visible, no se pudo confirmar el rostro o el texto del documento, posible fotocopia, documento vencido o medio inesperado (solo para empresas con esa verificación habilitada). También se devuelve cuando el país o el tipo del documento no se acepta para la empresa. |
| `7007` | Documento marcado como fraude en la revisión manual de un duplicado. |
| `8001` | Identidad no encontrada en el registro oficial. |
| `8002` | No verificado por el registro oficial (por ejemplo, un documento que ya no es válido). |
| `8003` | El registro oficial no respondió a tiempo. |
| `8004` / `8005` / `8006` | Error del registro oficial, error de comunicación, o no disponible para ese país. |
| `9001` | El rostro no coincide con la foto del documento (nivel de coincidencia por debajo del mínimo). |
| `9002` | El rostro no coincide con el rostro enrolado. |
| `9004` / `9005` | Error interno o de comunicación del motor biométrico. |
| `9007` | Error interno al procesar la captura. |
| `10010` / `10011` | Error de comunicación o interno del clasificador de documentos. |

## Códigos solo de la aplicación web

La aplicación web los informa en los eventos cuando la sesión de cámara termina
antes de tiempo. Nunca llegan al webhook ni a `query-transaction`.

| Código | Descripción |
| --- | --- |
| `9991` | La sesión de cámara se interrumpió (por ejemplo, por un error de red). |
| `9994` | Bloqueo después de demasiados intentos fallidos en la sesión de cámara. |
| `9995` | Error de cámara. |
| `9996` | Permiso de cámara denegado. |
| `9997` | Error inesperado, o el componente de cámara no inició a tiempo. |
| `9998` | La cámara no puede ejecutarse dentro de un frame sin permiso. |
| `500` | Error genérico cuando no se devolvió ningún código. |

Cuando el usuario cancela el escaneo del rostro o del documento, los eventos
informan `2041`.

## Niveles de coincidencia y grupos de edad

Los resultados de los pasos y los webhooks pueden incluir un `matchLevel` y un
`ageEstimateGroup`. Los niveles de coincidencia y sus tasas de falsa aceptación
los define FaceTec, el motor biométrico.

**Rostro contra el rostro enrolado** (verificación): niveles `0` a `15`.

| Nivel | Tasa de falsa aceptación |
| --- | --- |
| `15` | 1 en 125,000,000 |
| `14` | 1 en 95,000,000 |
| `13` | 1 en 70,000,000 |
| `12` | 1 en 50,000,000 |
| `11` | 1 en 25,000,000 |
| `10` | 1 en 12,800,000 |
| `9` | 1 en 2,000,000 |
| `8` | 1 en 1,000,000 |
| `7` | 1 en 500,000 |
| `6` | 1 en 100,000 |
| `5` | 1 en 10,000 |
| `4` | 1 en 1,000 |
| `3` | 1 en 500 |
| `2` | 1 en 250 |
| `1` | 1 en 100 |
| `0` | Sin coincidencia |

**Rostro contra la foto del documento** (enrolamiento): niveles `0` a `7`, con
las mismas tasas de la tabla anterior para cada nivel. Los rechazos varían según
las características de seguridad del documento y la antigüedad de su foto. Un
nivel por debajo del mínimo configurado para la empresa devuelve `9001`.

**Grupo de edad** (`ageEstimateGroup`; en transacciones de prueba de vida,
`ageEstimateGroupV2`). Cada grupo significa una confianza de 99.5 % o más de que
la persona tiene al menos esa edad:

| Valor | Significado |
| --- | --- |
| `0` | No disponible (no se pudo estimar la edad) |
| `1` | Fuera de cualquier rango válido (menor de 8) |
| `2` | 8 o más |
| `3` | 13 o más |
| `4` | 16 o más |
| `5` | 18 o más |
| `6` | 21 o más |
| `7` | 25 o más |
| `8` | 30 o más |
