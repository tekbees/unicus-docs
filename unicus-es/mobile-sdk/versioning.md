---
description: >-
  Versiones de los SDK móviles de Unicus, notas de la versión y política de
  compatibilidad.
---

# Versiones

{% hint style="warning" %}
**Próximamente.** Los SDK móviles de Unicus aún no están disponibles en
producción. Hoy solo está disponible el ambiente **DEV**; Tekbees anunciará el
lanzamiento en producción. Mientras tanto, solicita a Tekbees el paquete del SDK
y un Customer Token de DEV para preparar tu integración.
{% endhint %}

## Versiones

Los SDK de Android, iOS y Flutter comparten un mismo número de versión y un
mismo comportamiento: los mismos flujos, códigos de resultado y códigos de
error. El SDK de Flutter de una versión contiene los SDK nativos de esa misma
versión.

| SDK | Versión actual | Dónde leerla |
| --- | --- | --- |
| Android | `0.1.0` | `UnicusSdk.shared.version`; la dependencia `com.tekbees.unicus:unicus-sdk:<versión>` |
| iOS | `0.1.0` | `UnicusSdk.shared.version` |
| Flutter | `0.1.0` | `version` en el `pubspec.yaml` del plugin |

Incluye la versión cuando contactes a [Soporte](support.md).

## Distribución

Hoy Tekbees entrega cada SDK como un **paquete** (ZIP) con el SDK, un inicio
rápido, la lista de errores y una app de ejemplo. Los repositorios alojados de
Maven, Swift Package Manager y CocoaPods llegarán próximamente: con ellos,
actualizar será cambiar una versión en tu archivo de build.

## Política de compatibilidad

* Las versiones siguen `MAJOR.MINOR.PATCH`. Las versiones patch y minor son
  compatibles: correcciones, nueva configuración opcional, nuevos eventos y
  nuevos campos.
* Los cambios incompatibles (una opción eliminada, un nuevo caso en un enum
  sobre el que tu código hace `switch`) llegan en una nueva versión major y se
  anuncian con anticipación. Antes del lanzamiento en producción, Tekbees aún
  puede hacer pequeños cambios incompatibles; se listan más abajo.
* **Los flujos evolucionan sin actualizar el SDK.** Los nuevos tipos de paso de
  UI corren en modo WebView con la versión que tienes. Los nuevos tipos de paso
  de cámara o un formato de flujo más nuevo requieren actualizar el SDK:
  mientras tanto el SDK responde `flow_not_supported` antes de mostrar nada,
  nunca a mitad de un flujo.
* Si las pantallas del flujo requieren un protocolo de comunicación más nuevo
  que el que soporta tu versión del SDK, el SDK responde
  `bridge_protocol_mismatch`: actualiza el SDK.
* Los nuevos ambientes (STAGING, PRODUCTION) llegan con una nueva versión del
  SDK. Una versión anterior responde `environment_not_available`.

## 0.1.0 (versión preliminar, ambiente DEV)

Primera versión de los SDK nativos de Android e iOS y del SDK de Flutter
construido sobre ellos.

* **Configuración simple.** `UnicusSdkConfig(apiKey, environment)`: el ambiente
  selecciona el API, las pantallas del flujo y las llaves embebidas. Los
  ambientes aún no publicados responden `environment_not_available`.
* **Flujos modulares.** El flujo asignado en el portal corre en el dispositivo:
  pasos de cámara en pantallas nativas, pasos de UI en modo WebView (por
  defecto), en tus propias pantallas con el modo Custom (Android, iOS) o
  desactivados. Revisión del flujo antes de mostrar nada
  (`flow_not_supported`, `9020`). Eventos de avance por segmento y por paso.
* **Firma de documentos** (`sign_document`) en los modos WebView y Custom, con
  los códigos `4011`–`4014`.
* **Salir no es cancelar.** Salir de una pantalla devuelve `2003` `RESUMABLE`
  con el `tid`; solo una cancelación dentro de la cámara o `cancelActiveSession`
  cancela (`2041`). `cancelActiveSession` termina el flujo donde esté.
* **Reanudación automática.** `resumeOpenTransactions` (activo por defecto): el
  siguiente `start` con el mismo documento continúa la transacción abierta;
  `clearResumeData()` al cerrar sesión.
* **Resultados.** `rejectionReason` y `rejectionDetail` para `2052`,
  `isRetryable`, `isResumable`, `resumed`, resultados por paso.
* **Selección de país** para empresas con varios países activos.
* **Español e inglés.** `UnicusSdkConfig.language` (por defecto el idioma del
  dispositivo) para las pantallas de cámara, las pantallas del flujo, el
  diálogo de salida y `resultMessage`; mensajes de resultado dirigidos al
  usuario.
