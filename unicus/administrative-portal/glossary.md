---
description: The terms used by the portal and this documentation.
---

# Glossary

| Term | Meaning |
| --- | --- |
| **Transaction** | One verification of a person: an enrollment, a verification or a liveness check. It is identified by its **TID**. |
| **TID** | The transaction's unique identifier. Your page (events) and your backend (webhook) receive it, and you see it in the portal. |
| **Enrollment** | The first time Unicus registers a person: their face and document. |
| **Verification** | Checking that the person in front of the camera is someone already enrolled. |
| **Liveness check** | Checking that a real person is in front of the camera, not a photo, a video or a mask. |
| **Review queue** | Enrollments whose face matches others of your company and are waiting for your team's decision. |
| **Flow** | The steps the person goes through in a verification. It is built and published in the portal. |
| **Use case** | The transaction type a flow is assigned to: enrollment, verification, liveness or age estimation. |
| **Default flow / variant** | The flow that runs when your integration does not ask for another, and the alternative flows it asks for by identifier. |
| **Version** | Each publication of a flow. It never changes once published. |
| **Company ID (Customer Token)** | Your company's public identifier for the button and the mobile SDKs. |
| **API key** | Your backend's secret credential for the server endpoints. |
| **Webhook** | The URL of your backend where Unicus sends the result of every transaction. |
| **Unicus Link** | A verification link you send to the person instead of integrating the button. |
| **Certificate** | The PDF with a transaction's result, which can be checked in **Verify certificate**. |
| **Personal data** | The photos and the data read from the document. Only users with the permission see them. |
| **Test credentials** | Credentials provided by Tekbees to test with the real flow without mixing tests with real transactions. |
