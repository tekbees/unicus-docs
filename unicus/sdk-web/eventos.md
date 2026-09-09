---
description: >-
  For Unicus SDK Web you will receive the following events in your website which
  will help you know what step the process is executing and the result of a
  transaction.
---

# Events

### How to subscribe to SDK Web events?

{% hint style="warning" %}
**Interesting:** \
In addition to the Webhooks where you can have server-side response of the flow of your users, you can also do it from the following feature.
{% endhint %}

The following are the events that can be returned by _**Unicus**_ _**Button**_, followed by a description of when each event is triggered and its characteristics.

```javascript
/* Add listener pointing to the Unicus Button */
const unicusButton = document.createElement("unicus-btn");

  unicusButton.addEventListener("OnUnicus:loaded", ({ detail }) => {
    console.log("Unicus loaded with transaction detail", detail);
  });
  
  unicusButton.addEventListener("OnUnicus:details", ({ detail }) => {
    console.log("OnUnicus:details", detail);
  });
  
  unicusButton.addEventListener("OnUnicus:error", ({ detail }) => {
    console.log("OnUnicus:error", detail);
  });
  
  unicusButton.addEventListener("OnUnicus:finished", ({ detail }) => {
    console.log("OnUnicus:finished", detail);
  });
  
  unicusButton.addEventListener("OnUnicus:exit", ({ detail }) => {
    console.log("OnUnicus:exit", detail);
  });
```

#### Unicus button load

Add the following **listener** to detect when the Unicus Button is finished loading.

```javascript
/* Add listener by pointing to the Unicus Button */
const unicusButton = document.createElement("unicus-btn");

unicusButton.addEventListener('OnUnicus:loaded', function(event) {
  console.log('Unicus loaded with transaction detail: ', event.detail)
});
```

{% tabs %}
{% tab title="Paramenters" %}
Keep in mind the following convention to know what each object means in the response corresponding to this Event.

<table data-header-hidden data-search="false"><thead><tr><th width="275.3197616683218">Name</th><th>Type</th><th>Description</th></tr></thead><tbody><tr><td><strong>status_error</strong></td><td>Boolean</td><td>Defines whether an error occurred during service loading or related to the pending process.</td></tr><tr><td><strong>loaded</strong></td><td>Boolean</td><td>Defines whether the process loaded correctly</td></tr><tr><td><strong>message</strong></td><td>String</td><td>Provides an immediate response message to the customer to let them know how the process was completed</td></tr><tr><td><strong>link</strong></td><td>String</td><td>Gives the customer the option to open the Web help modal himself.</td></tr><tr><td><strong>transaction</strong></td><td>Object</td><td>Provides an object with transaction data</td></tr><tr><td><strong>transaction</strong>.<strong>transactionId</strong></td><td>String</td><td>Corresponds to the transaction ID</td></tr><tr><td><strong>transaction</strong>.<strong>clientid</strong></td><td>String</td><td>Corresponds to the identification of the customer with whom the transaction is to be carried out.</td></tr></tbody></table>
{% endtab %}

{% tab title="Response" %}
If all goes well, you should see the following response in the (developer) console in your application.

{% hint style="success" %}
successful
{% endhint %}

```javascript
{
  link: "URL/?token=<TransactionID>",
  loaded: true,
  message: "Unicus SDK: loaded successfully",
  status_error: false,
  transaction: {
    transactionId: "<TransactionID>",
    clientid: "<ClientID>",
  },
}
```

{% hint style="warning" %}
**Important**\
If the **status\_error** object is set to True, it means that the transaction cannot be executed.\
\
Please check carefully the values of the client and customerid attributes corresponding to the Unicus button to see if there are any errors or inconsistencies, ultimately contact support.
{% endhint %}
{% endtab %}
{% endtabs %}

### Events Details&#x20;

#### Enrollment with document and verification

Add the following _addEventListener_ to listen when the user has an active transaction.

{% code overflow="wrap" expandable="true" %}
```js
/* Add listener by pointing to the Unicus Button */
const unicusButton = document.createElement("unicus-btn");
unicusButton.addEventListener("OnUnicus:details", ({ detail }) => {
    console.log("Unicus with transaction details", detail);
  });
```
{% endcode %}

