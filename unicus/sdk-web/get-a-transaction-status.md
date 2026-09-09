# Get a transaction status

This Unicus API endpoint allows our customers to know the status of a transaction through a post type service and returns all the information related to the requested transaction, only when it has been completed successfully. The customer token is required.



## Query transaction endpoint

<mark style="color:green;">`POST`</mark> `<unicus-server-api-url>/query-transaction`

#### Headers

| Name                                            | Type   | Description        |
| ----------------------------------------------- | ------ | ------------------ |
| X-Customer-ID<mark style="color:red;">\*</mark> | String | **Customer Token** |

#### Request Body

| Name                                  | Type   | Description    |
| ------------------------------------- | ------ | -------------- |
| tid<mark style="color:red;">\*</mark> | String | Transaction ID |

#### Curl Execution

```
curl --location 'https://api.idunicus.com:8080/query-transaction' \
--header 'X-Customer-ID: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX' \
--header 'Content-Type: application/json' \
--data '{
    "tid": "tid" 
}'
```

#### Response

200: OK Response for a complete transaction

```json5
{
    "success": true,
    "wasProcessed": true,
    "error": false,
    "path": "query-transaction",
    "resultCode": 0,
    "resultMessage": "",
    "additionalSessionData": {
        "isAdditionalDataPartiallyIncomplete": true
    },
    "elapsedPerformanceTime": 1667,
    "tid": "<<tid transaction>>",
    "transactionType": "<<ENROLL WITH FACE-ID||VERIFY WITH FACE>>",
    "transactionResult": "<<SUCCESS||FAIL||>>",
    "transactionStatus": "<<MESSAGE STATUS>>",
    "transactionStatusId": STATUS_ID,
    "transactionDate": "2023-09-26T14:05:52.000+00:00",
    "document": "<<ID NUMBER>>",
    "documentType": "<<ID TYPE>>",
    "name": "<<NAME SCANED>>",
    "lastName": "<<LAST NAME SCANED>>",
    "country": "CO",
    "statusUser": USER STATUS ,
    "companyName": "<<COMPANY NAME>>",
    "feature": {
        "path": "enrollment-3d",
        "error": false,
        "success": true,
        "companyTin": 19,
        "resultCode": 0,
        "wasProcessed": true,
        "resultMessage": "success",
        "scanResultBlob": "<<ENCRIPTED SCAN PROCESS>>",
        "retryScreenEnumInt": 0,
        "additionalSessionData": {
            "appID": "web.idunicus.com",
            "platform": "web",
            "ipAddress": "<<IP DEVICE>>",
            "sessionID": "<<ID SESSION>>",
            "userAgent": "<<DETAIL DEVICE>>",
            "deviceModel": "<<MODEL DEVICE>>",
            "installationID": "<<ID DEVICE>>",
            "deviceSDKVersion": "VERSION SDK",
            "isAdditionalDataPartiallyIncomplete": false
        },
        "externalDatabaseRefID": "<<TID>>",
        "elapsedPerformanceTime": 2239,
        "faceScanSecurityChecks": {
            "replayCheckSucceeded": <<BOOLEAN>>,
            "sessionTokenCheckSucceeded": <<BOOLEAN>>,
            "faceScanLivenessCheckSucceeded": <<BOOLEAN>>,
            "auditTrailVerificationCheckSucceeded": <<BOOLEAN>>
        },
        "ageEstimateGroupEnumInt": <<INT>>,
        "ageEstimateGroupV2EnumInt": <<INT>>
    },
    "matchIdFeature": {
        "path": "match-3d-2d-idscan",
        "error": false,
        "success": <<BOOLEAN>>,
        "matchLevel": <<FLOAT>>,
        "resultCode": <<FLOAT>>,
        "wasProcessed": <<BOOLEAN>>,
        "resultMessage": "<<STRING>>",
        "completelyDone": <<BOOLEAN>>,
        "nfcStatusEnumInt": <<FLOAT>>,
        "fullIDStatusEnumInt": <<FLOAT>>,
        "isPossiblePhotocopy": <<BOOLEAN>>,
        "barcodeStatusEnumInt": <<FLOAT>>,
        "externalDatabaseRefID": "<<TID>>",
        "elapsedPerformanceTime": 5376.0,
        "matchLevelNFCToFaceMap": <<FLOAT>>,
        "photoIDNextStepEnumInt": <<FLOAT>>,
        "ageEstimateGroupEnumInt": <<FLOAT>>,
        "digitalIDSpoofStatusEnumInt": <<FLOAT>>,
        "faceOnDocumentStatusEnumInt": <<FLOAT>>,
        "textOnDocumentStatusEnumInt": <<FLOAT>>,
        "idScanAgeEstimateGroupEnumInt": <<FLOAT>>,
        "unexpectedMediaEncounteredAtLeastOnce": <<BOOLEAN>>,
        "scannedIDPhotoFaceFoundWithMinimumQuality": <<BOOLEAN>>
    },
    "liveness": {
        "replayCheckSucceeded": <<BOOLEAN>>,
        "sessionTokenCheckSucceeded": <<BOOLEAN>>,
        "auditTrailVerificationCheckSucceeded": <<BOOLEAN>>,
        "faceScanLivenessCheckSucceeded": <<BOOLEAN>>
    },
    "livenessStatus": <<INT>>,
    "matchLevel": {
        "match_level": <<INT>>
    },
    "matchLevelStatus": <<INT>>,
    "ocrStatus": -1,
    "infer": {
        "typeDetector": "Not Apply",
        "match_level": 0
    },
    "inferStatus": <<INT>>,
    "validNumberCheckStatus": <<INT>>,
    "governmentStatus": <<INT>>,
    "documentValidation": {
        "BackIsPossiblePhotocopy": false,
        "BackDigitalIdSpoofStatus": false,
        "BackFullIDStatus": false,
        "BackTextOnDocumentStatus": false,
        "BackFaceOnDocumentStatus": false
    },
    "documentValidationStatus": <<INT>>,
    "location": {
        "latitude": <<FLOAT>>,
        "longitude": <<FLOAT>>
    },
    "faceImageList": [
        {
            "image": {
                "bytes": "<<BASE64 TO SELFIE>>",
                "base64": "<<BASE64 TO SELFIE>>"
            },
            "folder": "<<FOLDER>>",
            "filename": "<<FILE NAME>>"
        }
    ],
    "frontDocument": "<<BASE64 FRONT DOCUMENT IMAGE>>",
    "backDocument": "<<BASE64 BACK DOCUMENT IMAGE>>",
    "frontDocumentWithoutSegment": "<<BASE64 FRONT DOCUMENT WITHOUT SEGMENT>>",
    "backDocumentWithoutSegment": "<<BASE64 BACK DOCUMENT WITHOUT SEGMENT>>",
    "searchDuplicated": [<<ARRAY WITH OTHER SEARCH ENROLLED >>],
    "hash": "<<HASH DATA>>"
}
```

200: OK Response for transaction in process or created

```json5
{
    "success": false,
    "wasProcessed": true,
    "error": false,
    "path": "query-transaction",
    "resultCode": -1,
    "resultMessage": "TRANSACTION IN PROCESS",
    "additionalSessionData": {
        "isAdditionalDataPartiallyIncomplete": true
    },
    "elapsedPerformanceTime": 500,
    "tid": "<<TID>>"
}
```

200: OK Response for transaction not found

{% code overflow="wrap" expandable="true" %}
```json5
{
    "success": false,
    "wasProcessed": true,
    "error": false,
    "path": "query-transaction",
    "resultCode": -1,
    "resultMessage": "TRANSACTION NOT FOUND",
    "additionalSessionData": {
        "isAdditionalDataPartiallyIncomplete": true
    },
    "elapsedPerformanceTime": 500,
    "tid": "<<TID>>"
}
```
{% endcode %}
