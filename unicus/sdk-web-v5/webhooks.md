---
description: >-
  The final result of every transaction, pushed to your backend: one signed
  TRANSACTION_FINALIZED event per transaction, retried until you acknowledge it.
---

# Webhooks

The webhook is the **authoritative result** of a verification. The browser
events of the button tell your page what happened, but anything the user's
browser reports can be tampered with: decide on your backend, with the webhook
(or [Get a transaction status](transaction-status.md)).

Unicus sends **one event per transaction**, when its result is final:

| Event | When |
| --- | --- |
| `TRANSACTION_FINALIZED` | Once per transaction, when it ends: approved, rejected, sent to review, expired, cancelled or deleted. |
| `TRANSACTION_REVIEW_RESOLVED` | Only for a transaction that ended in `REVIEW`, when Tekbees decides the review. |

{% hint style="warning" %}
**Coming from Web SDK 4.x?** The step webhooks (`LIVENESS_FACEMAP`,
`FRONT_DOCUMENT`, `BACK_DOCUMENT`, `MATCH_DOCUMENT`, `VERIFY_LIVENESS`) are no
longer sent. Your endpoint receives only the events on this page. See
[Migration from Web SDK 4.x](migration-from-v4.md).
{% endhint %}

## 1. Configure your endpoint

