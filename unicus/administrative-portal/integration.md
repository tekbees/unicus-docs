---
description: >-
  Where to find the Company ID in the portal, create API keys, set up and sign
  the webhook, and review its deliveries.
---

# Integration

**Settings → Company → Integration** has what your technical team needs to
connect to Unicus. It includes a guide, **How to integrate with Unicus**, with
the recommended order.

## Company ID

The **Company ID** is the [Customer Token](../sdk-web-v5/customer-token.md): the
`customerid` value of the button and the mobile SDKs. It is public and can go
in your page.

The portal shows only its first characters. **Copy ID** copies the whole ID,
and only the company administrator can do it; other users see "Only the
company administrator can copy the token". Every copy is recorded in the
[Audit log](users-roles-and-approvals.md#audit-log).

**Generate a new ID** (administrator only) invalidates the old one
immediately: the button and the apps that use it stop working until you
publish the new one. Do it only if you need to, and coordinate the change with
your team.

## API keys

API keys authorise your **backend** to call the server endpoints
(`init-api-transaction`, `query-transaction`, `query-id`,
`delete-transaction`). See [Server API and API keys](../sdk-web-v5/server-api.md).

* **Create key:** a name that says where you will use it, and its expiry
  (never, 30, 90, 180 or 365 days). The key is shown **only once**: store it in
  your backend's secret manager.
* **Revoke:** the key stops working immediately. To rotate it, create a new
  one, deploy it and then revoke the old one.
* Keys your company creates are **production** keys. Test credentials are
  provided by Tekbees and are marked as managed by Tekbees.

Only your company's administrators see and manage API keys.

## Webhook

In **Results → Webhook** enter your endpoint's `https://` URL and turn it on.
Unicus sends it the result of every transaction (see
[Webhooks](../sdk-web-v5/webhooks.md)).

**Webhook signing** (administrators): **Generate secret** creates the secret
each delivery is signed with. It is shown **only once**. You can also
**rotate** or **turn off** the secret; changes apply within a minute.

## Delivery log

**Platform → Webhooks** (administrators) shows every delivery of your webhook
over the last days: delivered, failed and queued, with the response.

* Find a delivery by the transaction's TID.
* **Retry** resends a failed delivery, or retry all failed ones at once.
  Failed deliveries can be resent for 7 days.

## Connections

* **Unicus Sign:** to use the Unicus Sign signature step, connect your Unicus
  Sign account's API key. It is stored encrypted and you can change or
  disconnect it.
