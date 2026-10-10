---
description: >-
  Query the status and result of a Unicus transaction from your backend with
  the query-transaction endpoint.
---

# Get a transaction status

Use `query-transaction` from your **backend** to read the status and result of
a transaction by its id (`tid`). The final webhook is the primary channel for
results; use this endpoint as a fallback and to reconcile (see
[Recommended usage](#recommended-usage)).

{% hint style="warning" %}
Call this endpoint only from your server. Never expose your API key or
Customer Token in a browser or mobile app.
{% endhint %}

## Request

<mark style="color:green;">`POST`</mark> `<unicus-server-api-url>/query-transaction`

### Headers

| Name | Required | Description |
| --- | --- | --- |
| `Authorization` | Yes | `Bearer <API_KEY>`: the company API key generated in the administrative portal. |
| `Content-Type` | Recommended | `application/json` |

The Customer Token (`X-Customer-ID`) is not accepted: it is public, because it
is rendered in your page. The API key identifies your company. How to create,
send and rotate it: [Server API and API keys](server-api.md).

### Body

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `tid` | string | Yes | Transaction id returned by Unicus when the transaction was created. |

### Example

{% code overflow="wrap" %}
```bash
curl --request POST '<unicus-server-api-url>/query-transaction' \
  --header 'Authorization: Bearer <API_KEY>' \
  --header 'Content-Type: application/json' \
  --data '{ "tid": "<TID>" }'
```
{% endcode %}

You can only query transactions of your own company. A `tid` of another
company answers as not found.

## Responses

Business answers always come with HTTP `200`. Read `success` and
`resultMessage`:

| Situation | `success` | `resultCode` | `resultMessage` |
| --- | --- | --- | --- |
| Transaction found (final, or never opened) | `true` | `0` | `""` |
| Transaction created or still in process | `false` | `-1` | `TRANSACTION IN PROCESS` |
| Transaction expired after 20 minutes of inactivity | `false` | `-1` | `TRANSACTION EXPIRED` |
| Unknown `tid`, or a `tid` of another company | `false` | `-1` | `TRANSACTION NOT FOUND` / `Transaction not found` |
| The Customer Token does not belong to an active company | `false` | `-1` | `COMPANY NOT FOUND` |

{% hint style="info" %}
The top-level `resultCode` is only `0` or `-1`. The verification result is in
`transactionResult` and `transactionStatusId`, see
[Result codes](result-codes.md).
{% endhint %}

### Transaction not available

{% code overflow="wrap" %}
```json
{
  "success": false,
  "wasProcessed": true,
  "error": false,
  "didError": false,
  "path": "query-transaction",
  "resultCode": -1,
  "resultMessage": "TRANSACTION IN PROCESS",
  "elapsedPerformanceTime": 0,
  "additionalSessionData": {},
  "tid": "<TID>"
}
```
{% endcode %}

### Transaction found

Fields without a value are omitted. Which groups are present depends on the
transaction type (enrolment, verification, liveness) and on the steps that
ran.

{% code overflow="wrap" expandable="true" %}
```json
{
  "success": true,
  "wasProcessed": true,
  "error": false,
  "didError": false,
  "path": "query-transaction",
  "resultCode": 0,
  "resultMessage": "",
  "elapsedPerformanceTime": 0,
  "tid": "<TID>",
  "transactionType": "<TRANSACTION_TYPE>",
  "transactionResult": "<TRANSACTION_RESULT>",
  "transactionStatus": "<STATUS_NAME>",
  "transactionStatusId": 2000,
  "transactionDate": "<ISO_DATE>",
  "transactionUpdate": "<ISO_DATE>",
  "companyTin": 0,
  "companyName": "<COMPANY_NAME>",
  "document": "<ID_NUMBER>",
  "documentType": "<ID_TYPE>",
  "name": "<NAME>",
  "lastName": "<LAST_NAME>",
  "country": "<COUNTRY_CODE>",
  "state": "<STATE>",
  "statusUser": 0,
  "userTid": "<ENROLLMENT_TID>",
  "comments": "<REVIEW_COMMENTS>",
  "liveness": { },
  "livenessStatus": 1,
  "matchLevel": { },
  "matchLevelStatus": 1,
  "ocr": { },
  "ocrStatus": 1,
  "infer": { },
  "inferStatus": -1,
  "validNumberCheck": { },
  "validNumberCheckStatus": 1,
  "government": { },
  "governmentStatus": 1,
  "documentValidation": { },
  "documentValidationStatus": 1,
  "feature": { },
  "matchIdFeature": { },
  "location": { },
  "searchDuplicated": [ ],
  "flowSteps": [
    {
      "stepId": "<STEP_ID>",
      "type": "otp",
      "outcome": "completed",
      "resultCode": 2000,
      "recordedAt": "<ISO_DATE>",
      "data": { "channel": "<CHANNEL>", "destination": "<DESTINATION>", "verifiedAt": "<ISO_DATE>" }
    }
  ],
  "faceImageList": [
    { "folder": "<FOLDER>", "filename": "<FILE_NAME>", "url": "<PRESIGNED_URL>" }
  ],
  "frontDocumentUrl": "<PRESIGNED_URL>",
  "backDocumentUrl": "<PRESIGNED_URL>",
  "frontDocumentWithoutSegmentUrl": "<PRESIGNED_URL>",
  "backDocumentWithoutSegmentUrl": "<PRESIGNED_URL>",
  "hash": "<SHA256>"
}
```
{% endcode %}

#### Transaction

| Field | Description |
| --- | --- |
| `tid` | Transaction id. |
| `transactionType` | Name of the transaction type. |
| `transactionResult` | Name of the overall result of the transaction. |
| `transactionStatus` / `transactionStatusId` | Name and number of the last code recorded on the transaction (see [Result codes](result-codes.md)). `transactionStatusId` is returned for enrolments. |
| `transactionDate` / `transactionUpdate` | Creation and last update dates. |
| `companyTin` / `companyName` | Your company. |
| `comments` | Comments of the manual review, if any. |
| `location` | Location reported by the device, if any. |

#### Person and document

{% hint style="warning" %}
Personal data. Store and process it according to your data protection
obligations.
{% endhint %}

| Field | Description |
| --- | --- |
| `document` / `documentType` | Document number and type. |
| `name` / `lastName` | Name of the person. |
| `country` / `state` | Country and state of the document. |
| `statusUser` | Status of the person in Unicus. |
| `userTid` | Verifications: `tid` of the person's enrolment. |
| `ocr` | Data read from the document. |
| `searchDuplicated` | Possible duplicates found in your company's 1:N search, with their data and images. |

#### Checks and biometrics

Each check has an object with its details and a status:

| Field | Status values |
| --- | --- |
| `liveness` / `livenessStatus` | `-1` not run, `0` pending, `1` passed, `2` failed. |
| `matchLevel` / `matchLevelStatus` | Same values. |
| `ocr` / `ocrStatus` | Same values. |
| `infer` / `inferStatus` | Same values. |
| `validNumberCheck` / `validNumberCheckStatus` | Document number check. Same values. |
| `government` / `governmentStatus` | Official registry check. Same values. |
| `documentValidation` / `documentValidationStatus` | Document authenticity checks. Same values. |

`feature` (liveness or enrolment) and `matchIdFeature` (document match) carry
the technical result of the biometric engine, including match levels and age
group (see [Match levels and age groups](result-codes.md#match-levels-and-age-groups)).

#### Flow steps

`flowSteps` is present for transactions that ran a flow from the portal: every
step with a recorded result, in flow order.

| Field | Description |
| --- | --- |
| `stepId` / `type` | Step id and type (`liveness`, `document`, `face_match`, `signature`, `otp`, `form`, `consent`, `info`, `age_check`). |
| `outcome` | `completed`, `failed` or `skipped` (optional step that failed). |
| `resultCode` | `2000` or the failure code. |
| `recordedAt` | When the result was recorded (UTC). |
| `data` | What the user gave: form `values`; OTP `channel`, `destination` and `verifiedAt`; signature hash, agreement text and an `imageUrl` valid for 24 hours; consent text and dates; document `validations`; `age_check` minimum and certified age. Personal data. |

#### Images

{% hint style="warning" %}
Images are personal (and biometric) data. Download them only if you need them
and store them securely.
{% endhint %}

| Field | Present |
| --- | --- |
| `faceImageList` | Face images captured during the transaction. Each item has a `url`, or the image bytes in `image` when no URL is available. |
| `frontDocumentUrl`, `backDocumentUrl` | Enrolments with a document: cropped front and back. |
| `frontDocumentWithoutSegmentUrl`, `backDocumentWithoutSegmentUrl` | Enrolments with a document: full front and back images. |
| `photoIDTamperingEvidenceFrontImageUrl`, `photoIDTamperingEvidenceBackImageUrl` | Enrolments whose last code is `7006` or `2041`: tampering evidence. |

Image URLs are pre-signed and expire after a few minutes (5 by default): query
again to get new ones. When an image has no URL, its base64 content comes in
the field without the `Url` suffix (`frontDocument`, `backDocument`,
`frontDocumentWithoutSegment`, `backDocumentWithoutSegment`,
`photoIDTamperingEvidenceFrontImage`, `photoIDTamperingEvidenceBackImage`).

#### Other fields

| Field | Description |
| --- | --- |
| `hash` | SHA-256 computed by Unicus over the response content. |
| `success`, `wasProcessed`, `error`, `didError`, `path`, `resultCode`, `resultMessage`, `elapsedPerformanceTime`, `additionalSessionData` | Technical envelope common to Unicus responses. |

## HTTP errors

| Status | When | Body |
| --- | --- | --- |
| `400` | The body is not valid JSON or not an object. | `application/problem+json` with a `title` such as `Malformed request [invalid-json]`. Decide by the HTTP status, not by the `title` text. |
| `401` | Missing, invalid, revoked or expired API key (for example, the request sent only `X-Customer-ID`). | `{"status":401,"title":"Unauthorized","detail":"..."}` and `WWW-Authenticate: Bearer`. |
| `429` | Rate limit exceeded for your credential (600 requests per minute by default). | `{"status":429,"title":"Too many requests"}` and `Retry-After` in seconds. |
| `500` | Unexpected error. | `application/problem+json`. Retry later. |

## Recommended usage

1. **Webhook first.** Unicus sends `TRANSACTION_FINALIZED` once per
   transaction when its result is final. Base your decision on it (see
   [Webhooks](webhooks.md)).
2. **Query as fallback and reconciliation.** Call `query-transaction` when a
   webhook did not arrive (for example your endpoint was down for longer than
   the retries), to fetch images that the webhook does not include, or in a
   periodic reconciliation job.
3. **Do not poll aggressively.** While the answer is `TRANSACTION IN PROCESS`
   the user may still be retrying: a transaction becomes final after its last
   step, an attempt limit, or 20 minutes of inactivity (24 hours if the link is
   never opened). If you must poll, use intervals of a minute or more and
   respect `Retry-After` on `429`.
4. **Treat `2013` as pending.** A transaction under review is decided later
   (`TRANSACTION_REVIEW_RESOLVED`).
