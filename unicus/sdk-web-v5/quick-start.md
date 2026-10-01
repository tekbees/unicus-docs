---
description: >-
  Add Unicus Web SDK 5.0 to a page in three steps: load the script, render the
  button, listen to the events.
---

# Quick start

```mermaid
flowchart LR
  A["1. Load the script"] --> B["2. Render unicus-btn"] --> C["3. Listen to OnUnicus events"] --> D["Confirm server side<br/>(webhook / query-transaction)"]
```

## 1. Load the script

Add the script once per page, with `defer`. It weighs about 6 KB and registers
the `<unicus-btn>` element.

{% code overflow="wrap" %}
```html
<script src="https://unicusbtn.idunicus.com/v5/sdkButton.js" defer></script>
```
{% endcode %}

The `/v5/` path receives compatible updates automatically (bug fixes, new
optional attributes). A breaking change will be published under a new path.

## 2. Render the button

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

* `customerid`: your company's Customer Token (Company → Settings in the
  administrative portal).
* `transactiontype`: `enrollment-verify` enrols the person if Unicus does not
  know them yet and verifies them if it does. `liveness` runs a liveness-only
  flow and needs no `clientid`.
* `clientid`: document type and number separated by `:` (`ID`, `FD`, `PP`,
  `DL`).

The button creates the transaction as soon as it is in the page. It is painted
neutral until Unicus answers with your company colours, then it shows the brand
colour and the label. One click opens the verification; if the user clicks
while the transaction is still being created, the flow opens as soon as it is
ready.

## 3. Listen to the events

{% code overflow="wrap" %}
```html
<script>
  const button = document.querySelector('#unicus-verification');

  button.addEventListener('OnUnicus:loaded', ({ detail }) => {
    // Transaction created. Keep the id to correlate the webhook later.
    console.log('tid', detail.transaction.transactionId);
  });

  button.addEventListener('OnUnicus:details', ({ detail }) => {
    // Progress: one event per step the user completes (or retries).
    console.log('step', detail.transaction.state);
  });

  button.addEventListener('OnUnicus:finished', ({ detail }) => {
    const { success, resultCode } = detail.transaction.state;
    if (success) {
      // Continue your process. Confirm with your webhook or query-transaction.
    } else if (resultCode === 2013) {
      // Completed, pending manual review in Unicus.
    } else {
      // Verification failed; show a retry option.
    }
  });

  button.addEventListener('OnUnicus:exit', () => {
    // The user closed the verification before finishing.
  });

  button.addEventListener('OnUnicus:error', ({ detail }) => {
    // The transaction could not be created or the flow could not continue.
    console.error(detail.message, detail.resultCode);
  });
</script>
```
{% endcode %}

Events bubble, so you can also listen on `document`:
`document.addEventListener('OnUnicus:finished', handler)`.

## Full HTML example

{% code overflow="wrap" %}
```html
<!doctype html>
<html lang="es">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <script src="https://unicusbtn.idunicus.com/v5/sdkButton.js" defer></script>
  </head>
  <body>
    <h1>Verify your identity</h1>

    <unicus-btn
      id="unicus-verification"
      customerid="<CUSTOMER_TOKEN>"
      transactiontype="enrollment-verify"
      clientid="ID:123456789"
      language="es">
    </unicus-btn>

    <p id="status"></p>

    <script>
      const button = document.querySelector('#unicus-verification');
      const status = document.querySelector('#status');
      let tid = null;

      button.addEventListener('OnUnicus:loaded', ({ detail }) => {
        tid = detail.transaction.transactionId;
      });

      button.addEventListener('OnUnicus:finished', async ({ detail }) => {
        const { success, resultCode } = detail.transaction.state;
        status.textContent = success
          ? 'Identity verified.'
          : resultCode === 2013
            ? 'Verification under review.'
            : 'We could not verify your identity. Please try again.';
        // Authoritative result: ask your backend, which queries Unicus with the tid.
        await fetch('/api/verification/' + tid + '/refresh', { method: 'POST' });
      });

      button.addEventListener('OnUnicus:error', ({ detail }) => {
        status.textContent = 'Verification is not available right now.';
        console.error('Unicus', detail);
      });
    </script>
  </body>
</html>
```
{% endcode %}

## Rendering the button from JavaScript

The element can be created dynamically. Set the attributes before appending it
to the DOM so the transaction is created once.

{% code overflow="wrap" %}
```js
const button = document.createElement('unicus-btn');
button.setAttribute('customerid', customerToken);
button.setAttribute('transactiontype', 'enrollment-verify');
button.setAttribute('clientid', `ID:${documentNumber}`);
button.addEventListener('OnUnicus:finished', onFinished);
container.appendChild(button);
```
{% endcode %}

Changing `customerid`, `clientid`, `transactiontype` or `data-flow-id` on a
rendered button discards the current transaction and creates a new one.

## Framework notes

### React

{% code overflow="wrap" %}
```jsx
import { useEffect, useRef } from 'react';

export function UnicusButton({ customerToken, documentNumber, onFinished }) {
  const ref = useRef(null);

  useEffect(() => {
    const el = ref.current;
    if (!el) return;
    const handler = (event) => onFinished(event.detail.transaction.state);
    el.addEventListener('OnUnicus:finished', handler);
    return () => el.removeEventListener('OnUnicus:finished', handler);
  }, [onFinished]);

  return (
    <unicus-btn
      ref={ref}
      customerid={customerToken}
      transactiontype="enrollment-verify"
      clientid={`ID:${documentNumber}`}
    />
  );
}
```
{% endcode %}

With TypeScript, declare the element once:

{% code overflow="wrap" %}
```ts
declare module 'react' {
  namespace JSX {
    interface IntrinsicElements {
      'unicus-btn': React.DetailedHTMLProps<React.HTMLAttributes<HTMLElement>, HTMLElement> & {
        customerid: string;
        transactiontype: 'enrollment-verify' | 'liveness';
        clientid?: string;
        language?: 'es' | 'en';
        label?: string;
        size?: 'lg';
      };
    }
  }
}
```
{% endcode %}

### Vue 3

Tell the compiler that `unicus-btn` is a custom element:

{% code overflow="wrap" %}
```js
// vite.config.js
import vue from '@vitejs/plugin-vue';

export default {
  plugins: [
    vue({ template: { compilerOptions: { isCustomElement: (tag) => tag === 'unicus-btn' } } }),
  ],
};
```
{% endcode %}

### Angular

Add `CUSTOM_ELEMENTS_SCHEMA` to the module or standalone component that renders
the button, and attach listeners with `@ViewChild` + `addEventListener`.
