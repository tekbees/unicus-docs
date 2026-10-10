---
description: Versioning policy and release notes of Unicus Web SDK 5.0.
---

# Versioning

{% hint style="warning" %}
**Coming soon.** Web SDK 5.0 is not yet available in production. Tekbees will
announce the release date. On that date Web SDK 4.x stops working and every
integration runs 5.0, including pages that still load the current script URL.
Until then, ask Tekbees for access to the sandbox environment to prepare.
{% endhint %}

The script is published under a major-version path:

| Path | Receives |
| --- | --- |
| `https://unicusbtn.idunicus.com/v5/sdkButton.js` | Every compatible 5.x update (bug fixes, new optional attributes, new events). Browsers may keep the previous copy for a few minutes; pages pick the update up on a later load with no action needed. |
| `https://unicusbtn.idunicus.com/sdkButton.js` | The address of 4.x integrations. From the 5.0 release date it serves the same script as `/v5/`, so existing pages move to 5.0 without changes. |
| `https://unicusbtn.idunicus.com/v6/…` (future) | Breaking changes. Announced in advance; the previous path keeps working during the transition. |

`customElements.get('unicus-btn').version` returns the exact version loaded
(include it when you contact [Support](support.md)).

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
* Button redesign: one piece with the Unicus mark, brand colours applied
  automatically and remembered, `label`, `size` and `radius` attributes, visible states, click during loading honoured, new
  transaction after every finished or exited flow.
* Events: `stepProgress` payloads per step, `resultCode` in `finished` and
  `error`, `2002` *not configured*, `2013` *under review*. Events bubble and
  are `composed`, so a listener on `document` receives them.
* Texts per flow in Spanish and English; language selector on the step
  screens; `label` attribute on the button.
* Security: short-lived sessions bound to one transaction, origin-checked
  messaging, strict iframe permissions, no referrer leakage.
* Public contract (attributes, events, `transactionId`) compatible with 4.x.
* Replaces 4.x for every integration on the release date; the 4.x script URL
  serves 5.0 from then on.
