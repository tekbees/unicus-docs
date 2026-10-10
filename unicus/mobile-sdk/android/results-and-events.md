---
description: >-
  The UnicusVerificationResult object, how to handle each outcome, the progress
  events and the sanitized API logs of the Unicus Android SDK.
---

# Results and events

## Two kinds of answer

| Answer | When | What it means |
| --- | --- | --- |
| `onSuccess(UnicusVerificationResult)` | The verification reached an end: verified, rejected, cancelled, left, or failed during the flow. | Business result. Read `outcome` and `resultCode`. A rejection is **not** an error. |
| `onError(UnicusSdkException)` | The verification could not start or could not continue for a technical or configuration reason. | `error.code` is stable (`not_configured`, `transaction_refused`, `flow_not_supported`, `network_error`…). See [Errors and troubleshooting](../errors-and-troubleshooting.md). |

With the `suspend` variants, `start` returns the result and throws
`UnicusSdkException`.

## Result fields

| Field | Type | Description |
| --- | --- | --- |
| `outcome` | `UnicusVerificationOutcome` | `SUCCESS`, `WARNING`, `FAILED`, `CANCELED`, `ERROR`, `RESUMABLE`, `UNKNOWN`. Route your app with it. |
| `resultCode` | `Int?` | Unicus result code (`2000`, `2003`, `2041`, `2052`…). See [Result codes](../result-codes.md). |
| `tid` | `String?` | Transaction id. Send it to your backend to match the webhook. |
| `success` | `Boolean` | `true` only for `SUCCESS`. |
| `isRetryable` | `Boolean` | `true` for `2054` and `4014`: start again. |
| `isResumable` | `Boolean` | `true` for `2003`: the transaction is still open. |
| `resumed` | `Boolean` | `true` when this `start` continued an open transaction of the same person. |
| `rejectionReason` | `String?` | Machine-readable reason of a `2052` (for example `STEP_OUT_OF_ORDER`, `SIGN_DECLINED`). Never a translated text. |
| `rejectionDetail` | `String?` | Detail after the reason, when Unicus gives one. |
| `resultMessage` | `String?` | Message of Unicus for the code (Spanish catalog). For logs; write your own user-facing texts. |
| `flowId` | `String?` | Id of the flow executed. |
| `steps` | `List<UnicusStepResult>` | Result of each step in flow order: `stepId`, `outcome` (`COMPLETED`, `FAILED`, `SKIPPED`), `resultCode`, `validations`. |
| `transactionResultCode` | `Int?` | Final code reported by Unicus for the transaction, when it answered. |
| `sessionError` | `Boolean` | `true` when the camera session ended because of a technical interruption. |
| `raw` | `Map<String, Any?>` | Diagnostic data (for example `raw["errorKey"]` of a flow screen error, `raw["leftOnStepId"]`). Do not depend on its keys for routing. |

## Handling every outcome

{% tabs %}
{% tab title="Kotlin" %}
{% code overflow="wrap" %}
```kotlin
fun handle(result: UnicusVerificationResult) {
    when (result.outcome) {
        UnicusVerificationOutcome.SUCCESS -> showVerified(result.tid)
        UnicusVerificationOutcome.WARNING -> showUnderReview(result.tid)          // 2013
        UnicusVerificationOutcome.RESUMABLE -> showContinueLater(result.tid)      // 2003
        UnicusVerificationOutcome.CANCELED -> showStartAgain()                    // 2041, 2051, 2061, 4001
        UnicusVerificationOutcome.FAILED -> showNotVerified(result.rejectionReason)
        UnicusVerificationOutcome.ERROR ->
            if (result.isRetryable) showTryAgain() else showTechnicalError(result.resultCode)
        UnicusVerificationOutcome.UNKNOWN -> showTechnicalError(result.resultCode)
    }
    result.tid?.let { backend.refreshVerification(it) }   // the backend decides with the webhook
}
```
{% endcode %}
{% endtab %}

