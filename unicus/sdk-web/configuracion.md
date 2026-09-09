# Appearance configuration

The Unicus verification experience uses the company appearance configured in the
Unicus administrative portal. Customer applications do not need to pass colors
or logos in the `<unicus-btn>` markup.

## Configure company appearance

1. Sign in to the [Unicus administrative portal](https://app.idunicus.com).
2. Open **Company**.
3. Open **Settings**.
4. Configure the company logo and colors.
5. Save the changes.

The next transactions created by Unicus Button will automatically use the saved
appearance.

## How the appearance is applied

After the button creates a transaction, the Unicus flow fetches the session
configuration using the transaction id. That configuration includes the company
logo, company name, colors, country settings, and other flow options.

The flow applies:

| Field | Where it is used |
| --- | --- |
| `logo` | Loading, authorization, and verification screens when available. |
| `windowColor` | Main action buttons and primary UI surfaces. |
| `buttonColor` | Verification progress, frame, and secondary accents. |
| `textColor` | Button text and readable foreground elements. |
| `customerName` | Authorization and consent text shown to the user. |

{% hint style="info" %}
If company colors or logo do not appear, confirm that the transaction was
created with the correct `customerid` and that the company settings are saved in
the same Unicus environment being tested.
{% endhint %}

## Button appearance

The `<unicus-btn>` element starts in a loading or disabled state while Unicus
creates the transaction. When the transaction is ready, the button uses the
colors returned by Unicus and emits `OnUnicus:loaded`.

The customer site can place the button inside its own layout, but should not
modify the internal iframe or verification screens directly.
