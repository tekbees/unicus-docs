---
description: >-
  Integra la verificación de identidad de Unicus en aplicaciones nativas Flutter
  para Android e iOS usando solo el SDK de Unicus.
---

# INTEGRACIÓN CON FLUTTER

El SDK de Unicus para Flutter permite que tu aplicación Flutter ejecute la
verificación de identidad nativa en Android e iOS. Tu app integra solo
**Unicus**. El SDK crea la transacción de Unicus, obtiene la configuración de la
sesión, aplica la marca de la empresa, abre las pantallas nativas de
verificación, procesa los datos biométricos cifrados a través de Unicus y
devuelve el resultado final.

{% hint style="info" %}
El SDK de Unicus para Flutter incluye todo lo necesario para la verificación de
identidad: captura de cámara, prueba de vida, escaneo de documentos, cifrado y
las llamadas a la API de Unicus. No agregues otros paquetes biométricos o de
captura de documentos para este flujo y no edites el código nativo dentro del
SDK.
{% endhint %}

## Lo que Unicus te entregará

Antes de iniciar la integración, solicita los siguientes valores a tu
administrador de Unicus o al equipo de soporte de Tekbees.

| Valor | Descripción | Ejemplo |
| --- | --- | --- |
| `baseUrl` | URL del ambiente de la API de Unicus. | `https://alpha.idunicus.com:8080` |
| `apiKey` | Customer Token generado para tu empresa. | `<UNICUS_CUSTOMER_TOKEN>` |

Las llaves internas de Unicus (el device id de sesión de Tekbees y la llave del
motor de verificación de Unicus) vienen incluidas dentro del SDK de Unicus para
Flutter. La app del cliente no debe solicitar, almacenar ni enviar esas llaves
internas.

La app del cliente usa `apiKey` solo para su Customer Token de Unicus. La
creación de la transacción usa ese customer token a través de `X-Customer-ID`.
La consulta de países activos usa el mismo customer token a través de
`X-Device-ID`, porque ese endpoint lo usa para resolver los países habilitados
del cliente. Las llamadas de sesión como `/get-restart-session` y
`/sdk-execution-keys` usan el device id de sesión de Tekbees incluido en el SDK a
través de `X-Device-ID`.

{% hint style="warning" %}
Usa los valores del ambiente correcto. Las credenciales del ambiente de pruebas
(sandbox), staging y producción son diferentes. No subas credenciales de
producción a repositorios públicos.
{% endhint %}

## Requisitos

| Plataforma | Requisito |
| --- | --- |
| Flutter | `3.19.0` o superior |
| Dart | `3.3.0` o superior |
| Android | `minSdkVersion 21` o superior |
| iOS | iOS `15.0` o superior |
| Dispositivos | Dispositivo físico Android o iOS con cámara para la validación completa |
| Permisos | La cámara es obligatoria. La ubicación es opcional y el SDK continúa si el usuario la rechaza. |

## 1. Revisa el paquete para clientes

Tekbees entrega un paquete para clientes con un nombre como este:

{% code overflow="wrap" %}
```text
unicus_sdk_flutter_0.1.0_customer_package.zip
```
{% endcode %}

Al descomprimirlo, contiene:

{% code overflow="wrap" %}
```text
unicus_sdk_flutter_0.1.0_customer_package/
  README.md
  sdk/
    unicus_sdk_flutter/
  example/
    TECHNICAL_INTEGRATION.md
    lib/
      main.dart
      sample_unicus_texts.dart
```
{% endcode %}

| Ruta | Propósito |
| --- | --- |
| `README.md` | Inicio rápido del paquete. |
| `sdk/unicus_sdk_flutter/` | Wrapper público de Flutter que integra la app del cliente. |
| `example/` | App Flutter ejecutable, ya configurada para usar el SDK incluido. |
| `example/TECHNICAL_INTEGRATION.md` | Referencia técnica para el equipo de desarrollo del cliente. |
| `example/lib/sample_unicus_texts.dart` | Mapas de idioma/texto editables que usan las llaves públicas `Unicus_`. |

El paquete contiene el wrapper público de Flutter, el núcleo nativo cerrado de
Android, el núcleo nativo cerrado de iOS con todos sus componentes internos,
recursos y un ejemplo ejecutable. Las aplicaciones del cliente integran solo
Unicus.

