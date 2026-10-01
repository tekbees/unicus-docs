---
description: >-
  Browser requirements, Content Security Policy, permissions and the security
  model of Web SDK 4.0. Read before going live.
---

# Compatibility and security

## Requirements

| Requirement | Why |
| --- | --- |
| HTTPS | Browsers only expose the camera to secure origins, and the hand-off links are HTTPS only. `http://localhost` is accepted for development. |
| JavaScript and custom elements | The button is a web component. |
| Third-party iframe allowed | The flow runs in an iframe of the Unicus domain. |
| Camera permission | Liveness and document steps. Microphone and geolocation are requested only when a flow needs them (geolocation is used to pre-select the document country). |
| Stable connection during capture | Uploads are small (about 1.5 MB per enrolment) but must complete. |

## Supported browsers

| Platform | Browsers |
| --- | --- |
| Android | Chrome and Samsung Internet, current and previous major version. |
| iOS / iPadOS | Safari 15 or later. Other iOS browsers use the same engine but may handle camera permissions differently; validate with Safari. |
| Desktop | Current Chrome, Edge, Firefox and Safari. Desktop can run the whole flow when it has a camera; document steps are better on a phone and the app offers the hand-off. |
| In-app browsers (webviews) | Supported when the host app grants camera permission to the webview. Facebook, Instagram and some banking in-app browsers do not; the user is advised to open the link in the system browser. |

## Content Security Policy

If your page sets a CSP, allow:

| Directive | Value |
| --- | --- |
| `script-src` | `https://unicusbtn.idunicus.com` |
| `frame-src` (or `child-src`) | `https://id.idunicus.com` |
| `connect-src` | `https://unicusapi.idunicus.com` (transaction creation from the button) |
| `img-src` | `data:` (the button draws its mark inline) |

Sandbox environments use other domains provided by Tekbees. The verification
iframe has its own policy; your page only needs the entries above.

## Permissions Policy

The button creates the iframe with
`allow="camera; microphone; geolocation; fullscreen"`. If your site sends a
`Permissions-Policy` header, it must not deny these features to the Unicus
origin, for example:

```
Permissions-Policy: camera=(self "https://id.idunicus.com"), geolocation=(self "https://id.idunicus.com")
```

## Security model

* **No secrets in the browser.** The Customer Token is a public identifier of
  your company. Transactions are authorised by a short-lived session issued by
  Unicus and bound to one transaction; it is never in a URL.
* **No transaction id in links.** QR, SMS and WhatsApp links carry a one-time
  token in the URL fragment, which browsers never send to servers. The web app
  sends no `Referer`.
* **Origin-checked messaging.** The web app only talks to the page that
  embedded it (origin learned during the handshake) and the button only accepts
  messages from the iframe it created.
* **Server authority.** Step order, completion and results are decided by
  Unicus, not by the browser.
* **Biometric data** never reaches the customer page. Events carry result
  codes and metadata only. Images and templates are processed by Unicus under
  the data processing agreement of your company.
* **Abandonment.** A verification closed before completion is recorded as
  cancelled by the user (`2041`). Only the device running the capture can
  cancel; a computer that is merely mirroring the phone cannot.

## Data usage

| Item | Size | When |
| --- | --- | --- |
| Button script | a few KB | Once per page load; cached by the browser. |
| Verification screens | tens of KB | When the flow opens; cached. |
| Biometric engine | several MB | Once per device; downloaded in the background during the first screens and cached for later transactions. |
| Uploads | about 0.5 MB per selfie and per document side | During the flow. |

## Pre-production checklist

1. Page served over HTTPS.
2. `https://unicusbtn.idunicus.com/v4/sdkButton.js` loads; the button changes
   from neutral to your brand colour.
3. `OnUnicus:loaded` arrives with a `tid`.
4. One complete enrolment from a phone, one from a computer with hand-off.
5. One verification of an already enrolled person.
6. `OnUnicus:finished` arrives on the page and the transaction appears in the
   portal and in your webhook or `query-transaction`.
7. Error path: open the button with an invalid `clientid` and confirm your page
   shows a recoverable message.
