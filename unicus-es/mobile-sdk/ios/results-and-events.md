---
description: >-
  El resultado de la verificación en iOS (UnicusVerificationResult), los errores
  (UnicusSdkError), los eventos de progreso y los logs sanitizados de la API.
---

# Resultados y eventos

`start` termina exactamente una vez con un `Result` de Swift:

* `.success(UnicusVerificationResult)`: la verificación se ejecutó. La persona
  puede haber quedado verificada o no: revisa `outcome`.
* `.failure(UnicusSdkError)`: la verificación no se pudo ejecutar
  (configuración, red, flujo no soportado…).

## Maneja el resultado

{% code overflow="wrap" %}
```swift
func handle(_ result: UnicusVerificationResult) {
    switch result.outcome {
    case .success:   approve(tid: result.tid)        // confirma en tu backend
    case .resumable: showPending()                   // 2003: inicia de nuevo con el mismo documento
    case .warning:   sendToReview(tid: result.tid)   // 2013
    case .failed:    showRejected(code: result.resultCode, reason: result.rejectionReason)
    case .canceled:  showStartAgain()                // 2041, 2051, reintentos agotados
    case .error:     result.isRetryable ? showTryAgain() : showSupport()
    case .unknown:   showSupport()
    @unknown default: showSupport()
    }
}
```
{% endcode %}

{% hint style="info" %}
Conserva `@unknown default`: el SDK es un framework binario y se pueden agregar
nuevos resultados. En el modo de lenguaje Swift 6 un `switch` sin él no compila.
{% endhint %}

El resultado en la app es para la interfaz. Decide el acceso en tu backend con
el [webhook](../../sdk-web-v5/webhooks.md) o con
[Consultar el estado de una transacción](../../sdk-web-v5/transaction-status.md)
para el `tid`. Significado de cada código:
[Códigos de resultado](../result-codes.md).

## `UnicusVerificationResult`

| Campo | Tipo | Significado |
| --- | --- | --- |
| `outcome` | `UnicusVerificationOutcome` | `.success`, `.warning`, `.failed`, `.canceled`, `.error`, `.resumable`, `.unknown`. Decide según este campo. |
| `success` | `Bool` | `outcome == .success`. |
| `tid` | `String?` | Id de la transacción de Unicus. Guárdalo con tu usuario o caso. |
| `resultCode` | `Int?` | Código de resultado de Unicus (2000, 2003, 2041…). |
| `resultMessage` | `String?` | Mensaje en español del código, o el mensaje del servidor. Para logs; muestra tus propios textos. |
| `isResumable` | `Bool` | 2003: la transacción está abierta y se puede continuar. |
| `isRetryable` | `Bool` | 2054 o 4014: falla temporal, inicia de nuevo. |
| `resumed` | `Bool` | `start` continuó una transacción abierta en lugar de crear una. |
| `rejectionReason` / `rejectionDetail` | `String?` | Para 2052 (paso rechazado): el motivo legible por máquina que envió el servidor (por ejemplo `STEP_OUT_OF_ORDER`). |
| `flowId` | `String?` | Flujo ejecutado; `nil` sin flujo modular. |
| `steps` | `[UnicusStepResult]` | Por paso: `stepId`, `outcome` (`completed`, `failed`, `skipped`), `resultCode`, `validations` del documento. |
| `transactionResultCode` | `Int?` | Código final reportado por Unicus para la transacción, cuando se conoce. |
| `sessionError` | `Bool` | La sesión terminó por un problema técnico. |
| `errorMessage` | `String?` | Detalle técnico, para logs. |
| `raw` | `[String: Any]` | Todos los campos como diccionario (también `resumeReason`: `userLeft`, `flowResumable`, `transactionOpen`). |

`UnicusResultCode` tiene las constantes (`success`, `resumable`, `userCanceled`,
`stepRejected`, `temporaryFailure`, `signDeclined`…) y
`UnicusResultCode.messageFor(code)` devuelve el mensaje en español de un código.

## `UnicusSdkError`