## 2. Ejecuta el ejemplo incluido

Antes de modificar tu propia app, ejecuta el ejemplo incluido para confirmar el
ambiente, el dispositivo, los permisos y el customer token.

{% code overflow="wrap" %}
```bash
cd unicus_sdk_flutter_0.1.0_customer_package/example
flutter pub get
flutter devices
flutter run -d <DEVICE_ID> \
  --dart-define=UNICUS_BASE_URL=<UNICUS_BASE_URL> \
  --dart-define=UNICUS_API_KEY=<UNICUS_CUSTOMER_TOKEN>
```
{% endcode %}

La pantalla del ejemplo solo pide:

1. Número de documento.
2. Tipo de documento: `ID`, `FD`, `PP` o `DL`.

El ejemplo también puede mostrar la solicitud de permiso de ubicación de la
plataforma. Si el usuario no comparte su ubicación, la transacción continúa sin
datos de ubicación. Si tu cuenta de Unicus tiene más de un país activo, el
ejemplo muestra un selector de país después de crear el id de la transacción. Si
solo hay un país activo, el SDK lo selecciona automáticamente y continúa.

El SDK crea la transacción, lee la configuración de sesión del cliente, aplica el
tema y los textos, abre la experiencia nativa de verificación, procesa las
solicitudes cifradas y devuelve el resultado.

Para una verificación de solo compilación:

{% code overflow="wrap" %}
```bash
flutter build apk --debug
flutter build ios --debug --simulator
```
{% endcode %}

Usa un dispositivo físico Android o iOS con cámara para la verificación completa
de punta a punta. Para dispositivos físicos iOS, usa un build firmado.

## 3. Agrega la dependencia

Para tu propia app, copia `sdk/unicus_sdk_flutter` del paquete para clientes al
repositorio de tu aplicación, por ejemplo:

{% code overflow="wrap" %}
```text
your_flutter_app/
  vendor/
    unicus_sdk_flutter/
```
{% endcode %}

Luego agrega la dependencia local al `pubspec.yaml` de tu aplicación:

{% code overflow="wrap" %}
```yaml
dependencies:
  flutter:
    sdk: flutter

  unicus_sdk_flutter:
    path: vendor/unicus_sdk_flutter
```
{% endcode %}

Luego instala la dependencia:

{% code overflow="wrap" %}
```bash
flutter pub get
```
{% endcode %}

El repositorio Git privado es solo para el desarrollo interno de Tekbees o para
pilotos privados basados en código fuente aprobados explícitamente por Tekbees:

{% code overflow="wrap" %}
```yaml
dependencies:
  unicus_sdk_flutter:
    git:
      url: git@bitbucket.org:tekbees/unicus_sdk_flutter.git
      ref: v0.1.0
```
{% endcode %}

No agregues dependencias nativas adicionales para este flujo y no solicites las
llaves internas de Tekbees.

## 4. Configura Android

Abre `android/app/src/main/AndroidManifest.xml` y agrega los permisos requeridos
antes de la etiqueta `<application>`.

{% code overflow="wrap" %}
```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <uses-permission android:name="android.permission.CAMERA" />
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />

    <application>
        ...
    </application>
</manifest>
```
{% endcode %}

Confirma que tu app usa Android embedding v2. La mayoría de las apps Flutter
actuales ya incluyen este metadata dentro de `<application>`:

{% code overflow="wrap" %}
```xml
<meta-data
    android:name="flutterEmbedding"
    android:value="2" />
```
{% endcode %}

No se requiere ninguna dependencia nativa adicional en Android.

## 5. Configura iOS

Abre `ios/Runner/Info.plist` y agrega las descripciones de uso de cámara y
ubicación.

{% code overflow="wrap" %}
```xml
<key>NSCameraUsageDescription</key>
<string>Camera access is required to verify your identity.</string>
<key>NSLocationWhenInUseUsageDescription</key>
<string>Location access helps complete the identity verification context.</string>
```
{% endcode %}

Si el permiso de ubicación se rechaza, está deshabilitado o no está configurado,
el SDK de Unicus continúa la transacción sin datos de ubicación.

El SDK de Unicus para Flutter usa CocoaPods para la integración en iOS. Si tu
proyecto Flutter tiene Swift Package Manager habilitado de forma global,
deshabilítalo en `pubspec.yaml`.

