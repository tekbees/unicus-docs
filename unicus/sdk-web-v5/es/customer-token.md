# Customer Token

{% hint style="info" %}
[English version](../customer-token.md)
{% endhint %}

Al iniciar sesión en el [panel de administración](https://app.idunicus.com) encontrarás una pestaña llamada **Compañía**; selecciónala, luego elige la opción **Configuraciones** y podrás ver tu token, hacer clic para copiarlo o generar uno nuevo si es necesario.

En las integraciones del botón Unicus, este valor se envía en el atributo
`customerid`:

{% code overflow="wrap" %}
```html
<unicus-btn
  customerid="<CUSTOMER_TOKEN>"
  transactiontype="enrollment-verify"
  clientid="ID:123456789">
</unicus-btn>
```
{% endcode %}

Usa el token que corresponde al mismo ambiente donde se ejecuta el botón.
Los tokens del ambiente de pruebas (sandbox), staging y producción son distintos. Con un token de otro
ambiente la transacción no se puede crear: el botón muestra *Reintentar* y
emite `OnUnicus:error` (consulta [Errores y solución de problemas](errors-and-troubleshooting.md)).

El token es un identificador público de tu empresa y es seguro incluirlo en tu
HTML. El botón lo envía con cada transacción que crea y lo usa para
recordar los colores de tu empresa en el navegador.

<figure><img src="../../.gitbook/assets/image (20).png" alt="" width="563"><figcaption></figcaption></figure>
