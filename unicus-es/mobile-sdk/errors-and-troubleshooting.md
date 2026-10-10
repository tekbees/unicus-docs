---
description: >-
  Todos los códigos de error de los SDK móviles de Unicus, qué significan y qué
  hacer, más los problemas de integración más comunes y cómo resolverlos.
---

# Errores y solución de problemas

## Errores frente a resultados

El SDK lanza un error solo cuando la verificación **no pudo correr**:
`UnicusSdkException` en Android y Flutter, `UnicusSdkError` en iOS (en
Objective-C, un `NSError` con dominio `com.tekbees.unicus.sdk` y el código en
`userInfo["UnicusErrorCode"]`). Una verificación que corrió siempre termina en
un resultado, también cuando la persona no quedó verificada (consulta
[Resultados y reanudación](results-and-resuming.md)).

| Campo | Descripción |
| --- | --- |
| `code` | Código de error estable (tabla abajo). Decide según él. |
| `message` | Texto para desarrolladores. No lo muestres al usuario. |
| `resultCode` | Código de resultado de Unicus, cuando Unicus respondió con uno (por ejemplo `2002` en `transaction_refused`). |
| Estado HTTP | `httpStatus` en Android e iOS, `statusCode` en Flutter: el estado HTTP de la respuesta de Unicus (401, 403, 429...). |
| Detalles | La respuesta de Unicus o la causa original, para soporte. |

## Códigos de error

Los códigos marcados con una plataforma solo existen allí; los demás existen en
todas las plataformas.

### Configuración

| `code` | Qué pasó | Qué hacer |
| --- | --- | --- |
| `environment_not_available` | `configure` con un ambiente que esta versión del SDK aún no trae (STAGING, PRODUCTION). El mensaje dice qué falta. | Usa `DEV`, o actualiza el SDK cuando Tekbees publique el ambiente. |
| `not_configured` | Se llamó una operación antes de `configure`. | Llama a `configure` una vez al iniciar tu app. |
| `invalid_configuration` | iOS, Flutter: sin ambiente ni URL base, URL base vacía, o un nombre de ambiente desconocido. | Usa la configuración con `apiKey` y `environment`. |
| `missing_api_key` | `apiKey` está vacío. | Configura el Customer Token de tu empresa. |
| `missing_session_device_id` | Una URL base propia de un ambiente para el que el SDK no tiene llaves. | Usa `environment` en lugar de una URL base. |
| `missing_embedded_sdk_device_key` | El build del SDK no tiene la llave del motor biométrico. | Build defectuoso: contacta a Tekbees. |
| `invalid_endpoint` | iOS: no se pudo construir la URL del API a partir de la URL base. | Usa `environment`. |

### Inicio de la verificación

| `code` | Qué pasó | Qué hacer |
| --- | --- | --- |
| `missing_document` | `start` sin documento y sin `tid`. | Inicia con `enrollmentVerify` y el documento. |
| `invalid_request` | iOS (Objective-C), Flutter: tipo de documento desconocido, o sin documento ni `tid`. | Usa `ID`, `FD`, `PP` o `DL`. |
| `session_active` | Ya hay otra verificación en curso. | Espera su resultado o llama a `cancelActiveSession`. |
| `transaction_refused` | Unicus no creó la transacción. `resultCode`: `2002` sin flujo asignado, `2012` persona no enrolada, `2052` usuario bloqueado. | Muestra un mensaje de negocio según el `resultCode`; revisa tu empresa en el portal. |
| `missing_tid` | Unicus respondió sin id de transacción. | Reintenta; contacta a soporte con los logs del API si persiste. |
| `flow_not_supported` | El flujo asignado no puede correr en esta app (`9020`): pasos de UI con el modo Disabled, un tipo de paso que tu proveedor Custom no soporta, una combinación de cámara no soportada, un flujo no publicado para móvil, o un formato de flujo que este SDK no sabe leer. La transacción se cierra. | Corrige el flujo en el portal o el `uiStepMode`; actualiza el SDK para flujos más nuevos. Consulta [Reglas de flujo en móvil](flows-and-ui-steps.md). |

