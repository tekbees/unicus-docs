---
description: >-
  Versiones mínimas, dispositivos y requisitos de WebView de los SDK móviles de
  Unicus, qué deben permitir los entornos enterprise, los datos que maneja el
  SDK y su modelo de seguridad. Léelo antes de salir a producción.
---

# Compatibilidad y seguridad

## Versiones mínimas

| | Android | iOS | Flutter |
| --- | --- | --- | --- |
| Sistema operativo | Android 5.0 (`minSdk 21`) | iOS 15.0 | Android e iOS como en las columnas nativas |
| Build | `compileSdk 34` o superior, Android Gradle Plugin 8.x o 9.x, Java 17 | Xcode 16 o superior | Flutter 3.19 o superior, Dart 3.3 o superior |
| Lenguaje | Kotlin (Kotlin Gradle Plugin 2.0 o superior) o solo Java | Swift 5.9; Objective-C mediante una fachada | Dart; en Android, Kotlin Gradle Plugin 2.0 o superior (tu versión de Flutter puede exigir más) |
| Dependencias | AndroidX. El SDK trae `appcompat`, `core-ktx`, `kotlinx-coroutines-android`, `androidx.webkit` y `androidx.browser`. | Solo frameworks del sistema | Sin dependencia de `webview_flutter` ni `http` |
| Gestor de paquetes | Repositorio Maven en el paquete | XCFrameworks (Embed & Sign) o CocoaPods | Dependencia `path:`; CocoaPods en iOS (Swift Package Manager aún no está soportado) |

Instalación por plataforma: [Android](android/installation.md),
[iOS](ios/installation.md), [Flutter](flutter/installation.md).

## Dispositivos

| Requisito | Por qué |
| --- | --- |
| Dispositivo físico con cámara | Prueba de vida y captura del documento. Los emuladores y simuladores compilan y ejecutan la app, pero no pueden completar una verificación. |
| Permiso de cámara | Lo pide el SDK la primera vez que abre la cámara. Si el usuario lo niega, el resultado es `9996`. |
| Permiso de ubicación (opcional) | Se registra con la transacción cuando tu app lo declara y el usuario lo permite. Si no, la verificación continúa sin él. |
| Conexión estable | Los envíos son pequeños pero deben completarse. El SDK reintenta automáticamente las fallas de red y nunca repite una solicitud que pudo haber llegado a Unicus cuando eso podría duplicar una transacción o un código OTP. |

## Requisitos de WebView

Solo para flujos con pasos de UI en modo WebView (el modo por defecto). Consulta
[Flujos y pasos de UI](flows-and-ui-steps.md).

| Plataforma | Requisito |
| --- | --- |
| Android | Android System WebView (Chromium) **90 o superior**, habilitado, con web message listeners. El SDK lo revisa antes de mostrar nada y falla con `webview_unavailable` en lugar de una página en blanco. |
| iOS | El WebKit del sistema (siempre presente en iOS 15+). Si tu app activa App-Bound Domains, consulta más abajo. |
| Flutter | El WebView vive en los SDK nativos; no hay conflicto de versiones con `webview_flutter` en tu app. |

## Entornos enterprise

### Listas de dominios permitidos (proxy, firewall, MDM, VPN por app)

Permite los dos dominios de tu ambiente:

| Ambiente | API de Unicus | Pantallas del flujo |
| --- | --- | --- |
| DEV | `alpha.idunicus.com` (puerto 8080) | `dev-id.idunicus.com` |
| STAGING | Se publican cuando se habilite el ambiente. | Se publican cuando se habilite el ambiente. |
| PRODUCTION | Se publican con el lanzamiento en producción. | Se publican con el lanzamiento en producción. |

Las pantallas del flujo cargan desde una dirección fija sin datos en la URL.
Funcionan detrás de un proxy corporativo. Las apps gestionadas (Intune App SDK,
BlackBerry Dynamics, AppConfig) deben permitir ambos dominios en su VPN por app o
túnel.

### App-Bound Domains en iOS

Si tu `Info.plist` declara `WKAppBoundDomains`, agrega el dominio de las
pantallas del flujo de tu ambiente. Si no, `start` se detiene antes de mostrar
nada con `webview_domain_not_allowed`.

### RASP y app shielding

Herramientas como Promon, Appdome, DexGuard o iXGuard pueden bloquear o alertar
sobre los WebView y la inyección de scripts. El SDK:

* no agrega scripts a la página ni usa `addJavascriptInterface`;
* se comunica con las pantallas del flujo por el canal de mensajes de la
  plataforma (`WebViewCompat.addWebMessageListener` en Android,
  `WKScriptMessageHandler` en iOS), restringido al marco principal y al origen
  de las pantallas del flujo.

Si tu herramienta exige una lista de permitidos, incluye las pantallas del SDK:

| Plataforma | Componentes |
| --- | --- |
| Android | `com.tekbees.unicus.sdk.internal.UnicusVerificationActivity`, `com.tekbees.unicus.sdk.internal.UnicusFlowWebActivity`, `com.tekbees.unicus.sdk.internal.UnicusPermissionActivity` |
| iOS | `UnicusFlowWebViewController` de `UnicusSDK.framework` |

### Certificate pinning y apps sin WebView