In the [administrative portal](https://app.idunicus.com/), go to
**Company → Settings → Webhook** and enter your endpoint URL, for example
`https://api.example.com/unicus/webhook`.

<figure><img src="../.gitbook/assets/image (25).png" alt="Webhook settings in the administrative portal" width="563"><figcaption><p>Webhook settings in the administrative portal.</p></figcaption></figure>

Requirements:

* A public `https://` URL. Hosts that resolve to private, loopback or
  link-local addresses are refused, and **redirects are not followed**: give the
  final URL.
* Answer any `2xx` within **30 seconds**. Anything else (or no answer) is
  retried.
* One URL per company and environment (sandbox and production are configured
  separately).

## 2. Turn on signatures (recommended)

In the same screen, **Webhook signing → Generate secret**. The portal shows the
secret (`whsec_…`) **once**: store it in your backend's secret manager, never in
browser code or a repository. Within a minute every webhook arrives signed.

| Header | Value |
| --- | --- |
| `X-Unicus-Event-Id` | Unique id of the event, the same on every retry (`fin_<tid>` or `rev_<tid>`). Also in `meta.event_id`. |
| `X-Unicus-Timestamp` | Unix time in seconds when this delivery was signed. |
| `X-Unicus-Signature` | `v1=` + hex HMAC-SHA256 of `timestamp + "." + raw body`, with the secret (the whole `whsec_…` string, UTF-8) as key. |

To verify a request:

1. Read the **raw body bytes** before parsing the JSON (a re-serialized JSON
   does not match).
2. Reject it if `X-Unicus-Timestamp` is more than **5 minutes** away from your
   clock.
3. Compute `HMAC-SHA256(secret, timestamp + "." + rawBody)` in hex, prefix
   `v1=`, and compare with `X-Unicus-Signature` in **constant time**.

**Rotate secret** issues a new one; the previous one stops validating within a
minute, so update your endpoint right away. `X-Unicus-Event-Id` is sent even
without a secret.

{% tabs %}
{% tab title="Node.js (Express)" %}
```js
import crypto from 'node:crypto';
import express from 'express';

const app = express();
const SECRET = process.env.UNICUS_WEBHOOK_SECRET; // whsec_...

app.post('/unicus/webhook', express.raw({ type: 'application/json' }), (req, res) => {
  const timestamp = req.get('X-Unicus-Timestamp') ?? '';
  const signature = req.get('X-Unicus-Signature') ?? '';
  const age = Math.abs(Date.now() / 1000 - Number(timestamp));
  const expected = 'v1=' + crypto.createHmac('sha256', SECRET)
    .update(`${timestamp}.`).update(req.body).digest('hex');
  const valid = age <= 300 && signature.length === expected.length
    && crypto.timingSafeEqual(Buffer.from(signature), Buffer.from(expected));
  if (!valid) return res.sendStatus(401);

  const event = JSON.parse(req.body.toString('utf8'));
  res.sendStatus(200);           // acknowledge first...
  handleUnicusEvent(event);      // ...then process (idempotently, by meta.event_id)
});
```
{% endtab %}

{% tab title="Python (Flask)" %}
```python
import hashlib, hmac, os, time
from flask import Flask, request, abort

app = Flask(__name__)
SECRET = os.environ["UNICUS_WEBHOOK_SECRET"].encode()  # whsec_...

@app.post("/unicus/webhook")
def unicus_webhook():
    timestamp = request.headers.get("X-Unicus-Timestamp", "")
    signature = request.headers.get("X-Unicus-Signature", "")
    raw = request.get_data()  # raw bytes, before parsing
    expected = "v1=" + hmac.new(SECRET, timestamp.encode() + b"." + raw, hashlib.sha256).hexdigest()
    if not timestamp.isdigit() or abs(time.time() - int(timestamp)) > 300 \
            or not hmac.compare_digest(signature, expected):
        abort(401)
    event = request.get_json()
    enqueue(event)  # process asynchronously, idempotently by event["meta"]["event_id"]
    return "", 200
```
{% endtab %}

{% tab title="Java" %}
```java
// rawBody: the request body bytes exactly as received.
static boolean isValid(String secret, String timestamp, String signature, byte[] rawBody) throws Exception {
    long ts;
    try { ts = Long.parseLong(timestamp); } catch (NumberFormatException e) { return false; }
    if (Math.abs(Instant.now().getEpochSecond() - ts) > 300) return false;
    Mac mac = Mac.getInstance("HmacSHA256");
    mac.init(new SecretKeySpec(secret.getBytes(StandardCharsets.UTF_8), "HmacSHA256"));
    mac.update((timestamp + ".").getBytes(StandardCharsets.UTF_8));
    String expected = "v1=" + HexFormat.of().formatHex(mac.doFinal(rawBody));
    return MessageDigest.isEqual(expected.getBytes(StandardCharsets.UTF_8),
            signature == null ? new byte[0] : signature.getBytes(StandardCharsets.UTF_8));
}
```
{% endtab %}

{% tab title="PHP" %}
```php
$secret    = getenv('UNICUS_WEBHOOK_SECRET'); // whsec_...
$raw       = file_get_contents('php://input');
$timestamp = $_SERVER['HTTP_X_UNICUS_TIMESTAMP'] ?? '';
$signature = $_SERVER['HTTP_X_UNICUS_SIGNATURE'] ?? '';
$expected  = 'v1=' . hash_hmac('sha256', $timestamp . '.' . $raw, $secret);

if (!ctype_digit($timestamp) || abs(time() - (int) $timestamp) > 300
    || !hash_equals($expected, $signature)) {
    http_response_code(401);
    exit;
}
$event = json_decode($raw, true);
// Store it, answer 200, process idempotently by $event['meta']['event_id'].
http_response_code(200);
```
{% endtab %}

{% tab title="C# (.NET)" %}
```csharp
static bool IsValid(string secret, string timestamp, string signature, byte[] rawBody)
{
    if (!long.TryParse(timestamp, out var ts)) return false;
    if (Math.Abs(DateTimeOffset.UtcNow.ToUnixTimeSeconds() - ts) > 300) return false;
    using var hmac = new HMACSHA256(Encoding.UTF8.GetBytes(secret));
    var prefix = Encoding.UTF8.GetBytes(timestamp + ".");
    var hash = hmac.ComputeHash(prefix.Concat(rawBody).ToArray());
    var expected = "v1=" + Convert.ToHexString(hash).ToLowerInvariant();
    return CryptographicOperations.FixedTimeEquals(
        Encoding.UTF8.GetBytes(expected), Encoding.UTF8.GetBytes(signature ?? ""));
}
```
{% endtab %}
{% endtabs %}

## 3. Handle deliveries

* **At least once.** A webhook can arrive more than once (a timeout on your
  side, a network error). Store `meta.event_id` and ignore an id you already
  processed.
* **Retries.** Any non-`2xx` answer is retried for about 22 hours: after 10 s,
  30 s, 2 min, 10 min, 30 min, 1 h, 2 h, 4 h, 6 h and 8 h. If your endpoint is
  down for longer, reconcile with [Get a transaction status](transaction-status.md).
* **Answer fast.** Persist the event and answer `200`; do the slow work
  (updating your systems, notifying the user) afterwards.
* **Order.** For one transaction, `TRANSACTION_FINALIZED` always comes before
  `TRANSACTION_REVIEW_RESOLVED`; across transactions there is no order.
* **Personal data.** The payload carries the person's document data (and
  optionally images): log only the `tid` and the outcome, never the body.