### Sesión y red

| `code` | Qué pasó | Qué hacer |
| --- | --- | --- |
| `network_error` | Sin conexión o tiempo agotado tras los reintentos automáticos. | Revisa la conexión y empieza de nuevo: la transacción abierta se retoma. |
| `rate_limited` | Demasiadas solicitudes (HTTP 429). | Espera un momento y empieza de nuevo. |
| `temporary_failure` | Falla temporal de Unicus (`2054`). No se procesó nada. | Empieza de nuevo. |
| `transaction_expired` | La transacción no existe o expiró (`2051`). | Inicia una nueva verificación. |
| `session_expired` | La sesión de la transacción venció y no se pudo renovar, o una pantalla del flujo la reportó vencida (`2051`, HTTP 401). | Empieza de nuevo. |
| `session_mismatch` | La sesión pertenece a otra transacción (HTTP 403). | Empieza de nuevo; contacta a soporte si persiste. |
| `session_refresh_failed` | iOS: no se pudo volver a leer el avance del flujo entre pasos. | Empieza de nuevo: la transacción se retoma. |
| `invalid_response` | Android: Unicus devolvió una respuesta que el SDK no pudo leer. | Reintenta; contacta a soporte si persiste. |
| `http_<status>` | Estado HTTP inesperado sin otro código (por ejemplo `http_500`). | Reintenta; contacta a soporte con el estado. |

### Pantallas del flujo (modo WebView)

| `code` | Qué pasó | Qué hacer |
| --- | --- | --- |
| `webview_unavailable` | Android: Android System WebView ausente, deshabilitado o anterior a Chromium 90. Reservado en iOS. | Pide al usuario que actualice *Android System WebView* desde Google Play, o usa el modo Custom. |
| `webview_domain_not_allowed` | iOS: tu app declara `WKAppBoundDomains` sin el dominio de la flow app de Unicus. | Agrega el dominio al `Info.plist`, o usa el modo Custom. |
| `webview_load_failed` | Las pantallas del flujo no cargaron tras un reintento. | Revisa la red, el proxy y las listas de dominios permitidos para el dominio de la flow app. |
| `bridge_protocol_mismatch` | Las pantallas del flujo usan una versión que este SDK no soporta. | Actualiza el SDK. |

### App y dispositivo

| `code` | Qué pasó | Qué hacer |
| --- | --- | --- |
| `no_view_controller` | iOS, Flutter en iOS: no hay un view controller visible desde el cual presentar, o la pantalla no apareció. | Llama a `start` con la app en primer plano, desde un controller que esté en una ventana. |
| `no_activity` | Flutter en Android: el plugin no está unido a una actividad. | Llama a `start` con la app en primer plano. |
| `host_activity_lost` | Android, Flutter en Android: no había una actividad viva para mostrar el siguiente paso. | Reintenta con la app en primer plano. |
| `unsupported_flow` | iOS: el modo de cámara pedido no está en este build. | Contacta a Tekbees. |
| `not_initialized`, `unicus_initialize_failed` | El motor biométrico no pudo iniciar. | Revisa la cámara, la fecha y hora del dispositivo y la red; contacta a soporte con el mensaje. |
| `plugin_unavailable` | Flutter: el plugin nativo no está registrado (plataforma no soportada, o una prueba sin mock). | Ejecuta en Android o iOS. |

### Pasos de UI propios (Custom)

