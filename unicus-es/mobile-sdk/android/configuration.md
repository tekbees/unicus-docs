---
description: >-
  Todas las opciones de UnicusSdkConfig, los ambientes, los logs, los textos y
  la marca del SDK Android de Unicus.
---

# Configuración

## Configuración mínima

{% tabs %}
{% tab title="Kotlin" %}
{% code overflow="wrap" %}
```kotlin
UnicusSdk.shared.configure(
    UnicusSdkConfig(apiKey = "<CUSTOMER_TOKEN>", environment = UnicusEnvironment.DEV)
)
```
{% endcode %}
{% endtab %}

{% tab title="Java" %}
{% code overflow="wrap" %}
```java
UnicusSdk.getShared().configure(new UnicusSdkConfig("<CUSTOMER_TOKEN>", UnicusEnvironment.DEV));
```
{% endcode %}
{% endtab %}
{% endtabs %}

`apiKey` es el [Customer Token](../../sdk-web-v5/customer-token.md) de tu
empresa para ese ambiente. Es la única credencial que maneja tu app: el SDK
trae sus propias llaves y no existe una opción para pasarlas.

Llama a `configure` una vez antes de cualquier otra operación (por ejemplo en
`Application.onCreate`). Si lo llamas de nuevo, reemplaza la configuración;
hazlo cuando no haya una verificación en curso. Sin `configure`, todas las
operaciones fallan con `not_configured`.

## Cambiar opciones

{% tabs %}
{% tab title="Kotlin" %}
{% code overflow="wrap" %}
```kotlin
val config = UnicusSdkConfig(apiKey = "<CUSTOMER_TOKEN>", environment = UnicusEnvironment.DEV)
    .copy(
        secureScreens = true,
        enableApiLogging = BuildConfig.DEBUG,
        verificationTextOverrides = misTextosEnEspanol
    )
UnicusSdk.shared.configure(config)
```
{% endcode %}
{% endtab %}

{% tab title="Java" %}
{% code overflow="wrap" %}
```java
UnicusSdkConfig config = new UnicusSdkConfig.Builder("<CUSTOMER_TOKEN>", UnicusEnvironment.DEV)
        .secureScreens(true)
        .enableApiLogging(BuildConfig.DEBUG)
        .verificationTextOverrides(misTextosEnEspanol)
        .build();
UnicusSdk.getShared().configure(config);
```
{% endcode %}
{% endtab %}
{% endtabs %}

## Opciones

