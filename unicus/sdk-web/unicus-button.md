---
description: >-
  Add the Unicus Web Button to a website and start enrollment verification or
  liveness from the customer page.
---

# Unicus Button

Unicus Button is a web component that starts the Unicus verification flow from
your web page. The customer application integrates **Unicus** only: add the
script, render `<unicus-btn>`, pass the required attributes, and listen for
events.

{% hint style="info" %}
Do not create Unicus transactions manually when using `<unicus-btn>`. The button
creates the transaction, opens the iframe, and reports progress through browser
events.
{% endhint %}

## 1. Add the script

Add the Unicus Button script once in your page, preferably with `defer`.

{% code overflow="wrap" %}
```html
<script
  type="text/javascript"
  src="https://unicusbtn.idunicus.com/sdkButton.js"
  defer>
</script>
```
{% endcode %}

## 2. Render the button

Use `transactiontype="enrollment-verify"` when the user must be enrolled if they
do not already exist in Unicus, or verified if they are already enrolled.

{% code overflow="wrap" %}
```html
<unicus-btn
  id="unicus-verification"
  customerid="<CUSTOMER_TOKEN>"
  transactiontype="enrollment-verify"
  clientid="ID:123456789">
</unicus-btn>
```
{% endcode %}

Use `transactiontype="liveness"` when the flow only needs a liveness check.
`clientid` is not required for liveness.

{% code overflow="wrap" %}
```html
<unicus-btn
  id="unicus-liveness"
  customerid="<CUSTOMER_TOKEN>"
  transactiontype="liveness">
</unicus-btn>
```
{% endcode %}

## Attribute reference

| Attribute | Required | Example | Description |
| --- | --- | --- | --- |
| `customerid` | Yes | `<CUSTOMER_TOKEN>` | Customer token generated in the Unicus administrative portal. This is the same value described as Customer Token in other API pages. |
| `transactiontype` | Yes | `enrollment-verify` | Public flow requested by the customer app. Supported values are `enrollment-verify` and `liveness`. |
| `clientid` | Required for `enrollment-verify` | `ID:123456789` | Document type and document number separated by `:`. The value before `:` is sent as `documentType`; the value after `:` is sent as `externalDatabaseRefID`. |
| `language` | Optional | `en` | Button label language. If omitted, the browser language is used when supported. The verification flow language is resolved by the Unicus flow configuration. |

Valid document types for `clientid`:

| Code | Description |
| --- | --- |
| `ID` | National ID document. |
| `FD` | Foreign document. |
| `PP` | Passport. |
| `DL` | Driver license. |

## What happens internally

For `transactiontype="enrollment-verify"`, the button sends this transaction
creation request to Unicus:

{% code overflow="wrap" %}
```json
{
  "documentType": "ID",
  "externalDatabaseRefID": "123456789",
  "process": "face-id"
}
```
{% endcode %}

The customer token is sent as the `X-Customer-ID` header. If the transaction is
created successfully, Unicus returns a `tid`, company colors, and the next flow
decision.

The button then converts the public `enrollment-verify` request into one of the
internal flow values:

| Backend decision | Internal flow opened by the button | User experience |
| --- | --- | --- |
| User is not enrolled | `enrollment` | Face verification plus document capture. |
| User is already enrolled | `verify` | Face verification only. |

For `transactiontype="liveness"`, the button creates a liveness transaction and
opens the liveness flow.

## Full HTML example

{% code overflow="wrap" %}
```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <script
      type="text/javascript"
      src="https://unicusbtn.idunicus.com/sdkButton.js"
      defer>
    </script>
  </head>
  <body>
    <unicus-btn
      id="unicus-verification"
      customerid="<CUSTOMER_TOKEN>"
      transactiontype="enrollment-verify"
      clientid="ID:123456789">
    </unicus-btn>

    <script>
      const unicusButton = document.querySelector('#unicus-verification');

      unicusButton.addEventListener('OnUnicus:loaded', ({ detail }) => {
        console.log('Unicus transaction created', detail.transaction.transactionId);
      });

      unicusButton.addEventListener('OnUnicus:finished', ({ detail }) => {
        console.log('Unicus transaction finished', detail);
      });

      unicusButton.addEventListener('OnUnicus:error', ({ detail }) => {
        console.error('Unicus transaction error', detail);
      });
    </script>
  </body>
</html>
```
{% endcode %}

## Framework notes

### React

React can render the custom element directly. Attach listeners with a `ref`.

{% code overflow="wrap" %}
```jsx
import { useEffect, useRef } from 'react';

export function UnicusVerificationButton() {
  const ref = useRef(null);

  useEffect(() => {
    const button = ref.current;
    if (!button) return;

    const onFinished = (event) => {
      console.log('Unicus finished', event.detail);
    };

    button.addEventListener('OnUnicus:finished', onFinished);
    return () => button.removeEventListener('OnUnicus:finished', onFinished);
  }, []);

  return (
    <unicus-btn
      ref={ref}
      customerid="<CUSTOMER_TOKEN>"
      transactiontype="enrollment-verify"
      clientid="ID:123456789"
    />
  );
}
```
{% endcode %}

### Vue

Vue 3 can render custom elements directly. If your build warns about unresolved
components, configure Vue to treat `unicus-btn` as a custom element.

{% code overflow="wrap" %}
```js
// vite.config.js
import vue from '@vitejs/plugin-vue';

export default {
  plugins: [
    vue({
      template: {
        compilerOptions: {
          isCustomElement: (tag) => tag === 'unicus-btn',
        },
      },
    }),
  ],
};
```
{% endcode %}

### Angular

Add `CUSTOM_ELEMENTS_SCHEMA` to the Angular module that renders the button.

{% code overflow="wrap" %}
```ts
import { CUSTOM_ELEMENTS_SCHEMA, NgModule } from '@angular/core';

@NgModule({
  schemas: [CUSTOM_ELEMENTS_SCHEMA],
})
export class AppModule {}
```
{% endcode %}

## Security and iframe behavior

The button opens the Unicus flow inside an iframe. The transaction id and flow
type are included in the iframe URL, but the customer token is not added to that
URL. The token is sent to the iframe through a secure `postMessage` handshake
after the iframe is ready.

If your site has a Content Security Policy, allow:

| Directive | Required value |
| --- | --- |
| `script-src` | `https://unicusbtn.idunicus.com` |
| `frame-src` or `child-src` | The Unicus flow domain provided by Tekbees, for example `https://id.idunicus.com`. |
| `connect-src` | The Unicus API and flow domains provided by Tekbees. |
| `img-src` | `https:` and `data:` if your policy is restrictive. |

The iframe requires access to camera, geolocation, encrypted media, and
microphone permissions. Users must grant camera access to complete verification.

## Common mistakes

| Symptom | What to check |
| --- | --- |
| The button stays disabled or gray | Confirm `customerid` is valid and the Unicus API environment matches the token. |
| `OnUnicus:error` says `clientid` is invalid | Confirm `clientid` uses `DOCUMENT_TYPE:DOCUMENT_NUMBER`, for example `ID:123456789`. |
| User is blocked | The transaction creation response can return result code `2052`; contact Tekbees support or review the user status in Unicus. |
| Events never fire | Attach listeners to the existing DOM element with `document.querySelector(...)`, not to a detached element created only in JavaScript. |
| The iframe does not open | Check browser popup/script blockers, CSP `frame-src`, and that the script loaded from `https://unicusbtn.idunicus.com/sdkButton.js`. |
