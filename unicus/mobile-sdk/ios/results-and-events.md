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
| `resultMessage` | `String?` | Message of the code addressed to the user, in the language of `UnicusSdkConfig.language`, or the reason written by Unicus for a rejected step. Route by `outcome` and `resultCode`, never by this text. |
| `isResumable` | `Bool` | 2003: the transaction is open and can be continued. |
| `isRetryable` | `Bool` | 2054 or 4014: temporary failure, start again. |
| `resumed` | `Bool` | `start` continued an open transaction instead of creating one. |
| `rejectionReason` / `rejectionDetail` | `String?` | For 2052 (step rejected): the machine-readable reason sent by the server (for example `STEP_OUT_OF_ORDER`). |
| `flowId` | `String?` | Flow executed; `nil` without a modular flow. |
| `steps` | `[UnicusStepResult]` | Per step: `stepId`, `outcome` (`completed`, `failed`, `skipped`), `resultCode`, document `validations`. |
| `transactionResultCode` | `Int?` | Final code reported by Unicus for the transaction, when known. |
| `sessionError` | `Bool` | The session ended because of a technical problem. |
| `errorMessage` | `String?` | Technical detail (server text or why the camera session ended), for logs. |
| `serverResponse` | `UnicusProcessResponse?` | Last Unicus answer to a capture, for diagnostics. It carries only `responseType`, `success`, `resultCode`, `isCompletelyDone`, `photoIDNextStepEnumInt` and `isFrontSide`; the other fields are empty (see [What the result and events carry](#what-the-result-and-events-carry)). `expectedMediaStatusEnumInt` and the deprecated `digitalIdSpoofStatusEnumInt` are not filled: a document that could not be confirmed as physical shows as result code `7006`. |
| `raw` | `[String: Any]` | Every field as a dictionary (also `resumeReason`: `userLeft`, `flowResumable`, `transactionOpen`). |

`UnicusResultCode` has the constants (`success`, `resumable`, `userCanceled`,
`stepRejected`, `temporaryFailure`, `signDeclined`…) and
`UnicusResultCode.messageFor(code)` returns the Spanish message of a code
(`messageFor(code, language: .english)` the English one; see
[Texts and languages](texts-and-languages.md#result-messages)).

## What the result and events carry

The result and the events are built on the device. From each Unicus answer to a
camera capture they keep only `responseType`, `success`, `resultCode`,
`isCompletelyDone`, `photoIDNextStepEnumInt` and `isFrontSide` (the `raw` maps
add the SDK's own labels `resolvedResponseType` and `eventName`). Document
data, match levels, the status of each check, age groups, session data and any
field Unicus adds later never reach your app: your server reads them with the
[webhook](../../sdk-web-v5/webhooks.md) or [Get a transaction
status](../../sdk-web-v5/transaction-status.md). The results of the flow steps
keep their `validations` (type, status, result code and profile).

{% hint style="warning" %}
Because they are computed on the device, the result and the events can be
altered and may never arrive (the app is closed, the network fails). Decide
access only on your server, with the `TRANSACTION_FINALIZED` webhook or
`query-transaction` and your server API key.
{% endhint %}

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
main queue. `UnicusSdkEvent` has `name`, `tid`, `message`, `response` (the reduced
Unicus answer, when there is one: see
[What the result and events carry](#what-the-result-and-events-carry)), `raw`,
plus `stepId` and `segmentKind`.

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
always removed. Answers to camera captures keep only what the SDK reads: type,
flags, codes, messages, flow state, summaries of validations and step results,
and the document check status. `includeSensitiveApiLogData = true` keeps tokens
and payloads: use it only locally and never in a production build. Turn
`enableApiLogging` off in production.
