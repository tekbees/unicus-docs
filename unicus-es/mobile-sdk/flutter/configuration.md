---
description: >-
  Todas las opciones de UnicusSdkConfig en el SDK de Flutter: ambientes, modo de
  pasos de interfaz, reanudación, ubicación, logs, textos y marca.
---

# Configuración

Llama `configure` una vez antes de cualquier otra operación, por ejemplo al
iniciar tu app o justo antes de la primera verificación. Llamarlo de nuevo
reemplaza la configuración para el siguiente `start`.

{% code overflow="wrap" %}
```dart
final unicus = UnicusSdkFlutter();

await unicus.configure(const UnicusSdkConfig(
  apiKey: '<CUSTOMER_TOKEN>',
  environment: UnicusEnvironment.dev,
));
```
{% endcode %}

Esa es toda la configuración obligatoria: tu
[Customer Token](../../sdk-web-v5/customer-token.md) y el ambiente. La
dirección del API, la de las pantallas del flujo y las llaves internas vienen
con el SDK para cada ambiente.

{% hint style="warning" %}
No escribas el Customer Token en el código que subes a tu control de
versiones. Cárgalo desde la configuración de tu build o desde tu backend, y
usa un token por ambiente.
{% endhint %}

## Ambientes

| `UnicusEnvironment` | Estado |
| --- | --- |
| `dev` | Disponible. Úsalo para desarrollo y pruebas. |
| `staging` | Pendiente: `configure` lanza `environment_not_available`. |
| `production` | Pendiente: `configure` lanza `environment_not_available`. Tekbees lo anunciará. |

`environment_not_available` lo lanza `configure`, y su mensaje indica qué
falta. Cuando Tekbees publique un ambiente solo actualizas el paquete del SDK:
tu código conserva el mismo valor de `environment`.

## Todas las opciones

| Opción | Por defecto | Cuándo cambiarla |
| --- | --- | --- |
| `apiKey` | obligatoria | Tu Customer Token. |
| `environment` | `null` | Defínelo siempre (`UnicusEnvironment.dev` hoy). Obligatorio salvo que Tekbees te entregue un `baseUrl`. |
| `baseUrl` | `''` | Avanzado. Solo cuando Tekbees te pida usar una dirección explícita del API; reemplaza la dirección de `environment`. |
| `flowAppUrl` | `null` | Avanzado. Origen `https` de las pantallas del flujo, solo para un ambiente que el SDK no conoce. |
| `uiStepMode` | `UnicusUiStepMode.webView` | `UnicusUiStepMode.disabled` si tu app no debe mostrar contenido web (flujos solo de cámara). Consulta [Pasos de interfaz](ui-steps.md). |
| `resumeOpenTransactions` | `true` | `false` para crear siempre una transacción nueva en vez de continuar la abierta del mismo documento. |
| `secureScreens` | `false` | `true` para proteger las pantallas del flujo: sin capturas ni grabación de pantalla en Android, cubiertas en el selector de apps de iOS. Recomendado para apps reguladas. |
| `prependConsent` | `false` | `true` para mostrar primero un paso de consentimiento cuando el flujo no tiene uno (como en la web). Desactivado porque las apps móviles suelen recoger el consentimiento por su cuenta. |
| `collectLocationOnStart` | `true` | `false` para no pedir nunca la ubicación. Con `true`, se recoge solo si tu app declara el permiso y el usuario lo concede. |
| `requestTimeout` | 2 minutos | Tiempo máximo de cada llamada a Unicus. Súbelo solo para redes muy lentas. |
| `additionalHeaders` | `{}` | Encabezados HTTP extra en cada llamada a Unicus, por ejemplo uno que exige tu proxy corporativo. |
| `verificationTextOverrides` | `{}` | Tu redacción o idioma para las pantallas de cámara. Consulta [Textos e idiomas](texts-and-languages.md). |
| `verificationOcrLocalization` | `null` | Etiquetas de la pantalla de confirmación de datos del documento. Coordina el diccionario con soporte de Tekbees. |
| `androidLogoResourceName` | `null` | Solo Android: nombre de un drawable de tu app que se muestra como logo en las pantallas de cámara (por ejemplo `customer_logo`). |
| `enableApiLogging` | `false` | `true` en desarrollo para recibir logs sanitizados en `unicus.apiLogs`. |
| `includeSensitiveApiLogData` | `false` | Solo para depuración: conserva payloads cifrados y tokens en los logs. Nunca en un build de release. |
| `apiLogStringLimit` | `1200` | Longitud máxima de cada texto en los logs antes de truncarlo. |

## Configuración recomendada

{% code overflow="wrap" %}
```dart
await unicus.configure(UnicusSdkConfig(
  apiKey: customerToken, // cargado desde tu configuración
  environment: UnicusEnvironment.dev,
  secureScreens: true,
  verificationTextOverrides: myUnicusTexts, // opcional, ver Textos e idiomas
  androidLogoResourceName: 'customer_logo', // opcional, logo en Android
));
```
{% endcode %}

## Logs

Los logs del API están desactivados por defecto. Actívalos solo mientras
desarrollas:

{% code overflow="wrap" %}
```dart
await unicus.configure(const UnicusSdkConfig(
  apiKey: '<CUSTOMER_TOKEN>',
  environment: UnicusEnvironment.dev,
  enableApiLogging: true,
));

unicus.apiLogs.listen((entry) {
  debugPrint(entry.summary); // "/path -> 200 (350 ms)"
});
```
{% endcode %}

Los blobs de solicitud y respuesta, los datos del documento, los resultados de
OCR y los tokens de sesión se reemplazan por `<redacted>` salvo que
`includeSensitiveApiLogData` sea `true`. Mantén ambas opciones desactivadas en
los builds de release. Consulta
[Resultados y eventos](results-and-events.md#logs-del-api).

## Marca

Los colores y el logo vienen de la configuración de tu compañía en el portal
administrativo; no los pasas en código.

* **Colores** (fondo, principal, botones, texto): se aplican a las pantallas
  de cámara y a las del flujo en ambas plataformas.
* **Logo en iOS:** el SDK carga el logo configurado en el portal (un PNG o JPG
  por `https`, una imagen raster en línea o un SVG simple hecho de trazos
  básicos). Si no lo puede decodificar, usa el logo de Unicus.
* **Logo en Android:** las pantallas de cámara solo pueden mostrar una imagen
  de los recursos de tu app. Agrega tu logo en
  `android/app/src/main/res/drawable` y pasa su nombre en
  `androidLogoResourceName`; si no, se muestra el logo de Unicus. Las
  pantallas del flujo (vista web) usan el logo del portal en ambas
  plataformas.

Siguiente: [Iniciar una verificación](start-a-verification.md).
