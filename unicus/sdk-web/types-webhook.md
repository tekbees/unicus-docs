---
description: >-
  Once the enrollment or verification process is completed, it is necessary for
  you to expose a service in order to send you the response of the client(s) in
  real time.
---

# Webhooks

### How to configure and/or customize the Webhook? :pencil2:&#x20;

When logging into the [administrative panel](https://app.idunicus.com/) you will find a tab called **Company**, select it followed by the option Settings and you will be able to see the webhook you have configured to edit it, in case you do not have it you can add it.

<figure><img src="../.gitbook/assets/image (25).png" alt="" width="563"><figcaption></figcaption></figure>

## &#x20;Process Types

### Enrollment

The Enrolment process has several steps and each step returns its own results.&#x20;

The most common steps include:

* liveness +  FaceMap Extraction (LIVENESS\_FACEMAP)
* OCR extraction data, and document validation from the front side of the document (FRONT\_DOCUMENT)
* OCR extraction data, and document validation from the back side of the document (BACK\_DOCUMENT)
* Match of face against document photo (MATCH\_DOCUMENT)

The last type you will get when a transaction is completed is "MATCH\_DOCUMENT". If for any reason you don't get it, is because something happened in the previous steps, which will be described in the response codes.

**LIVENESS\_FACEMAP**

{% tabs %}
{% tab title="Fields" %}
| **process**          | String | Process being executed                                                     |
| -------------------- | ------ | -------------------------------------------------------------------------- |
| **ageEstimateGroup** | Int    | Identifier of the age group in which the person is most likely to be found |
| **result\_code**     | Int    | Process result code                                                        |
| **result\_message**  | String | Process result message                                                     |
| **tid**              | String | Transaction ID                                                             |
| **faceImage**        | String | Image on base 64 of the face                                               |
{% endtab %}

{% tab title="Response" %}
```json5
{
    "data": {
        "process": "LIVENESS_FACEMAP",
        "ageEstimateGroup": int,
        "result_message": string,
        "result_code": int,
        "tid": string,
        "faceImage": string
    },
    "meta": {
        "code": 200,
        "ok": true
    }
}
```
{% endtab %}
{% endtabs %}

#### FRONT\_DOCUMENT

{% tabs %}
{% tab title="Fields" %}


<table data-header-hidden><thead><tr><th width="273.66666666666663"></th><th>Tipo</th><th>Descripción</th></tr></thead><tbody><tr><td><strong>process</strong></td><td>String</td><td>Process being executed</td></tr><tr><td><strong>result_code</strong></td><td>Int</td><td>Process result code</td></tr><tr><td><strong>result_message</strong></td><td>String</td><td>Process result message</td></tr><tr><td><strong>success</strong></td><td>Boolean</td><td>Result if the process was satisfactory or not</td></tr><tr><td><strong>idNumber</strong></td><td>String</td><td>The document number of the person to be enrolled</td></tr><tr><td><strong>tid</strong></td><td>String</td><td>Transaction ID</td></tr><tr><td><strong>docBack</strong></td><td>String</td><td>Base 64 image of the back of the document</td></tr><tr><td><strong>docFront</strong></td><td>String</td><td>Base 64 image of the front of the document</td></tr><tr><td><strong>match_level</strong></td><td>String</td><td>Match level between the document and the person's face can be in the range of 0 - 7 acceptable values are: 2 - 7 unacceptable values are from 0 - 1</td></tr><tr><td><strong>age_estimate_group</strong></td><td>Int</td><td>Identifier of the age group in which the photo of the person's document face is likely to be found.</td></tr><tr><td><strong>digital_id_spoof</strong></td><td>Int</td><td>Physical and morphological document validation</td></tr><tr><td><strong>face_on_document_status</strong></td><td>Int</td><td>Validation of the face contained in the document</td></tr><tr><td><strong>full_id_status</strong></td><td>Int</td><td>Validates if the document is complete</td></tr><tr><td><strong>text_on_document_status</strong></td><td>Int</td><td>Validates the document text</td></tr><tr><td><strong>unexpectedMediaEncounteredAtLeastOnce</strong></td><td>boolean</td><td>Validates if the material in the document was unexpected</td></tr><tr><td><strong>location</strong></td><td><strong>jsonObject</strong> </td><td>Return the latitude and longitude of the client, this parameter isn't present if the client not active her location mobile</td></tr></tbody></table>
{% endtab %}

{% tab title="Response" %}


{% hint style="success" %}
200: OK
{% endhint %}

```json5
{
    "data": {
        "process": "FRONT_DOCUMENT",
        "result_message": string,
        "age_estimate_group": int,
        "digital_id_spoof": int,
        "text_on_document_status": int,
        "idNumber": int,        
        "tid": string,
        "docBack": string,
        "face_on_document_status": Int,
        "full_id_status": Int,
        "success": Boolean,
        "unexpectedMediaEncounteredAtLeastOnce": Boolean,
        "docFront": string,
        "result_code": Int,
        "location": "{"latitude":float,"longitude":float}",
        "match_level": Int
    },
    "meta": {
        "code": 200,
        "ok": true
    }
}
```
{% endtab %}
{% endtabs %}

#### BACK\_DOCUMENT

{% tabs %}
{% tab title="Fields" %}
<table data-header-hidden><thead><tr><th width="273.66666666666663"></th><th>Tipo</th><th>Descripción</th></tr></thead><tbody><tr><td><strong>process</strong></td><td>String</td><td>Process being executed</td></tr><tr><td><strong>result_code</strong></td><td>Int</td><td>Process result code</td></tr><tr><td><strong>result_message</strong></td><td>String</td><td>Process result message</td></tr><tr><td><strong>success</strong></td><td>Boolean</td><td>Result if the process was satisfactory or not</td></tr><tr><td><strong>idNumber</strong></td><td>String</td><td>The document number of the person to be enrolled</td></tr><tr><td><strong>tid</strong></td><td>String</td><td>Transaction ID</td></tr><tr><td><strong>docFront</strong></td><td>String</td><td>Base 64 image of the front of the document</td></tr><tr><td><strong>docBack</strong></td><td>String</td><td>Image on base 64 of the back of the document</td></tr><tr><td><strong>match_level</strong></td><td>String</td><td>Match level between the document and the person's face can be in the range of 0 - 7 acceptable values are: 2 - 7 unacceptable values are from 0 - 1</td></tr><tr><td><strong>age_estimate_group</strong></td><td>Int</td><td>Identifier of the age group in which the photo of the person's document face is likely to be found.</td></tr><tr><td><strong>digital_id_spoof</strong></td><td>Int</td><td>Physical and morphological document validation</td></tr><tr><td><strong>face_on_document_status</strong></td><td>Int</td><td>Validation of the face contained in the document</td></tr><tr><td><strong>full_id_status</strong></td><td>Int</td><td>Validates if the document is complete</td></tr><tr><td><strong>text_on_document_status</strong></td><td>Int</td><td>Validates the document text</td></tr><tr><td><strong>unexpectedMediaEncounteredAtLeastOnce</strong></td><td>Boolean</td><td>Validates if the material in the document was unexpected</td></tr><tr><td><strong>location</strong></td><td><strong>JsonObject</strong></td><td>Return the latitude and longitude of the client, this parameter isn't present if the client not active her location mobile</td></tr></tbody></table>
{% endtab %}

{% tab title="Response" %}


{% hint style="success" %}
200: OK
{% endhint %}

```json5
{
    "data": {
        "process": "BACK_DOCUMENT",
        "result_message": string,
        "age_estimate_group": int,
        "digital_id_spoof": int,
        "text_on_document_status": int,
        "idNumber": string,
        "tid": string,
        "unexpectedMediaEncounteredAtLeastOnce": Boolean,
        "docBack": string,
        "face_on_document_status": Int,
        "full_id_status": Int,
        "success": Boolean,
        "docFront": string,             
        "result_code": Int,
        "location: "{"latitude":float,"longitude":float}",
        "match_level": Int
    },
    "meta": {
        "code": 200,
        "ok": true
    }
}
```
{% endtab %}
{% endtabs %}

#### MATCH\_DOCUMENT

{% tabs %}
{% tab title="Fields" %}
<table data-header-hidden><thead><tr><th width="273.66666666666663"></th><th>Tipo</th><th>Descripción</th></tr></thead><tbody><tr><td><strong>process</strong></td><td>String</td><td>Process being executed</td></tr><tr><td><strong>result_code</strong></td><td>Int</td><td>Process result code</td></tr><tr><td><strong>result_message</strong></td><td>String</td><td>Process result message</td></tr><tr><td><strong>success</strong></td><td>Boolean</td><td>Result if the process was satisfactory or not</td></tr><tr><td><strong>idType</strong></td><td>String</td><td>Type of document of the person to be enrolled</td></tr><tr><td><strong>idNumber</strong></td><td>String</td><td>The document number of the person to be enlisted</td></tr><tr><td><strong>idName</strong></td><td>String</td><td>Names of person to be enrolled</td></tr><tr><td><strong>idLastName</strong></td><td>String</td><td>Last name of person to be enrolled</td></tr><tr><td><strong>birthDate</strong></td><td>String</td><td>Date of birth</td></tr><tr><td><strong>placeBirth</strong></td><td>String</td><td>Place of birth</td></tr><tr><td><strong>issue_date</strong></td><td>String</td><td>Date of issue of the document</td></tr><tr><td><strong>issue_place</strong></td><td>String</td><td>Place of issue of the document</td></tr><tr><td><strong>heigth</strong></td><td>String</td><td>Height recorded in the document</td></tr><tr><td><strong>gender</strong></td><td>String</td><td>Person's gender</td></tr><tr><td><strong>tid</strong></td><td>String</td><td>Transaction ID</td></tr><tr><td><strong>docFront</strong></td><td>String</td><td>Base 64 image of the front of the document</td></tr><tr><td><strong>docBack</strong></td><td>String</td><td>Image on base 64 of the back of the document</td></tr><tr><td><strong>match_level</strong></td><td>String</td><td>Match level between the document and the person's face can be in the range of 0 - 7 acceptable values are: 2 - 7 unacceptable values are from 0 - 1</td></tr><tr><td><strong>age_estimate_group</strong></td><td>Int</td><td>Identifier of the age group in which the photo of the person's document face is likely to be found.</td></tr><tr><td><strong>digital_id_spoof</strong></td><td>Int</td><td>Physical and morphological document validation</td></tr><tr><td><strong>face_on_document_status</strong></td><td>Int</td><td>Validation of the face contained in the document</td></tr><tr><td><strong>full_id_status</strong></td><td>Int</td><td>Validates if the document is complete</td></tr><tr><td><strong>text_on_document_status</strong></td><td>Int</td><td>Validates the document text</td></tr><tr><td><strong>unexpectedMediaEncounteredAtLeastOnce</strong></td><td>boolean</td><td>Validates if the material in the document was unexpected</td></tr><tr><td><strong>idNumberOCR</strong></td><td>string</td><td>idNumber returned by the OCR process</td></tr><tr><td><strong>location</strong></td><td><strong>JsonObject</strong></td><td>Return the latitude and longitude of the client, this parameter isn't present if the client not active her location mobile</td></tr></tbody></table>


{% endtab %}

{% tab title="Response" %}
{% hint style="success" %}
200: OK
{% endhint %}

<pre class="language-json5"><code class="lang-json5">{
    "data": {
        "process": "MATCH_DOCUMENT",
        "result_code":int
        "result_message": string,
        "success": boolean,
        "idType": string,
        "idNumber": string,
        "idName": string,
        "idLastName": string,
<strong>        "birthDate": string,
</strong>        "placeBirth": string,
        "issue_date": string,
        "issue_place": string,
        "height": string,
        "gender": string,
        "tid": string,
        "docFront": string,
        "docBack": string,
        "match_level": int,
        "age_estimate_group": int,
        "digital_id_spoof": int,
        "face_on_document_status": int,
        "full_id_status": int,
        "text_on_document_status": int,
        "unexpectedMediaEncounteredAtLeastOnce":boolean,
        "location":"{"latitude":float,"longitude":float}",
        "idNumberOCR": string},
    },
    "meta": {
        "code": 200,
        "ok": true
    }
}
</code></pre>
{% endtab %}
{% endtabs %}

