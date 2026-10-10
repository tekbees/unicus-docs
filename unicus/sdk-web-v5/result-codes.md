---
description: >-
  Result codes returned in Web SDK 5.0 events, the final webhook and the
  query-transaction endpoint: what each one means, whether it is final and
  how it maps to the webhook outcome.
---

# Result codes

The same number means the same thing everywhere it appears:

| Where | Field |
| --- | --- |
| Events | `OnUnicus:finished` and `OnUnicus:details` (`state.resultCode`), `OnUnicus:error` (`resultCode`). |
| Final webhook | `data.result_code` (the code that ended the transaction) and `data.last_failure.code` (the last failed attempt). |
| `query-transaction` | `transactionStatusId` (last code recorded on the transaction) and `flowSteps[].resultCode`. The top-level `resultCode` of that response is not a result code: it is `0` or `-1`, see [Get a transaction status](transaction-status.md). |

## Final or retryable

A failure does **not** end the transaction by itself. The user can try again
with the same link until one of these happens:

| Limit | Result |
| --- | --- |
| A code listed as **final** below. | The transaction ends at once. |
| 10 failed face attempts (liveness or verification). | Ends `REJECTED` with the last failure code. |
| 10 failed document-match attempts. | Ends `REJECTED` with the last failure code. |
| 10 attempts on one side of the document (front or back). | Ends `REJECTED` with `2061`. |
| 20 minutes without activity after the user opened the link. | Ends `REJECTED` with the last failure code, or `EXPIRED` (`6003`) when the last thing that happened was not a failure. |
| The link was never opened in 24 hours. | Ends `EXPIRED` (`6003`). |

Errors of Unicus or of a provider (`5001`, `5002`, `5004`, `5005`, `7003`,
`7004`, `7005`, `8003`–`8006`, `9004`, `9005`, `9007`, `10010`, `10011`) never
use up an attempt. Once the transaction is final its link no longer works
(`2051`) and Unicus sends the final webhook.

## Webhook outcome

`TRANSACTION_FINALIZED` carries an `outcome` and a `result_code`:

| `outcome` | `result_code` | When |
| --- | --- | --- |
| `APPROVED` | `2000` | The last stage succeeded, or the flow's last required step was completed. |
| `REVIEW` | `2013` | Possible duplicate identity: held for manual review. `TRANSACTION_REVIEW_RESOLVED` follows with `APPROVED` (`2000`) or `REJECTED` (`2013` or `7007` when marked as fraud, `8002` when rejected). |
| `REJECTED` | A final code below, or the last failure | A final code, an attempt limit, or 20 minutes of inactivity after a failure. |
| `EXPIRED` | `6003` | Opened and inactive for 20 minutes without a pending failure (`finalized_by: INACTIVITY`), or never opened in 24 hours (`finalized_by: NOT_STARTED`). |
| `CANCELLED` | `2041` | The user cancelled the verification. |
| `DELETED` | `2053` | The transaction was deleted with `delete-transaction`. |

## Final codes

| Code | Meaning | Outcome |
| --- | --- | --- |
| `2000` | Every required step passed. Legacy flows may report `0` or `200` in events. | `APPROVED` |
| `2013` | The face was found in the company's 1:N search (possible duplicate). Treat as pending, not as a failure; the decision arrives with `TRANSACTION_REVIEW_RESOLVED`. | `REVIEW` |
| `2011` | The person is already enrolled. | `REJECTED` |
| `2012` | The person is not enrolled (verification, or a flow that matches against an enrolment). Also returned when creating the transaction. | `REJECTED` |
| `2014` | The face was found in the fraud 1:N group. | `REJECTED` |
| `2061` | The attempts on one side of the document were used up. | `REJECTED` |
| `4001` | The OTP step ran out of verification attempts. | `REJECTED` |
| `9010` | `age_check` step: the face is not certified to be at least the flow's minimum age. | `REJECTED` |
| `9011` | `age_check` step: the age could not be estimated from the face. | `REJECTED` |
| `10020` | The person was not verified because of an age restriction. | `REJECTED` |
| `1001`, `7001`, `7002`, `8001`–`8006`, `6009` | One of the flow's document validations failed when the document capture completed (see below). | `REJECTED` |
| `2041` | The user cancelled the camera session. Closing or leaving the page does not cancel. | `CANCELLED` |
| `2053` | The transaction was deleted. | `DELETED` |
| `6003` | The transaction expired by inactivity or was never started. Also used for a service timeout during a step, which is retryable. | `EXPIRED` |

