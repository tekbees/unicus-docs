---
description: >-
  Read the UnicusVerificationResult of the Flutter SDK, route by outcome,
  listen to progress events and to the sanitized API logs.
---

# Results and events

`start` always ends with one `UnicusVerificationResult` when the verification
ran, also when the person was not verified. It throws `UnicusSdkException`
only when the verification could not run (see
[Errors and troubleshooting](../errors-and-troubleshooting.md)).

{% hint style="info" %}
The result in the app drives the user experience. To grant access, confirm in
your backend with the [webhook](../../sdk-web-v5/webhooks.md) or
[Get a transaction status](../../sdk-web-v5/transaction-status.md) using the
`tid`.
{% endhint %}

## Handle the outcome

{% code overflow="wrap" %}
```dart
Future<void> verify(UnicusVerificationRequest request) async {
  try {
    final result = await unicus.start(request);
    switch (result.outcome) {
      case UnicusVerificationOutcome.success:
      case UnicusVerificationOutcome.warning:
        await confirmInBackend(result.tid!); // 2000, or 2013 with a warning
      case UnicusVerificationOutcome.resumable:
        showContinueLater(); // 2003: start again with the same document
      case UnicusVerificationOutcome.failed:
        showNotVerified(result.resultCode, result.rejectionKind);
      case UnicusVerificationOutcome.canceled:
        showStartAgain(); // 2041 cancelled, 2051 expired, 2061 attempts…
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

`resumable` was added for 2003 (it was reported as `warning` before): an
exhaustive `switch` must include it. The meaning of every code is in
[Result codes](../result-codes.md); what to show the user, in
[Results and resuming](../results-and-resuming.md).

## Result fields

| Field | Type | Meaning |
| --- | --- | --- |
| `outcome` | `UnicusVerificationOutcome` | `success`, `warning`, `failed`, `canceled`, `error`, `resumable` or `unknown`. Route by this. |
| `success` | `bool` | `true` only for `success`. |
| `resultCode` | `int?` | Unicus result code (2000, 2003, 2041, 2052…). The server's final answer takes priority over the camera exit. |
| `resultMessage` | `String?` | Message of the result code. For developers; do not show it as is. |
| `tid` | `String?` | Transaction id. Send it to your backend. |
| `resumed` | `bool` | `true` when `start` continued the open transaction of the same document. |
| `isResumable` | `bool` | `true` for 2003: the transaction is still open and can be continued. |
| `resumeReason` | `String?` | Why it is still open: `userLeft`, `flowResumable` (a cancel arrived after the camera session) or `transactionOpen` (steps remain). |
| `isRetryable` | `bool` | `true` for 2054 and 4014: nothing was processed, start again. |
| `rejectionReason` / `rejectionDetail` | `String?` | For a step rejected by the server (2052), the machine-readable reason (`STEP_OUT_OF_ORDER`…) and its detail. |
| `rejectionKind` | `UnicusStepRejectionKind?` | Typed `rejectionReason` (`formInvalid`, `otpExpired`, `signDeclined`…). |
| `flowId` | `String?` | Flow executed; `null` for transactions without a modular flow. |
| `steps` | `List<UnicusStepResult>` | Result of each flow step: `stepId`, `outcome` (`completed`, `failed`, `skipped`), `resultCode`, document `validations`. |
| `transactionResultCode` | `int?` | Final code reported by the transaction status, when it answered. |
| `sessionError` | `bool` | The camera session ended on a technical interruption. |
| `status`, `nativeStatusCode` | `String`, `int?` | Exit status of the camera screen, for diagnostics. |
| `errorMessage`, `serverResponse`, `raw` | | Diagnostic details. Do not rely on them for routing. |

## Events

`unicus.events` is a broadcast `Stream<UnicusSdkEvent>` with the progress of
the running verification. Use it for analytics or a progress indicator, never
for the final decision.

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
// subscription.cancel() when your screen is disposed.
```
{% endcode %}

| Event (`event.name`) | When | Useful fields |
| --- | --- | --- |
| `sessionPrepared` | The transaction and its session are ready. | `tid` |
| `transactionResumed` | `start` continued an open transaction of the same document. | `tid` |
| `locationCollected` / `locationSkipped` | Location obtained, or skipped (no permission, denied, off). | `message` |
| `segmentStarted` | A group of steps begins. | `segmentKind` (`ui`, `biometric`, `server`), `flowId` |
| `stepStarted` | A flow step begins. | `stepId`, `stepType`, `flowId` |
| `stepCompleted` | A step ended well or was skipped. | `stepResult` |
| `stepFailed` | A step failed. | `stepResult` (`resultCode`, `validations`) |
| `pageError` | A flow screen shows a recoverable error (network, 2054, 429) with "try again". | `message` (error key) |
| `stepProgress` | Progress of a `sign_document` step: `signStatus`, `envelopeId`, `reason`. | Emitted by the native Custom mode only; Flutter apps normally do not receive it. |
| `processRequest`, `livenessProcessed`, `idScanProcessed` | Camera captures sent to Unicus. | `response` |
| `completed` | Unicus finalised the verification. | `response`, `message` |
| `nativeExit` | The camera screen closed. | `message` (exit status) |
| `error` | Error during the session (the result or exception follows). | `message` |

Every event also has `flow`, `tid`, `message` and `raw`. `stepType` values are
in `UnicusStepType` (`consent`, `info`, `liveness`, `document`, `face_match`,
`signature`, `otp`, `form`, `age_check`, `sign_document`). Events never carry
the resume key or session tokens.

## API logs

With `enableApiLogging: true`, `unicus.apiLogs` emits one `UnicusApiLogEntry`
per call to Unicus: `timestamp`, `path`, `statusCode`, `duration`,
`requestBody`, `responseBody`, `error`, plus `isSuccess`, `summary` and
`toPrettyJson()`.

{% code overflow="wrap" %}
```dart
unicus.apiLogs.listen((entry) {
  if (!entry.isSuccess) debugPrint(entry.toPrettyJson());
});
```
{% endcode %}

The SDK redacts `requestBlob`, `responseBlob`, `response`, `documentData`,
`ocrResults`, `unicusSession` and `sessionToken` (`<redacted>`) and truncates
long strings (`apiLogStringLimit`). `includeSensitiveApiLogData: true` disables
the redaction: debug builds only. Logs can still contain the document number
and the `tid`; do not send them to third-party services. When you contact
[Support](../support.md), share logs captured with sensitive logging off.
