---
description: >-
  The iOS verification result (UnicusVerificationResult), errors
  (UnicusSdkError), progress events and sanitized API logs.
---

# Results and events

`start` completes exactly once with a Swift `Result`:

* `.success(UnicusVerificationResult)`: the verification ran. The person may or
  may not be verified: read `outcome`.
* `.failure(UnicusSdkError)`: the verification could not run (configuration,
  network, flow not supported…).

## Handle the outcome

{% code overflow="wrap" %}
```swift
func handle(_ result: UnicusVerificationResult) {
    switch result.outcome {
    case .success:   approve(tid: result.tid)        // confirm in your backend
    case .resumable: showPending()                   // 2003: start again with the same document
    case .warning:   sendToReview(tid: result.tid)   // 2013
    case .failed:    showRejected(code: result.resultCode, reason: result.rejectionReason)
    case .canceled:  showStartAgain()                // 2041, 2051, retries exhausted
    case .error:     result.isRetryable ? showTryAgain() : showSupport()
    case .unknown:   showSupport()
    @unknown default: showSupport()
    }
}
```
{% endcode %}

{% hint style="info" %}
Keep `@unknown default`: the SDK is a binary framework and new outcomes may be
added. In the Swift 6 language mode a `switch` without it does not compile.
{% endhint %}

The app result is for the user interface. Decide access in your backend with the
[webhook](../../sdk-web-v5/webhooks.md) or
[Get a transaction status](../../sdk-web-v5/transaction-status.md) for the `tid`.
Meaning of each code: [Result codes](../result-codes.md).

## `UnicusVerificationResult`

| Field | Type | Meaning |
| --- | --- | --- |
| `outcome` | `UnicusVerificationOutcome` | `.success`, `.warning`, `.failed`, `.canceled`, `.error`, `.resumable`, `.unknown`. Route by this. |
| `success` | `Bool` | `outcome == .success`. |
| `tid` | `String?` | Unicus transaction id. Store it with your user or case. |
| `resultCode` | `Int?` | Unicus result code (2000, 2003, 2041…). |
| `resultMessage` | `String?` | Spanish message of the code, or the server message. For logs; show your own texts. |
| `isResumable` | `Bool` | 2003: the transaction is open and can be continued. |
| `isRetryable` | `Bool` | 2054 or 4014: temporary failure, start again. |
| `resumed` | `Bool` | `start` continued an open transaction instead of creating one. |
| `rejectionReason` / `rejectionDetail` | `String?` | For 2052 (step rejected): the machine-readable reason sent by the server (for example `STEP_OUT_OF_ORDER`). |
| `flowId` | `String?` | Flow executed; `nil` without a modular flow. |
| `steps` | `[UnicusStepResult]` | Per step: `stepId`, `outcome` (`completed`, `failed`, `skipped`), `resultCode`, document `validations`. |
| `transactionResultCode` | `Int?` | Final code reported by Unicus for the transaction, when known. |
| `sessionError` | `Bool` | The session ended because of a technical problem. |
| `errorMessage` | `String?` | Technical detail, for logs. |
| `raw` | `[String: Any]` | Every field as a dictionary (also `resumeReason`: `userLeft`, `flowResumable`, `transactionOpen`). |

`UnicusResultCode` has the constants (`success`, `resumable`, `userCanceled`,
`stepRejected`, `temporaryFailure`, `signDeclined`…) and
`UnicusResultCode.messageFor(code)` returns the Spanish message of a code.

## `UnicusSdkError`

| Field | Meaning |
| --- | --- |
| `code` | Stable identifier: route by it (`not_configured`, `environment_not_available`, `session_active`, `transaction_refused`, `network_error`, `flow_not_supported`, `no_view_controller`…). |
| `message` | Developer text. Do not show it to end users. |
| `resultCode` / `resultMessage` | Unicus values when the server answered (for example `transaction_refused` with 2002: no flow assigned). |
| `httpStatus` | HTTP status, when the error came from an HTTP answer. |
| `details` | Extra data (for example the missing values of `environment_not_available`). |

Every code with its fix: [Errors and troubleshooting](../errors-and-troubleshooting.md).

## Progress events

{% code overflow="wrap" %}
```swift
UnicusSdk.shared.eventHandler = { event in
    switch event.name {
    case UnicusSdkEvent.transactionResumed:
        print("continuing", event.tid ?? "")
    case UnicusSdkEvent.stepStarted, UnicusSdkEvent.stepCompleted, UnicusSdkEvent.stepFailed:
        print(event.name, event.stepId ?? "", event.raw["stepType"] ?? "")
    default:
        print(event.name, event.message ?? "")
    }
}
```
{% endcode %}

Events are informative: never decide the result from them. They arrive on the
main queue. `UnicusSdkEvent` has `name`, `tid`, `message`, `response` (parsed
server answer, when there is one), `raw`, plus `stepId` and `segmentKind`.

| Event | When |
| --- | --- |
| `locationCollected` / `locationSkipped` | Optional location obtained or skipped. |
| `transactionResumed` | `start` continued the open transaction of the same document (same `tid`). |
| `sessionPrepared` | Transaction and session ready; the screens are about to open. |
| `segmentStarted` | A group of steps starts; `event.segmentKind`: `.ui`, `.biometric`, `.server`. |
| `stepStarted` / `stepCompleted` / `stepFailed` | A flow step changed; `event.stepId`, `raw`: `flowId`, `stepType`, `resultCode`, `validations` (document step), `templateName` (`sign_document`). |
| `stepProgress` | `sign_document` in custom mode: the signature status changed (`raw["signStatus"]`, `envelopeId`, `reason`). |
| `pageError` | A flow screen showed a non-fatal error with its own retry. |
| `processRequest` | An encrypted capture is being sent to Unicus. |
| `livenessProcessed`, `enrollmentProcessed`, `authenticationComplete`, `idScanProcessed` | Unicus processed a capture. |
| `idScanFrontRetry`, `idScanBackRequired`, `idScanBackRetry`, `idScanUserConfirmation`, `idScanComplete`, `idScanNfcFallback` | Document capture progress. |
| `enrollmentRetry`, `verificationRetry`, `serverError` | The capture must be repeated, or the server answered an error. |
| `completed` | Unicus finalized the transaction. |
| `nativeExit` | The camera screen closed (`event.message`: its status). |
| `error` | A request to Unicus failed. |

## API logs

For development, receive a sanitized log of every Unicus API call:

{% code overflow="wrap" %}
```swift
let config = UnicusSdkConfig(apiKey: "<CUSTOMER_TOKEN>", environment: .dev, enableApiLogging: true)
UnicusSdk.shared.configure(config)
UnicusSdk.shared.apiLogHandler = { entry in print(entry.toPrettyJson()) }
```
{% endcode %}

`UnicusApiLogEntry` has `timestamp`, `path`, `statusCode`, `duration`,
`requestBody`, `responseBody`, `error`, `isSuccess` and `summary`.

Logs are redacted: encrypted biometric data, document data, session tokens and
the personal data of flow steps (codes, phone, e-mail, form values, signature
image, document number, location) are removed or truncated. The resume key is
always removed. `includeSensitiveApiLogData = true` keeps tokens and payloads:
use it only locally and never in a production build. Turn `enableApiLogging`
off in production.
