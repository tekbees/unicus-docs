---
description: >-
  Your company's settings in the portal: details and logo, appearance,
  biometrics, integration, security, and countries and documents.
---

# Company settings

**Settings → Company** groups your company's settings in five sections. Users
without permission to edit them see them read-only.

## General

* **Company details:** the name (the one the person sees on WhatsApp and SMS)
  and the tax ID. To change them, write to [Support](../sdk-web-v5/support.md).
* **Logo:** PNG or JPG up to 2 MB. It appears in the verification, in e-mails
  and in the portal.

## Appearance

* **Capture appearance:** your brand colour, the colour of text on it and the
  colour of the camera buttons. The preview shows the start, the face capture
  and the document capture, and warns you when the contrast is hard to read.
  These colours apply to the button and the verification (see
  [Button reference](../sdk-web-v5/button-reference.md)).
* **Your portal** (administrators): the portal colour for your team and, if
  Tekbees assigned one, your portal's own address to copy and share.

## Biometrics

**Face/ID match level:** how similar the face must be to the document photo to
approve, from **Quick** to **Maximum**. A higher level rejects more
impersonation attempts and also more legitimate attempts with poor-quality
photos. Before you save, the portal simulates how each level would have
worked with your recent enrollments.

## Integration

The credentials (Company ID and API keys), the webhook and its signing, and
connections with other services. See [Integration](integration.md).

## Security

Administrators only:

* **Access only from the company's IPs:** the portal only accepts sign-ins
  from the addresses or ranges (CIDR) you add; it is checked at sign-in and at
  least every hour during the session. Use **Use my current IP** so you
  do not lock yourself out; the portal will not save a list that would block
  your own session.
* **Second-person approval:** publishing a flow and creating a production API
  key need another administrator's approval (see
  [Approvals](users-roles-and-approvals.md#approvals)). It requires at least two
  active administrators.
* **E-mailed code for everyone:** besides the password, every user of your
  company types a code sent to their e-mail when signing in.

## Countries and documents

**Settings → Countries and documents** shows the countries and document types
your company accepts in verifications. The active countries also define which
phone numbers can receive SMS and WhatsApp. This section is read-only:
countries and documents are assigned by Tekbees. To add a country or a
document, write to [Support](../sdk-web-v5/support.md).
