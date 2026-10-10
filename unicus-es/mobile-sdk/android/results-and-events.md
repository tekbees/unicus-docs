---
description: >-
  El objeto UnicusVerificationResult, cómo manejar cada outcome, los eventos de
  progreso y los logs sanitizados del API en el SDK Android de Unicus.
---

# Resultados y eventos

## Dos tipos de respuesta

| Respuesta | Cuándo | Qué significa |
| --- | --- | --- |
| `onSuccess(UnicusVerificationResult)` | La verificación llegó a un final: verificado, rechazado, cancelado, abandonado o fallido durante el flujo. | Resultado de negocio. Lee `outcome` y `resultCode`. Un rechazo **no** es un error. |
| `onError(UnicusSdkException)` | La verificación no pudo iniciar o no pudo continuar por un motivo técnico o de configuración. | `error.code` es estable (`not_configured`, `transaction_refused`, `flow_not_supported`, `network_error`…). Consulta [Errores y solución de problemas](../errors-and-troubleshooting.md). |

Con las variantes `suspend`, `start` devuelve el resultado y lanza
`UnicusSdkException`.

## Campos del resultado

| Campo | Tipo | Descripción |
| --- | --- | --- |
| `outcome` | `UnicusVerificationOutcome` | `SUCCESS`, `WARNING`, `FAILED`, `CANCELED`, `ERROR`, `RESUMABLE`, `UNKNOWN`. Enruta tu app con él. |
| `resultCode` | `Int?` | Código de resultado de Unicus (`2000`, `2003`, `2041`, `2052`…). Consulta [Códigos de resultado](../result-codes.md). |
| `tid` | `String?` | Id de la transacción. Envíalo a tu backend para cruzarlo con el webhook. |
| `success` | `Boolean` | `true` solo para `SUCCESS`. |
| `isRetryable` | `Boolean` | `true` para `2054` y `4014`: inicia de nuevo. |
| `isResumable` | `Boolean` | `true` para `2003`: la transacción sigue abierta. |
| `resumed` | `Boolean` | `true` cuando este `start` continuó una transacción abierta de la misma persona. |
| `rejectionReason` | `String?` | Motivo legible por máquina de un `2052` (por ejemplo `STEP_OUT_OF_ORDER`, `SIGN_DECLINED`). Nunca un texto traducido. |
| `rejectionDetail` | `String?` | Detalle después del motivo, cuando Unicus lo envía. |
| `resultMessage` | `String?` | Mensaje de Unicus para el código (catálogo en español). Para logs; escribe tus propios textos para el usuario. |
| `flowId` | `String?` | Id del flujo ejecutado. |
| `steps` | `List<UnicusStepResult>` | Resultado de cada paso en el orden del flujo: `stepId`, `outcome` (`COMPLETED`, `FAILED`, `SKIPPED`), `resultCode`, `validations`. |
| `transactionResultCode` | `Int?` | Código final que reporta Unicus para la transacción, cuando respondió. |
| `sessionError` | `Boolean` | `true` cuando la sesión de cámara terminó por una interrupción técnica. |
| `raw` | `Map<String, Any?>` | Datos de diagnóstico (por ejemplo `raw["errorKey"]` de un error de una pantalla del flujo, `raw["leftOnStepId"]`). No dependas de sus llaves para enrutar. |

## Manejar cada outcome

{% tabs %}
{% tab title="Kotlin" %}
{% code overflow="wrap" %}
```kotlin
fun manejar(result: UnicusVerificationResult) {
    when (result.outcome) {
        UnicusVerificationOutcome.SUCCESS -> mostrarVerificado(result.tid)
        UnicusVerificationOutcome.WARNING -> mostrarEnRevision(result.tid)        // 2013
        UnicusVerificationOutcome.RESUMABLE -> mostrarContinuarLuego(result.tid)  // 2003
        UnicusVerificationOutcome.CANCELED -> mostrarEmpezarDeNuevo()             // 2041, 2051, 2061, 4001
        UnicusVerificationOutcome.FAILED -> mostrarNoVerificado(result.rejectionReason)
        UnicusVerificationOutcome.ERROR ->
            if (result.isRetryable) mostrarReintentar() else mostrarErrorTecnico(result.resultCode)
        UnicusVerificationOutcome.UNKNOWN -> mostrarErrorTecnico(result.resultCode)
    }
    result.tid?.let { backend.actualizarVerificacion(it) }   // el backend decide con el webhook
}
```
{% endcode %}
{% endtab %}