## Transaction creation

Returned by the button in `OnUnicus:error` when the transaction cannot be
created.

| Code | Meaning | Fix |
| --- | --- | --- |
| `2002` | No valid flow for the company and transaction type, unknown `data-flow-id`, or a flow not compatible with the transaction type. The reason is in `resultMessage`. | Assign or publish the flow in the administrative portal. |
| `2012` | The flow matches the face against an enrolment and the person is not enrolled. | Enrol the person first. |
| `2052` | Unicus refused to create the transaction. The button reports it as `user is currently blocked`. | Review the person in the portal or contact support. |

## Session and link

| Code | Meaning | What to do |
| --- | --- | --- |
| `2051` | The transaction was not found, is already final, or the link expired. | Create a new transaction. |
| `2054` | Temporary failure in Unicus. Nothing was used up: the same link and transaction remain valid. | The web app offers **Retry**. Do not create a new transaction; if it persists, contact support. |
| `2003` | The camera steps finished but the flow has more required steps (signature, OTP, form). Seen in step results, never as a final code. | Nothing; the flow continues. |

## Step rejected by the server (`2052` during the flow)

While the flow runs, `2052` means the Unicus API refused a step. The reason is
machine-readable in `resultMessage` (`REASON` or `REASON:detail`) and the web
app shows the user an appropriate message. Main reasons:

| `resultMessage` | Meaning |
| --- | --- |
| `STEP_OUT_OF_ORDER:<stepIds>` | A step was submitted before the previous required ones. The client resumes at the right step. |
| `STEP_NOT_IN_FLOW:<stepIds>` / `STEP_TYPE_MISMATCH:<stepId>` | The step does not belong to the transaction's flow or has another type (stale client). |
| `FLOW_ALREADY_FAILED:<stepId>` | A required step already failed; the transaction cannot continue. |
| `NO_FLOW` | The transaction has no stored flow. |
| `FORM_INVALID:<problems>` | Form rejected. Each problem is `<KIND>:<field>` with `KIND` `REQUIRED`, `INVALID_TYPE`, `PATTERN_MISMATCH` or `UNKNOWN_FIELD`. The user sees the errors next to the fields. |
| `SIGNATURE_INVALID:<what>` | Signature missing or not valid. |
| `CONSENT_INVALID` / `CONSENT_NOT_RECORDED` | The consent screen was not recorded correctly. |
| `OTP_INVALID_CODE` | Wrong code; the user can retry until `4001`. |
| `OTP_EXPIRED` | The code expired (5 minutes); a new one is requested. |
| `OTP_RESEND_TOO_SOON:<seconds>` / `OTP_SEND_LIMIT` | Too early to resend, or no more sends allowed. |
| `OTP_SEND_FAILED:<channel>:<reason>` | The code could not be sent by that channel. `reason` is `NO_OTP_TEMPLATE`, `OTP_TEMPLATE_INVALID:<code>` (configuration) or `PROVIDER_ERROR` (provider). |

### OTP limits

