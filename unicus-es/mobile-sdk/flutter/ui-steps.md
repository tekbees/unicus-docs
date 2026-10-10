---
description: >-
  Cómo muestra el SDK de Flutter los pasos no biométricos de un flujo: la vista
  web segura (por defecto), el modo Disabled y por qué el modo Custom es solo
  nativo.
---

# Pasos de interfaz

Un flujo de Unicus combina **pasos biométricos** (prueba de vida, documento,
comparación facial), que siempre se ejecutan en las pantallas nativas de
cámara del SDK, **pasos de servidor** (verificación de edad), que no tienen
pantalla, y **pasos de interfaz**: consentimiento, información, formulario,
firma, OTP y `sign_document`. `uiStepMode` decide cómo se ejecutan los pasos
de interfaz. Los conceptos están en
[Flujos y pasos de interfaz](../flows-and-ui-steps.md).

| `uiStepMode` | Pasos de interfaz | Flujos que se pueden ejecutar |
| --- | --- | --- |
| `UnicusUiStepMode.webView` (por defecto) | Los muestran las pantallas del flujo de Unicus en una vista web segura propiedad del SDK. | Todos. |
| `UnicusUiStepMode.disabled` | No permitidos. | Solo flujos con pasos biométricos y de servidor. |
| Custom (tus propias pantallas) | Los dibuja tu app. | **No disponible desde Flutter.** Solo en los SDK nativos de Android e iOS. |

## WebView (por defecto)

No hay nada que programar: el SDK abre las pantallas del flujo cuando llega un
paso de interfaz y vuelve a la cámara o a tu app cuando termina. Las pantallas
usan la marca de tu compañía y los textos del flujo del portal.

* La vista web es nativa (Android `WebView`, iOS `WKWebView`) y pertenece al
  SDK: tu app no necesita `webview_flutter` y no hay conflicto de versiones
  con él.
* Carga solo las pantallas del flujo de Unicus de tu ambiente; la sesión
  viaja por un canal privado, nunca en la URL.
* `secureScreens: true` bloquea las capturas de pantalla en Android y cubre
  las pantallas en el selector de apps de iOS.
* Android necesita un *Android System WebView* actualizado (Chromium 90 o
  superior); si no, `webview_unavailable`. En iOS, las apps con
  `WKAppBoundDomains` deben incluir el dominio de las pantallas del flujo; si
  no, `webview_domain_not_allowed`. Consulta
  [Compatibilidad y seguridad](../compatibility-and-security.md).

## Disabled

Para apps que no deben mostrar ningún contenido web:

{% code overflow="wrap" %}
```dart
await unicus.configure(const UnicusSdkConfig(
  apiKey: '<CUSTOMER_TOKEN>',
  environment: UnicusEnvironment.dev,
  uiStepMode: UnicusUiStepMode.disabled,
));
```
{% endcode %}

Antes de mostrar nada, el SDK comprueba que cada paso pendiente se pueda
ejecutar. Si el flujo tiene algún paso de interfaz, `start` lanza
`flow_not_supported` (9020) y la transacción se cierra. Asigna a tu compañía
un flujo solo de cámara en el portal y recoge el consentimiento en tu propia
app si lo necesitas.

## El modo Custom es solo nativo

Los SDK nativos ofrecen un modo Custom en el que la app dibuja consentimiento,
formulario, firma, OTP y `sign_document` con sus propias pantallas mientras el
SDK hace las llamadas a Unicus. **No está expuesto a Dart.** Si tu app Flutter
no puede usar la vista web y necesita pasos de interfaz, integra directamente
los SDK nativos (por ejemplo en un módulo add-to-app o una pantalla específica
de cada plataforma):

* [Pasos de interfaz Custom en Android](../android/custom-ui-steps.md)
* [Pasos de interfaz Custom en iOS](../ios/custom-ui-steps.md)

El evento `stepProgress` (progreso de `sign_document`) viene de ese modo, así
que las apps Flutter normalmente no lo reciben. En modo WebView un paso
`sign_document` se reporta con `stepCompleted` / `stepFailed`, y los códigos
de resultado 4011–4014 son los mismos en todas las plataformas.
