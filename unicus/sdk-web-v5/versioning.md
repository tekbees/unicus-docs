---
description: Versioning policy and release notes of Unicus Web SDK 5.0.
---

# Versioning

The script is published under a major-version path:

| Path | Receives |
| --- | --- |
| `https://unicusbtn.idunicus.com/v5/sdkButton.js` | Every compatible 5.x update (bug fixes, new optional attributes, new events). Pages pick it up on the next load; no action needed. |
| `https://unicusbtn.idunicus.com/v6/…` (future) | Breaking changes. Announced in advance; the previous path keeps working during the transition. |

`customElements.get('unicus-btn').version` returns the exact version loaded.

## 5.0.0

* New web application: modular flows composed in the administrative portal
  (consent, info, liveness, document with server validations, face match,
  signature, OTP, form, age check).
* Lightweight start; biometric engine prefetched in the background and cached;
  designed for low-bandwidth mobile networks.
* Desktop hand-off by QR, WhatsApp or SMS with a live mirror of the phone's
  progress and the final result on the computer.
* One-time hand-off tokens in the URL fragment; the transaction id never
  travels in a link.
* Button redesign: brand colours applied automatically and remembered, `label`
  and `size` attributes, visible states, click during loading honoured, new
  transaction after every finished or exited flow.
* Events: `stepProgress` payloads per step, `resultCode` in `finished` and
  `error`, `2002` *not configured*, `2013` *under review*.
* Texts per flow in Spanish and English; language selector on every screen;
  `label` attribute on the button.
* Security: short-lived sessions bound to one transaction, origin-checked
  messaging, strict iframe permissions, no referrer leakage.
* Public contract (attributes, events, `transactionId`) compatible with 4.x.
