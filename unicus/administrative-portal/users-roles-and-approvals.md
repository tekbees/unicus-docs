---
description: >-
  Invite your team, define what each role can do and review the audit log.
---

# Users and roles

## Users

**Administration → Users** lists the people of your company with their role,
status (active, pending or inactive) and last access.

* **Create user:** e-mail, names, last names and role. The person receives an
  invitation by e-mail and creates their own password (the link expires in 72
  hours). An e-mail can belong to only one Unicus account; the e-mail of a
  deleted user of your company can be used again.
* **Edit:** their details, role and access. In **Security** you can send them
  the password reset e-mail (the link expires in 24 hours and works once).
* **Delete:** the person loses access. Your company's main administrator
  cannot be edited or deleted, and nobody can delete themselves.

Only the company administrator gives the administrator role or edits an
administrator. Deactivating a user or changing their role ends their sessions
immediately.

**Sign-in e-mail:** nobody changes their own. An administrator's is changed by
another administrator or by Tekbees, and the main administrator's only by
Tekbees. When it changes, all of the person's sessions are closed and the old
address is notified.

## Roles

**Administration → Roles** defines what each group of people can do. Every role
has permissions per service (enrollment, verification, liveness, personal
data, reports, Dashboard and analytics, Audit log, flows, users, roles, Unicus
Link…) with the actions **view, create, edit and delete** each service
supports.

Every company has four **default** roles, the same in every company:

| Role | What it can do |
| --- | --- |
| **Operador** (Operator) | Reviews transactions and reports, sends Unicus Link. |
| **Revisor de fraude** (Fraud reviewer) | What the Operator does, plus personal data and the duplicate decision. |
| **Consulta** (Viewer) | Read-only access to transactions, reports and the Dashboard. |
| **Integrador** (Integrator) | Flows, settings and integration (no API keys). |

Default roles cannot be edited or deleted: if you need a variant, use
**Duplicate** and adjust the copy.

* The administrator role is protected.
* A role can only hold the services your company has.
* You can only grant the permissions you hold.
* You cannot modify your own role.
* A role that users hold cannot be deleted: give them another one first.
* Some names are reserved (for example, those that read as the administrator's).
* Some actions are reserved for administrators, whatever the role (see
  [Overview](overview.md#who-sees-what)).

{% hint style="info" %}
Give each role only what it needs. For example, an operations team can review
transactions without the **View personal data** permission: they see the
name, the document number and the results, but not the photos or the rest of
the personal data.
{% endhint %}

## Audit log

**Administration → Audit log** records what your company's users do in the
portal: sign-ins, changes to flows, users, roles, API keys and settings, every
copy of the Company ID and every full read of personal data. Records cannot be
edited or deleted.

Filter by user, resource and action, and export the result as CSV.
