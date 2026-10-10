---
description: >-
  Todas las opciones de UnicusSdkConfig en iOS: ambientes, modo de pasos de UI,
  reanudación, ubicación, logs, textos y marca.
---

# Configuración

Configura la instancia compartida una vez, antes de la primera verificación
(por ejemplo al iniciar la app). Si llamas a `configure` de nuevo, la nueva
configuración aplica desde la siguiente verificación.

{% code overflow="wrap" %}
```swift
import UnicusSDK

UnicusSdk.shared.configure(UnicusSdkConfig(apiKey: "<CUSTOMER_TOKEN>", environment: .dev))
```
{% endcode %}

Esa línea basta para la mayoría de las apps. El ambiente selecciona la API de
Unicus, la app de flujos y las llaves incluidas en el SDK; tu app nunca maneja
URL ni llaves internas.

## Cambiar opciones

El inicializador simple también acepta las opciones más comunes:

{% code overflow="wrap" %}
```swift
let config = UnicusSdkConfig(
    apiKey: "<CUSTOMER_TOKEN>",
    environment: .dev,
    uiStepMode: .webView,           // .webView (por defecto) | .custom(provider) | .disabled
    resumeOpenTransactions: true,
    collectLocationOnStart: false,
    enableApiLogging: false
)
UnicusSdk.shared.configure(config)
```
{% endcode %}

Las demás opciones son propiedades `var` que asignas antes de llamar a
`configure`:

{% code overflow="wrap" %}
```swift
var config = UnicusSdkConfig(apiKey: "<CUSTOMER_TOKEN>", environment: .dev)
config.secureScreens = true
config.verificationTextOverrides = [UnicusVerificationTextKey.actionImReady: "ESTOY LISTO"]
UnicusSdk.shared.configure(config)
```
{% endcode %}

## Opciones

| Opción | Por defecto | Cuándo cambiarla |
| --- | --- | --- |
| `apiKey` | — (obligatoria) | El [Customer Token](../../sdk-web-v5/customer-token.md) de tu empresa para el ambiente. La única credencial que entregas. |
| `environment` | — (obligatoria) | `.dev` por ahora. Consulta [Ambientes](#ambientes). |
| `uiStepMode` | `.webView` | Cómo se muestran las pantallas del flujo (consentimiento, información, formulario, firma, OTP, `sign_document`). `.custom(provider)` para mostrarlas con tu UI ([Pasos de interfaz propios](custom-ui-steps.md)); `.disabled` para flujos solo de cámara. Consulta [Flujos y pasos de UI](../flows-and-ui-steps.md). |
| `resumeOpenTransactions` | `true` | `false` para crear siempre una transacción nueva en lugar de continuar la abierta del mismo documento. Consulta [Resultados y reanudación](../results-and-resuming.md). |
| `collectLocationOnStart` | `true` | `false` cuando tu app no debe pedir ubicación. Se puede cambiar por solicitud. |
| `secureScreens` | `false` | `true` para ocultar las pantallas del flujo en la vista previa del selector de apps. |
| `prependConsent` | `false` | `true` para mostrar primero una pantalla de consentimiento de Unicus cuando el flujo no tiene paso de consentimiento. Las apps móviles suelen recoger el consentimiento por su cuenta. |
| `verificationTextOverrides` | `[:]` | Textos o idioma de las pantallas de cámara. Consulta [Textos e idiomas](texts-and-languages.md). |
| `verificationOcrLocalization` | `nil` | Etiquetas de la pantalla de confirmación de datos del documento. Déjala en `nil` salvo que Tekbees te entregue un diccionario. |
| `requestTimeout` | `120` s | Tiempo máximo de cada llamada a la API de Unicus. Redúcelo solo si tu experiencia lo requiere. |
| `additionalHeaders` | `[:]` | Encabezados adicionales en cada llamada a la API de Unicus (por ejemplo, un requisito de un proxy corporativo). |
| `enableApiLogging` | `false` | `true` en desarrollo para recibir logs sanitizados de la API en `apiLogHandler`. Consulta [Resultados y eventos](results-and-events.md). |
| `includeSensitiveApiLogData` | `false` | Conserva tokens y datos cifrados en los logs. Solo para depuración local; nunca en producción. |
| `apiLogStringLimit` | `1200` | Los textos más largos se truncan en los logs. |
| `flowAppUrl` | `nil` | No se necesita con `environment`. Solo para un ambiente que el SDK no conoce (https). |

El inicializador avanzado `UnicusSdkConfig(baseUrl:apiKey:…)` existe para
orígenes de API que no son un ambiente conocido. Prefiere `environment`: con un
origen desconocido las llamadas de sesión fallan con `missing_session_device_id`.

## Ambientes

| `UnicusEnvironment` | Estado |
| --- | --- |
| `.dev` | Disponible. |
| `.staging` | Pendiente. |
| `.production` | Pendiente. Tekbees lo anunciará. |

Con un ambiente pendiente, todas las llamadas (`start`,
`prepareEnrollmentVerify`, `getCompanyCountries`) fallan de inmediato con
`environment_not_available`; `error.details` indica qué falta. Puedes revisarlo
antes:

{% code overflow="wrap" %}
```swift
if !UnicusEnvironment.production.isAvailable {
    print(UnicusEnvironment.production.missingValues)
}
```
{% endcode %}

Cada ambiente tiene su propio Customer Token. Mantén el token y el ambiente
juntos en la configuración de compilación (por ejemplo, un `.xcconfig` por
esquema) para que un token de DEV nunca llegue a una compilación de producción.

## Logs y eventos

Dos handlers en la instancia compartida, ambos invocados en la cola principal:

{% code overflow="wrap" %}
```swift
UnicusSdk.shared.eventHandler = { event in print(event.name, event.message ?? "") }
UnicusSdk.shared.apiLogHandler = { entry in print(entry.summary) }   // requiere enableApiLogging
```
{% endcode %}

Consulta [Resultados y eventos](results-and-events.md).

## Marca

Tu app no configura colores ni logo. El SDK lee la marca de la empresa (colores
y logo) configurada en el portal administrativo para la transacción y la aplica
a las pantallas de cámara y del flujo. Si falta un color, el SDK elige uno
legible por defecto; si el logo no se puede cargar, muestra el logo de Unicus.

## Otros miembros

| Miembro | Uso |
| --- | --- |
| `UnicusSdk.shared.configuration` | Configuración actual, `nil` antes de `configure`. |
| `UnicusSdk.shared.version` | Versión del SDK (inclúyela en las solicitudes de soporte). |
| `UnicusSdk.shared.isSessionActive` | `true` mientras una verificación se prepara o se muestra. |
| `UnicusSdk.shared.clearResumeData()` | Olvida las llaves de reanudación guardadas (llámalo al cerrar sesión). |
