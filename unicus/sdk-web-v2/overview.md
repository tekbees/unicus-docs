---
description: >-
  What Unicus Web SDK 2.0 is, what changed from the previous button, and what a
  customer web application has to implement.
---

# Overview

Unicus Web SDK 2.0 is the new web integration of the Unicus identity
verification platform. From the customer page it is still one script and one
HTML element: `<unicus-btn>`. Everything else — transaction creation, the
verification screens, the camera, the hand-off to the user's phone and the
result — is handled by Unicus.

{% hint style="info" %}
Web SDK 2.0 keeps the public contract of the previous button: the same
attributes (`customerid`, `transactiontype`, `clientid`, `language`), the same
`OnUnicus:*` browser events and the same `transactionId` property. An existing
integration only has to change the script URL. See
[Migration from Web SDK 1.x](migration-from-v1.md).
{% endhint %}

## What is new

| Area | Web SDK 1.x | Web SDK 2.0 |
| --- | --- | --- |
| Flow | Fixed: face, then document. | **Modular.** Your company composes the flow in the Unicus administrative portal: consent, instructions, liveness, document, face match, electronic signature, OTP, data form, age check. The web app runs whatever flow is assigned to the transaction type. |
| Weight | ~9 MB and 7 sequential requests before the camera opened. | ~30 KB of application code; the biometric engine (~8.7 MB) is downloaded once in the background while the user reads the first screens and cached for later transactions. Designed for low-bandwidth mobile networks. |
| Desktop users | Hand-off by QR code. | Hand-off by QR code, WhatsApp or SMS (channels enabled per company). The desktop page mirrors the phone's progress step by step and shows the final result. |
| Hand-off links | Transaction id in the URL. | One-time hand-off tokens in the URL fragment. The transaction id never travels in a link, a Referer, or a server log. |
| Button | Fixed label, loading state. | Brand colours applied automatically, custom label, two sizes, visible loading / ready / verifying / error states, one click opens the flow even while the transaction is still being created. |
| Errors | Generic messages. | Machine-readable result codes and rejection reasons in every event (see [Result codes](result-codes.md)). |
| Security | Secrets in the bundle, `postMessage('*')`. | No secrets in the browser bundle, origin-checked messaging, 15-minute device session bound to the transaction, strict iframe permissions. |

## How it works

1. The customer page loads `sdkButton.js` and renders `<unicus-btn>` with the
   company's **Customer Token** and the user's document.
2. The button creates the transaction in Unicus as soon as it is rendered and
   emits `OnUnicus:loaded` with the transaction id (`tid`).
3. When the user clicks, the button opens the Unicus web app in a full-screen
   iframe. The web app fetches the session: company branding and the **flow**
   assigned to the transaction.
4. The web app runs the flow. On a desktop computer it runs the pre-camera
   steps locally and hands the camera steps off to the user's phone; the
   desktop mirrors the progress.
5. Each step reports progress to the customer page through `OnUnicus:details`.
   The end of the flow produces `OnUnicus:finished` (or `OnUnicus:exit` if the
   user left before finishing).
6. Your backend receives the authoritative result through your
   [webhook](../sdk-web/types-webhook.md) or by calling
   [Get a transaction status](../sdk-web/get-a-transaction-status.md).

The customer application never calls `/start-process-transaction`,
`/get-restart-session` or `/process-request` directly and never creates the
iframe manually.

## What you need from Unicus

| Value | Where it goes | Description |
| --- | --- | --- |
| Customer Token | `customerid` attribute | Public token of your company, generated in the administrative portal (Company → Settings). It is safe to render in HTML: it identifies the company, it does not authorise anything by itself. One token per environment. |
| Script URL | `<script src>` | `https://unicusbtn.idunicus.com/v2/sdkButton.js` in production. Tekbees provides the sandbox URL. |
| Flow | Administrative portal | At least one flow assigned to the transaction type you use. Without it the button shows "Verification not configured" (result code `2002`). |

For enrollment and verification you also need the user's document type and
document number (`clientid` attribute).

## Reading order

1. [Quick start](quick-start.md): script, element, events, full example.
2. [Button reference](button-reference.md): attributes, states, appearance,
   lifecycle.
3. [Events](events.md): payload of every event and how to react.
4. [Flows and hand-off](flows-and-handoff.md): what the user sees for each step
   type and how the phone hand-off works.
5. [Result codes](result-codes.md) and
   [Errors and troubleshooting](errors-and-troubleshooting.md).
6. [Compatibility and security](compatibility-and-security.md) before going
   live.
