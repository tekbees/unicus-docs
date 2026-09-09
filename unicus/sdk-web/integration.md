---
description: >-
  Choose the right Unicus web integration and understand what the customer
  application must implement.
---

# Integration overview

Unicus provides two web integration options:

| Option | Best for | What the customer application does |
| --- | --- | --- |
| Unicus Button | Embedding verification directly inside a web page or web app. | Adds the Unicus script, renders `<unicus-btn>`, passes the customer token and document data, and listens for events. |
| Unicus Link | Sending a verification URL by email, SMS, WhatsApp, or another channel. | Calls the Unicus API to create a link and sends that link to the user. |

For most web applications, start with **Unicus Button**. It is the fastest path
because the customer application does not need to create iframes, call the
verification endpoints directly, or manage the biometric provider.

## What Unicus Button handles

When the customer renders `<unicus-btn>`, the button handles the standard flow:

1. Create a transaction through Unicus.
2. Receive the transaction id, called `tid`.
3. Detect whether the user must be enrolled or only verified.
4. Open the Unicus verification flow in an iframe.
5. Let the flow fetch company branding, colors, logo, country options, and
   session configuration.
6. Run the verification screens.
7. Emit browser events back to the customer page.

The customer application should not call `/start-process-transaction`,
`/get-restart-session`, `/process-request`, or create the iframe manually when
using Unicus Button.

## Recommended reading order

1. [Unicus Button](unicus-button.md): add the script, render the button, and pass
   the right attributes.
2. [Events](eventos.md): listen for transaction progress and final result.
3. [Appearance configuration](configuracion.md): configure colors and logo in
   the Unicus administrative portal.
4. [Compatibility](compatibilidad.md): confirm browser, iframe, camera, and CSP
   requirements.
5. [Result Codes and References](result-codes-and-references.md): interpret final
   transaction status codes.

## Data required from Unicus

Before integrating, request these values from your Unicus administrator or the
Tekbees support team:

| Value | Used as | Description |
| --- | --- | --- |
| Customer Token | `customerid` attribute | Token assigned to the company in the Unicus administrative portal. |
| Environment | Script/API domain | Sandbox, staging, or production domain provided by Tekbees. |

For enrollment verification you also need the user's document type and document
number.