{% code overflow="wrap" %}
```yaml
flutter:
  config:
    enable-swift-package-manager: false
```
{% endcode %}

Luego instala los pods:

{% code overflow="wrap" %}
```bash
cd ios
pod install
cd ..
```
{% endcode %}

No se requieren cambios en `AppDelegate`, `SceneDelegate` ni en ningún archivo
nativo de iOS.

## 6. Configura el SDK

Crea una instancia de `UnicusSdkFlutter` en la parte de tu app que maneja el
flujo de verificación.

{% code overflow="wrap" %}
```dart
import 'package:unicus_sdk_flutter/unicus_sdk_flutter.dart';

final UnicusSdkFlutter unicus = UnicusSdkFlutter();

Future<void> configureUnicus() async {
  await unicus.configure(
    const UnicusSdkConfig(
      baseUrl: '<UNICUS_BASE_URL>',
      apiKey: '<UNICUS_CUSTOMER_TOKEN>',
      collectLocationOnStart: true,
    ),
  );
}
```
{% endcode %}

Llama a `configureUnicus()` antes de iniciar la primera verificación. Un lugar
común es cuando el usuario llega a la pantalla donde puede comenzar la
verificación de identidad. `collectLocationOnStart` es opcional y su valor por
defecto es `true`. Ponlo en `false` solo cuando tu app no deba solicitar la
ubicación para este flujo.

## 7. Inicia una verificación

Envía el tipo de documento y el número de documento del usuario a `start`.

{% code overflow="wrap" %}
```dart
Future<void> startUnicusVerification() async {
  final UnicusVerificationResult result = await unicus.start(
    const UnicusVerificationRequest.enrollmentVerify(
      document: UnicusDocument(
        type: UnicusDocumentType.id,
        externalDatabaseRefId: '123456789',
      ),
    ),
  );

  if (result.success) {
    // La verificación de identidad fue exitosa.
  } else {
    // Muestra una ruta de reintento, rechazo o soporte según tu flujo de negocio.
  }
}
```
{% endcode %}

Cuando se llama a `start`, el SDK crea internamente la transacción usando el
proceso móvil estándar de Unicus:

{% code overflow="wrap" %}
```json
{
  "documentType": "ID",
  "externalDatabaseRefID": "123456789",
  "process": "ENROLLMENT-VERIFY"
}
```
{% endcode %}

Tu app no debe llamar manualmente a `/start-mobile-transaction` ni a
`/get-restart-session` en la integración estándar con Flutter. Tampoco debe
pedirle al usuario ni al desarrollador de la aplicación que seleccione un valor
de proceso.

La secuencia estándar es:

1. Tu app llama a `unicus.start(...)`.
2. El SDK llama a `/start-mobile-transaction` y recibe un nuevo id de
   transacción `tid`.
3. El SDK solicita la ubicación a la plataforma cuando está habilitado. Si el
   usuario rechaza el permiso o el dispositivo no puede entregar la ubicación,
   el SDK continúa.
4. El SDK llama a `/company-countries`, lee los países activos del cliente y
   selecciona automáticamente el único país activo cuando aplica.
5. El SDK llama a `/get-restart-session` usando el `tid` y el país seleccionado,
   cuando hay uno disponible.
6. El SDK aplica los colores, el logo y los textos de verificación de Unicus de
   la empresa.
7. El SDK abre la pantalla nativa de verificación.
8. El SDK envía los datos de verificación cifrados a Unicus.
9. Tu app recibe un `UnicusVerificationResult`.

## Selección de país

En la mayoría de las integraciones basta con llamar a `start(...)`. Si tu cuenta
de Unicus tiene un solo país activo, el SDK lo selecciona automáticamente. Si la
cuenta tiene varios países activos y tu app debe permitir que el usuario elija,
usa primero `prepareEnrollmentVerify(...)`.

{% code overflow="wrap" %}
```dart
Future<void> startWithCountrySelection() async {
  final prepared = await unicus.prepareEnrollmentVerify(
    document: const UnicusDocument(
      type: UnicusDocumentType.id,
      externalDatabaseRefId: '123456789',
    ),
  );

  final String? selectedCountry = prepared.requiresCountrySelection
      ? await showYourCountryPicker(prepared.countries)
      : prepared.defaultCountry?.code;

  final result = await unicus.start(
    prepared.toRequest(country: selectedCountry),
  );

  // Maneja aquí el UnicusVerificationResult.
}
```
{% endcode %}

