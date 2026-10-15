---
description: >-
  What you can do in the Unicus administrative portal, how it is organised and
  who sees each section.
---

# Overview

The administrative portal is where your company sets up Unicus and reviews the
results of its verifications. From there you:

* compose the **flows** the person goes through (consent, liveness, document,
  signature…) and assign them to each transaction type;
* get your integration **credentials** (Company ID, API keys) and set up the
  **webhook**;
* review every **enrollment, verification and liveness check**, with its PDF
  certificate;
* send a **Unicus Link** to a person by WhatsApp or SMS;
* manage your **team**: users, roles and approvals.

## How to get in

Your company gets access when Tekbees approves its registration (see
[Company registration](sign-up.md)). Sign in at the
[administrative portal](https://app.idunicus.com/) or at your company's own
address, if Tekbees gave you one (for example `yourcompany.idunicus.com`). See
[Access and your account](access-and-account.md).

{% hint style="info" %}
The portal is available in Spanish and English, in light or dark mode. Change
it from your profile menu (bottom left). Dates and times are shown in your
browser's time zone.
{% endhint %}

## How it is organised

| Menu group | Sections | What for |
| --- | --- | --- |
| — | Dashboard, Analytics | The figures of your verifications for the period. See [Dashboard](dashboard.md). |
| Operations | Enrollments, Review queue, Verifications, Liveness checks, Age checks, Reports | Day-to-day work with transactions. See [Transactions and reports](transactions.md). |
| Tools | Unicus Link, E-mail validation, Verify certificate | Send verification links, review e-mail validations and check that a certificate is genuine. |
| Platform | Flows, Webhooks | What the person does in each verification, and your webhook's delivery log. See [Verification flows](flows.md) and [Integration](integration.md). |
| Administration | Users, Roles, Approvals, Audit log | Who has access, what they can do and the record of what each one did. See [Users, roles and approvals](users-roles-and-approvals.md). |
| Settings | Company, Countries and documents | Brand, biometrics, integration and security. See [Company settings](company-settings.md). |

Press **Ctrl + K** (**Cmd + K** on a Mac) for quick search: find a transaction
by its TID, the document number or the person's name.

## Who sees what

Every user has a **role**, and every role has permissions per service: view,
create, edit and delete. A section shows a lock and "Not included in your plan"
when your company does not have that service or your role lacks the
permission; that section cannot be opened.

Some actions are reserved for your company's **administrators**, whatever a
custom role allows: access security, approvals, API keys, webhook signing and
the webhook delivery log, the portal's address and colour, copying or
regenerating the Company ID, and assigning the administrator role or editing an
administrator.

## Getting started

While your company completes its setup, the Dashboard shows the **Getting
started** guide:

1. Customise your brand (logo and colours).
2. Assign a flow to each transaction type you will use.
3. Connect your webhook and turn on its signing.
4. Create an API key for your backend.
5. Invite your team.
6. Run your first verification, for example with a Unicus Link.

Then follow the Web SDK [Integration checklist](../sdk-web-v5/integration-checklist.md).
