---
description: >-
  Cómo obtener ayuda con una integración de un SDK móvil de Unicus y qué enviar
  para que el equipo pueda diagnosticarla rápido.
---

# Soporte

Lo más importante para nosotros es que tu integración funcione. Contáctanos por:

* Correo electrónico: [support@tekbees.com](mailto:support@tekbees.com)
* Tu gerente de proyecto (asignado una vez iniciado el contrato).

Antes de escribir, revisa [Errores y solución de problemas](errors-and-troubleshooting.md)
y la lista de verificación de lanzamiento de tu plataforma:
[Android](android/release-checklist.md), [iOS](ios/release-checklist.md),
[Flutter](flutter/release-checklist.md).

## Qué enviar

| Dato | Cómo obtenerlo |
| --- | --- |
| `tid` de la transacción | `result.tid`, o la lista de transacciones del portal. |
| Plataforma y versión del SDK | Android, iOS o Flutter, y la versión (consulta [Versiones](versioning.md)). |
| Ambiente | `DEV`, `STAGING` o `PRODUCTION`. Nunca envíes el Customer Token. |
| Dispositivo y versión del sistema | Por ejemplo "Samsung A14, Android 14" o "iPhone 13, iOS 18.1". En Android, también la versión de Android System WebView para problemas con las pantallas del flujo. |
| Qué recibió tu app | El `outcome`, `resultCode` y `rejectionReason` del resultado, o el `code`, `resultCode`, estado HTTP y `message` del error. |
| Configuración | `uiStepMode`, y si `secureScreens`, `prependConsent` y `resumeOpenTransactions` están activos. |
| Logs del API | Con `enableApiLogging` activo e `includeSensitiveApiLogData` **desactivado**. |
| Fecha y hora | Con la zona horaria. |
| Pasos para reproducirlo | Qué hizo el usuario, y una captura o video si el problema se ve en pantalla. |

{% hint style="danger" %}
Nunca envíes logs capturados con `includeSensitiveApiLogData` activo, fotos de
documentos de identidad reales ni Customer Tokens. Los logs sanitizados y el
`tid` son suficientes: Tekbees puede consultar la transacción en Unicus.
{% endhint %}
