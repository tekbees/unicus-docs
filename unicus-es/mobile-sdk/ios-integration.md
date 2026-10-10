---
description: >-
  Integra la verificación de identidad de Unicus en una app nativa de iOS
  (Swift u Objective-C) con el SDK de Unicus para iOS.
---

# Integración iOS

El SDK de Unicus para iOS ejecuta una verificación de identidad completa dentro
de tu app nativa de iOS. Tu app configura el SDK con su Customer Token, llama a
`start` con el documento de la persona y recibe un resultado. El SDK crea la
transacción, aplica la marca de tu empresa, ejecuta el flujo asignado en el
portal administrativo (pantallas de cámara y pantallas del flujo como
consentimiento, formulario u OTP) y se comunica con Unicus. Tu app solo importa
`UnicusSDK`: no agregues otras librerías biométricas o de captura de documentos.

{% hint style="warning" %}
**Próximamente.** Los SDK móviles aún no están disponibles en producción. Hoy
solo está disponible el ambiente DEV; Tekbees anunciará el lanzamiento en
producción. Solicita a Tekbees un Customer Token de DEV para preparar tu
integración.
{% endhint %}

{% code overflow="wrap" %}
```swift
import UnicusSDK

UnicusSdk.shared.configure(UnicusSdkConfig(apiKey: "<CUSTOMER_TOKEN>", environment: .dev))

UnicusSdk.shared.start(
    .enrollmentVerify(document: UnicusDocument(type: .id, externalDatabaseRefId: "123456789")),
    from: self
) { result in
    // Result<UnicusVerificationResult, UnicusSdkError>
}
```
{% endcode %}

## Páginas de esta sección

| Página | Qué encuentras |
| --- | --- |
| [Inicio rápido](ios/quick-start.md) | Dependencia, `Info.plist`, diez líneas de código y los resultados. |
| [Instalación](ios/installation.md) | Contenido del paquete, Xcode / CocoaPods / Swift Package Manager, permisos, la fase de compilación para Debug en dispositivo. |
| [Configuración](ios/configuration.md) | Todas las opciones de `UnicusSdkConfig`, ambientes, logs, marca. |
| [Iniciar una verificación](ios/start-a-verification.md) | Documento, selección de país, transacción existente, ubicación, cancelación, hilos. |
| [Resultados y eventos](ios/results-and-events.md) | `UnicusVerificationResult`, resultados, errores, eventos de progreso, logs de la API. |
| [Pasos de interfaz propios](ios/custom-ui-steps.md) | Muestra las pantallas del flujo con tu propia UI (proveedor `.custom`), incluido `sign_document`. |
| [Textos e idiomas](ios/texts-and-languages.md) | Claves de texto `Unicus_*` y el idioma de cada pantalla. |
| [Objective-C](ios/objective-c.md) | La fachada `UnicusSdkObjC`. |
| [Lista de salida a producción](ios/release-checklist.md) | Qué revisar antes de salir a producción. |

Compartidas por todos los SDK móviles: [Descripción general](overview.md),
[Flujos y pasos de UI](flows-and-ui-steps.md),
[Resultados y reanudación](results-and-resuming.md),
[Códigos de resultado](result-codes.md),
[Errores y solución de problemas](errors-and-troubleshooting.md),
[Compatibilidad y seguridad](compatibility-and-security.md),
[Versiones](versioning.md) y [Soporte](support.md).

## Requisitos

| Elemento | Requisito |
| --- | --- |
| iOS | 15.0 o superior |
| Xcode | 16 o superior |
| Swift | 5.9 o superior (completion handlers y `async/await`). Objective-C mediante una fachada. |
| Dispositivo | iPhone físico con cámara para verificaciones reales. El simulador compila y ejecuta la app, pero no puede ejecutar las pantallas de cámara. |
| Permisos | Cámara (obligatorio). Ubicación (opcional; la verificación continúa sin ella). |
| Red | Acceso HTTPS a los dominios de la API de Unicus y de la app de flujos de tu ambiente. Consulta [Compatibilidad y seguridad](compatibility-and-security.md). |
| Credenciales | El Customer Token de tu empresa para el ambiente. Nada más: el SDK trae todas las demás llaves. |
| Distribución | Paquete entregado por Tekbees (ZIP). Repositorios alojados de Swift Package Manager y CocoaPods: próximamente. |
