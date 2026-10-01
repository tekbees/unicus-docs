---
description: >-
  Move an existing Unicus Button integration to Web SDK 5.0. One line changes in
  the page; the rest is configuration in the portal.
---

# Migration from Web SDK 4.x

## In the page

Replace the script URL. Attributes, events and the `transactionId` property are
unchanged.

{% code overflow="wrap" %}
```html
<!-- before -->
<script src="https://unicusbtn.idunicus.com/sdkButton.js" defer></script>
<!-- after -->
<script src="https://unicusbtn.idunicus.com/v5/sdkButton.js" defer></script>
```
{% endcode %}

Everything below is optional.

| 4.x behaviour | 5.0 behaviour | Action |
| --- | --- | --- |
| Button label fixed. | `label` attribute. | None unless you want another text. |
| Colours from the transaction response. | Same, plus remembered in the browser and overridable with `color` / `textcolor` or CSS variables. | None. |
| `OnUnicus:details` carried only biometric step results. | Also carries `stepProgress` payloads for every step of the flow (consent, signature, OTP, form…). | If your code reads `state.path`, keep it; it still arrives in biometric payloads as `responseType`. Add handling for `state.stepProgress` if you show progress. |
| `OnUnicus:finished` state `{ exited, success }`. | Adds `resultCode`. `2013` means under review. | Treat `success: false` with `resultCode: 2013` as pending. |
| `OnUnicus:error` message only. | Adds `resultCode` and the `2002` *not configured* case. | Show a different message for `2002` if you want. |
| Flow fixed. | Flow configured in the portal. | **Assign a flow to each transaction type before switching the script.** Without it the button shows "Verification not configured". |
| Hand-off by QR only. | QR, WhatsApp, SMS (per company). | Enable the channels you want in the portal. |
| Screen texts fixed. | Titles, descriptions, consent, instructions and form labels per flow, in Spanish and English. | Fill them in the flow editor; empty fields use the Unicus defaults. |

## In the portal

1. Create the flow(s) that reproduce what your users do today (for example
   consent → liveness → document → face match) and assign them to the
   transaction types you use (`enrollment-verify`, `liveness`).
2. Review the company branding: logo and hex colours. The 5.0 screens apply
   them everywhere, including the camera screens.
3. Enable the hand-off channels (QR, WhatsApp, SMS). WhatsApp needs the
   authentication template approved for your company.

## CSP

Add `https://id.idunicus.com` to `frame-src` if your 4.x policy pointed at a
different flow domain, and keep `connect-src` for the API.

## Running both versions

4.x and 5.0 can coexist on different pages of the same site during the
transition. They use the same Customer Token and the same transactions appear
in the portal and webhooks.