## `TRANSACTION_FINALIZED`

{% code overflow="wrap" %}
```json
{
  "meta": {
    "event": "TRANSACTION_FINALIZED",
    "event_id": "fin_6f1c2a4e-1b7d-4c8e-9a51-2f3b4c5d6e7f",
    "version": 2,
    "occurred_at": "2026-10-05T15:04:05Z",
    "ok": true,
    "code": 200,
    "livemode": true
  },
  "data": {
    "tid": "6f1c2a4e-1b7d-4c8e-9a51-2f3b4c5d6e7f",
    "type": "ENROLLMENT",
    "outcome": "APPROVED",
    "result_code": 2000,
    "result_message": "Success",
    "finalized_by": "PROCESS",
    "created_at": "2026-10-05T14:51:10Z",
    "started_at": "2026-10-05T14:52:02Z",
    "finalized_at": "2026-10-05T15:04:05Z",
    "attempts": { "face": 0, "document_front": 1, "document_back": 0, "document": 0,
                  "face_captures": 1, "document_front_captures": 2, "document_back_captures": 1 },
    "idNumber": "<DOCUMENT NUMBER>",
    "location": "{\"latitude\":4.7398,\"longitude\":-74.1137}",
    "document": {
      "idType": "ID",
      "idNumberOCR": "<DOCUMENT NUMBER READ FROM THE DOCUMENT>",
      "idName": "<FIRST NAMES>",
      "idLastName": "<LAST NAMES>",
      "birthDate": "<DATE>",
      "match_level": 6,
      "digital_id_spoof": 1,
      "face_on_document_status": 1,
      "full_id_status": 1,
      "text_on_document_status": 1,
      "unexpectedMediaEncounteredAtLeastOnce": false
    },
    "biometrics": { "ageEstimateGroup": 4, "livenessCheck": true },
    "flow": { "id": "onboarding", "version": 3 }
  }
}
```
{% endcode %}

### `meta`

