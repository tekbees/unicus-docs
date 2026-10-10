---
description: >-
  Integra la verificación de identidad de Unicus en una app Flutter para
  Android e iOS: qué hace el SDK de Flutter, las páginas de esta sección y los
  requisitos.
---

# Integración Flutter

{% hint style="warning" %}
**Próximamente.** Los SDK móviles de Unicus aún no están disponibles en
producción. Hoy solo está disponible el ambiente **DEV**; `staging` y
`production` responden `environment_not_available`. Tekbees anunciará el
lanzamiento en producción.
{% endhint %}

El SDK de Unicus para Flutter (`unicus_sdk_flutter`) es una capa delgada en
Dart sobre los SDK nativos de Unicus para Android e iOS. Tu app llama
`configure` una vez y `start` con el documento del usuario; el SDK crea (o
retoma) la transacción, ejecuta el flujo que tu compañía asignó en el portal
administrativo (los pasos de cámara en pantallas nativas; consentimiento,
formulario, firma y OTP en la vista web segura del SDK) y devuelve un
`UnicusVerificationResult`. No editas `MainActivity` ni `AppDelegate`, no
llamas el API de Unicus y no agregas otros paquetes biométricos. El único
secreto que pasa tu app es el
[Customer Token](../sdk-web-v5/customer-token.md), como `apiKey`.

{% code overflow="wrap" %}
```dart
final unicus = UnicusSdkFlutter();
await unicus.configure(
  const UnicusSdkConfig(apiKey: '<CUSTOMER_TOKEN>', environment: UnicusEnvironment.dev),
);
final result = await unicus.start(
  const UnicusVerificationRequest.enrollmentVerify(
    document: UnicusDocument(type: UnicusDocumentType.id, externalDatabaseRefId: '123456789'),
  ),
);
```
{% endcode %}

## Páginas de esta sección

| Página | Qué encontrarás |
| --- | --- |
| [Inicio rápido](flutter/quick-start.md) | Instalar, configurar, iniciar y manejar el resultado en pocas líneas. |
| [Instalación](flutter/installation.md) | Contenido del paquete, `pubspec.yaml`, configuración de Gradle en Android, Podfile e `Info.plist` en iOS, builds Debug en un iPhone físico. |
| [Configuración](flutter/configuration.md) | Todas las opciones de `UnicusSdkConfig`, ambientes, logs, marca. |
| [Iniciar una verificación](flutter/start-a-verification.md) | Tipos de documento, selección de país, transacciones existentes, ubicación, cancelación, reglas de hilos. |
| [Resultados y eventos](flutter/results-and-events.md) | `UnicusVerificationResult`, resultados, los streams `events` y `apiLogs`. |
| [Pasos de interfaz](flutter/ui-steps.md) | Modos WebView (por defecto) y Disabled; por qué Custom es solo nativo. |
| [Textos e idiomas](flutter/texts-and-languages.md) | Llaves de texto `Unicus_*`, personalización e idiomas de las pantallas del flujo. |
| [Lista de salida a producción](flutter/release-checklist.md) | Qué probar y revisar antes de publicar. |

Compartidas por los tres SDK móviles: [Visión general](overview.md),
[Flujos y pasos de interfaz](flows-and-ui-steps.md),
[Resultados y reanudación](results-and-resuming.md),
[Códigos de resultado](result-codes.md),
[Errores y solución de problemas](errors-and-troubleshooting.md),
[Compatibilidad y seguridad](compatibility-and-security.md),
[Versiones](versioning.md) y [Soporte](support.md).

## Requisitos

| | Mínimo |
| --- | --- |
| Flutter / Dart | Flutter 3.19, Dart 3.3 |
| Android | `minSdk` 21, `compileSdk` 34, Android Gradle Plugin 8.x o 9.x, Kotlin Gradle Plugin 2.0, JDK 17. Tu versión de Flutter puede exigir más: Flutter 3.47 exige AGP 8.11.1 y Kotlin Gradle Plugin 2.2.20. |
| Dispositivo Android | Android System WebView basado en Chromium 90 o superior (para flujos con pasos de interfaz), cámara. |
| iOS | iOS 15.0, CocoaPods (Swift Package Manager no es compatible en esta versión). |
| Permisos | Cámara (Android: la agrega el plugin; iOS: `NSCameraUsageDescription`). La ubicación es opcional. |
| Credenciales | El Customer Token de tu compañía. No hay URL ni llave interna que configurar: el SDK las trae según el ambiente. |

Hoy la distribución es un paquete que entrega Tekbees (ZIP). Los repositorios
públicos (pub, Maven, CocoaPods, Swift Package Manager) llegarán pronto.

{% hint style="info" %}
El SDK de Flutter ejecuta el mismo motor nativo que los SDK de Android e iOS,
así que los flujos, los códigos de resultado y los errores son idénticos en los
tres. Solo el modo de pasos de interfaz *Custom* (tus propias pantallas para
consentimiento, formularios, firma y OTP) no está disponible desde Dart; las
apps que lo necesitan integran directamente el SDK de
[Android](android-integration.md) o de [iOS](ios-integration.md).
{% endhint %}