### Verificación

{% tabs %}
{% tab title="Fields" %}
| Field name           | Type    | Description                                                                |
| -------------------- | ------- | -------------------------------------------------------------------------- |
| **process**          | String  | Process being executed                                                     |
| **tid**              | String  | Transaction ID                                                             |
| **auditTrailImage**  | Base64  | Image on base 64 of the face                                               |
| **ageEstimateGroup** | Int     | Identifier of the age group in which the person is most likely to be found |
| **result\_code**     | Int     | Transaction result code                                                    |
| **result\_message**  | String  | Transaction result message                                                 |
| **match\_level**     | String  | Match level for verification can only be: 0 not valid and 15 valid.        |
| **liveness\_check**  | Boolean | Liveness validation result                                                 |
{% endtab %}

{% tab title="Response" %}
{% hint style="success" %}
200: OK
{% endhint %}

```json5
{
    "data": {
        "process": "VERIFY_LIVENESS",
        "auditTrailImage": <FaceImageInBase64>,
        "ageEstimateGroup": ageEstimateGroup,
        "result_message": "string",
        "matchLevel": matchLevel,
        "livenessCheck": boolean,
        "result_code": resultCode,
        "tid": idTransaction
    },
    "meta": {
        "code": 200,
        "ok": true
    }
}
```
{% endtab %}
{% endtabs %}

ageEstimateGroup, matchLevel, result\_code return different codes which you can find described in the following section [Result codes and References](result-codes-and-references.md)

### Practical URL example

```http
https://<domain>/api/results-verification
```

> This is an example of the definition of your service which must receive the parameters mentioned below in JSON structure. This Url of the webhook must be previously configured in the Unicus administrative application, in general settings.
