# Customer Token

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

<figure><img src="../.gitbook/assets/image (20).png" alt="" width="563"><figcaption></figcaption></figure>

## Modo de prueba (sandbox)

Para probar toda tu integración en producción sin mezclarla con transacciones
reales, Tekbees puede darle a tu empresa **credenciales de prueba**:

* un **device id de prueba** (`sbx_...`), que se usa en lugar del Customer Token
  en el botón (`customerid`) o en los SDK;
* una **API key de prueba** (`unk_test_...`), que se usa en lugar de una key
  real en las llamadas de tu backend.

Con ellas corre el flujo real (las mismas pantallas y validaciones), pero la
transacción se guarda como prueba:

* no aparece con tus transacciones en el portal: activa **Ver pruebas** en
  los listados para verla;
* su rostro no entra a la búsqueda de duplicados (1:N), así que las personas de
  prueba nunca coinciden con las reales;
* sus webhooks llevan `meta.livemode: false`;
* se borra 30 días después de creada.

Solo Tekbees crea y revoca las credenciales de prueba (pídelas a
[Soporte](support.md)); los administradores de tu empresa las ven en
Compañía → Configuraciones → API keys. Cuando se revoca la última, el modo de
prueba termina y **Ver pruebas** desaparece.