#### Liveness

{% tabs %}
{% tab title="Parameters" %}
<table><thead><tr><th width="202.55989583333331">Name</th><th width="144">Type</th><th>Description</th></tr></thead><tbody><tr><td><strong>status_error</strong></td><td>Booleano</td><td>Defines whether an error occurred during service loading or related to the pending process.</td></tr><tr><td><strong>message</strong></td><td>String</td><td>Provides an immediate response message to the customer to let them know how the process was completed</td></tr><tr><td><strong>transaction</strong></td><td>Object</td><td>Provides an object with data from the transaction process</td></tr><tr><td><strong>transaction.state</strong></td><td>Object</td><td>Provides an object with data on the status of the transaction performed.</td></tr><tr><td><strong>transaction.state.additionalSessionData</strong></td><td>Object</td><td>Provides an object with transaction session data.</td></tr><tr><td><strong>transaction.state.faceScanSecurityChecks</strong></td><td>Object</td><td>Provides an object with data from the performed face scan.</td></tr></tbody></table>
{% endtab %}

{% tab title="Response" %}
{% hint style="success" %}
Successful

Note that this event will be repeated several times as the customer repeats the document validation process.
{% endhint %}

```json5
{ 
  status_error: false,
  message: "Unicus SDK: User is active on transaction",
  status_error: false,
    transaction:{
     exited: false
     state: {
          success: true,
          wasProcessed: true,
          error: false,
          path: "match-3d-3d",
          resultCode: 0,
          resultMessage: "The match request was processed  and the Match Level was  15",
          additionalSessionData: {
                isAdditionalDataPartiallyIncomplete: false,
                platform: "web",
                appID: "192.168.100.70",
                installationID: "ad2b8bd8-fc54-4446-bebf-23e2634f8d5a",
                deviceModel: "EB2103",
                deviceSDKVersion: "9.6.18",
                sessionID: "231d724f-6706-4790-920a-314604817cbe",
                userAgent: "Mozilla/5.0 (Linux; Android 13; EB2103)",
                ipAddress: "157.100.137.115"
          },
          elapsedPerformanceTime: 1623,
          externalDatabaseRefID: a84785fd-c821-11ed-b3bf-12dee90996cb,
          faceScanSecurityChecks: {
               success: true,
               replayCheckSucceeded: true,
               sessionTokenCheckSucceeded: true,
               auditTrailVerificationCheckSucceeded: true,
               faceScanLivenessCheckSucceeded: true
          },
          ageEstimateGroupEnumInt: 1,
          matchLevel: 15,
          retryScreenEnumInt: 0,
          scanResultBlob: "AAEAAABTAAAAAAAAAPCFkLOa4Lfa5VYTYLjY8VVcx1kIH+CtvK77KaGtuTozuNB7HNabe7/ms19h86bhjOZMmuGcvgP2vQFo/b2ijxN/LvB9+XwyPwmg2YMDk1kbb92k"
     },
    }
}
```

{% hint style="danger" %}
Errors \
The following is a list of errors you might get.
{% endhint %}

* **USER\_ALREADY\_ENROLL:**

> It occurs if the user re-enrolls. This error is only possible during an internal application failure since the application automatically detects which process it should direct to.

```javascript
{ 
   status_error: false,
   message: "Unicus auth: User is active on transaction",
   transaction: {
      exited: false,
      transactionId: "c84c962d-29a7-11eb-8376-16aff0d1ab49",
      ageEstimateGroupEnumInt: 0,
      externalDatabaseRefID: "102349666",
      resultCode: 2011,
      resultMessage: "USER_ALREADY_ENROLL",
      success: false,
      error: false,    
   },
}
```
{% endtab %}
{% endtabs %}

#### Enrollment

