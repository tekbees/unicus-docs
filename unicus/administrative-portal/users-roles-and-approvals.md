---
description: >-
  Invite your team, define what each role can do, approve sensitive changes and
  review the audit log.
---

# Users, roles and approvals

## Users

**Administration → Users** lists the people of your company with their role,
status (active, pending or inactive) and last access.

* **Create user:** e-mail, names, last names and role. The person receives an
  invitation by e-mail and creates their own password. An e-mail can belong to
  only one Unicus account.
* **Edit:** their details, role and access. In **Security** you can send them
  the password reset e-mail (the link expires in 24 hours and works once).
* **Delete:** the person loses access. Your company's main administrator
  cannot be edited or deleted, and nobody can delete themselves.

## Roles

**Administration → Roles** defines what each group of people can do. Every role
has permissions per service (enrollment, verification, liveness, personal
data, reports, Dashboard, audit log, flows, users, roles, Unicus Link…) with
the actions **view, create, edit and delete** each service supports.

* The administrator role is protected.
* A role can only hold the services your company has.
* Some actions are reserved for administrators, whatever the role (see
  [Overview](overview.md#who-sees-what)).

{% hint style="info" %}
Give each role only what it needs. For example, an operations team can review
transactions without the **View personal data** permission: they see the
results, but not the photos or the document data.
{% endhint %}

## Approvals

If your company turned on **second-person approval** (in
[Security](company-settings.md#security)), these actions need another
administrator's approval:

* publishing a flow;
* creating a production API key.

**Administration → Approvals** lists pending, approved, rejected and withdrawn
requests. Nobody approves their own request; a rejection carries a reason. An
approved request lets the requester make the change once, within the next 24
hours.

## Audit log

**Administration → Audit log** records what your company's users do in the
portal: sign-ins, changes to flows, users, roles, API keys and settings, and
every full read of personal data. Records cannot be edited or deleted.

Filter by user, resource and action, and export the result as CSV.
