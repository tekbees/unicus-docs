---
description: >-
  Send the user a verification link instead of embedding the button: from the
  administrative portal by WhatsApp or SMS, or from your backend.
---

# Unicus Link

Unicus Link sends the person a verification link. The person opens it and goes
through the same Web SDK 5.0 verification as with the button: the flow
assigned in the portal, your company's colours and logo, and the hand-off to
the phone when the link is opened on a computer.

Use it when the person does not go through your website or web app. To verify
them inside your page, use the [button](quick-start.md) instead.

{% hint style="info" %}
Do not combine Unicus Link and the button for the same person and process. The
button creates and opens its own transaction; a link creates another one.
{% endhint %}

## Option 1: from the administrative portal

In the [administrative portal](https://app.idunicus.com/), open
**Tools → Unicus Link**:

1. **Channel.** Choose WhatsApp or SMS. Only the channels enabled for your
   company appear; if you see none, ask [Support](support.md).
2. **The person's phone.** The number, with its country code. Messages are
   only sent to numbers in your company's active countries.
3. **The person's document.** The type (national ID, foreigner ID, passport or
   driving licence) and the number.
4. Select **Send**.

Before sending, the portal shows **what the person receives**. The WhatsApp
message uses a template approved by WhatsApp and cannot be customised.

* The link is valid for a short time and can be used once.
* The transaction appears in the portal with the others, and your backend
  receives the result through the [webhook](webhooks.md).
* To send the link through another channel (e-mail, your own chat) or to pick
  a flow variant, create it from your backend (option 2).

Using Unicus Link in the portal requires the Unicus Link permission in your
role (see
[Users, roles and approvals](../administrative-portal/users-roles-and-approvals.md)).

## Option 2: from your backend

<mark style="color:green;">`POST`</mark> `<unicus-server-api-url>/init-api-transaction`

Call it only from your server, with your company's [API key](server-api.md).
Unicus returns the link and you send it through the channel you prefer.

### Headers

| Name | Required | Description |
| --- | --- | --- |
| `Authorization` | Yes | `Bearer <API_KEY>` of the target environment. |
| `Content-Type` | Yes | `application/json` |

### Body

| Name | Required | Example | Description |
| --- | --- | --- | --- |
| `document` | Yes | `123456789` | The person's document number. |
| `documentType` | Yes | `ID` | Document type: `ID` (national ID), `FD` (foreigner ID), `PP` (passport) or `DL` (driving licence). |
| `flowId` | No | `credit-with-signature` | Identifier of a flow variant to use instead of the default flow. |

See every field (including how to ask for a new link for an open transaction)
in [Server API and API keys](server-api.md#init-api-transaction).

### Example

{% code overflow="wrap" %}
```bash
curl --request POST '<unicus-server-api-url>/init-api-transaction' \
  --header 'Authorization: Bearer <API_KEY>' \
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
  "resultMessage": "Enrollment Transaction was created.",
  "tid": "<TID>",
  "url": "<link to the Unicus web app>"
}
```
{% endcode %}

Send the `url` to the person and store the `tid` with your user: the webhook
and the result query refer to it.

## What the person sees

* **On a phone or tablet,** the whole verification runs in the browser.
* **On a computer,** the steps before the first camera step run there, and the
  person continues on the phone by QR code, WhatsApp or SMS (see
  [Flows and hand-off](flows-and-handoff.md)).
* The address bar is cleaned as soon as the link is read: the transaction id
  does not stay in the browser history.
* The link works once. If the person opens it again, or opens it after the
  transaction finished or expired, they see "This link is no longer active".
  To give them another link for the same open transaction, use `reissueLink`
  (see [Server API and API keys](server-api.md#init-api-transaction)).
* The verification opens in the language of the person's browser (Spanish or
  English); they can change it on the step screens.

## Getting the result

A link has no page of yours around it, so there are no `OnUnicus:*` browser
events. Your backend receives the result through the
[webhook](webhooks.md) (`TRANSACTION_FINALIZED`) or by calling
[Get a transaction status](transaction-status.md) with the `tid`. In the
portal, the transaction appears in
[Transactions](../administrative-portal/transactions.md).

## Good practices

* The link identifies one transaction: send it only to the person who has to
  verify, and do not publish it or reuse it for someone else.
* Without a flow assigned to the transaction type, the link cannot be created
  (result code `2002`): assign the flow in the portal first.
* If the person lost the link or it expired, ask for a new link for the same
  transaction (`reissueLink`) or create a new one.
