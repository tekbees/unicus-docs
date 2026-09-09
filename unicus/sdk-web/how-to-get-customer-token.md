# How to get Customer Token?

When you log in to the [administration panel](https://app.idunicus.com) you will find a tab called **Company**, select it followed by the option **Settings** and you will be able to view your token and click to copy it or generate a new one if necessary.

In Unicus Button integrations, this value is passed in the `customerid`
attribute:

{% code overflow="wrap" %}
```html
<unicus-btn
  customerid="<CUSTOMER_TOKEN>"
  transactiontype="enrollment-verify"
  clientid="ID:123456789">
</unicus-btn>
```
{% endcode %}

Use the token that belongs to the same environment where the button is running.
Sandbox, staging, and production tokens are different.

<figure><img src="../.gitbook/assets/image (20).png" alt="" width="563"><figcaption></figcaption></figure>
