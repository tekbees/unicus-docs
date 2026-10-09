---
description: >-
  Browser requirements, Content Security Policy, permissions and the security
  model of Web SDK 5.0. Read before going live.
---

# Compatibility and security

## Requirements

| Requirement | Why |
| --- | --- |
| HTTPS | Browsers only expose the camera to secure origins, and the hand-off links are HTTPS only. `http://localhost` is accepted for development. |
| JavaScript and custom elements | The button is a web component. |
| No domain registration | Your page can be served from any domain. The Customer Token identifies your company; you do not need to send the domain of your site to Tekbees. |
| Third-party iframe allowed | The flow runs in an iframe of the Unicus domain. |
| Camera permission | Liveness and document steps, on the phone or tablet that runs them. Before the camera opens the web app asks the browser for the location, which is recorded with the transaction; the user can refuse and the flow continues without it. |
| Stable connection during capture | Uploads are small (about 1.5 MB per enrolment) but must complete. |

## Supported browsers

| Platform | Browsers |
| --- | --- |
| Android | Chrome and Samsung Internet, current and previous major version. |
| iOS / iPadOS | Safari 15 or later. Other iOS browsers use the same engine but may handle camera permissions differently; validate with Safari. |
| Desktop | Current Chrome, Edge, Firefox and Safari. A desktop or laptop never runs the camera steps, even with a webcam: it runs the steps before the first camera step and hands the rest off to a phone (QR, WhatsApp or SMS). |
| In-app browsers (webviews) | Supported when the host app grants camera permission to the webview. Facebook, Instagram and some banking in-app browsers do not; the user is advised to open the link in the system browser. |

## Content Security Policy

If your page sets a CSP, allow:

| Directive | Value |
| --- | --- |
| `script-src` | `https://unicusbtn.idunicus.com` |
| `frame-src` (or `child-src`) | `https://id.idunicus.com` |
| `connect-src` | `https://unicusapi.idunicus.com` (transaction creation from the button) |

{% code overflow="wrap" %}
```
Content-Security-Policy: script-src 'self' https://unicusbtn.idunicus.com; frame-src https://id.idunicus.com; connect-src 'self' https://unicusapi.idunicus.com
```
{% endcode %}

No `img-src` or `style-src` entry is needed: the button draws its mark as SVG
elements and styles itself with a constructable stylesheet, which a strict
`style-src` does not block. Browsers without constructable stylesheets (for
example Safari before 16.4) fall back to a `<style>` element inside the
button, which needs `style-src 'unsafe-inline'`; without it the button still
works but is shown unstyled there. The button creates its elements with DOM
calls only, so it also works on pages that enforce Trusted Types.

Sandbox environments use other domains provided by Tekbees. The verification
iframe has its own policy; your page only needs the entries above.

If your page sets `Referrer-Policy: no-referrer`, the verification still works:
it learns the origin of your page through an origin-checked handshake with the
button.

## Permissions Policy

The button creates the iframe with
`allow="camera; microphone; geolocation; fullscreen"`, which grants these
features to the Unicus origin only. If your site sends a `Permissions-Policy`
header, it must not deny them to the Unicus origin, for example:

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
* **Abandonment.** Closing or reloading the verification never cancels the
  transaction: progress is kept in Unicus, so the user can resume (a reload
  continues at the first pending step; on a computer, the user can send a new
  link to the phone). Only a camera session cancelled by the user on the
  device running it ends the transaction as cancelled (`2041`); a computer
  that is merely mirroring the phone cannot cancel. A transaction nobody
  resumes expires.
* **Local storage.** The button keeps the company colours in `localStorage`
  (key `unicus-btn:brand:<Customer Token>`) to paint the brand colour on the
  first frame of later visits. No personal data is stored.

## Data usage

| Item | Size | When |
| --- | --- | --- |
| Button script | a few KB | Once per page load; cached by the browser. |
| Verification screens | tens of KB | When the flow opens; cached. |
| Biometric engine | several MB | Once per device; downloaded in the background during the first screens and cached for later transactions. |
| Uploads | about 0.5 MB per selfie and per document side | During the flow. |

## Pre-production checklist

1. Page served over HTTPS.
2. `https://unicusbtn.idunicus.com/v5/sdkButton.js` loads; the button changes
   from neutral to your brand colour.
3. `OnUnicus:loaded` arrives with a `tid`.
4. One complete enrolment from a phone, one from a computer with hand-off.
5. One verification of an already enrolled person.
6. `OnUnicus:finished` arrives on the page and the transaction appears in the
   portal and in your webhook or `query-transaction`.
7. Error path: render the button with `clientid="ID:"` (no number) and
   confirm your page shows a recoverable message on `OnUnicus:error`.
8. Close the verification before finishing and confirm your page handles
   `OnUnicus:exit`; click again and confirm a new `tid` arrives in
   `OnUnicus:loaded`.
