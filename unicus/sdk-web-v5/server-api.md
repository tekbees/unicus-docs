---
description: >-
  The endpoints your backend calls (create a link, query, look up by document,
  delete) and the API key every one of them requires.
---

# Server API and API keys

Your **backend** can call four Unicus endpoints directly. Every one of them
requires your company's **API key**:

| Endpoint | What it does |
| --- | --- |
| `init-api-transaction` | Creates a transaction and returns a verification link to send to the person. |
| `query-transaction` | Returns the status and result of a transaction by its `tid`. See [Get a transaction status](transaction-status.md). |
| `query-id` | Returns the enrollment of a person by their document. |
| `delete-transaction` | Deletes an enrollment (and its face from the 1:N search) or a verification. |

{% hint style="danger" %}
**The Customer Token does not open these endpoints.** The Customer Token is
public: it is rendered in your page as the button's `customerid`. A request
that sends only `X-Customer-ID` / `X-Device-ID` gets `401`. Use the API key,
and only from your server: never put it in a page, a mobile app or a
repository.
{% endhint %}

The button (`<unicus-btn>`) and the mobile SDKs keep working with the Customer
Token: they do not use these endpoints.

## 1. Create an API key

In the [administrative portal](https://app.idunicus.com/), go to your company's
settings → **API keys** and create a key (requires the company's ADMIN role).

* The key looks like `unk_<8 characters>_<32 characters>`.
* The portal shows it **once**. Store it in your backend's secret manager or
  environment variables.
* Each environment (sandbox, production) has its own keys.
* To rotate: create a new key, deploy it to your backend, then revoke the old
  one. A revoked or expired key answers `401` immediately.

## 2. Send it

Send the key in the `Authorization` header, with `Content-Type:
application/json`:

{% code overflow="wrap" %}
```http
POST /query-transaction HTTP/1.1
Host: <unicus-server-api-url>
Authorization: Bearer unk_Ab12Cd34_0123456789abcdefghijABCDEFGHIJ
Content-Type: application/json

{ "tid": "<TID>" }
```
{% endcode %}

The key identifies your company: you only reach your own company's
transactions, and any `X-Customer-ID` / `X-Device-ID` header you send is
ignored.

## `init-api-transaction`

Creates a transaction for a person and returns a one-time link to the Unicus
web app. Use it when the person is not on your website (for example to send
the link by email or chat). Unicus enrolls the person if it does not know them
yet and verifies them if it does, with the flow assigned in the portal.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `document` | string | Yes* | Document number. |
| `documentType` | string | Yes* | Document type code, as in the button's `clientid`: `ID`, `FD`, `PP`, `DL`. |
| `flowId` | string | No | Slug of a published flow of your company to use instead of the one assigned to the transaction type (`2002` if it is not compatible). |
| `customParameter` | string | No | Only for age-estimation transactions, which are created without a document. |

\* Without `document`, an age-estimation transaction is created.

{% code overflow="wrap" %}
```bash
curl --request POST '<unicus-server-api-url>/init-api-transaction' \
  --header 'Authorization: Bearer <API_KEY>' \
  --header 'Content-Type: application/json' \
  --data '{ "document": "123456789", "documentType": "ID" }'
```
{% endcode %}

```json
{
  "success": true,
  "resultCode": 0,
  "resultMessage": "Enrollment Transaction was created.",
  "tid": "<TID>",
  "url": "<link to the Unicus web app>"
}
```

* Store the `tid` with your user: the [webhook](webhooks.md) and
  `query-transaction` refer to it.
* The `url` carries a single-use token: once the person opens it, the same
  link does not open again. Send a new one with another call if needed.
* `resultMessage` says whether an enrollment or a verification was created.
* On failure `success` is `false` and `resultCode` / `resultMessage` say why
  (for example `2002`: no flow assigned). See [Result codes](result-codes.md).

## `query-transaction`

Status and result of a transaction by its `tid`. Request, response fields and
recommended usage: [Get a transaction status](transaction-status.md).

## `query-id`

The enrollment of a person, looked up by their document instead of by `tid`.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `externalDatabaseRefID` | string | Yes | Document number. |
| `documentType` | string | Yes | Document type code (`ID`, `FD`, `PP`, `DL`). |

{% code overflow="wrap" %}
```bash
curl --request POST '<unicus-server-api-url>/query-id' \
  --header 'Authorization: Bearer <API_KEY>' \
  --header 'Content-Type: application/json' \
  --data '{ "externalDatabaseRefID": "123456789", "documentType": "ID" }'
```
{% endcode %}

* **Enrolled** in your company: `200` with the same body `query-transaction`
  returns for their successful enrollment transaction (person data, document,
  images).
* **Not enrolled**, or the enrollment did not succeed: HTTP `404`.

{% hint style="warning" %}
The answer carries the person's identity data and images. Call it only for a
legitimate reason in your process, and do not log the body.
{% endhint %}

## `delete-transaction`

Deletes a transaction of your company: for an enrollment, the person, their
result and their face in the 1:N search; for a verification, its result.
Deleting must be enabled for your company: ask Tekbees.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `tid` | string | Yes | Transaction id. |

{% code overflow="wrap" %}
```bash
curl --request POST '<unicus-server-api-url>/delete-transaction' \
  --header 'Authorization: Bearer <API_KEY>' \
  --header 'Content-Type: application/json' \
  --data '{ "tid": "<TID>" }'
```
{% endcode %}

* Deleted: `success: true`, `resultCode` `2053`. Your webhook receives
  `TRANSACTION_FINALIZED` with `outcome` `DELETED` if the transaction was still
  open.
* Not deleted: `success: false` with the reason in `resultMessage` (unknown
  `tid`, deleting not enabled for your company, or a temporary error: retry
  later).

## Errors

| HTTP | When | What to do |
| --- | --- | --- |
| `401` | No API key, a key with the wrong format, revoked or expired, or only the Customer Token was sent. Comes with `WWW-Authenticate: Bearer`. | Check the `Authorization` header and the key's status in the portal. |
| `429` | More than 600 requests per minute for your key (default). | Wait the seconds in `Retry-After`. |
| `400` | The body is not valid JSON. | Send a JSON object with the fields above. |
| `5xx` | Temporary failure. | Retry with backoff. |

Business answers (transaction not found, not enrolled in `query-transaction`,
flow not configured) come with HTTP `200` and `success: false`, except
`query-id`, which answers `404` when the person is not enrolled.

## Checklist

- [ ] API key created per environment and stored in your secret manager.
- [ ] Every server call sends `Authorization: Bearer <API_KEY>`.
- [ ] No API key in front-end code, mobile apps, logs or repositories.
- [ ] `401` alerts in your monitoring (a revoked or expired key).
- [ ] A rotation procedure: new key → deploy → revoke the old one.
