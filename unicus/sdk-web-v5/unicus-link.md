---
description: >-
  Send the user a verification link instead of embedding the button: create it
  in the administrative portal or from your backend.
---

# Unicus Link

Unicus Link creates a verification link that you send to the user by email,
SMS, WhatsApp or any other channel. The user opens it and goes through the same
Web SDK 5.0 verification as with the button: the flow assigned in the portal,
the company colours and logo, and the hand-off to the phone when the link is
opened on a computer.

Use it when the user does not go through your website or web app. To verify
the user inside your page, use the [button](quick-start.md) instead.

{% hint style="info" %}
Do not combine Unicus Link and the button for the same user and process. The
button creates and opens its own transaction; a link creates another one.
{% endhint %}

## Option 1: from the administrative portal

1. Sign in to the [Unicus administrative portal](https://app.idunicus.com/).
2. Open **Unicus Link** and select **Generate Link**.
3. Enter the user's document data.
4. Send the generated link to the user through your preferred channel.

<figure><img src="../.gitbook/assets/image (21).png" alt="Generate Link option in the administrative portal" width="375"><figcaption></figcaption></figure>

## Option 2: from your backend

<mark style="color:green;">`POST`</mark> `<unicus-server-api-url>/init-api-transaction`

Call it from your server only: the request carries your Customer Token.

### Headers

| Name | Required | Description |
| --- | --- | --- |
| `X-Customer-ID` | Yes | [Customer Token](customer-token.md) of the target environment. |
| `Content-Type` | Yes | `application/json` |

### Body

| Name | Required | Example | Description |
| --- | --- | --- | --- |
| `document` | Yes | `123456789` | Document number of the user. |
| `documentType` | Yes | `ID` | Document type: `ID`, `FD`, `PP` or `DL`. |

### Example

{% code overflow="wrap" %}
```bash
curl --request POST '<unicus-server-api-url>/init-api-transaction' \
  --header 'X-Customer-ID: <CUSTOMER_TOKEN>' \
  --header 'Content-Type: application/json' \
  --data '{ "document": "123456789", "documentType": "ID" }'
```
{% endcode %}

### Response

{% code overflow="wrap" %}
```json
{
  "success": true,
  "resultCode": 0,
  "resultMessage": "A Transaction Enrollment was created.",
  "url": "https://id.idunicus.com/?token=<TID>&process=enrollment"
}
```
{% endcode %}

Send the `url` value to the user. The `tid` in it is the transaction id: keep
it to correlate the webhook and to query the result.

## What the user sees

* **On a phone or tablet,** the whole verification runs in the browser.
* **On a computer,** the steps before the first camera step run there, and the
  user continues on the phone by QR code, WhatsApp or SMS (see
  [Flows and hand-off](flows-and-handoff.md)).
* The address bar is cleaned as soon as the link is read: the transaction id
  does not stay in the browser history.
* Reopening the link after the transaction finished or expired shows "the
  session expired or was already used".

## Getting the result

A link has no page of yours around it, so there are no `OnUnicus:*` browser
events. Your backend receives the result through the
[webhook](webhooks.md) (`TRANSACTION_FINALIZED`) or by calling
[Get a transaction status](transaction-status.md) with the `tid`.

## Good practices

* The link identifies one transaction: send it only to the person who has to
  verify, and do not publish it or reuse it for someone else.
* Without a flow assigned to the transaction type, the link cannot be used
  (result code `2002`): assign the flow in the portal first.
* If the user lost the link or it expired, create a new one.