{% tabs %}
{% tab title="Parameters" %}
<table><thead><tr><th width="192.33333333333331">Name</th><th width="144">Type</th><th>Description</th></tr></thead><tbody><tr><td><strong>status_error</strong></td><td>Booleano</td><td>Defines whether an error occurred during service loading or related to the pending process.</td></tr><tr><td><strong>message</strong></td><td>String</td><td>Provides an immediate response message to the customer to let them know how the process was completed</td></tr><tr><td><strong>transaction</strong></td><td>Object</td><td>Provides an object with data from the transaction process</td></tr><tr><td><strong>transaction.state</strong></td><td>Object</td><td>Provides an object with data on the status of the transaction performed.</td></tr><tr><td><strong>transaction.state.additionalSessionData</strong></td><td>Object</td><td>Provides an object with transaction session data.</td></tr><tr><td><strong>transaction.state.faceScanSecurityChecks</strong></td><td>Object</td><td>Provides an object with data from the performed face scan.</td></tr></tbody></table>
{% endtab %}

{% tab title="Response" %}
{% hint style="success" %}
Success

Note that this event will be repeated several times as the customer repeats the document validation process.
{% endhint %}

```json5
{ 
  status_error: false,
  message: "Unicus SDK: User is active on transaction",
  status_error: false,
    transaction:{
     exited: false
     state: "",
     ageEstimateGroupEnumInt: 0,
     barcodeStatusEnumInt: 0,
     digitalIDSpoofStatusEnumInt: 0,
     documentData: "{<DOCUMENT_INFORMATION>}",
     error: false,
     externalDatabaseRefID: "<ID_TRANSACTION>",
     faceOnDocumentStatusEnumInt: 1,
     fullIDStatusEnumInt: 0,
     idScanAgeEstimateGroupEnumInt: 4,
     isCompletelyDone: false,
     isFrontSide: true,
     matchLevel: 4,
     matchLevelNFCToFaceMap: 0,
     nfcStatusEnumInt: 0,
     ocrResults: "{<OCR_SERVICE_DOCUMENT>}",
     path: "match-3d-2d-idscan",
     resultCode: 200,
     resultMessage: "Success",
     scanResultBlob: "AAEAAAA6MwAAAAAAANA5/JQ6E4/qlhw2Kb1KbXl5bPFg0xsTJ,
     success: true,
     textOnDocumentStatusEnumInt: 1,
     wasProcessed: true
    }
  transactionId: "70a440e5-c7e2-11ec-b35b-160220c5a63b" 
}
```


{% endtab %}
{% endtabs %}

#### Verify

{% tabs %}
{% tab title="Parameters" %}
<table><thead><tr><th width="192.33333333333331">Name</th><th width="144">Type</th><th>Description</th></tr></thead><tbody><tr><td><strong>status_error</strong></td><td>Booleano</td><td>Defines whether an error occurred during service loading or related to the pending process.</td></tr><tr><td><strong>message</strong></td><td>String</td><td>Provides an immediate response message to the customer to let them know how the process was completed</td></tr><tr><td><strong>transaction</strong></td><td>Object</td><td>Provides an object with data from the transaction process</td></tr><tr><td><strong>transaction.state</strong></td><td>Object</td><td>Provides an object with data on the status of the transaction performed.</td></tr><tr><td><strong>transaction.state.additionalSessionData</strong></td><td>Object</td><td>Provides an object with transaction session data.</td></tr><tr><td><strong>transaction.state.faceScanSecurityChecks</strong></td><td>Object</td><td>Provides an object with data from the performed face scan.</td></tr></tbody></table>
{% endtab %}

{% tab title="Response" %}
{% hint style="success" %}
Success

Note that this event will be repeated several times as the customer repeats the document validation process.
{% endhint %}

