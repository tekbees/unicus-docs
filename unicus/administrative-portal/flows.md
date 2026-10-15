---
description: >-
  Create and publish your company's verification flows, and choose which one
  runs for each transaction type.
---

# Verification flows

A **flow** is the sequence of steps the person goes through: consent,
instructions, liveness, document, face match, OTP code, form, signature… Flows
are built in **Platform → Flows**, and the button, Unicus Link and the mobile
SDKs use the flow you assign, without changing your code. To see what each
step looks like for the person, read
[Flows and handoff to the phone](../sdk-web-v5/flows-and-handoff.md).

## Use cases, default flow and variants

Flows are grouped by **use case**: enrollment, verification, liveness and age
estimation.

* Each use case has a **default flow**: it runs when your integration does
  not ask for a specific one.
* You can have **variants** (another product, an A/B test). Your integration
  asks for them by their identifier with `data-flow-id` on the button or
  `flowId` in `init-api-transaction`.
* Without a default flow, that transaction type cannot be created (code
  `2002`, "Verification not configured").

If your company has no flows yet, use **Create default flows**.

## Create a flow

1. **New flow:** internal name, **identifier** (the one your integration will
   use; it cannot be changed later), use case, and a template or blank.
2. Drag steps from the palette. The editor shows which steps run in the
   browser and which on the phone.
3. Select a step to edit its texts (in Spanish and English) and its settings.
4. The editor validates the flow as you work and tells you what to fix. The
   draft is saved automatically.

### Available steps

| Group | Steps |
| --- | --- |
| Screens | Consent (always first, before any camera step), Information |
| Biometric capture | Liveness, Document, Face match |
| Face analysis | Age check |
| Add-ons | Electronic signature (handwritten signature), OTP code (SMS, WhatsApp or e-mail), Form, Unicus Sign signature (the "Document signing" step: signs a Unicus Sign document prefilled with the flow data) |

Some steps depend on the services in your plan (for example the OTP code or
signing with Unicus Sign); the ones you do not have are disabled. Signing with
Unicus Sign also needs your Unicus Sign account connected (see
[Integration](integration.md#connections)). Besides
steps, a flow can include server-side **checks** (for example list screening)
if your plan includes them.

## Publish

**Publish version** turns the draft into a new version, with optional notes. A
published version never changes: to modify it, edit the draft and publish
another one.

* New transactions use the published version; those already in progress
  finish with the one they started with.
* If your company turned on **second-person approval**, the publication waits
  until another administrator approves it (see
  [Approvals](users-roles-and-approvals.md#approvals)).
* **Versions** shows the history: compare versions or restore one as the
  draft.

## Assign flows

**Flow assignments** defines which flow runs for each use case:

* **Always the latest published:** each publication applies immediately.
* **A fixed version:** the assignment does not change until you update it.
  Useful to try a new version without affecting your users.

## Good practices

* Test every new version with [test credentials](../sdk-web-v5/customer-token.md)
  before assigning it in production.
* Keep texts short and clear: they are shown on screen and in the progress list
  (see [Texts and languages](../sdk-web-v5/texts-and-languages.md)).
* From the list you can **duplicate** a flow to create a variant, or **export
  it as JSON** as a backup.