{% tab title="Java" %}
{% code overflow="wrap" %}
```java
void manejar(UnicusVerificationResult result) {
    switch (result.getOutcome()) {
        case SUCCESS: mostrarVerificado(result.getTid()); break;
        case WARNING: mostrarEnRevision(result.getTid()); break;
        case RESUMABLE: mostrarContinuarLuego(result.getTid()); break;
        case CANCELED: mostrarEmpezarDeNuevo(); break;
        case FAILED: mostrarNoVerificado(result.getRejectionReason()); break;
        case ERROR:
            if (result.isRetryable()) mostrarReintentar(); else mostrarErrorTecnico(result.getResultCode());
            break;
        default: mostrarErrorTecnico(result.getResultCode());
    }
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% hint style="warning" %}
Si usas un `when` exhaustivo, agrega una rama para cada valor. `RESUMABLE` se
agregó en esta versión; el código escrito contra una versión preliminar debe
manejarlo.
{% endhint %}

{% hint style="info" %}
Usa el resultado solo para la pantalla del usuario. Otorga el acceso desde tu
backend con el [webhook](../../sdk-web-v5/webhooks.md) o con
[Consultar el estado de una transacción](../../sdk-web-v5/transaction-status.md).
Qué mostrar en cada caso: [Resultados y reanudación](../results-and-resuming.md).
{% endhint %}

## Eventos

Los eventos de progreso sirven para registrar o mostrar el avance. Nunca traen
el resultado final: ese llega en el callback de `start`.

{% tabs %}
{% tab title="Kotlin" %}
{% code overflow="wrap" %}
```kotlin
UnicusSdk.shared.setEventListener { event ->
    when (event.name) {
        UnicusSdkEvent.STEP_STARTED -> analitica.paso(event.stepId, event.stepType)
        UnicusSdkEvent.STEP_FAILED -> analitica.pasoFallido(event.stepId, event.stepResult?.resultCode)
        UnicusSdkEvent.TRANSACTION_RESUMED -> Log.i("Unicus", "Retomada ${event.tid}")
        else -> Log.d("Unicus", "${event.name} ${event.message}")
    }
}
```
{% endcode %}
{% endtab %}

{% tab title="Java" %}
{% code overflow="wrap" %}
```java
UnicusSdk.getShared().setEventListener(event ->
    Log.d("Unicus", event.getName() + " " + event.getStepId() + " " + event.getMessage()));
```
{% endcode %}
{% endtab %}
{% endtabs %}

Pasa `null` para quitar el listener. Todos los eventos tienen `name`, `tid` y
`message`; los demás campos dependen del evento.

| `name` | Cuándo | Campos útiles |
| --- | --- | --- |
| `locationCollected` / `locationSkipped` | Ubicación opcional obtenida u omitida. | `message` |
| `transactionResumed` | `start` continuó una transacción abierta de la misma persona. | `tid` |
| `sessionPrepared` | Transacción y sesión listas; la primera pantalla está por abrirse. | `tid` |
| `segmentStarted` | Empieza un grupo de pasos. | `segmentKind` (`UI`, `BIOMETRIC`, `SERVER`), `raw["stepIds"]`, `raw["flowId"]` |
| `stepStarted` | Empieza un paso. | `stepId`, `stepType`, `raw["flowId"]` |
| `stepCompleted` / `stepFailed` | Terminó un paso. | `stepId`, `stepType`, `stepResult` (`outcome`, `resultCode`, `validations`) |
| `stepProgress` | Avance de la firma de documentos en modo Custom. | `raw["signStatus"]`, `raw["envelopeId"]`, `raw["reason"]` |
| `pageError` | Una pantalla del flujo muestra un error recuperable (red, falla temporal) con "reintentar". | `message` (llave del error) |
| `processRequest` | Se está enviando una captura biométrica a Unicus. | — |
| `livenessProcessed`, `enrollmentProcessed`, `authenticationComplete`, `idScanProcessed`, `idScanFrontRetry`, `idScanBackRequired`, `idScanBackRetry`, `idScanUserConfirmation`, `idScanComplete`, `idScanNfcFallback`, `enrollmentRetry`, `verificationRetry`, `serverError` | Unicus respondió una captura biométrica. | `response` (sanitizada) |
| `completed` | Unicus finalizó la transacción. | `response` |
| `nativeExit` | Se cerró la pantalla de cámara. | `message` (estado de salida) |
| `error` | Falló una solicitud a Unicus. | `message` |

## Logs del API

Mientras integras, puedes ver cada llamada que el SDK hace a Unicus:

{% code overflow="wrap" %}
```kotlin
UnicusSdk.shared.configure(
    UnicusSdkConfig(apiKey = token, environment = UnicusEnvironment.DEV).copy(enableApiLogging = true)
)
UnicusSdk.shared.setApiLogListener { entry ->
    Log.d("Unicus API", entry.summary)        // "/ruta -> 200 (123 ms)"
    Log.v("Unicus API", entry.toPrettyJson()) // solicitud y respuesta sanitizadas
}
```
{% endcode %}

Cada `UnicusApiLogEntry` tiene `timestamp` (UTC), `path`, `statusCode`,
`durationMillis`, `requestBody`, `responseBody`, `error` e `isSuccess`.

Las entradas vienen **sanitizadas**: datos biométricos, datos del documento y
del OCR, tokens de sesión, la referencia de la transacción, valores de
formularios, códigos y destinos OTP, imágenes y textos de firma, ubicación,
teléfono y correo se reemplazan por `<redacted>`, y los textos largos se
recortan (`apiLogStringLimit`). La llave de reanudación siempre se oculta.

{% hint style="danger" %}
Mantén `enableApiLogging` desactivado en los builds de producción y nunca
actives `includeSensitiveApiLogData` fuera de una sesión local de depuración.
Cuando envíes logs a [Soporte](../support.md), envíalos sanitizados.
{% endhint %}

Siguiente: [Pasos de UI personalizados](custom-ui-steps.md).