| `code` | Qué pasó | Qué hacer |
| --- | --- | --- |
| `custom_provider_failed` | Android: el `present` de tu proveedor lanzó una excepción. | Corrige el proveedor (consulta la causa). |
| `step_failed` | iOS: falló una llamada de paso de tu pantalla Custom (red, sesión). | Muestra un reintento en tu pantalla. |
| `step_inactive` | iOS: una llamada al contexto después de que el paso terminó. | No reutilices contextos de pasos anteriores. |
| `sign_status_rejected` | Android: Unicus rechazó la consulta del estado de la firma (por ejemplo `STEP_OUT_OF_ORDER`). | Termina el paso con `fail`. |
| `unsupported_operation` | Android: un método de firma llamado sobre un contexto de paso que no creó el SDK. | Usa el contexto que te entrega el SDK. |

### Internos

| `code` | Qué pasó | Qué hacer |
| --- | --- | --- |
| `unicus_start_failed`, `unicus_prepare_failed`, `unicus_countries_failed`, `unicus_error` | Error inesperado en esa operación; trae adjunta la causa original. | Contacta a soporte con el mensaje y los logs del API. |
| `cancelled` | Interno: un paso se detuvo por una cancelación. Tu `start` recibe en su lugar el resultado `2041` o `2003`. | Ninguna. |
| `transaction_status_failed` | iOS, interno: no se entrega a tu app. | Ninguna. |

## Problemas comunes

| Síntoma | Revisa |
| --- | --- |
| `environment_not_available` al iniciar | Hoy solo está disponible `DEV`. Usa el mismo ambiente de tu Customer Token. |
| `transaction_refused` con `2002` | Asigna un flujo a tu empresa en el portal y publícalo para móvil. |
| `flow_not_supported` | Abre el flujo en el portal: un `face_match` sin `liveness` anterior, un `document` solo en un grupo de cámara, un flujo publicado solo para web, o pasos de UI con `uiStepMode` Disabled (o que faltan en tu proveedor Custom). |
| La transacción no se crea con un token que funciona en la web | El Customer Token pertenece a otro ambiente. Los tokens son distintos por ambiente. |
| La cámara nunca abre | La app debe correr en un dispositivo físico. iOS: `NSCameraUsageDescription` debe estar en el `Info.plist`. Resultado `9996`: el usuario negó la cámara; pídele que la permita en la configuración del sistema. |
| Las pantallas del flujo quedan en blanco o fallan con `webview_load_failed` | Proxy corporativo, MDM, VPN por app o firewall que bloquea el dominio de la flow app. Consulta [Compatibilidad y seguridad](compatibility-and-security.md). |
| `webview_unavailable` en algunos Android | Android System WebView antiguo o deshabilitado (flotas gestionadas, kioscos). Actualízalo, o usa el modo Custom. |
| La verificación termina con `2003` | El usuario salió de una pantalla: no es un error. Ofrece continuar; el siguiente `start` con el mismo documento la retoma. |
| Cada `start` crea una transacción nueva | `resumeOpenTransactions` está desactivado, cambió el documento, el flujo se cambió en el portal, pasaron más de 20 minutos, o Android 5.x. |
| Un build Debug de iOS en un iPhone físico no puede iniciar la verificación | Los builds Debug en un iPhone físico necesitan el cambio al framework de desarrollo. Consulta [Instalación en iOS](ios/installation.md). |
| Nunca se envía la ubicación | Es opcional. Android: declara los permisos de ubicación en tu app. iOS: agrega `NSLocationWhenInUseUsageDescription`. El usuario puede negarla; la verificación continúa. |
| El logo no aparece en Android | Android muestra el logo desde un drawable de tu app (`setAndroidLogoResourceName`); los logos remotos no se muestran. Consulta [Configuración en Android](android/configuration.md). |

## Logs del API

Para diagnosticar, activa `enableApiLogging` y registra un listener de logs del
API. Los logs se sanitizan: se ocultan los datos biométricos cifrados, los datos
del documento, los tokens de sesión y los datos personales de los pasos del
flujo. **No** actives `includeSensitiveApiLogData` en producción, y nunca envíes
a nadie logs capturados con esa opción activa.

## Qué enviar a soporte

Consulta [Soporte](support.md).