| Opción | Valor por defecto | Cuándo cambiarla |
| --- | --- | --- |
| `apiKey` | — (obligatoria) | Tu Customer Token. Uno por ambiente. |
| `environment` | — (obligatoria en la forma simple) | `DEV`, `STAGING` o `PRODUCTION`. Consulta [Ambientes](#ambientes). |
| `uiStepMode` | `UnicusUiStepMode.WebView` | `Custom(provider)` para mostrar consentimiento, formulario, firma, OTP y firma de documentos con tus propias pantallas; `Disabled` para flujos solo biométricos. Consulta [Pasos de UI personalizados](custom-ui-steps.md) y [Flujos y pasos de UI](../flows-and-ui-steps.md). |
| `secureScreens` | `false` | `true` bloquea capturas y grabación de pantalla (`FLAG_SECURE`) en las pantallas del flujo. Recomendado en producción. |
| `resumeOpenTransactions` | `true` | Cuando el usuario sale y vuelves a llamar a `start` con el mismo documento, el SDK continúa la transacción abierta. Usa `false` para crear siempre una nueva. Consulta [Resultados y reanudación](../results-and-resuming.md). |
| `collectLocationOnStart` | `true` | `false` nunca pide la ubicación, aunque tu app declare el permiso. |
| `verificationTextOverrides` | vacío | Llaves `Unicus_*` para cambiar los textos o el idioma de las pantallas de cámara. Consulta [Textos e idiomas](texts-and-languages.md). |
| `verificationOcrLocalization` | `null` | Textos de la pantalla donde el usuario confirma los datos leídos del documento. Pide el formato a Tekbees. |
| `prependConsent` | `false` | `true` agrega un paso de consentimiento al inicio cuando el flujo asignado no tiene uno. En modo WebView lo muestran las pantallas del flujo; en modo Custom tu proveedor debe soportar `consent`. |
| `requestTimeoutMillis` | `120000` | Tiempo máximo de cada llamada a Unicus. |
| `additionalHeaders` | vacío | Encabezados HTTP adicionales para cada llamada a Unicus (por ejemplo para un proxy corporativo). |
| `enableApiLogging` | `false` | `true` emite logs del API sanitizados a `setApiLogListener`. Solo en builds de depuración. |
| `includeSensitiveApiLogData` | `false` | Mantiene en los logs los campos que normalmente se ocultan. Nunca en producción. |
| `apiLogStringLimit` | `1200` | Longitud máxima de un texto en los logs del API antes de recortarlo. |
| `flowAppUrl` | `null` | Solo si Tekbees te da un ambiente no estándar. |
| `baseUrl` | según `environment` | Constructor avanzado `UnicusSdkConfig(baseUrl, apiKey, …)`. Solo si Tekbees te lo pide. |
| `wrapper` | `null` | Reservado para los SDK envoltorio de Unicus (Flutter). Déjalo en `null`. |

## Ambientes

| `UnicusEnvironment` | Estado en esta versión |
| --- | --- |
| `DEV` | Disponible. |
| `STAGING` | Pendiente: `configure` lanza `environment_not_available`. |
| `PRODUCTION` | Pendiente: `configure` lanza `environment_not_available`. Tekbees lo anunciará con una actualización del SDK. |

Las direcciones y llaves de cada ambiente vienen embebidas en la versión del
SDK, así que un ambiente nuevo llega con una versión nueva del SDK. Puedes
comprobarlo antes de configurar:

{% code overflow="wrap" %}
```kotlin
val env = UnicusEnvironment.fromWireValue(BuildConfig.UNICUS_ENV) ?: UnicusEnvironment.DEV
if (!env.isAvailable) {
    Log.w("Unicus", "El ambiente $env aún no está disponible: ${env.missingValues}")
}
```
{% endcode %}

`configure` lanza `UnicusSdkException` (no comprobada en Java) con código
`environment_not_available` cuando el ambiente no se puede usar.

No guardes el Customer Token en el control de versiones: inyéctalo por tipo de
build (`buildConfigField`, un archivo de propiedades ignorado por Git o los
secretos de tu CI).

## Logs

Para ver las llamadas que hace el SDK mientras integras, activa
`enableApiLogging` y registra un listener; las entradas vienen sanitizadas
(datos biométricos, datos del documento, tokens, valores de formularios,
códigos OTP, teléfono, correo y ubicación se ocultan). Consulta
[Resultados y eventos](results-and-events.md#logs-del-api).

## Marca

Los colores salen de la marca de la empresa en el portal administrativo y se
aplican automáticamente a las pantallas de cámara y a las del flujo. Sin color
de texto, el SDK elige blanco o casi negro, el que mejor contraste con tu color
de marca; sin color de botón, usa el color de marca.

En Android, el logo que muestran las pantallas nativas del SDK (pantallas de
cámara y barra superior de las pantallas del flujo) debe ser un **drawable de
tu app**. Ahí no se cargan URLs remotas de logo:

{% code overflow="wrap" %}
```kotlin
UnicusSdk.shared.setAndroidLogoResourceName("logo_mi_empresa") // res/drawable/logo_mi_empresa.png
```
{% endcode %}

Si no lo defines, se muestra el logo de Unicus.

## Otros miembros

| Miembro | Uso |
| --- | --- |
| `UnicusSdk.shared.version` | Versión del SDK (envíala a [Soporte](../support.md)). |
| `UnicusSdk.shared.configuration` | Configuración actual, `null` antes de `configure`. |
| `UnicusSdk.shared.isSessionActive` | `true` mientras una verificación se prepara o se muestra. |
| `UnicusSdk.shared.clearResumeData(context)` | Olvida las llaves de reanudación guardadas. Llámalo al cerrar sesión. |

Siguiente: [Iniciar una verificación](start-a-verification.md).
