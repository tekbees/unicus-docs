---
description: >-
  Your team signs in to the portal with the company account (Microsoft Entra
  ID, Google Workspace or Okta) and its users are created and deactivated
  automatically (SCIM).
---

# Single sign-on (SSO)

With single sign-on, your team signs in to the portal with the company account
instead of a Unicus password. Your identity provider (Entra ID, Google Workspace
or Okta) confirms who each person is and applies its own rules: multi-factor
authentication, account lockout, sign-in hours.

A **company administrator** sets it up in **Settings → Company → Security**,
together with whoever manages the identity provider. It uses the OpenID Connect
(OIDC) standard.

{% hint style="info" %}
SSO does not create users by itself: only people who already have a user in
your company in Unicus can sign in. Create them in
[Users](users-roles-and-approvals.md) or let your provider create them with
[automatic provisioning (SCIM)](#automatic-provisioning-scim).
{% endhint %}

## Before you start

* Your company's **e-mail domain** or domains (for example `yourcompany.com`). A
  domain can belong to only one company in Unicus, and free e-mail domains
  (gmail.com, outlook.com…) are not accepted.
* The **redirect URL** the portal shows in **Security → Single sign-on (SSO)**,
  for example `https://app.idunicus.com/sso/callback`. If your team signs in
  through your company's own address (`yourcompany.idunicus.com`), register
  `https://yourcompany.idunicus.com/sso/callback` too.
* Administrator access to your identity provider.

## Microsoft Entra ID (Azure AD)

1. In the [Entra admin center](https://entra.microsoft.com), go to **Identity →
   Applications → App registrations → New registration**.
   * **Supported account types:** accounts in this organizational directory only
     (single tenant).
   * **Redirect URI:** platform **Web**, with the portal's redirect URL.
2. In **Certificates & secrets**, create a **client secret** and copy its value
   (it is shown only once).
3. In **Token configuration**, add the optional claim **email** to the ID token.
   If a user has no e-mail in Entra, Unicus uses their sign-in name (UPN) when it
   is shaped like an e-mail.
4. Copy the **Application (client) ID** and the **Directory (tenant) ID** from
   **Overview**.
5. In the portal, in **Security → Single sign-on (SSO)**:
   * **Issuer:** `https://login.microsoftonline.com/<tenant-id>/v2.0`
   * **Client ID** and **client secret:** those of steps 2 and 4.
   * **Domains:** your company's.

Unicus only accepts tokens from that tenant; a `common` or `organizations`
address cannot be saved.

## Google Workspace

1. In the [Google Cloud console](https://console.cloud.google.com), with a
   project of your organization, open **APIs & Services → OAuth consent screen**
   and choose the **Internal** type: only accounts of your Workspace can then
   sign in.
2. In **Credentials → Create credentials → OAuth client ID**, choose **Web
   application** and add the portal's redirect URL under **Authorized redirect
   URIs**.
3. Copy the **Client ID** and the **Client secret**.
4. In the portal:
   * **Issuer:** `https://accounts.google.com`
   * **Client ID** and **client secret:** those of step 3.
   * **Domains:** your Workspace's.

Unicus checks that the account belongs to the Workspace of that domain: a
personal Google account created with a work e-mail does not get in.

{% hint style="warning" %}
Google Workspace does not send users over SCIM to applications like Unicus. With
Google, create the users in the portal; they then sign in with their Google
account.
{% endhint %}

## Okta

1. In the Okta admin console, go to **Applications → Create App Integration**,
   choose **OIDC - OpenID Connect** and **Web Application**.
2. **Sign-in redirect URIs:** the portal's redirect URL. **Grant type:**
   Authorization Code.
3. Assign the application to the people or groups who should use Unicus.
4. Copy the **Client ID** and the **Client secret**.
5. In the portal:
   * **Issuer:** `https://<your-org>.okta.com` (or
     `https://<your-org>.okta.com/oauth2/default` if you use the default
     authorization server).
   * **Client ID**, **client secret** and **domains**.

## Save and test

When you save, the portal reads your provider's public configuration at the
issuer URL; if it cannot find it, it tells you and saves nothing. The secret is
stored encrypted and never shown again: to change it, type a new one; left empty,
the current one is kept.

To test it, sign out, type an e-mail of your domain on the sign-in screen and
choose **Sign in with corporate SSO**.

### Make it mandatory

Turn on **Required for these domains** once your whole team signs in with SSO.
From then on, the Unicus password no longer works for e-mails of those domains
and the portal tells those users to use the corporate sign-in. Before turning it
on, make sure at least one administrator signs in fine with SSO.

## Automatic provisioning (SCIM)

With SCIM, your identity provider creates in Unicus the people you assign to the
application and deactivates them when they leave the company or lose access. It
works with Microsoft Entra ID and Okta.

1. In the portal, in **Security → Automatic provisioning (SCIM)**, copy the
   **SCIM base URL** and create a **token** with a name that identifies it (for
   example "Entra ID production"). Copy it: it is shown only once.
2. In your provider:
   * **Entra ID:** **Enterprise applications →** your application **→
     Provisioning →** mode **Automatic**. **Tenant URL:** the SCIM base URL;
     **Secret token:** the token. Test the connection and turn provisioning on.
   * **Okta:** in the application, **General → Provisioning: SCIM**; in
     **Provisioning**, the SCIM base URL as **SCIM connector base URL**, **Unique
     identifier field:** `userName`, **HTTP Header** authentication with the
     token. Turn on **Create Users**, **Update User Attributes** and **Deactivate
     Users**.

What Unicus does with each change:

* **New user:** created in your company with the **Operator** role and no
  password (they sign in with SSO). Change the role in
  [Users](users-roles-and-approvals.md) if they need another one.
* **Name or e-mail changes:** applied.
* **Removal or deactivation:** the user is deactivated and their sessions are
  closed; never deleted, and reactivated if you assign them again.
* Your company's **main administrator** is never changed by SCIM.

Revoke a token in the same section if it is no longer used or you think it was
exposed; your provider can no longer sync until you give it a new one.

## Troubleshooting

| Message in the portal | What to check |
| --- | --- |
| We could not read the provider's configuration | The issuer URL: it must be exact, with `https://`. |
| Free e-mail domains are not accepted | Type your company's domain, not a free e-mail one (gmail.com, outlook.com…). |
| The domain is already in use | That domain is set up in another Unicus company: contact us. |
| Your e-mail is not allowed | The e-mail your provider sends is not of the configured domains, or (Google) the account is not of your Workspace. |
| There is no user with that e-mail | The person has no active user in your company in Unicus: create it or assign it over SCIM. |
| The sign-in was cancelled | The person cancelled at the provider, or your provider did not give them access to the application. |
| Your company uses corporate sign-in | SSO is required for that domain: use **Sign in with corporate SSO**. |
