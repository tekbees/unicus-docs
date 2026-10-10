---
description: >-
  Lee el UnicusVerificationResult del SDK de Flutter, decide según el resultado
  y escucha los eventos de progreso y los logs sanitizados del API.
---

# Resultados y eventos

`start` siempre termina con un `UnicusVerificationResult` cuando la
verificación se ejecutó, también cuando la persona no quedó verificada. Lanza
`UnicusSdkException` solo cuando la verificación no pudo ejecutarse (consulta
[Errores y solución de problemas](../errors-and-troubleshooting.md)).

{% hint style="info" %}
El resultado en la app guía la experiencia del usuario. Para otorgar acceso,
confirma en tu backend con el [webhook](../../sdk-web-v5/webhooks.md) o
[Consultar el estado de una transacción](../../sdk-web-v5/transaction-status.md)
usando el `tid`.
{% endhint %}

## Maneja el resultado

{% code overflow="wrap" %}
```dart
Future<void> verify(UnicusVerificationRequest request) async {
  try {
    final result = await unicus.start(request);
    switch (result.outcome) {
      case UnicusVerificationOutcome.success:
      case UnicusVerificationOutcome.warning:
        await confirmInBackend(result.tid!); // 2000, o 2013 con advertencia
      case UnicusVerificationOutcome.resumable:
        showContinueLater(); // 2003: inicia de nuevo con el mismo documento
      case UnicusVerificationOutcome.failed:
        showNotVerified(result.resultCode, result.rejectionKind);
      case UnicusVerificationOutcome.canceled:
        showStartAgain(); // 2041 cancelada, 2051 caducada, 2061 intentos…
      case UnicusVerificationOutcome.error:
      case UnicusVerificationOutcome.unknown:
        showTryAgain(retryable: result.isRetryable);
    }
  } on UnicusSdkException catch (e) {
    showError(e.code, e.resultCode);
  }
}
```
{% endcode %}

`resumable` se agregó para 2003 (antes se reportaba como `warning`): un
`switch` exhaustivo debe incluirlo. El significado de cada código está en
[Códigos de resultado](../result-codes.md); qué mostrar al usuario, en
[Resultados y reanudación](../results-and-resuming.md).

## Campos del resultado

| Campo | Tipo | Significado |
| --- | --- | --- |
| `outcome` | `UnicusVerificationOutcome` | `success`, `warning`, `failed`, `canceled`, `error`, `resumable` o `unknown`. Decide con este campo. |
| `success` | `bool` | `true` solo para `success`. |
| `resultCode` | `int?` | Código de resultado de Unicus (2000, 2003, 2041, 2052…). La respuesta final del servidor tiene prioridad sobre la salida de la cámara. |
| `resultMessage` | `String?` | Mensaje del código. Para desarrolladores; no lo muestres tal cual. |
| `tid` | `String?` | Id de la transacción. Envíalo a tu backend. |
| `resumed` | `bool` | `true` cuando `start` continuó la transacción abierta del mismo documento. |
| `isResumable` | `bool` | `true` para 2003: la transacción sigue abierta y se puede continuar. |
| `resumeReason` | `String?` | Por qué sigue abierta: `userLeft` (el usuario salió), `flowResumable` (llegó una cancelación después de la sesión de cámara) o `transactionOpen` (faltan pasos). |
| `isRetryable` | `bool` | `true` para 2054 y 4014: no se procesó nada, inicia de nuevo. |
| `rejectionReason` / `rejectionDetail` | `String?` | Para un paso rechazado por el servidor (2052), el motivo legible por máquina (`STEP_OUT_OF_ORDER`…) y su detalle. |
| `rejectionKind` | `UnicusStepRejectionKind?` | `rejectionReason` tipado (`formInvalid`, `otpExpired`, `signDeclined`…). |
| `flowId` | `String?` | Flujo ejecutado; `null` en transacciones sin flujo modular. |
| `steps` | `List<UnicusStepResult>` | Resultado de cada paso del flujo: `stepId`, `outcome` (`completed`, `failed`, `skipped`), `resultCode`, `validations` del documento. |
| `transactionResultCode` | `int?` | Código final que reportó el estado de la transacción, cuando respondió. |
| `sessionError` | `bool` | La sesión de cámara terminó por una interrupción técnica. |
| `status`, `nativeStatusCode` | `String`, `int?` | Estado de salida de la pantalla de cámara, para diagnóstico. |
| `errorMessage`, `serverResponse`, `raw` | | Detalles de diagnóstico. No dependas de ellos para decidir. |

