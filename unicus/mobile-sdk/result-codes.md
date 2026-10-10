---
description: >-
  Every result code a Unicus mobile SDK can return in a verification result:
  meaning, outcome reported by the SDK and what to do.
---

# Result codes

The result of `start` carries a `resultCode` and an `outcome`. The codes are
the same as in [Web SDK 5.0](../sdk-web-v5/result-codes.md), the final webhook
and the transaction status, plus a few codes produced by the SDK itself.

The outcome in the tables is the one the SDK reports. When it is `ERROR` for a
rejection of the person (for example `9001`), route on the code.

## Flow and transaction

| Code | Meaning | Outcome | What to do |
| --- | --- | --- | --- |
| `2000` | Every required step passed. Some older flows report `0` or `200`. | `SUCCESS` | Confirm on your backend. |
| `2003` | The transaction is still open: the user left, or steps are pending. | `RESUMABLE` | Offer to continue. See [Resuming](results-and-resuming.md#resuming). |
| `2013` | Possible duplicate identity, held for manual review. | `WARNING` | Treat as pending; wait for `TRANSACTION_REVIEW_RESOLVED`. |
| `2002` | No flow assigned to your company, or no valid flow for the transaction. | `FAILED` | Assign and publish a flow in the portal. |
| `2011` | The person is already enrolled. | `FAILED` | Business rule of your company. |
| `2012` | The person is not enrolled (the flow matches against an enrolment). | `FAILED` | Enrol the person first. |
| `2041` | Cancelled by the user inside the camera, or by your app with `cancelActiveSession`. | `CANCELED` | Let the user start again. |
| `2051` | Transaction not found, already final, or expired. | `CANCELED` | Start a new verification. |
| `2052` | Unicus refused a step. `rejectionReason` holds the reason (see below). When creating the transaction: user blocked. | `FAILED` | See [Step rejected](#step-rejected-2052). |
| `2054` | Temporary failure in Unicus. Nothing was processed. | `ERROR` | Retry (`isRetryable`). |
| `2061` | The attempts on one side of the document were used up. | `CANCELED` | Start a new verification. |
| `4001` | The OTP step ran out of attempts. | `CANCELED` | Start a new verification. |
| `9020` | The assigned flow cannot run in this app (see [Flow rules](flows-and-ui-steps.md#flow-rules-for-mobile)). Normally reported as the error `flow_not_supported`. | `ERROR` | Fix the flow or the `uiStepMode`. |

## Document signature (`sign_document`)

| Code | Meaning | Outcome | What to do |
| --- | --- | --- | --- |
| `4011` | The signer declined the document. | `FAILED` | Business message. |
| `4012` | No valid e-mail for the signer in the data the step uses. | `ERROR` | Fix the flow configuration or the data collected before the step. |
| `4013` | Unicus Sign is not enabled for your company. | `ERROR` | Contact Tekbees. |
| `4014` | The signature service did not answer. | `ERROR` | Retry (`isRetryable`). |

## Age check

| Code | Meaning | Outcome |
| --- | --- | --- |
| `9010` | The face is not certified to be at least the flow's minimum age. | `FAILED` |
| `9011` | The age could not be estimated from the face. | `FAILED` |
| `10020` | The person was not verified because of an age restriction. | `ERROR` |

## Camera and document

Reported when they end the transaction, and in the step results. Unless the
attempt limits are reached, the user can retry inside the flow.

| Code | Meaning | Outcome |
| --- | --- | --- |
| `1001` | The document number read from the ID does not match the document of the transaction. | `FAILED` |
| `5003` | Liveness could not be confirmed. | `FAILED` |
| `5001`, `5002`, `5004`, `5005` | The selfie could not be saved or processed. | `FAILED` |
| `6003` | A service did not answer in time, or the transaction expired. | `FAILED` |
| `6004` | Document type not supported, or its country or type is not enabled. | `FAILED` |
| `6009` | The document classifier rejected the document. | `FAILED` |
| `7001`, `7002` | Low quality or unreadable front / back. | `FAILED` |
| `7003`, `7004`, `7005` | The document could not be read. | `FAILED` |
| `7006` | The document did not pass the authenticity checks. | `FAILED` |
| `7007` | Document marked as fraud in a duplicate review. | `FAILED` |
| `8001`, `8002` | Not found or not verified in the official registry. | `FAILED` |
| `8003`–`8006` | Official registry timeout, error or not available. | `FAILED` |
| `9001` | The face does not match the document photo. | `ERROR` |
| `9002` | The face does not match the enrolled face. | `ERROR` |
| `9004`, `9005`, `9007` | Biometric engine or processing error. | `ERROR` |
| `10010`, `10011` | Document classifier error. | `ERROR` |

Descriptions in detail: [Camera and document codes](../sdk-web-v5/result-codes.md#camera-and-document-codes).

## Step rejected (`2052`)

`rejectionReason` and `rejectionDetail` come from the machine-readable reason
sent by Unicus (`REASON` or `REASON:detail`), never from translated text.

| `rejectionReason` | Meaning |
| --- | --- |
| `STEP_OUT_OF_ORDER` | A step was submitted before the previous required ones. |
| `STEP_NOT_IN_FLOW`, `STEP_TYPE_MISMATCH` | The step is not in the transaction's flow, or has another type. Usually the flow changed while the transaction was open. |
| `FLOW_ALREADY_FAILED` | A required step already failed. |
| `FORM_INVALID` | Form rejected; the detail lists the fields. |
| `SIGNATURE_INVALID` | Signature missing or not valid. |
| `CONSENT_INVALID`, `CONSENT_NOT_RECORDED` | Consent not recorded correctly. |
| `OTP_INVALID_CODE`, `OTP_EXPIRED`, `OTP_RESEND_TOO_SOON`, `OTP_SEND_LIMIT`, `OTP_SEND_FAILED` | OTP problems; limits as in the [web OTP limits](../sdk-web-v5/result-codes.md#otp-limits). |
| `SIGN_EMAIL_MISSING`, `SIGN_NOT_CONNECTED`, `SIGN_DECLINED`, `STEP_NOT_AVAILABLE` | Document signature problems. |

In WebView mode the flow screens show these to the user and let them correct
the step; your app receives them only when the flow ends. In Custom mode your
screens receive them for each submission.

## Codes from the device

Produced by the SDK when the camera session or a flow screen ends early. They
never reach the webhook or the transaction status.

| Code | Meaning | Outcome |
| --- | --- | --- |
| `9991` | The camera session was interrupted (for example a network error). | `ERROR` |
| `9994` | Locked out after too many failed attempts in the camera session. | `ERROR` |
| `9995` | Camera error. | `ERROR` |
| `9996` | Camera permission denied. Ask the user to allow the camera in the system settings. | `ERROR` |
| `9997` | Unexpected error of the camera component. | `ERROR` |
| `9009` | Internal SDK error, or a fatal error of a flow screen (`raw["errorKey"]` names it). | `ERROR` |
| `500` | Generic error when no code was returned. | `FAILED` |

When the user cancels the face or document scan, the SDK reports `2041`.

## Messages

`UnicusResultCode.messageFor(code)` returns a short Spanish message for each
code, the same catalogue Unicus uses. Use your own texts if you need another
language or wording.
