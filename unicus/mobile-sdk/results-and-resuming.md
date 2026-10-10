---
description: >-
  Outcomes of a mobile verification, the difference between leaving and
  cancelling, how an open transaction is resumed, and how to confirm the result
  on your backend.
---

# Results and resuming

## Result or error

`start` ends in one of two ways:

| Ending | Meaning |
| --- | --- |
| **Result** (`UnicusVerificationResult`) | The verification ran. This includes a person who was not verified, a cancellation and an open transaction. Route on `outcome`. |
| **Error** (`UnicusSdkException` / `UnicusSdkError`) | The verification could not run: configuration, network, a refused transaction, a flow that cannot run on mobile. Route on `error.code`. See [Errors and troubleshooting](errors-and-troubleshooting.md). |

A business rejection is never an error: it arrives as a result.

## Outcomes

| Outcome | Result codes | What to do |
| --- | --- | --- |
| `SUCCESS` | `2000` | Verified. Send the `tid` to your backend and confirm there. |
| `WARNING` | `2013` | Possible duplicate identity: held for manual review. Treat as pending; the decision arrives with the webhook `TRANSACTION_REVIEW_RESOLVED`. |
| `RESUMABLE` | `2003` | The transaction is still open: the user left, or steps are still pending. Offer to continue; see [Resuming](#resuming). |
| `FAILED` | `2052`, `4011`, `9010`, `9011`, and other codes below `9000` | Not verified. Show a business message; `rejectionReason` explains a `2052`. |
| `CANCELED` | `2041`, `2051`, `2061`, `4001` | Cancelled, expired, or no attempts left. Let the user start again. |
| `ERROR` | `2054`, `4012`–`4014`, `9xxx` | Technical failure. When `isRetryable` is true (`2054`, `4014`) just try again. |
| `UNKNOWN` | none | No code could be determined. Retry, or contact support with the `tid`. |

Every code is listed in [Result codes](result-codes.md). Native code shows the
outcomes in the platform's style: `UnicusVerificationOutcome.SUCCESS` on
Android, `.success` on iOS and Flutter.

{% hint style="warning" %}
Some rejections have codes of `9000` or higher (for example `9001`, the face
does not match the document) and the SDK reports them with outcome `ERROR`.
For those, use `resultCode` to tell the user that the person was not
verified rather than "technical problem". See [Result codes](result-codes.md).
{% endhint %}

Useful fields of the result:

| Field | Description |
| --- | --- |
| `tid` | Transaction id. Send it to your backend. |
| `outcome`, `resultCode` | See above. |
| `isRetryable` | `true` for `2054` and `4014`: repeating is safe. |
| `isResumable` | `true` for `2003`. |
| `resumed` | `true` when this `start` continued an open transaction. |
| `rejectionReason`, `rejectionDetail` | For `2052`: the reason sent by Unicus (`STEP_OUT_OF_ORDER`, `FORM_INVALID`, `SIGN_DECLINED`...). |
| `flowId`, `steps` | The flow that ran and the result of each step (completed, failed, skipped). |

Exact names per platform: [Android](android/results-and-events.md),
[iOS](ios/results-and-events.md), [Flutter](flutter/results-and-events.md).

## Leaving is not cancelling

| What happens | Result | Transaction |
| --- | --- | --- |
| The user closes a flow screen and confirms, presses back, swipes the app away, or the system closes the screen. A Custom provider calls `cancel()`. | `2003` `RESUMABLE`, with the `tid` | Stays open and can be resumed. |
| The user cancels inside the camera screen. | `2041` `CANCELED` | Cancelled. |
| Your app calls `cancelActiveSession`. | `2041` `CANCELED` | Cancelled. |
| The flow already passed its camera session when the cancel arrives. | `2003` `RESUMABLE` | Unicus keeps it open: the remaining steps can still be completed. |

`cancelActiveSession` closes the SDK screens wherever the user is (camera,
flow screens, your Custom screen, between steps, or while `start` is still
preparing) and delivers the result to the `start` callback first. It never
fails. Leaving a screen never sends a cancellation.

## Resuming

A transaction stays open in Unicus until it is final or **20 minutes pass
without activity**. While it is open, the user can continue at the first
pending step. Completed steps are not repeated.

### Automatic resume (recommended)

The configuration option `resumeOpenTransactions` is on by default. When the
user leaves, the next `start` **with the same document** continues the same
transaction:

1. Unicus returns a resume key when the transaction is created. The SDK stores
   it encrypted on the device (Android Keystore, iOS Keychain), linked to your
   Customer Token and the document.
2. The next `start` for the same person sends it. If the transaction can
   continue (not final, not expired, same flow), Unicus resumes it: same
   `tid`, the event `transactionResumed` and `result.resumed == true`.
3. Otherwise a new transaction is created, with no error.

The key is deleted when the transaction reaches a final outcome and expires on
the device after 24 hours. It never appears in results, events or logs.

{% hint style="info" %}
Call `clearResumeData()` when the user logs out of your app, so another person
on the same device never continues that transaction. On Android 5.0 and 5.1
(API 21–22) the key is not stored and every `start` creates a new transaction.
{% endhint %}

Set `resumeOpenTransactions` to `false` if every `start` must create a new
transaction.

### Resume a known transaction

To continue a specific transaction, start with
`existingTransaction(tid)` and the `tid` of the `RESUMABLE` result. The SDK
continues at the first pending step. It works until the transaction expires.

### Expiry

| Situation | Result |
| --- | --- |
| 20 minutes without activity after a failed attempt. | The transaction ends `REJECTED` with the last failure code. |
| 20 minutes without activity otherwise. | The transaction ends `EXPIRED` (`6003`). |
| Resuming an expired or final transaction by `tid`. | Error `transaction_expired` (`2051`): start a new verification. |

Attempt limits are the same as in the web: see
[Final or retryable](../sdk-web-v5/result-codes.md#final-or-retryable).

## What to show the user

| Outcome | Suggested screen |
| --- | --- |
| `SUCCESS` | "Identity verified." Continue your process once your backend confirms. |
| `WARNING` | "We are reviewing your information." No retry. |
| `RESUMABLE` | "You have a verification in progress." Button **Continue**, which calls `start` again with the same document. |
| `FAILED` | "We could not verify your identity", with the reason for your business. Offer a new attempt when it makes sense. |
| `CANCELED` | "Verification cancelled." Button **Start again**. |
| `ERROR` | "Something went wrong." Button **Try again**; contact support if it persists. |

## Confirm on your backend

The result in your app is for the user experience. Your backend decides with
data that comes from Unicus:

* **Webhook.** Unicus sends one `TRANSACTION_FINALIZED` per transaction when
  it is final, and `TRANSACTION_REVIEW_RESOLVED` after a manual review. See
  [Webhooks](../sdk-web-v5/webhooks.md).
* **Transaction status.** Your backend calls
  [Get a transaction status](../sdk-web-v5/transaction-status.md) with the
  `tid` that your app sent.

A `RESUMABLE` transaction has no webhook yet: the webhook arrives when it is
finished or expires.