| Campo | Significado |
| --- | --- |
| `code` | Identificador estable: decide según él (`not_configured`, `environment_not_available`, `session_active`, `transaction_refused`, `network_error`, `flow_not_supported`, `no_view_controller`…). |
| `message` | Texto para desarrolladores. No lo muestres al usuario final. |
| `resultCode` / `resultMessage` | Valores de Unicus cuando el servidor respondió (por ejemplo `transaction_refused` con 2002: sin flujo asignado). |
| `httpStatus` | Estado HTTP, cuando el error vino de una respuesta HTTP. |
| `details` | Datos adicionales (por ejemplo, los valores que faltan en `environment_not_available`). |

Todos los códigos con su solución:
[Errores y solución de problemas](../errors-and-troubleshooting.md).

## Eventos de progreso

{% code overflow="wrap" %}
```swift
UnicusSdk.shared.eventHandler = { event in
    switch event.name {
    case UnicusSdkEvent.transactionResumed:
        print("continuando", event.tid ?? "")
    case UnicusSdkEvent.stepStarted, UnicusSdkEvent.stepCompleted, UnicusSdkEvent.stepFailed:
        print(event.name, event.stepId ?? "", event.raw["stepType"] ?? "")
    default:
        print(event.name, event.message ?? "")
    }
}
```
{% endcode %}

Los eventos son informativos: nunca decidas el resultado con ellos. Llegan en la
cola principal. `UnicusSdkEvent` tiene `name`, `tid`, `message`, `response`
(respuesta del servidor ya interpretada, cuando existe), `raw`, además de
`stepId` y `segmentKind`.

| Evento | Cuándo |
| --- | --- |
| `locationCollected` / `locationSkipped` | Ubicación opcional obtenida u omitida. |
| `transactionResumed` | `start` continuó la transacción abierta del mismo documento (mismo `tid`). |
| `sessionPrepared` | Transacción y sesión listas; las pantallas están por abrirse. |
| `segmentStarted` | Inicia un grupo de pasos; `event.segmentKind`: `.ui`, `.biometric`, `.server`. |
| `stepStarted` / `stepCompleted` / `stepFailed` | Cambió un paso del flujo; `event.stepId`, `raw`: `flowId`, `stepType`, `resultCode`, `validations` (paso de documento), `templateName` (`sign_document`). |
| `stepProgress` | `sign_document` en modo personalizado: cambió el estado de la firma (`raw["signStatus"]`, `envelopeId`, `reason`). |
| `pageError` | Una pantalla del flujo mostró un error no fatal con su propio reintento. |
| `processRequest` | Se está enviando una captura cifrada a Unicus. |
| `livenessProcessed`, `enrollmentProcessed`, `authenticationComplete`, `idScanProcessed` | Unicus procesó una captura. |
| `idScanFrontRetry`, `idScanBackRequired`, `idScanBackRetry`, `idScanUserConfirmation`, `idScanComplete`, `idScanNfcFallback` | Avance de la captura del documento. |
| `enrollmentRetry`, `verificationRetry`, `serverError` | Hay que repetir la captura, o el servidor respondió un error. |
| `completed` | Unicus finalizó la transacción. |
| `nativeExit` | Se cerró la pantalla de cámara (`event.message`: su estado). |
| `error` | Falló una solicitud a Unicus. |

## Logs de la API

Durante el desarrollo, recibe un log sanitizado de cada llamada a la API de
Unicus:

{% code overflow="wrap" %}
```swift
let config = UnicusSdkConfig(apiKey: "<CUSTOMER_TOKEN>", environment: .dev, enableApiLogging: true)
UnicusSdk.shared.configure(config)
UnicusSdk.shared.apiLogHandler = { entry in print(entry.toPrettyJson()) }
```
{% endcode %}

`UnicusApiLogEntry` tiene `timestamp`, `path`, `statusCode`, `duration`,
`requestBody`, `responseBody`, `error`, `isSuccess` y `summary`.

Los logs están depurados: los datos biométricos cifrados, los datos del
documento, los tokens de sesión y los datos personales de los pasos del flujo
(códigos, teléfono, correo, valores de formulario, imagen de la firma, número de
documento, ubicación) se eliminan o se truncan. La llave de reanudación siempre
se elimina. `includeSensitiveApiLogData = true` conserva tokens y datos: úsalo
solo en local y nunca en una compilación de producción. Desactiva
`enableApiLogging` en producción.
