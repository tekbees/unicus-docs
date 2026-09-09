# Unicus link

This Unicus integration method allows our customers to use our validation platform, without having to integrate it into a web page or application, it is possible to generate a link with the defined validation flow that can be shared to your customers by virtual means.

{% hint style="info" %}
There are two ways to integrate Unicus link, using the administration portal or with a request to our API.
{% endhint %}

### Administration Portal

Navigating to the [administration portal](https://app.idunicus.com/) You will find the **Unicus link** section, in this section you will find the option _Generate Link_ to send messages via [**WhatsApp**](#user-content-fn-1)[^1], just fill out the form to send the validation flow to your customer.





<figure><img src="../.gitbook/assets/image (21).png" alt="" width="375"><figcaption></figcaption></figure>

### API

Navigating to the  [administration portal](https://app.idunicus.com/) In this section you will find the **Company** section, where you will find the endpoint to make use of our API and obtain the link with the validation flow that you can send to your customer.

<figure><img src="../.gitbook/assets/image (22).png" alt="" width="563"><figcaption></figcaption></figure>

#### API use



## Unicus link endpoint

<mark style="color:green;">`POST`</mark> `<unicus-server-api-url>/init-api-transaction`

#### Headers

| Name          | Type   | Description                                     |
| ------------- | ------ | ----------------------------------------------- |
| X-Customer-ID | String | [Customer\_Token](how-to-get-customer-token.md) |

#### Request Body

| Name                                           | Type   | Description                                                                                      |
| ---------------------------------------------- | ------ | ------------------------------------------------------------------------------------------------ |
| document<mark style="color:red;">\*</mark>     | String | Identification number                                                                            |
| documentType<mark style="color:red;">\*</mark> | String | <p>Identification Type ID </p><p>(national identified) , FD(foreign document), PP(passport),</p> |

{% tabs %}
{% tab title="200: OK " %}
```json
{
    "success": true,
    "wasProcessed": true,
    "error": false,
    "path": "init-api-transaction",
    "resultCode": 0,
    "resultMessage": "A Transaction Enrollment was created.",
    "additionalSessionData": {
        "isAdditionalDataPartiallyIncomplete": true
    },
    "elapsedPerformanceTime": 644,
    "url": "https://id.idunicus.com/?token=3a407c4a-c362-11ed-b3bf-12dee90996cb&process=enrollment"
}
```
{% endtab %}
{% endtabs %}

```
curl --location '<unicus-server-api-url>/init-api-transaction' \
--header 'X-Customer-ID: CUSTOMER-TOKEN' \
--header 'Content-Type: application/json' \
--data '{
    "document": "<document-client-number>",
    "documentType": "<document-type-client>"
}
```

The _**url**_ parameter is the only thing required for your customer to start the validation process remotely, quickly and easily.&#x20;

When your customer finishes the identity validation process, you will receive the response of the process through [webhook](types-webhook.md).

[^1]: charges might apply
