# Customer Token

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
Sandbox, staging, and production tokens are different. With a token of another
environment the transaction cannot be created: the button shows *Retry* and
emits `OnUnicus:error` (see [Errors and troubleshooting](errors-and-troubleshooting.md)).

The token is a public identifier of your company and is safe to render in your
HTML. The button sends it with each transaction it creates and uses it to
remember your company colours in the browser.

<figure><img src="../.gitbook/assets/image (20).png" alt="" width="563"><figcaption></figcaption></figure>

## Test mode (sandbox)

To try your whole integration in production without mixing it with real
transactions, Tekbees can give your company **test credentials**:

* a **test device id** (`sbx_...`), used in place of the Customer Token in the
  button (`customerid`) or in the SDKs;
* a **test API key** (`unk_test_...`), used in place of a live key in your
  backend calls.

With them the real flow runs (same screens, same checks), but the transaction
is stored as a test:

* it is not shown with your transactions in the portal: switch on
  **Show tests** in the lists to see it;
* its face is not added to the duplicate search (1:N), so test people never
  match real ones;
* its webhooks carry `meta.livemode: false`;
* it is deleted 30 days after it was created.

Test credentials are created and revoked by Tekbees only (ask
[Support](support.md)); your company admins can see them in
Company → Settings → API keys. When the last one is revoked, test mode ends
and **Show tests** disappears.