## Eventos

`unicus.events` es un `Stream<UnicusSdkEvent>` de difusión con el progreso de
la verificación en curso. Úsalo para analítica o un indicador de progreso,
nunca para la decisión final.

{% code overflow="wrap" %}
```dart
final subscription = unicus.events.listen((event) {
  switch (event.name) {
    case UnicusSdkEvent.stepStarted:
      analytics.log('unicus_step', {'type': event.stepType ?? ''});
    case UnicusSdkEvent.stepFailed:
      analytics.log('unicus_step_failed', {'code': '${event.stepResult?.resultCode}'});
    case UnicusSdkEvent.transactionResumed:
      analytics.log('unicus_resumed', {'tid': event.tid ?? ''});
  }
});
// subscription.cancel() cuando se destruya tu pantalla.
```
{% endcode %}

| Evento (`event.name`) | Cuándo | Campos útiles |
| --- | --- | --- |
| `sessionPrepared` | La transacción y su sesión están listas. | `tid` |
| `transactionResumed` | `start` continuó una transacción abierta del mismo documento. | `tid` |
| `locationCollected` / `locationSkipped` | Se obtuvo la ubicación, o se omitió (sin permiso, negada, apagada). | `message` |
| `segmentStarted` | Comienza un grupo de pasos. | `segmentKind` (`ui`, `biometric`, `server`), `flowId` |
| `stepStarted` | Comienza un paso del flujo. | `stepId`, `stepType`, `flowId` |
| `stepCompleted` | Un paso terminó bien o se omitió. | `stepResult` |
| `stepFailed` | Un paso falló. | `stepResult` (`resultCode`, `validations`) |
| `pageError` | Una pantalla del flujo muestra un error recuperable (red, 2054, 429) con "intentar de nuevo". | `message` (llave del error) |
| `stepProgress` | Progreso de un paso `sign_document`: `signStatus`, `envelopeId`, `reason`. | Lo emite solo el modo Custom nativo; las apps Flutter normalmente no lo reciben. |
| `processRequest`, `livenessProcessed`, `idScanProcessed` | Capturas de cámara enviadas a Unicus. | `response` |
| `completed` | Unicus finalizó la verificación. | `response`, `message` |
| `nativeExit` | Se cerró la pantalla de cámara. | `message` (estado de salida) |
| `error` | Error durante la sesión (luego llega el resultado o la excepción). | `message` |

Todos los eventos tienen además `flow`, `tid`, `message` y `raw`. Los valores
de `stepType` están en `UnicusStepType` (`consent`, `info`, `liveness`,
`document`, `face_match`, `signature`, `otp`, `form`, `age_check`,
`sign_document`). Los eventos nunca llevan la llave de reanudación ni tokens
de sesión.

## Logs del API

Con `enableApiLogging: true`, `unicus.apiLogs` emite un `UnicusApiLogEntry`
por cada llamada a Unicus: `timestamp`, `path`, `statusCode`, `duration`,
`requestBody`, `responseBody`, `error`, además de `isSuccess`, `summary` y
`toPrettyJson()`.

{% code overflow="wrap" %}
```dart
unicus.apiLogs.listen((entry) {
  if (!entry.isSuccess) debugPrint(entry.toPrettyJson());
});
```
{% endcode %}

El SDK oculta `requestBlob`, `responseBlob`, `response`, `documentData`,
`ocrResults`, `unicusSession` y `sessionToken` (`<redacted>`) y trunca los
textos largos (`apiLogStringLimit`). `includeSensitiveApiLogData: true`
desactiva el ocultamiento: solo en builds de depuración. Los logs aún pueden
contener el número de documento y el `tid`; no los envíes a servicios de
terceros. Cuando contactes a [Soporte](../support.md), comparte logs
capturados con el registro sensible desactivado.