`prepareEnrollmentVerify(...)` crea el id de la transacción de Unicus,
opcionalmente obtiene la ubicación y devuelve los países activos. No abre la
pantalla nativa de verificación. Llama a `start(...)` con
`prepared.toRequest(...)` después de que tu app haya seleccionado el país. El
valor del país debe ser un código ISO 3166-1 alpha-2, como `CO`, `US` o `MX`.

## Tipos de documento

Usa el enum que provee el SDK.

| Valor en Dart | Valor en la API | Descripción |
| --- | --- | --- |
| `UnicusDocumentType.id` | `ID` | Documento de identidad nacional |
| `UnicusDocumentType.foreignDocument` | `FD` | Documento extranjero |
| `UnicusDocumentType.passport` | `PP` | Pasaporte |
| `UnicusDocumentType.driverLicense` | `DL` | Licencia de conducción |

## Comportamiento del flujo nativo

El SDK maneja internamente el flujo nativo por defecto. La aplicación del cliente
no necesita ningún campo de flujo.

Unicus revisa el estado actual de la sesión:

| Estado de la sesión | Comportamiento |
| --- | --- |
| El usuario ya está enrolado | El SDK inicia la autenticación facial. |
| El usuario no está enrolado | El SDK inicia el enrolamiento facial y del documento. |

En la integración estándar con Flutter, la aplicación solo debe entregar el tipo
de documento y el número de documento.

## Lee el resultado

`start` devuelve un `UnicusVerificationResult`.

| Campo | Descripción |
| --- | --- |
| `success` | `true` cuando la verificación terminó exitosamente. |
| `outcome` | Categoría normalizada del resultado: success, warning, failed, canceled, error o unknown. |
| `tid` | Id de la transacción de Unicus creada por el SDK. |
| `resultCode` | Código de resultado de la transacción de Unicus, cuando está disponible. Si Unicus ya finalizó la transacción, este valor tiene prioridad sobre el estado de la pantalla nativa. |
| `resultMessage` | Mensaje de resultado legible, cuando está disponible. |
| `status` | Estado de la pantalla nativa para diagnóstico técnico. |
| `sessionError` | `true` solo ante una interrupción técnica, una cancelación o un error nativo/de sesión. Los resultados de negocio deben manejarse con `outcome` y `resultCode`. |

Manejo recomendado del resultado:

{% code overflow="wrap" %}
```dart
switch (result.outcome) {
  case UnicusVerificationOutcome.success:
    // Continúa con el usuario verificado.
    break;
  case UnicusVerificationOutcome.warning:
    // Continúa o envía a revisión manual según tus reglas de negocio.
    break;
  case UnicusVerificationOutcome.canceled:
    // Permite que el usuario lo intente de nuevo.
    break;
  case UnicusVerificationOutcome.failed:
  case UnicusVerificationOutcome.error:
    // Muestra el flujo de falla configurado.
    break;
  case UnicusVerificationOutcome.unknown:
    // Muestra una ruta de soporte o de reintento.
    break;
}
```
{% endcode %}

## Escucha los eventos de progreso

Puedes suscribirte a los eventos del SDK antes de iniciar la verificación.

{% code overflow="wrap" %}
```dart
final subscription = unicus.events.listen((event) {
  debugPrint('Unicus event: ${event.name}');
  debugPrint('Transaction id: ${event.tid}');
  debugPrint('Message: ${event.message}');
});
```
{% endcode %}

Cancela la suscripción cuando se descarte la pantalla:

{% code overflow="wrap" %}
```dart
await subscription.cancel();
```
{% endcode %}

## Personaliza los textos de verificación

El SDK incluye textos por defecto en inglés basados en la configuración actual
del SDK web de Unicus. Si tu aplicación necesita otro idioma u otra redacción,
entrega textos personalizados al configurar Unicus.

La app de ejemplo incluye mapas de textos de Unicus completos en inglés y
español, basados en la configuración de idioma actual del SDK web, en
`example/lib/sample_unicus_texts.dart`. Usa ese archivo como punto de partida
para tu propio archivo de idioma. La pantalla de inicio del ejemplo pide a
propósito solo el número y el tipo de documento; los textos se cambian en el
código, no desde la interfaz de la demo.

