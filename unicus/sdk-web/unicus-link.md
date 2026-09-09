---
description: >-
  Create a Unicus verification link from the administrative portal or API and
  send it to the user outside an embedded web page.
---

# Unicus Link

Unicus Link lets you create a verification URL that can be sent to a user by
email, SMS, WhatsApp, or another communication channel.

Use this option when the user does not need to complete the process inside your
website or web app. If you want to embed the verification flow directly in your
page, use [Unicus Button](unicus-button.md) instead.

{% hint style="info" %}
Do not combine Unicus Link and Unicus Button for the same transaction. Unicus
Button creates and opens its own transaction automatically.
{% endhint %}

## Option 1: Create a link from the administrative portal

1. Sign in to the [Unicus administrative portal](https://app.idunicus.com/).
2. Open **Unicus Link**.
3. Select **Generate Link**.
4. Enter the user's document data.
5. Send the generated URL to the user through your preferred channel.

<figure><img src="../.gitbook/assets/image (21).png" alt="" width="375"><figcaption></figcaption></figure>

## Option 2: Create a link from the API

Use the API option when your backend needs to create the verification URL
programmatically.

### Endpoint

<mark style="color:green;">`POST`</mark> `<unicus-server-api-url>/init-api-transaction`

### Headers

| Name | Required | Description |
| --- | --- | --- |
| `X-Customer-ID` | Yes | [Customer Token](how-to-get-customer-token.md) for the target environment. |

### Request body

| Name | Required | Example | Description |
| --- | --- | --- | --- |
| `document` | Yes | `123456789` | User document number. |
| `documentType` | Yes | `ID` | User document type. Supported values are `ID`, `FD`, `PP`, and `DL`. |

### Curl example

{% code overflow="wrap" %}
```bash
curl --location '<unicus-server-api-url>/init-api-transaction' \
  --header 'X-Customer-ID: <CUSTOMER_TOKEN>' \
  --header 'Content-Type: application/json' \
  --data '{
    "document": "123456789",
    "documentType": "ID"
  }'
```
{% endcode %}

### Successful response

{% code overflow="wrap" %}
```json
{
  "success": true,
  "wasProcessed": true,
  "error": false,
  "path": "init-api-transaction",
  "resultCode": 0,
  "resultMessage": "A Transaction Enrollment was created.",
  "additionalSessionData": {
    "isAdditionalDataPartiallyIncomplete": true
  },
  "elapsedPerformanceTime": 644,
  "url": "https://id.idunicus.com/?token=<TID>&process=enrollment"
}
```
{% endcode %}

Send the `url` value to the user. When the user finishes the identity validation
process, Unicus can notify your backend through the configured
[webhook](types-webhook.md). You can also query the final result with
[Get a transaction status](get-a-transaction-status.md).
