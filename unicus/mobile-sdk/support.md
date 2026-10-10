---
description: >-
  How to get help with a Unicus mobile SDK integration and what to send so the
  team can diagnose it quickly.
---

# Support

The most important thing for us is that your integration works. Contact us
through:

* E-mail: [support@tekbees.com](mailto:support@tekbees.com)
* Your project manager (assigned when the contract started).

Before writing, check [Errors and troubleshooting](errors-and-troubleshooting.md)
and the release checklist of your platform:
[Android](android/release-checklist.md), [iOS](ios/release-checklist.md),
[Flutter](flutter/release-checklist.md).

## What to send

| Item | How to get it |
| --- | --- |
| `tid` of the transaction | `result.tid`, or the transaction list of the portal. |
| Platform and SDK version | Android, iOS or Flutter, and the version (see [Versioning](versioning.md#versions)). |
| Environment | `DEV`, `STAGING` or `PRODUCTION`. Never send the Customer Token itself. |
| Device and OS version | For example "Samsung A14, Android 14" or "iPhone 13, iOS 18.1". On Android, also the Android System WebView version for problems with the flow screens. |
| What your app received | The `outcome`, `resultCode` and `rejectionReason` of the result, or the `code`, `resultCode`, HTTP status and `message` of the error. |
| Configuration | `uiStepMode`, and whether `secureScreens`, `prependConsent` and `resumeOpenTransactions` are on. |
| API logs | With `enableApiLogging` on and `includeSensitiveApiLogData` **off**. |
| Date and time | With the time zone. |
| Steps to reproduce | What the user did, and a screenshot or video if the problem is visible. |

{% hint style="danger" %}
Never send logs captured with `includeSensitiveApiLogData` turned on, photos of
real identity documents, or Customer Tokens. The sanitized logs and the `tid`
are enough: Tekbees can look up the transaction in Unicus.
{% endhint %}