{% code overflow="wrap" %}
```dart
await unicus.configure(
  const UnicusSdkConfig(
    baseUrl: '<UNICUS_BASE_URL>',
    apiKey: '<UNICUS_CUSTOMER_TOKEN>',
    verificationTextOverrides: sampleUnicusTextOverrides,
  ),
);
```
{% endcode %}

Unicus combina tus textos personalizados con los textos por defecto y envía el
mapa de textos final a las pantallas nativas de verificación de Android e iOS
antes de abrir la sesión. El SDK adapta las llaves de texto nativas internas, y
los saltos de línea `<br/>` se convierten en saltos de línea nativos.

Llaves de texto comunes:

| Llave en Dart | Texto en pantalla |
| --- | --- |
| `UnicusVerificationTextKey.actionImReady` | Botón de listo. |
| `UnicusVerificationTextKey.actionContinue` | Botón de continuar. |
| `UnicusVerificationTextKey.actionTryAgain` | Botón de reintentar. |
| `UnicusVerificationTextKey.feedbackCenterFace` | Indicación de alineación del rostro. |
| `UnicusVerificationTextKey.initializingCamera` | Mensaje de inicialización de la cámara. |
| `UnicusVerificationTextKey.idScanTypeSelectionHeader` | Título del escaneo del documento. |
| `UnicusVerificationTextKey.resultFaceScanUploadMessage` | Mensaje de carga del rostro. |

Para las etiquetas avanzadas de confirmación del OCR, coordina el diccionario
`verificationOcrLocalization` con el soporte de Tekbees.

## Logs de API opcionales para pruebas

Durante las pruebas en el ambiente de pruebas (sandbox), los logs de API pueden
ayudar a tu equipo a confirmar las respuestas recibidas de Unicus.

{% hint style="warning" %}
Los logs de API son solo para desarrollo y QA. No habilites logs sensibles en
builds de producción.
{% endhint %}

{% code overflow="wrap" %}
```dart
await unicus.configure(
  const UnicusSdkConfig(
    baseUrl: '<UNICUS_BASE_URL>',
    apiKey: '<UNICUS_CUSTOMER_TOKEN>',
    enableApiLogging: true,
  ),
);

unicus.apiLogs.listen((entry) {
  debugPrint(entry.toPrettyJson());
});
```
{% endcode %}

Por defecto, los logs se sanitizan. Los blobs biométricos cifrados, los payloads
de OCR, los datos del documento y los tokens de sesión se ocultan o se truncan.

## Ejemplo completo de botón

{% code overflow="wrap" expandable="true" %}
```dart
import 'package:flutter/material.dart';
import 'package:unicus_sdk_flutter/unicus_sdk_flutter.dart';

class UnicusVerificationButton extends StatefulWidget {
  const UnicusVerificationButton({super.key});

  @override
  State<UnicusVerificationButton> createState() =>
      _UnicusVerificationButtonState();
}

class _UnicusVerificationButtonState extends State<UnicusVerificationButton> {
  final UnicusSdkFlutter unicus = UnicusSdkFlutter();
  bool running = false;

  @override
  void initState() {
    super.initState();
    configureSdk();
  }

  Future<void> configureSdk() async {
    await unicus.configure(
      const UnicusSdkConfig(
        baseUrl: '<UNICUS_BASE_URL>',
        apiKey: '<UNICUS_CUSTOMER_TOKEN>',
      ),
    );
  }

  Future<void> startVerification() async {
    setState(() => running = true);

    try {
      final result = await unicus.start(
        const UnicusVerificationRequest.enrollmentVerify(
          document: UnicusDocument(
            type: UnicusDocumentType.id,
            externalDatabaseRefId: '123456789',
          ),
        ),
      );

      if (!mounted) {
        return;
      }

      final message = result.success
          ? 'Identity verified successfully'
          : result.resultMessage ?? 'Identity could not be verified';

      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text(message)),
      );
    } finally {
      if (mounted) {
        setState(() => running = false);
      }
    }
  }

  @override
  Widget build(BuildContext context) {
    return FilledButton(
      onPressed: running ? null : startVerification,
      child: Text(running ? 'Verifying...' : 'Verify identity'),
    );
  }
}
```
{% endcode %}