{% tab title="Java" %}
{% code overflow="wrap" %}
```java
void handle(UnicusVerificationResult result) {
    switch (result.getOutcome()) {
        case SUCCESS: showVerified(result.getTid()); break;
        case WARNING: showUnderReview(result.getTid()); break;
        case RESUMABLE: showContinueLater(result.getTid()); break;
        case CANCELED: showStartAgain(); break;
        case FAILED: showNotVerified(result.getRejectionReason()); break;
        case ERROR:
            if (result.isRetryable()) showTryAgain(); else showTechnicalError(result.getResultCode());
            break;
        default: showTechnicalError(result.getResultCode());
    }
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% hint style="warning" %}
Add a branch for every value if you use an exhaustive `when`. `RESUMABLE` was
added in this version; code written against a preview build must handle it.
{% endhint %}

{% hint style="info" %}
Use the result for the user's screen only. Grant access from your backend with
the [webhook](../../sdk-web-v5/webhooks.md) or
[Get a transaction status](../../sdk-web-v5/transaction-status.md). What to
show for each case: [Results and resuming](../results-and-resuming.md).
{% endhint %}

## Events

Progress events help you log or show progress. They never carry the final
result: that is the `start` callback.

{% tabs %}
{% tab title="Kotlin" %}
{% code overflow="wrap" %}
```kotlin
UnicusSdk.shared.setEventListener { event ->
    when (event.name) {
        UnicusSdkEvent.STEP_STARTED -> analytics.step(event.stepId, event.stepType)
        UnicusSdkEvent.STEP_FAILED -> analytics.stepFailed(event.stepId, event.stepResult?.resultCode)
        UnicusSdkEvent.TRANSACTION_RESUMED -> Log.i("Unicus", "Resumed ${event.tid}")
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

Pass `null` to remove the listener. Every event has `name`, `tid` and
`message`; the other fields depend on the event.

| `name` | When | Useful fields |
| --- | --- | --- |
| `locationCollected` / `locationSkipped` | Optional location obtained or skipped. | `message` |
| `transactionResumed` | `start` continued an open transaction of the same person. | `tid` |
| `sessionPrepared` | Transaction and session ready; the first screen is about to open. | `tid` |
| `segmentStarted` | A group of steps begins. | `segmentKind` (`UI`, `BIOMETRIC`, `SERVER`), `raw["stepIds"]`, `raw["flowId"]` |
| `stepStarted` | A step begins. | `stepId`, `stepType`, `raw["flowId"]` |
| `stepCompleted` / `stepFailed` | A step ended. | `stepId`, `stepType`, `stepResult` (`outcome`, `resultCode`, `validations`) |
| `stepProgress` | Document signing progress in Custom mode. | `raw["signStatus"]`, `raw["envelopeId"]`, `raw["reason"]` |
| `pageError` | A flow screen shows a recoverable error (network, temporary failure) with "try again". | `message` (error key) |
| `processRequest` | A biometric capture is being sent to Unicus. | — |
| `livenessProcessed`, `enrollmentProcessed`, `authenticationComplete`, `idScanProcessed`, `idScanFrontRetry`, `idScanBackRequired`, `idScanBackRetry`, `idScanUserConfirmation`, `idScanComplete`, `idScanNfcFallback`, `enrollmentRetry`, `verificationRetry`, `serverError` | Unicus answered a biometric capture. | `response` (sanitized) |
| `completed` | Unicus finalized the transaction. | `response` |
| `nativeExit` | The camera screen closed. | `message` (exit status) |
| `error` | A request to Unicus failed. | `message` |

## API logs

While you integrate, you can see every call the SDK makes to Unicus:

{% code overflow="wrap" %}
```kotlin
UnicusSdk.shared.configure(
    UnicusSdkConfig(apiKey = token, environment = UnicusEnvironment.DEV).copy(enableApiLogging = true)
)
UnicusSdk.shared.setApiLogListener { entry ->
    Log.d("Unicus API", entry.summary)        // "/path -> 200 (123 ms)"
    Log.v("Unicus API", entry.toPrettyJson()) // sanitized request and response
}
```
{% endcode %}

Each `UnicusApiLogEntry` has `timestamp` (UTC), `path`, `statusCode`,
`durationMillis`, `requestBody`, `responseBody`, `error` and `isSuccess`.

The entries are **sanitized**: biometric data, document and OCR data, session
tokens, the transaction reference, form values, OTP codes and destinations,
signature images and texts, location, phone and e-mail are replaced by
`<redacted>`, and long strings are truncated (`apiLogStringLimit`). The resume
key is always redacted.

{% hint style="danger" %}
Keep `enableApiLogging` off in production builds and never turn on
`includeSensitiveApiLogData` outside a local debug session. When you send logs
to [Support](../support.md), send them sanitized.
{% endhint %}

Next: [Custom UI steps](custom-ui-steps.md).