| Field | Type | Description |
| --- | --- | --- |
| `event` | string | `TRANSACTION_FINALIZED`. |
| `event_id` | string | `fin_<tid>`. Never changes between retries: use it to deduplicate. |
| `version` | int | Payload version, currently `2`. |
| `occurred_at` | string | When the transaction ended (ISO 8601, UTC). |
| `ok` / `code` | bool / int | `true` / `200` only when `outcome` is `APPROVED`; otherwise `false` / `400`. Prefer `data.outcome`. |
| `livemode` | bool | `false` for a test transaction (created with a test API key or a test device id, see [Test mode](customer-token.md#test-mode-sandbox)); `true` otherwise. Never act on a `false` one as if it were real. |

### `data`

| Field | Type | Description |
| --- | --- | --- |
| `tid` | string | Transaction id: the one `OnUnicus:loaded` gave your page. |
| `type` | string | `ENROLLMENT`, `VERIFY`, `LIVENESS` or `FACE_AGE_ID`. |
| `outcome` | string | `APPROVED`, `REJECTED`, `REVIEW`, `EXPIRED`, `CANCELLED` or `DELETED` (see below). |
| `result_code` | int | `2000` when approved; otherwise the code that ended it. See [Result codes](result-codes.md). |
| `result_message` | string | Description of `result_code`. |
| `finalized_by` | string | What ended it: `PROCESS` (a capture result), `FLOW` (the flow's last step or a failed step), `INACTIVITY`, `NOT_STARTED`, `CANCEL`, `DELETE`. |
| `created_at` | string | When the transaction was created. |
| `started_at` | string | When the user first opened it. Absent if never opened. |
| `finalized_at` | string | When it ended. |
| `attempts` | object | Failed attempts per capture: `face`, `document_front`, `document_back`, `document` (face against document). Technical errors on Unicus' side are not counted. Also every capture made, the successful one included: `face_captures` (face captures, for example the selfies of a verification), `document_front_captures`, `document_back_captures`. |
| `last_failure` | object | `{ code, message, step }` of the last failed attempt. Absent when approved or when nothing failed. |
| `idNumber` | string | Document number the transaction was created with (`clientid`). |
| `location` | string | JSON text `{"latitude":…,"longitude":…}` when the user shared their location. Absent otherwise. |
| `document` | object | Document data and checks, when a document was captured (see below). |
| `biometrics` | object | Face results, when the face was captured: `ageEstimateGroup`, `livenessCheck`, `matchLevel` (verification). |
| `flow` | object | `{ id, version }` of the portal flow the transaction ran. Absent for transactions without a flow. |
| `images` | object | Only if Tekbees enabled images for your company: `document_front`, `document_back` (base64 JPEG). Otherwise get them with [Get a transaction status](transaction-status.md). |

**`document`** can contain: `idType`, `idNumberOCR` (the number read from the
document), `idName`, `idLastName`, `birthDate`, `placeBirth`, `issue_date`,
`issue_place`, `height`, `bloodType`, `gender`, `idCountryCode`, `address1`–`address3`,
`custom_field_1`–`custom_field_5`, and the checks `match_level` (face against
the document photo, `0`–`7`), `age_estimate_group`, `digital_id_spoof`,
`face_on_document_status`, `full_id_status`, `text_on_document_status` and
`unexpectedMediaEncounteredAtLeastOnce`. Which fields are present depends on
the country and the document template. Compare `idNumberOCR` with `idNumber` if
you need to make sure the document belongs to the person you expected.

### Outcomes

| `outcome` | When | What to do |
| --- | --- | --- |
| `APPROVED` | Every required step passed (`result_code` `2000`). | Continue your process. |
| `REJECTED` | A definitive failure (for example the retry limit of a capture was reached, or too many wrong OTP codes), or the user failed a step and then left for 20 minutes. | Do not continue. Offer a new verification if appropriate. `last_failure` says what failed. |
| `REVIEW` | The verification completed but Unicus holds it for manual review (for example a possible duplicate identity, `2013`). | Treat as pending. Wait for `TRANSACTION_REVIEW_RESOLVED`. |
| `EXPIRED` | Opened and then abandoned for 20 minutes without a pending failure, or the link was never opened in 24 hours (`finalized_by: NOT_STARTED`). `result_code` `6003`. | Create a new transaction if the user comes back. |
| `CANCELLED` | The user cancelled the verification (`2041`). | Offer to start again. |
| `DELETED` | The transaction was deleted. | — |

Until the transaction is final, a failed attempt is **not** final: the user can
retry with the same link (up to 10 attempts per capture). Once the event is
sent, the transaction is closed and its link no longer works.

## `TRANSACTION_REVIEW_RESOLVED`

{% code overflow="wrap" %}
```json
{
  "meta": {
    "event": "TRANSACTION_REVIEW_RESOLVED",
    "event_id": "rev_6f1c2a4e-1b7d-4c8e-9a51-2f3b4c5d6e7f",
    "version": 2,
    "occurred_at": "2026-10-05T16:20:00Z",
    "ok": true,
    "code": 200,
    "livemode": true
  },
  "data": {
    "tid": "6f1c2a4e-1b7d-4c8e-9a51-2f3b4c5d6e7f",
    "type": "ENROLLMENT",
    "previous_outcome": "REVIEW",
    "outcome": "APPROVED",
    "result_code": 2000,
    "result_message": "Success",
    "review_decision": "enroll",
    "resolved_at": "2026-10-05T16:20:00Z"
  }
}
```
{% endcode %}

| `review_decision` | `outcome` |
| --- | --- |
| `enroll` | `APPROVED` |
| `fraud`, `fail` | `REJECTED` |

## Testing your endpoint

* Use the sandbox environment and its own Customer Token and webhook URL.
* Your endpoint must be reachable from the internet. For local development use
  a tunnel (for example ngrok or Cloudflare Tunnel) and register its `https`
  URL in the sandbox.
* Run one transaction to the end and one that you abandon: after 20 minutes of
  inactivity you receive an `EXPIRED` (or `REJECTED`) event, which is the case
  most integrations forget.
* If nothing arrives, check in the portal that the webhook is active and that
  the URL answers `2xx` to a `POST` with a JSON body.
