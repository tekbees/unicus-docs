---
description: >-
  Review enrollments, verifications and liveness checks, their detail and PDF
  certificate; decide possible duplicates and export reports.
---

# Transactions and reports

## The lists

**Operations → Enrollments**, **Verifications** and **Liveness checks** list
your company's transactions, newest first. Enrollments update live.

* Search by document number or by name and last name.
* Filter by status: successful, processing, failed, in review, suspicious,
  deleted or expired.
* Pick the date range (up to 90 days) and the columns you want to see.
* **Show tests** appears if your company has test credentials: it shows only
  the test transactions (see [Customer Token](../sdk-web-v5/customer-token.md)).

## A transaction's detail

Open a transaction to see, in tabs:

| Tab | What it shows |
| --- | --- |
| **Summary** | Result, status, person, document, dates and channel. |
| **Biometrics** | Liveness and the face comparison (against the document or the enrollment), compared with your company's minimum level. |
| **Document** | The captured document, the data read and its checks. If a capture step was rejected, the reason. |
| **Flow** | Each step of the flow with its result, in order. |
| **Location** | Where the verification took place, approximately. |
| **Technical evidence** | Technical capture data, for support. |

### Personal data

Photos (face, document, evidence) and the personal data read from the document
are only shown to users with the **View personal data** permission. Without
it, that data is hidden (`***`) and the location is rounded; the name, the
document and the results stay visible.

Every full read of personal data is recorded in the
[Audit log](users-roles-and-approvals.md#audit-log).

### PDF certificate

**Download PDF** generates the transaction's certificate (enrollment, identity
verification or liveness). Unicus keeps the fingerprint (SHA-256) of every
certificate it generates, with who generated it and when. Without the personal
data permission, the certificate leaves that data out and says so.

To check that a certificate someone gave you is genuine, use **Tools → Verify
certificate** and drop the PDF: the file never leaves your browser, only its
fingerprint does. The result is **Genuine certificate**, **Modified document**
or **Certificate not registered**.

## Review queue

When a new face matches other enrollments of your company, the enrollment is
put **in review** and appears in **Operations → Review queue** (the menu shows
how many are pending).

For each case you decide:

* **Valid enrollment:** it is a different person and the enrollment goes on.
* **Same person: impersonation:** the enrollment is rejected.

Each decision needs a comment and cannot be undone from the portal. Your
backend receives the outcome through the [webhook](../sdk-web-v5/webhooks.md).
Deciding requires the permission to edit enrollments.

## Reports

**Operations → Reports** finds the transactions of a range of up to one month
and downloads them as CSV: TID, result, status, document, name, number of
liveness checks and date.

{% hint style="info" %}
**Download** exports the page of results you are viewing. For reconciliation
or automated reports, get the results from your backend through the
[webhook](../sdk-web-v5/webhooks.md) or
[query-transaction](../sdk-web-v5/transaction-status.md).
{% endhint %}

## E-mail validation

**Tools → E-mail validation** lists your company's e-mail validations with
their status and, in the detail, the domain and mail provider information.

## Deleting a transaction

The portal does not delete transactions. To delete an enrollment (and its face
from the duplicate search) or a verification, your backend calls
`delete-transaction` (see [Server API](../sdk-web-v5/server-api.md)).