Los WebView no aplican el certificate pinning de tu app (el WebView de Android
ignora los pins de `network_security_config`). Si tu política exige pinning en
todas las pantallas, o prohíbe los WebView con contenido remoto, usa el modo
**Custom** (Android, iOS) o el modo **Disabled** para flujos solo biométricos.
Las apps Flutter que necesiten el modo Custom deben integrar los SDK nativos.

### Capturas de pantalla y selector de apps

Pon `secureScreens` en `true` para proteger las pantallas del flujo:
`FLAG_SECURE` en Android (sin capturas ni grabación de pantalla) y una cubierta
de privacidad en el selector de apps de iOS. Por defecto: `false`.

## Modelo de seguridad

* **Una sola credencial.** Tu app entrega solo el Customer Token (`apiKey`). El
  SDK trae los demás identificadores que necesita; no hay opción para pasarlos.
  Mantén el Customer Token de cada ambiente fuera de repositorios públicos.
* **Autoridad del servidor.** El orden de los pasos, el avance y los resultados
  los decide Unicus. El dispositivo nunca decide que un paso pasó.
* **Captura cifrada.** Los escaneos del rostro y las imágenes del documento se
  cifran en el dispositivo con el motor biométrico y se envían solo a Unicus.
  Nunca llegan a tu app, al resultado ni a los eventos.
* **Sesión ligada a una transacción.** El SDK guarda la sesión de la transacción
  en memoria y la pasa a las pantallas del flujo por un canal privado, nunca en
  una URL, almacenamiento ni logs.
* **WebView restringido.** JavaScript solo para el origen de las pantallas del
  flujo; navegación a cualquier otro origen bloqueada; sin acceso a archivos,
  contenido mixto ni ventanas emergentes; almacenamiento y cookies del flujo
  borrados al cerrar la pantalla. El SDK no activa la depuración del WebView.
* **Enlaces externos.** Los avisos de privacidad y los documentos para firmar
  se abren solo por `https`, en Custom Tabs (Android) o en una vista de Safari
  (iOS).
* **Llave de reanudación cifrada.** La llave que retoma una transacción abierta
  se guarda cifrada con Android Keystore (Android 6.0 o superior; en versiones
  anteriores no se guarda) o en el Llavero de iOS (solo este dispositivo,
  después del primer desbloqueo). Está ligada a un hash de tu Customer Token y
  del documento, caduca a las 24 horas y nunca se registra en logs. Consulta
  [Retomar](results-and-resuming.md#retomar).
* **Ofuscación.** El código interno del SDK está ofuscado; los identificadores
  embebidos están enmascarados en el binario. En Android el SDK trae sus propias
  reglas de R8 / ProGuard: tu app no necesita reglas adicionales.

## Datos que maneja el SDK

| Dato | Origen | A dónde va |
| --- | --- | --- |
| Customer Token | Configuración de tu app | Unicus, para crear la transacción. |
| Tipo y número de documento | Tu app (`start`) | Unicus. |
| Escaneo del rostro e imágenes del documento | Cámara | Cifrados, solo a Unicus. |
| Datos leídos del documento | Unicus | Se muestran al usuario para que los confirme; no van en el resultado. |
| Ubicación (opcional) | Dispositivo | Unicus, como metadato de la transacción. |
| Consentimiento, valores del formulario, teléfono o correo del OTP, firma | Usuario, en los pasos del flujo | Unicus. |
| Versión del SDK y plataforma | SDK | Unicus, en el user agent. |
| Llave de reanudación | Unicus | Cifrada en el dispositivo; se borra con un resultado final. |

El resultado en tu app trae solo el `tid`, códigos y los resultados de los
pasos. Los datos personales y las imágenes están disponibles para tu backend
mediante
[Consultar el estado de una transacción](../sdk-web-v5/transaction-status.md), y
Unicus los procesa bajo el acuerdo de tratamiento de datos de tu empresa.

## Logs

Los logs del API están desactivados por defecto. Cuando los activas
(`enableApiLogging`), se sanitizan: se ocultan los datos biométricos cifrados,
los datos del documento, los tokens de sesión y los datos personales de los
pasos del flujo (códigos, teléfonos, correos, valores de formulario, firmas,
número de documento, ubicación), y se recortan los valores largos. La llave de
reanudación siempre se oculta.

`includeSensitiveApiLogData` muestra los datos personales. Úsalo solo en tus
propios dispositivos de prueba, nunca en un build de producción.

## Privacidad

* Explica a tus usuarios por qué verificas su identidad. Agrega un paso
  `consent` a tu flujo, o activa `prependConsent` (consulta
  [Flujos y pasos de UI](flows-and-ui-steps.md)).
* **Android:** declarar los permisos de ubicación implica informar la ubicación
  (recolectada, no compartida) en el formulario *Seguridad de los datos* de
  Google Play. El SDK por sí mismo no agrega permisos de ubicación.
* **iOS:** el SDK incluye un manifiesto de privacidad. Escribe textos claros para
  `NSCameraUsageDescription` y, si recolectas la ubicación,
  `NSLocationWhenInUseUsageDescription`, y refleja los datos de arriba en los
  detalles de privacidad de tu app en el App Store.
* Llama a `clearResumeData()` cuando un usuario cierre sesión, para que otra
  persona en el mismo dispositivo nunca continúe su transacción.
