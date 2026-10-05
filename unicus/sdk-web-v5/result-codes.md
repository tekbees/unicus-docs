---
description: >-
  Result codes returned in Web SDK 5.0 events, webhooks and the
  query-transaction endpoint, and the rejection reasons attached to them.
---

# Result codes

Codes appear in `OnUnicus:finished` (`state.resultCode`), in `OnUnicus:details`
(step results), in `OnUnicus:error`, in the webhook payloads and in
`query-transaction`. The same number means the same thing everywhere.

## Final outcomes

| Code | Outcome | Meaning |
| --- | --- | --- |
| `2000` | Success | Every required step passed. Legacy flows may report `0` or `200`. |
| `2013` | Under review | The flow completed but Unicus holds the transaction for manual review (for example a possible duplicate identity). Treat as pending, not as a failure. The final decision arrives through the webhook. |
| `2003` | Open | Camera steps finished but the transaction has pending steps (signature, OTP, form). Seen in step results, never as the final code of `finished`. |
| `2041` | Cancelled | The user closed the verification before finishing (`OnUnicus:exit`). Also sent in the `CANCEL_TRANSACTION` webhook (or the code the SDK reported). |
| `2053` | Deleted | The transaction was deleted with `delete-transaction` (`DELETE_TRANSACTION` webhook). |
| `2051` | Expired or used | The transaction, session or link is no longer valid. A new transaction is needed. |
| `2061` | Retry limit | The user exhausted the retries of a camera step. |
| `4001` | OTP exhausted | Too many wrong one-time codes. |

## Transaction creation

| Code | Meaning | Fix |
| --- | --- | --- |
| `2002` | No flow assigned to the company and transaction type, or unknown `data-flow-id`. | Assign or publish the flow in the administrative portal. |
| `2052` | The transaction could not be started (for example the person is blocked). The reason is in `resultMessage`. | Review the person or the request in the portal. |

## Temporary failure

| Code | Meaning | What to do |
| --- | --- | --- |
| `2054` | Unicus could not process the request at that moment. Nothing was used up: the same link and the same transaction remain valid. | The web app shows the user a "try again" screen with a **Retry** button. Do not create a new transaction; if it persists, contact support. |

## Step rejected by the server (`2052` during the flow)

While the flow runs, `2052` means the Unicus API refused a step. The reason is
machine-readable in `resultMessage` and the web app shows the user an
appropriate message.

| `resultMessage` | Meaning |
| --- | --- |
| `STEP_OUT_OF_ORDER:<stepIds>` | A step was submitted before the previous ones. The client resumes at the right step. |
| `STEP_NOT_IN_FLOW:<stepIds>` | The step does not belong to the transaction's flow (stale client). |
| `NO_FLOW` | The transaction lost its flow assignment. |
| `FORM_INVALID:<KIND>:<field>,…` | Form rejected. `KIND` is `REQUIRED`, `INVALID_TYPE`, `PATTERN_MISMATCH` or `UNKNOWN_FIELD`; the fields are listed. The user sees the errors next to the fields. |
| `SIGNATURE_INVALID:<field>` | Signature missing or unreadable. |
| `OTP_INVALID` | Wrong code; the user can retry until `4001`. |
| `OTP_EXPIRED` | The code expired; a new one is requested. |
| `OTP_SEND_FAILED:<channel>` | The code could not be sent by that channel (for example WhatsApp template not configured). |
| `CONSENT_NOT_RECORDED` | A camera capture (liveness, document, face match) arrived before the person's consent was recorded. Web transactions (button or link) need the consent first; in portal flows the consent step must come before the first camera step. |
| `TRANSACTION_ALREADY_FINISHED` | `save-sdk-status` was called for a transaction that already has a final result; nothing changes. |

## Camera and document codes

Reported in `OnUnicus:details` step results and as the final code when a
required camera step fails.

| Code | Description |
| --- | --- |
| `1001` | Invalid document verification (id number check). |
| `5003` | Liveness could not be determined. |
| `6001` / `6002` | Invalid front / back of the document. |
| `6003` | Timeout capturing the document. Also the code of an expired transaction (sent once). |
| `6004` | Document type not supported. |
| `6006` | Front and back do not match. |
| `6009` | Invalid document material (photocopy or screen). |
| `7001` / `7002` | Low quality front / back image. |
| `7003`–`7005` | OCR service timeout, internal error or communication error. |
| `7006` | Document text could not be read. |
| `7007` | Document failed validation. |
| `8001` | Person not found in the official registry. |
| `8002` | Not verified by the official registry. |
| `8003`–`8006` | Official registry timeout, internal error, communication error or unavailable. |
| `9001` | Selfie does not match the document photo. |
| `9002` | Selfie does not match the enrolled face. |
| `9004` / `9005` | Biometric engine internal or communication error. |
| `9992`–`9997` | The camera session ended early: `9992` the user cancelled the face scan, `9993` the user cancelled the document scan, `9994` locked out after too many attempts, `9995` camera error, `9996` camera permission denied, `9997` the camera could not start. |

## Match levels and age groups

Step results and webhooks can carry a `matchLevel` and an `ageEstimateGroup`.

**Face against the enrolled face** (verification): levels `0` to `15`.

| Level | False acceptance rate |
| --- | --- |
| `15` | 1 in 125,000,000 |
| `14` | 1 in 95,000,000 |
| `13` | 1 in 70,000,000 |
| `12` | 1 in 50,000,000 |
| `11` | 1 in 25,000,000 |
| `10` | 1 in 12,800,000 |
| `9` | 1 in 2,000,000 |
| `8` | 1 in 1,000,000 |
| `7` | 1 in 500,000 |
| `6` | 1 in 100,000 |
| `5` | 1 in 10,000 |
| `4` | 1 in 1,000 |
| `3` | 1 in 500 |
| `2` | 1 in 250 |
| `1` | 1 in 100 |
| `0` | No match |

**Face against the document photo** (enrolment): levels `0` to `7`, with the
same rates as the table above for each level. Rejections vary with the security
features of the document and the age of its photo.

**Age group** (`ageEstimateGroup`):

| Value | Meaning |
| --- | --- |
| `0` | Not available |
| `1` | Under 8 |
| `2` | Over 8 |
| `3` | Over 13 |
| `4` | Over 18 |
| `5` | Over 21 |
| `6` | Over 25 |
| `7` | Over 30 |
