# Compatibility

Use this page before going live to confirm that the customer site can load the
Unicus Button and run the verification flow correctly.

## Required browser capabilities

| Requirement | Why it matters |
| --- | --- |
| HTTPS | Browsers require HTTPS for camera access and secure iframe behavior. |
| JavaScript enabled | The button is a web component loaded by the Unicus script. |
| Camera permission | Verification requires access to the user's camera. |
| Third-party iframe allowed | The button opens the Unicus flow in an iframe. |
| Stable network connection | The verification flow uploads encrypted biometric and document payloads. |

## Supported browsers

| Platform | Recommended browsers |
| --- | --- |
| Desktop | Latest Chrome, Edge, Firefox, or Safari. |
| Android mobile browser | Latest Chrome or Samsung Internet. |
| iOS mobile browser | Latest Safari. |

{% hint style="warning" %}
iOS browsers other than Safari can behave differently because all iOS browsers
depend on Apple's WebKit engine and camera permission model. For production
validation on iPhone, test with Safari.
{% endhint %}

## Iframe and Content Security Policy

If the customer site uses a Content Security Policy, allow the Unicus domains
provided by Tekbees. At minimum, review these directives:

| Directive | Allow |
| --- | --- |
| `script-src` | `https://unicusbtn.idunicus.com` |
| `frame-src` or `child-src` | The Unicus flow domain, for example `https://id.idunicus.com`. |
| `connect-src` | The Unicus API and flow domains. |
| `img-src` | `https:` and `data:` if the policy is restrictive. |

Do not block the iframe from using camera and geolocation permissions. The
Unicus iframe is created with permission attributes for camera, geolocation,
encrypted media, and microphone.

## Mobile behavior

On mobile devices, the flow can use a full-screen iframe experience so the user
does not lose the connection with the customer page. Users should complete the
process in portrait orientation and avoid refreshing or closing the browser
during verification.

## Pre-production checklist

1. Open the page over HTTPS.
2. Confirm `https://unicusbtn.idunicus.com/sdkButton.js` loads successfully.
3. Confirm `<unicus-btn>` appears and becomes enabled after the transaction is
   created.
4. Confirm the browser asks for camera permission.
5. Complete one successful enrollment verification transaction.
6. Complete one liveness transaction if your application uses liveness.
7. Confirm `OnUnicus:loaded`, `OnUnicus:details`, and `OnUnicus:finished` are
   received by the customer page.
8. Confirm the final transaction appears in the Unicus administrative portal or
   in your webhook/status integration.