* **Seguridad.** WebView restringido, logs del API sanitizados, llave de
  reanudación cifrada, SDK ofuscado, `secureScreens` opcional.
* Fachada **Objective-C** en iOS; API amigable con Java y corrutinas de Kotlin
  en Android.

### Cambios durante la versión preliminar

Si recibiste un paquete anterior de `0.1.0`, revisa estos cambios:

| Cambio | Qué hacer |
| --- | --- |
| Nuevo outcome `RESUMABLE` para `2003` (antes `WARNING`). | Agrega el caso a los `when` / `switch` exhaustivos sobre el outcome. |
| Android y Flutter en Android: el SDK ya no agrega los permisos de ubicación. | Declara `ACCESS_FINE_LOCATION` / `ACCESS_COARSE_LOCATION` en tu app si quieres la ubicación. |
| Flutter: `rejectionReason` y `rejectionDetail` vienen de los SDK nativos. | Ninguna. |
| Flutter en Android: Kotlin Gradle Plugin 2.0 o superior (antes 2.3). | Ninguna; ahora se aceptan versiones de Kotlin más antiguas. |
| Iniciar con una URL base es una opción avanzada. | Prefiere `UnicusSdkConfig(apiKey, environment)`. |
| Opción de idioma `UnicusSdkConfig.language` (por defecto: idioma del dispositivo). Un dispositivo en español ahora ve las pantallas de cámara en español sin reemplazos, y un dispositivo en inglés recibe `resultMessage` en inglés. | Para mantener todo en español, fija el idioma en español. Si entregabas la tabla completa de textos en español solo para traducir las pantallas, reemplázala por la opción. Textos e idiomas: [Android](android/texts-and-languages.md), [iOS](ios/texts-and-languages.md), [Flutter](flutter/texts-and-languages.md). |
| `resultMessage` / `messageFor` le hablan al usuario (por ejemplo `4001` «Se agotaron los intentos.»; por error decía «Proceso cancelado por el usuario.»). Los códigos y outcomes no cambian. | Si tu app comparaba textos de mensajes, compara `resultCode`. |
| Las pantallas de cámara se ven como las del Web SDK 5.0: colores, animaciones, textos y etiquetas de los datos del documento. Sin colores de la empresa, el tema por defecto es el del Web SDK (azul `#2c6ba9`), ya no amarillo y azul oscuro. Las instrucciones de captura del documento son más cortas para que quepan en el recuadro. | Ninguna. Revisa las capturas de pantalla de tus propias guías. |
| Logo cuando no hay logo de la empresa: en iOS las pantallas de cámara no muestran logo (igual que en la web); Android muestra el logo de Unicus a color. | Ninguna. |
| Motor de cámara más reciente (paquetes desde el 14 de octubre de 2026). Corrige la pantalla de «gira el dispositivo» que no desaparecía en los últimos iPhone y versiones de iOS. | Actualiza el SDK. |
| Algunos textos de `resultMessage` ahora dicen a quién contactar (`2002`, `4012`, `4013`: «Contacta a quien te envió el enlace.»); `2013` dice que la validación quedó en revisión manual; los códigos de salida nativos `9991`–`9998` y de revisión del documento `10001`–`10020` tienen su propio mensaje. Los códigos y outcomes no cambian. | Si tu app comparaba textos de mensajes, compara `resultCode`. |
| `4000` (`INVALID_REQUEST`, outcome `ERROR`) en Android, iOS y Flutter. | Manéjalo como un error de integración; consulta [Códigos de resultado](result-codes.md). |
| El diálogo de salida dice «¿Salir de la validación?» y, también en iOS, que el avance queda guardado y la validación se puede retomar. Textos de cámara con nueva redacción. | Ninguna. Revisa las capturas de pantalla de tus propias guías. |
| `resultMessage` le habla a la persona cuando la sesión de cámara termina antes de tiempo (por ejemplo `2061` «Se agotaron los intentos de esta validación.») y cuando sale de la cámara después de un `7006` («No pudimos confirmar que el documento sea físico. Intenta de nuevo con el documento original.»); antes traía el texto técnico. El detalle queda en `errorMessage`. La pantalla de reintento del documento pide «tu documento original». Códigos y outcomes sin cambios. | Si tu app guardaba `resultMessage` para diagnóstico, usa `errorMessage`. |
| Nuevo `serverResponse.expectedMediaStatusEnumInt` (`0` documento probablemente físico, `1` no confirmado). `digitalIdSpoofStatusEnumInt` queda deprecado con el mismo valor; antes llegaba siempre en `0`. | Si lo leías, cambia a `expectedMediaStatusEnumInt`. El código anterior sigue compilando, con una advertencia. |