## Apariencia

El SDK aplica automáticamente la apariencia de la empresa configurada en Unicus y
devuelta por el endpoint de sesión.

| Campo | Comportamiento |
| --- | --- |
| `backgroundColor` | Color de fondo de la pantalla nativa de verificación. |
| `windowColor` | Color principal de la verificación nativa. |
| `buttonColor` | Color de acento de botones, progreso, marco y OCR. |
| `textColor` | Color del texto de botones e indicaciones. |
| `logo` | Logo de la empresa. |

iOS puede usar la URL remota del logo que devuelve Unicus cuando la URL apunta a
una imagen rasterizada como PNG o JPG. Android requiere que el logo sea un
recurso drawable nativo, por lo que Android aplica los colores automáticamente.
Si tu integración en Android requiere un logo dentro de la pantalla nativa de
verificación, coordina el nombre del recurso drawable con el soporte de Tekbees.

## Lista de verificación de pruebas

Usa un dispositivo físico para la validación final.

1. Ejecuta `flutter pub get`.
2. Ejecuta la app en Android y acepta el permiso de cámara.
3. Ejecuta la app en un iPhone físico firmado y acepta el permiso de cámara.
4. Prueba un documento válido con el tipo de documento `ID`, `FD`, `PP` o `DL`.
5. Prueba el permiso de ubicación aceptado y rechazado. Ambos caminos deben
   continuar.
6. Si la cuenta tiene un país activo, confirma que el flujo continúa sin mostrar
   un selector de país.
7. Si la cuenta tiene varios países activos, confirma que tu app muestra un
   selector de país antes de abrir la pantalla nativa de verificación.
8. Confirma que se abre la pantalla nativa de verificación.
9. Confirma que los colores y el logo de la empresa se muestran como se espera.
10. Confirma que tu app recibe un `UnicusVerificationResult`.
11. Confirma que la transacción aparece en el portal administrativo de Unicus.

Comandos útiles:

{% code overflow="wrap" %}
```bash
flutter devices
flutter run -d <DEVICE_ID> \
  --dart-define=UNICUS_BASE_URL=<UNICUS_BASE_URL> \
  --dart-define=UNICUS_API_KEY=<UNICUS_CUSTOMER_TOKEN>
```
{% endcode %}

Para dispositivos físicos iOS, no uses `--no-codesign`. Usa un build firmado
desde Flutter o Xcode.

## Solución de problemas

| Problema | Qué revisar |
| --- | --- |
| El SDK dice que no está configurado | Confirma que `unicus.configure(...)` se ejecuta antes de `unicus.start(...)`. |
| Se rechazó el permiso de cámara | Confirma el permiso `CAMERA` en Android o `NSCameraUsageDescription` en iOS. |
| Se rechazó el permiso de ubicación | Esto no detiene la transacción. El SDK continúa sin datos de ubicación. |
| No aparece el selector de país | Confirma que la cuenta del cliente tiene más de un país activo en Unicus. Con un solo país activo, el SDK lo selecciona automáticamente. |
| No se devuelve un id de transacción | Confirma `baseUrl`, `apiKey`, el tipo de documento y el número de documento. |
| El build de iOS falla después de agregar el paquete | Confirma que CocoaPods está instalado y que Swift Package Manager está deshabilitado para esta app. |
| La verificación nativa no se abre | Confirma que estás ejecutando en un dispositivo físico compatible con acceso a la cámara. |
| No aparecen los colores de la empresa | Confirma que `/get-restart-session` devuelve `windowColor`, `buttonColor` y `textColor`. |
| No aparece el logo en iOS | Confirma que `/get-restart-session` devuelve `logo` como una URL HTTPS de PNG o JPG. Las URL de SVG no son válidas para el logo móvil nativo. |
| No aparece el logo en Android | Confirma si se coordinó el nombre de un recurso drawable con el soporte de Tekbees. |

## Soporte

Al contactar a soporte, incluye:

1. URL del ambiente.
2. Plataforma y versión de la app.
3. Modelo del dispositivo y versión del sistema operativo.
4. Id de la transacción de Unicus `tid`, si se creó.
5. Código de resultado y mensaje de resultado, si están disponibles.
6. Una breve descripción del paso en el que ocurrió el problema.