| Limit | Value | When it is reached |
| --- | --- | --- |
| Wrong codes per `otp` step (shared by all resends) | 5 | The transaction ends `REJECTED` with `4001`. |
| Code validity | 5 minutes | `OTP_EXPIRED`; the user asks for a new code. |
| Codes sent per `otp` step | 5 | `OTP_SEND_LIMIT` |
| Wait before a resend | 30 seconds (configurable in the step) | `OTP_RESEND_TOO_SOON:<seconds>` |
| Codes sent to one phone or e-mail | 5 per hour | `OTP_SEND_LIMIT:rate` |
| Codes requested from one IP address | 10 per 10 minutes | `OTP_SEND_LIMIT:rate` |
| Codes sent for your company | 1,000 per day (UTC) | `OTP_SEND_LIMIT:rate` |

If you expect more than 1,000 OTP codes a day (a campaign, a migration), ask
[Support](support.md) to raise your company's limit beforehand.

## Camera and document codes

Reported in `OnUnicus:details` step results, as `last_failure.code` in the
webhook, and as the final code when they end the transaction. Unless listed in
[Final codes](#final-codes), the user can retry within the limits above.

| Code | Description |
| --- | --- |
| `1001` | The document number read from the ID does not match the one of the transaction (`clientid`), or could not be compared (flow validation `id_number_match`). |
| `5003` | Liveness could not be confirmed. |
| `5001` / `5002` | The selfie or the audit image could not be saved. |
| `5004` / `5005` | Internal or communication error while processing the selfie. |
| `6003` | A service did not answer in time (for example the document classifier). |
| `6004` | Document type not supported: the document did not match any known template, or its country or type is not enabled. |
| `6009` | The document classifier rejected the document. |
| `7001` / `7002` | Low quality or unreadable front / back of the document (for Colombian ID cards, also a number that is not only digits or an unreadable issue date). |
| `7003` / `7004` / `7005` | Document reading failed: timeout, data could not be read, communication error. |
| `7006` | The document did not pass the authenticity checks: the ID could not be confirmed as a physical document (digital spoof), the full ID was not visible, the face or text on the document could not be confirmed, possible photocopy, expired document, or unexpected media (only for companies with that check enabled). Also returned when the document's country or type is not accepted for the company. |
| `7007` | Document marked as fraud in the review of a duplicate. |
| `8001` | Identity not found in the official registry. |
| `8002` | Not verified by the official registry (for example a document that is no longer valid). |
| `8003` | The official registry did not answer in time. |
| `8004` / `8005` / `8006` | Official registry error, communication error, or not available for that country. |
| `9001` | The face does not match the document photo (match level below the minimum). |
| `9002` | The face does not match the enrolled face. |
| `9004` / `9005` | Biometric engine internal or communication error. |
| `9007` | Internal error processing the capture. |
| `10010` / `10011` | Communication or internal error of the document classifier. |

## Codes from the web app only

The web app reports these in events when the camera session ends early. They
never reach the webhook or `query-transaction`.

| Code | Description |
| --- | --- |
| `9991` | The camera session was interrupted (for example a network error). |
| `9994` | Locked out after too many failed attempts in the camera session. |
| `9995` | Camera error. |
| `9996` | Camera permission denied. |
| `9997` | Unexpected error, or the camera component did not start in time. |
| `9998` | The camera cannot run inside a frame without permission. |
| `500` | Generic error when no code was returned. |

When the user cancels the face or document scan, events report `2041`.

## Match levels and age groups

Step results and webhooks can carry a `matchLevel` and an `ageEstimateGroup`.
Match levels and their false acceptance rates are defined by FaceTec, the
biometric engine.

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
features of the document and the age of its photo. A level below the minimum
configured for the company returns `9001`.

**Age group** (`ageEstimateGroup`; in liveness transactions `ageEstimateGroupV2`).
Each group means 99.5 % or more confidence that the person is at least that age:

| Value | Meaning |
| --- | --- |
| `0` | Not available (the age could not be estimated) |
| `1` | Not in any valid range (under 8) |
| `2` | 8 or over |
| `3` | 13 or over |
| `4` | 16 or over |
| `5` | 18 or over |
| `6` | 21 or over |
| `7` | 25 or over |
| `8` | 30 or over |
