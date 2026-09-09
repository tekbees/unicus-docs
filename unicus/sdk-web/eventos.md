---
description: >-
  Listen to Unicus Web Button events from the customer page and react to
  transaction progress, completion, and errors.
---

# Events

Unicus Button emits browser events from the `<unicus-btn>` element. Use these
events to update your application UI, store the transaction id, and react when
the user finishes or leaves the flow.

{% hint style="info" %}
Server-side webhooks are still recommended for authoritative back-office
processing. Browser events are useful for the user interface and immediate page
behavior.
{% endhint %}

## Subscribe to events

Attach listeners to the `<unicus-btn>` element that exists in the page.

{% code overflow="wrap" %}
```html
<unicus-btn
  id="unicus-verification"
  customerid="<CUSTOMER_TOKEN>"
  transactiontype="enrollment-verify"
  clientid="ID:123456789">
</unicus-btn>

<script>
  const unicusButton = document.querySelector('#unicus-verification');

  unicusButton.addEventListener('OnUnicus:loaded', ({ detail }) => {
    console.log('Transaction created', detail.transaction.transactionId);
  });

  unicusButton.addEventListener('OnUnicus:details', ({ detail }) => {
    console.log('Transaction progress', detail.transaction.state);
  });

  unicusButton.addEventListener('OnUnicus:finished', ({ detail }) => {
    console.log('Transaction finished', detail);
  });

  unicusButton.addEventListener('OnUnicus:error', ({ detail }) => {
    console.error('Transaction error', detail);
  });
</script>
```
{% endcode %}

Do not use `document.createElement("unicus-btn")` only to subscribe to events.
Listeners must be attached to the element that is rendered in the DOM.

## Event summary

| Event | When it happens | Recommended action |
| --- | --- | --- |
| `OnUnicus:loaded` | The button created the transaction and received a `tid`. | Store `transaction.transactionId` if your application needs to correlate the transaction. Enable any UI that depends on the transaction being ready. |
| `OnUnicus:details` | The verification flow reports progress. This event can fire several times. | Update progress indicators. Do not treat this as final success or failure. |
| `OnUnicus:finished` | The user completed the flow or closed the final Unicus screen. | Close local modals, refresh user state, and optionally call `query-transaction` or wait for your webhook. |
| `OnUnicus:error` | The button or flow could not continue. | Show a recoverable error and log the payload with the transaction id when available. |
| `OnUnicus:exit` | Legacy exit signal, if emitted by older flows. | Treat the flow as closed. Prefer `OnUnicus:finished` for new integrations. |

## `OnUnicus:loaded`

This event confirms that Unicus created the transaction.

{% code overflow="wrap" %}
```json
{
  "status_error": false,
  "loaded": true,
  "message": "Unicus SDK: loaded successfully",
  "link": "https://id.idunicus.com/?token=<TID>",
  "transaction": {
    "transactionId": "<TID>",
    "clientid": "123456789"
  }
}
```
{% endcode %}

For liveness transactions, `transaction.clientid` can be an empty string.

## `OnUnicus:details`

This event reports intermediate progress while the verification flow is running.
It can fire more than once during the same transaction.

{% code overflow="wrap" %}
```json
{
  "status_error": false,
  "message": "Unicus SDK: User is active on transaction",
  "transaction": {
    "exited": false,
    "transactionId": "<TID>",
    "state": {
      "path": "match-3d-2d-idscan",
      "success": true,
      "wasProcessed": true,
      "resultCode": 200,
      "resultMessage": "Success",
      "isFrontSide": true,
      "matchLevel": 4
    }
  }
}
```
{% endcode %}

The `state.path` value helps identify which step reported the event. Common
values include:

| `state.path` | Meaning |
| --- | --- |
| `match-3d-2d-idscan` | Face and document step. |
| `match-3d-3d` | Face verification step. |
| `liveness-3d` | Liveness step. |

Only use fields that your application needs. The payload can include additional
technical fields depending on the verification step.

## `OnUnicus:finished`

This event is emitted when the Unicus flow ends from the customer page
perspective.

{% code overflow="wrap" %}
```json
{
  "status_error": false,
  "message": "Unicus SDK: User ended transaction",
  "transaction": {
    "exited": false,
    "transactionId": "<TID>",
    "state": {
      "exited": true,
      "success": true
    }
  }
}
```
{% endcode %}

For the final authoritative transaction result, use your configured webhook or
call [Get a transaction status](get-a-transaction-status.md) with the `tid`.
Browser events can be affected by page refreshes, browser navigation, or network
conditions.

## `OnUnicus:error`

This event is emitted when the button cannot create or continue a transaction.

{% code overflow="wrap" %}
```json
{
  "status_error": true,
  "message": "Unicus SDK: cannot connect with server, check [customerid] or [clientid] might be wrong"
}
```
{% endcode %}

Common causes:

| Cause | What to check |
| --- | --- |
| Invalid customer token | Confirm the `customerid` attribute contains the Customer Token for the same environment as the script. |
| Invalid document data | Confirm `clientid` uses `DOCUMENT_TYPE:DOCUMENT_NUMBER`, for example `ID:123456789`. |
| User blocked | The transaction creation response can return result code `2052`. Review the user status in Unicus or contact Tekbees support. |
| CSP or iframe blocked | Confirm your site allows the Unicus script, iframe, and network domains. |
| Browser permission denied | Ask the user to allow camera access and retry the transaction. |

## Recommended implementation pattern

Use `OnUnicus:loaded` to store the `tid`, `OnUnicus:details` for progress UI,
and `OnUnicus:finished` to close your local UI or trigger a status refresh.
Replace the helper functions in this example with your application logic.

{% code overflow="wrap" %}
```js
let unicusTid = null;

unicusButton.addEventListener('OnUnicus:loaded', ({ detail }) => {
  unicusTid = detail.transaction.transactionId;
});

unicusButton.addEventListener('OnUnicus:details', ({ detail }) => {
  renderProgress(detail.transaction.state);
});

unicusButton.addEventListener('OnUnicus:finished', async () => {
  if (!unicusTid) return;
  await refreshTransactionStatus(unicusTid);
});

unicusButton.addEventListener('OnUnicus:error', ({ detail }) => {
  showUnicusError(detail.message);
});
```
{% endcode %}