```json5
{ 
  status_error: false,
  message: "Unicus SDK: User is active on transaction",
  status_error: false,
    transaction:{
     exited: false
     state: {
          success: true,
          wasProcessed: true,
          error: false,
          path: "match-3d-3d",
          resultCode: 0,
          resultMessage: "The match request was processed  and the Match Level was  15",
          additionalSessionData: {
                isAdditionalDataPartiallyIncomplete: false,
                platform: "web",
                appID: "192.168.100.70",
                installationID: "ad2b8bd8-fc54-4446-bebf-23e2634f8d5a",
                deviceModel: "EB2103",
                deviceSDKVersion: "9.6.18",
                sessionID: "231d724f-6706-4790-920a-314604817cbe",
                userAgent: "Mozilla/5.0 (Linux; Android 13; EB2103)",
                ipAddress: "157.100.137.115"
          },
          elapsedPerformanceTime: 1623,
          externalDatabaseRefID: "a84785fd-c821-11ed-b3bf-12dee90996cb",
          faceScanSecurityChecks: {
               success: true,
               replayCheckSucceeded: true,
               sessionTokenCheckSucceeded: true,
               auditTrailVerificationCheckSucceeded: true,
               faceScanLivenessCheckSucceeded: true
          },
          ageEstimateGroupEnumInt: 1,
          matchLevel: 15,
          retryScreenEnumInt: 0,
          scanResultBlob: "AAEAAABTAAAAAAAAAPCFkLOa4Lfa5VYTYLjY8VVcx1kIH+CtvK77KaGtuTozuNB7HNabe7/ms19h86bhjOZMmuGcvgP2vQFo/b2ijxN/LvB9+XwyPwmg2YMDk1kbb92k"
     },
    }
}
```
{% endtab %}
{% endtabs %}

[Check result codes](result-codes-and-references.md#event-codes)

### **Transaction Completion**

Add the following addEventListener to listen when the user has a completed transaction.

```javascript
/* Add listener by pointing to the Unicus Button */
const unicusButton = document.createElement("unicus-btn");

unicusButton.addEventListener('OnUnicus:finished', ({ detail }) => {
  console.log('loaded payload', detail)
});
```

{% tabs %}
{% tab title="Parameters" %}
| status\_error             | Boolean | Defines whether an error occurred during service loading or related to the pending process.           |
| ------------------------- | ------- | ----------------------------------------------------------------------------------------------------- |
| message                   | String  | Provides an immediate response message to the customer to let them know how the process was completed |
| transaction               | Object  | Provides an object including the final state of the transaction                                       |
| transaction.exited        | Boolean | Provides a value given the case true or false                                                         |
| transaction.transactionId | String  | Corresponds to the ID                                                                                 |
{% endtab %}

{% tab title="Response" %}
{% hint style="danger" %}
Errors \
The following is a list of errors that you might get
{% endhint %}

* **USER\_ALREADY\_ENROLL:**

> It occurs if the user re-enrolls. This error is only possible during an internal application failure because the application automatically detects which process it should direct to.

```javascript
  status_error: false,
  message: "Unicus auth: User is active on transaction",
  transaction: {
    exited: false,
    transactionId: "c84c962d-29a7-11eb-8376-16aff0d1ab49",
    ageEstimateGroupEnumInt: 0,
    externalDatabaseRefID: "1022349666",
    resultCode: 2011,
    resultMessage: "USER_ALREADY_ENROLL",
    success: false,
    error: false,    
  },

```
{% endtab %}
{% endtabs %}

### Transaction error

Add the following addEventListener to listen when the user has a completed transaction.

```javascript
/* Add listener pointing to the Unicus Button */
const unicusButton = document.createElement("unicus-btn");

unicusButton.addEventListener('OnUnicus:error', ({ detail }) => {
  console.log('Unicus:OnError', detail)
}); 
```

{% tabs %}
{% tab title="Parameters" %}
| status\_error | Boolean | Defines whether an error occurred during service loading or related to the pending process.           |
| ------------- | ------- | ----------------------------------------------------------------------------------------------------- |
| message       | String  | Provides an immediate response message to the customer to let them know how the process was completed |
| details       | Object  | Contains the error detail or description                                                              |
{% endtab %}

{% tab title="Responses" %}
{% hint style="danger" %}
Errors \
The following is a list of errors that you might get
{% endhint %}

* **USER\_ALREADY\_ENROLL:**

> It occurs if the user re-enrolls. This error is only possible during an internal application failure because the application automatically detects which process it should direct to.

```javascript
  status_error: false,
  message: "Unicus auth: User is active on transaction",
  transaction: {
    exited: false,
    transactionId: "c84c962d-29a7-11eb-8376-16aff0d1ab49",
    ageEstimateGroupEnumInt: 0,
    externalDatabaseRefID: "1022349666",
    resultCode: 2011,
    resultMessage: "USER_ALREADY_ENROLL",
    success: false,
    error: false,    
  },

```
{% endtab %}
{% endtabs %}

