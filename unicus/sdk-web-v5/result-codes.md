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
| `2041` | Cancelled | The user closed the verification before finishing (`OnUnicus:exit`). |
| `2051` | Expired or used | The transaction, session or link is no longer valid. A new transaction is needed. |
| `2061` | Retry limit | The user exhausted the retries of a camera step. |
| `4001` | OTP exhausted | Too many wrong one-time codes. |

## Transaction creation

| Code | Meaning | Fix |
| --- | --- | --- |
| `2002` | No flow assigned to the company and transaction type, or unknown `data-flow-id`. | Assign or publish the flow in the administrative portal. |
| `2052` | The person is blocked. | Review the person in the portal. |

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

## Camera and document codes

Reported in `OnUnicus:details` step results and as the final code when a
required camera step fails.

| Code | Description |
| --- | --- |
| `1001` | Invalid document verification (id number check). |
| `5003` | Liveness could not be determined. |
| `6001` / `6002` | Invalid front / back of the document. |
| `6003` | Timeout capturing the document. |
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

The match levels (`matchLevel`) and age groups (`ageEstimateGroup`) that
accompany step results are the same as in Web SDK 4.x; see
[Result Codes and References](../sdk-web/result-codes-and-references.md).
