---
description: >-
  Outcomes de una verificación móvil, la diferencia entre salir y cancelar,
  cómo se retoma una transacción abierta y cómo confirmar el resultado en tu
  backend.
---

# Resultados y reanudación

## Resultado o error

`start` termina de una de dos formas:

| Final | Significado |
| --- | --- |
| **Resultado** (`UnicusVerificationResult`) | La verificación corrió. Incluye a una persona que no quedó verificada, una cancelación y una transacción abierta. Decide según `outcome`. |
| **Error** (`UnicusSdkException` / `UnicusSdkError`) | La verificación no pudo correr: configuración, red, transacción rechazada, un flujo que no puede correr en móvil. Decide según `error.code`. Consulta [Errores y solución de problemas](errors-and-troubleshooting.md). |

Un rechazo de negocio nunca es un error: llega como resultado.

## Outcomes

| Outcome | Códigos de resultado | Qué hacer |
| --- | --- | --- |
| `SUCCESS` | `2000` | Verificado. Envía el `tid` a tu backend y confírmalo allí. |
| `WARNING` | `2013` | Posible identidad duplicada: en revisión manual. Trátalo como pendiente; la decisión llega con el webhook `TRANSACTION_REVIEW_RESOLVED`. |
| `RESUMABLE` | `2003` | La transacción sigue abierta: el usuario salió, o aún hay pasos pendientes. Ofrece continuar; consulta [Retomar](#retomar). |
| `FAILED` | `2052`, `4011`, `9001`, `9002`, `9006`, `9010`, `9011`, `10001`, `10002`, `10020` y otros códigos menores que `9000` | No verificado. Muestra un mensaje de negocio; `rejectionReason` explica un `2052`. |
| `CANCELED` | `2041`, `2051`, `2061`, `4001` | Cancelado, caducado o sin intentos. Permite que el usuario empiece de nuevo. |
| `ERROR` | `2054`, `4012`–`4014`, los demás códigos `9xxx` y `10xxx` | Falla técnica. Cuando `isRetryable` es `true` (`2054`, `4014`) basta con reintentar. |
| `UNKNOWN` | ninguno | No se pudo determinar un código. Reintenta, o contacta a soporte con el `tid`. |

Todos los códigos están en [Códigos de resultado](result-codes.md). El código
nativo muestra los outcomes en el estilo de cada plataforma:
`UnicusVerificationOutcome.SUCCESS` en Android, `.success` en iOS y Flutter.


Campos útiles del resultado:

| Campo | Descripción |
| --- | --- |
| `tid` | Id de la transacción. Envíalo a tu backend. |
| `outcome`, `resultCode` | Ver arriba. |
| `isRetryable` | `true` para `2054` y `4014`: repetir es seguro. |
| `isResumable` | `true` para `2003`. |
| `resumed` | `true` cuando este `start` continuó una transacción abierta. |
| `rejectionReason`, `rejectionDetail` | Para `2052`: el motivo que envía Unicus (`STEP_OUT_OF_ORDER`, `FORM_INVALID`, `SIGN_DECLINED`...). |
| `flowId`, `steps` | El flujo que corrió y el resultado de cada paso (completado, fallido, omitido). |

Nombres exactos por plataforma: [Android](android/results-and-events.md),
[iOS](ios/results-and-events.md), [Flutter](flutter/results-and-events.md).

## Salir no es cancelar

| Qué pasa | Resultado | Transacción |
| --- | --- | --- |
| El usuario cierra una pantalla del flujo y confirma, presiona atrás, descarta la app desde las recientes, o el sistema cierra la pantalla. Un proveedor Custom llama a `cancel()`. | `2003` `RESUMABLE`, con el `tid` | Sigue abierta y se puede retomar. |
| El usuario cancela dentro de la pantalla de cámara. | `2041` `CANCELED` | Cancelada. |
| Tu app llama a `cancelActiveSession`. | `2041` `CANCELED` | Cancelada. |
| El flujo ya pasó su sesión de cámara cuando llega la cancelación. | `2003` `RESUMABLE` | Unicus la mantiene abierta: los pasos restantes aún se pueden completar. |

`cancelActiveSession` cierra las pantallas del SDK donde esté el usuario
(cámara, pantallas del flujo, tu pantalla Custom, entre pasos, o mientras `start`
todavía se prepara) y entrega primero el resultado al callback de `start`. Nunca
falla. Salir de una pantalla nunca envía una cancelación.

## Retomar

Una transacción sigue abierta en Unicus hasta que es final o hasta que pasan
**20 minutos sin actividad**. Mientras está abierta, el usuario puede continuar
en el primer paso pendiente. Los pasos completados no se repiten.

### Reanudación automática (recomendada)

La opción de configuración `resumeOpenTransactions` está activa por defecto.
Cuando el usuario sale, el siguiente `start` **con el mismo documento** continúa
la misma transacción:

1. Unicus devuelve una llave de reanudación al crear la transacción. El SDK la
   guarda cifrada en el dispositivo (Android Keystore, Llavero de iOS), ligada a
   tu Customer Token y al documento.
2. El siguiente `start` de la misma persona la envía. Si la transacción puede
   continuar (no es final, no expiró, mismo flujo), Unicus la retoma: mismo
   `tid`, el evento `transactionResumed` y `result.resumed == true`.
3. Si no, se crea una transacción nueva, sin error.

La llave se borra cuando la transacción llega a un resultado final y caduca en
el dispositivo a las 24 horas. Nunca aparece en resultados, eventos ni logs.

{% hint style="info" %}
Llama a `clearResumeData()` cuando el usuario cierre sesión en tu app, para que
otra persona en el mismo dispositivo nunca continúe esa transacción. En Android
5.0 y 5.1 (API 21–22) la llave no se guarda y cada `start` crea una transacción
nueva.
{% endhint %}

Pon `resumeOpenTransactions` en `false` si cada `start` debe crear una
transacción nueva.

### Retomar una transacción conocida

Para continuar una transacción específica, inicia con
`existingTransaction(tid)` y el `tid` del resultado `RESUMABLE`. El SDK continúa
en el primer paso pendiente. Funciona hasta que la transacción expira.

### Expiración

| Situación | Resultado |
| --- | --- |
| 20 minutos sin actividad después de un intento fallido. | La transacción termina `REJECTED` con el código de la última falla. |
| 20 minutos sin actividad en otro caso. | La transacción termina `EXPIRED` (`6003`). |
| Retomar por `tid` una transacción expirada o final. | Error `transaction_expired` (`2051`): inicia una nueva verificación. |

Los límites de intentos son los mismos de la web: consulta
[Final o con reintento](../sdk-web-v5/result-codes.md#final-o-con-reintento).

## Qué mostrar al usuario

| Outcome | Pantalla sugerida |
| --- | --- |
| `SUCCESS` | "Identidad verificada." Continúa tu proceso cuando tu backend lo confirme. |
| `WARNING` | "Estamos revisando tu información." Sin reintento. |
| `RESUMABLE` | "Tienes una verificación en curso." Botón **Continuar**, que vuelve a llamar a `start` con el mismo documento. |
| `FAILED` | "No pudimos verificar tu identidad", con el motivo para tu negocio. Ofrece un nuevo intento cuando tenga sentido. |
| `CANCELED` | "Verificación cancelada." Botón **Empezar de nuevo**. |
| `ERROR` | "Algo salió mal." Botón **Reintentar**; contacta a soporte si persiste. |

## Confirma en tu backend

El resultado en tu app es para la experiencia del usuario. Tu backend decide con
datos que vienen de Unicus:

* **Webhook.** Unicus envía un `TRANSACTION_FINALIZED` por transacción cuando
  es final, y `TRANSACTION_REVIEW_RESOLVED` después de una revisión manual.
  Consulta [Webhooks](../sdk-web-v5/webhooks.md).
* **Estado de la transacción.** Tu backend llama a
  [Consultar el estado de una transacción](../sdk-web-v5/transaction-status.md)
  con el `tid` que envió tu app.

Una transacción `RESUMABLE` aún no tiene webhook: el webhook llega cuando
termina o expira.
