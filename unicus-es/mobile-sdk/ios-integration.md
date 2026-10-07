---
description: >-
  Integra la verificación de identidad de Unicus en aplicaciones nativas de iOS
  (Swift) usando únicamente el SDK de Unicus.
---

# INTEGRACIÓN EN IOS

El SDK de Unicus para iOS permite que tu aplicación nativa de iOS ejecute la
verificación de identidad. Tu app integra únicamente **Unicus**. El SDK crea la
transacción de Unicus, obtiene la configuración de la sesión, aplica la marca de
la empresa, presenta las pantallas nativas de verificación, procesa los datos
biométricos cifrados a través de Unicus y devuelve el resultado final.

{% hint style="info" %}
El SDK de Unicus para iOS incluye todo lo necesario para la verificación de
identidad: captura con cámara, prueba de vida, escaneo de documentos, cifrado y
las llamadas a la API de Unicus. No agregues otras librerías biométricas o de
captura de documentos para este flujo. Tu app solo importa `UnicusSDK`.
{% endhint %}

## Lo que Unicus te entregará

Antes de iniciar la integración, solicita los siguientes valores a tu
administrador de Unicus o al equipo de soporte de Tekbees.

| Valor | Descripción | Ejemplo |
| --- | --- | --- |
| `baseUrl` | URL del ambiente de la API de Unicus. | `https://alpha.idunicus.com:8080` |
| `apiKey` | Customer Token generado para tu empresa. | `<UNICUS_CUSTOMER_TOKEN>` |

Las llaves internas de Unicus (el device id de sesión de Tekbees y la llave del
motor de verificación de Unicus) vienen embebidas dentro del SDK de Unicus para
iOS. La app del cliente no debe solicitar, almacenar ni enviar esas llaves
internas.

La app del cliente usa `apiKey` solo para su Customer Token de Unicus. La
creación de la transacción usa ese Customer Token a través de `X-Customer-ID`.
La consulta de países activos usa el mismo Customer Token a través de
`X-Device-ID`, porque ese endpoint lo usa para resolver los países habilitados
del cliente. Las llamadas de sesión como `/get-restart-session` y
`/sdk-execution-keys` usan el device id de sesión de Tekbees embebido en el SDK
a través de `X-Device-ID`.

{% hint style="warning" %}
Usa los valores del ambiente correcto. Las credenciales del ambiente de pruebas
(sandbox), staging y producción son diferentes. No subas credenciales de
producción a repositorios públicos.
{% endhint %}

## Requisitos

| Plataforma | Requisito |
| --- | --- |
| iOS | iOS `15.0` o superior |
| Xcode | Xcode `16` o superior |
| Lenguaje | Swift `5.9` o superior (se admiten tanto completion handlers como `async/await`) |
| Dispositivos | iPhone físico con cámara para la validación completa |
| Permisos | La cámara es obligatoria. La ubicación es opcional y el SDK continúa si el usuario la rechaza. |

## 1. Revisa el paquete para clientes

Tekbees entrega un paquete para clientes con un nombre como este:

{% code overflow="wrap" %}
```text
unicus_sdk_ios_0.1.0_customer_package.zip
```
{% endcode %}

Al descomprimirlo, contiene:

{% code overflow="wrap" %}
```text
unicus_sdk_ios_0.1.0_customer_package/
  README.md
  TECHNICAL_INTEGRATION.md
  sdk/
    Frameworks/
      UnicusSDK.xcframework
      <Unicus verification engine>.xcframework
      <Unicus verification engine>ForDevelopment.xcframework
    UnicusSDK.podspec
    Package.swift
  example/
    UnicusSDKExample.xcodeproj
    UnicusSDKExample/
      Config/Unicus.xcconfig
      ContentView.swift
      ExampleViewModel.swift
      SampleUnicusTexts.swift
```
{% endcode %}

| Ruta | Propósito |
| --- | --- |
| `README.md` | Guía rápida del paquete. |
| `TECHNICAL_INTEGRATION.md` | Referencia técnica para el equipo de desarrollo del cliente. |
| `sdk/Frameworks/UnicusSDK.xcframework` | Módulo público del SDK de Unicus (slices para dispositivo y simulador). |
| `sdk/Frameworks/*.xcframework` | Binarios del motor de verificación de Unicus que requiere el SDK. El framework cuyo nombre termina en `ForDevelopment` se usa solo en builds Debug en dispositivos físicos. |
| `sdk/UnicusSDK.podspec` | Archivo de integración con CocoaPods. |
| `sdk/Package.swift` | Archivo de integración con Swift Package Manager. |
| `example/` | Proyecto de Xcode ejecutable (SwiftUI) ya configurado para usar los binarios incluidos. |
| `example/UnicusSDKExample/SampleUnicusTexts.swift` | Mapas de idioma/textos editables que usan las llaves públicas `Unicus_`. |

El paquete contiene el SDK cerrado de Unicus con todos sus componentes
internos, recursos y un ejemplo ejecutable. Las aplicaciones de los clientes
integran únicamente Unicus.

## 2. Ejecuta el ejemplo incluido

Antes de modificar tu propia app, ejecuta el ejemplo incluido para confirmar el
ambiente, el dispositivo, los permisos y el Customer Token.

1. Abre `example/UnicusSDKExample.xcodeproj` en Xcode.
2. Edita `example/UnicusSDKExample/Config/Unicus.xcconfig` con los valores
   entregados por Tekbees:

{% code overflow="wrap" %}
```properties
UNICUS_BASE_URL = https:/$()/alpha.idunicus.com:8080
UNICUS_API_KEY = <UNICUS_CUSTOMER_TOKEN>
```
{% endcode %}

{% hint style="info" %}
En los archivos `xcconfig`, `//` inicia un comentario. Escribe la URL como
`https:/$()/host` para que se conserve la doble barra.
{% endhint %}

3. Selecciona tu equipo de desarrollo en *Signing & Capabilities*.
4. Ejecuta el scheme `UnicusSDKExample` en un iPhone físico.

La pantalla del ejemplo solo pide:

1. Número de documento.
2. Tipo de documento: `ID`, `FD`, `PP` o `DL`.

El ejemplo también puede mostrar la solicitud de permiso de ubicación de iOS. Si
el usuario no comparte su ubicación, la transacción continúa sin datos de
ubicación. Si tu cuenta de Unicus tiene más de un país activo, el ejemplo
muestra un selector de país después de crear el id de la transacción. Si solo
hay un país activo, el SDK lo selecciona automáticamente y continúa.

El SDK crea la transacción, lee la configuración de sesión del cliente, aplica
el tema y los textos, presenta la experiencia nativa de verificación, procesa
las solicitudes cifradas y devuelve el resultado. El ejemplo también muestra las
respuestas sanitizadas de la API de Unicus recibidas durante el flujo.

Para una verificación de solo compilación en el simulador:

{% code overflow="wrap" %}
```bash
cd unicus_sdk_ios_0.1.0_customer_package/example
xcodebuild -project UnicusSDKExample.xcodeproj -scheme UnicusSDKExample \
  -sdk iphonesimulator -destination 'generic/platform=iOS Simulator' \
  CODE_SIGNING_ALLOWED=NO build
```
{% endcode %}

Usa un iPhone físico con cámara para la verificación completa de punta a punta.
El simulador no puede ejecutar las pantallas nativas de verificación.

## 3. Agrega la dependencia

Para tu propia app, copia la carpeta `sdk` del paquete para clientes en el
repositorio de tu aplicación, por ejemplo:

{% code overflow="wrap" %}
```text
your_ios_app/
  Vendor/
    Unicus/
      Frameworks/
      UnicusSDK.podspec
      Package.swift
```
{% endcode %}

Elige un método de integración.

### Opción A: Xcode (frameworks manuales)

1. Arrastra cada `.xcframework` de `Vendor/Unicus/Frameworks` al target de tu
   app en *General > Frameworks, Libraries, and Embedded Content*.
2. Configura cada uno como **Embed & Sign**.

### Opción B: CocoaPods

{% code overflow="wrap" %}
```ruby
platform :ios, '15.0'

target 'YourApp' do
  use_frameworks!
  pod 'UnicusSDK', :path => 'Vendor/Unicus'
end
```
{% endcode %}

Luego instala los pods:

{% code overflow="wrap" %}
```bash
pod install
```
{% endcode %}

### Opción C: Swift Package Manager

En Xcode selecciona *File > Add Package Dependencies... > Add Local...*, elige
la carpeta `Vendor/Unicus` y vincula el producto `UnicusSDK` al target de tu
app.

Todos los frameworks de `sdk/Frameworks` forman parte del SDK de Unicus y deben
embeberse juntos. No solicites las llaves internas de Tekbees.

## 4. Configura iOS

Abre el `Info.plist` de tu app y agrega las descripciones de uso de cámara y
ubicación.

{% code overflow="wrap" %}
```xml
<key>NSCameraUsageDescription</key>
<string>Camera access is required to verify your identity.</string>
<key>NSLocationWhenInUseUsageDescription</key>
<string>Location access helps complete the identity verification context.</string>
```
{% endcode %}

Si el permiso de ubicación se rechaza, está deshabilitado o no está
configurado, el SDK de Unicus continúa la transacción sin datos de ubicación.

No se requieren cambios en `AppDelegate`, `SceneDelegate` ni en ningún otro
archivo nativo. El SDK presenta las pantallas de verificación sobre el view
controller que le pases o, si no pasas ninguno, sobre el view controller
superior.

### Builds Debug en dispositivos físicos

El motor de verificación de Unicus se entrega como un binario de producción y
un binario de desarrollo (el framework cuyo nombre termina en
`ForDevelopment`). Los builds Debug instalados en un dispositivo físico deben
usar el binario de desarrollo. El ejemplo incluido lo hace automáticamente con
una fase de build *Run Script*. Copia esa fase a tu propio target cuando
ejecutes builds Debug en dispositivos:

{% code overflow="wrap" expandable="true" %}
```bash
set -euo pipefail
if [[ "${CONFIGURATION}" != "Debug" || "${PLATFORM_NAME}" != "iphoneos" ]]; then
  exit 0
fi
if [[ "${CODE_SIGNING_ALLOWED:-YES}" == "NO" || -z "${EXPANDED_CODE_SIGN_IDENTITY:-}" ]]; then
  exit 0
fi
DEVELOPMENT_FRAMEWORK="${SRCROOT}/Vendor/Unicus/Frameworks/$(ls "${SRCROOT}/Vendor/Unicus/Frameworks" | grep ForDevelopment)"
EMBEDDED_FRAMEWORK=$(find "${TARGET_BUILD_DIR}/${FRAMEWORKS_FOLDER_PATH}" -maxdepth 1 -name "*.framework" -not -name "UnicusSDK.framework" | head -1)
if [[ -d "${DEVELOPMENT_FRAMEWORK}" && -d "${EMBEDDED_FRAMEWORK}" && -f "${EMBEDDED_FRAMEWORK}/swap-development-framework.sh" ]]; then
  sh "${EMBEDDED_FRAMEWORK}/swap-development-framework.sh" "${DEVELOPMENT_FRAMEWORK}"
fi
```
{% endcode %}

Agrega la fase después de *Embed Frameworks*. Los builds Release y de App Store
conservan el binario de producción.

## 5. Configura el SDK

Configura la instancia compartida `UnicusSdk` en la parte de tu app que maneja
el flujo de verificación.

{% code overflow="wrap" %}
```swift
import UnicusSDK

func configureUnicus() {
    UnicusSdk.shared.configure(
        UnicusSdkConfig(
            baseUrl: "<UNICUS_BASE_URL>",
            apiKey: "<UNICUS_CUSTOMER_TOKEN>",
            collectLocationOnStart: true
        )
    )
}
```
{% endcode %}

Llama a `configureUnicus()` antes de iniciar la primera verificación. Un lugar
habitual es cuando el usuario llega a la pantalla donde puede comenzar la
verificación de identidad. `collectLocationOnStart` es opcional y su valor por
defecto es `true`. Configúralo en `false` solo cuando tu app no deba solicitar
la ubicación para este flujo.

## 6. Inicia una verificación

Envía el tipo de documento y el número de documento del usuario a `start`. Pasa
el view controller que debe presentar la verificación (opcional). Los
completions se entregan en la main queue.

{% code overflow="wrap" %}
```swift
import UnicusSDK

func startUnicusVerification(from viewController: UIViewController) {
    UnicusSdk.shared.start(
        .enrollmentVerify(
            document: UnicusDocument(
                type: .id,
                externalDatabaseRefId: "123456789"
            )
        ),
        from: viewController
    ) { result in
        switch result {
        case .success(let verification):
            if verification.success {
                // La verificación de identidad fue exitosa.
            } else {
                // Muestra una ruta de reintento, rechazo o soporte según tu flujo de negocio.
            }
        case .failure(let error):
            // No se pudo iniciar la verificación (error de configuración, red o sesión).
            print(error.code, error.message)
        }
    }
}
```
{% endcode %}

Con `async/await`:

{% code overflow="wrap" %}
```swift
do {
    let result = try await UnicusSdk.shared.start(request, from: viewController)
    // Maneja aquí el UnicusVerificationResult.
} catch let error as UnicusSdkError {
    // error.code y error.message describen la falla.
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
`/get-restart-session` en la integración estándar de iOS. Tampoco debe pedirle
al usuario ni al desarrollador de la aplicación que seleccione un valor de
proceso.

La secuencia estándar es:

1. Tu app llama a `UnicusSdk.shared.start(...)`.
2. El SDK llama a `/start-mobile-transaction` y recibe un nuevo id de
   transacción `tid`.
3. El SDK solicita la ubicación a la plataforma cuando está habilitada. Si el
   usuario rechaza el permiso o el dispositivo no puede entregar la ubicación,
   el SDK continúa.
4. El SDK llama a `/company-countries`, lee los países activos del cliente y
   selecciona automáticamente el único país activo cuando aplica.
5. El SDK llama a `/get-restart-session` usando el `tid` y el país
   seleccionado, cuando hay uno disponible.
6. El SDK aplica los colores y el logo de la empresa, y los textos de
   verificación de Unicus.
7. El SDK presenta la pantalla nativa de verificación.
8. El SDK envía los datos de verificación cifrados a Unicus.
9. Tu app recibe un `UnicusVerificationResult`.

Solo puede ejecutarse una verificación a la vez. Llamar a `start` mientras otra
verificación está activa falla con el código de error `session_active`.

## Selección de país

Para la mayoría de las integraciones, basta con llamar a `start(...)`. Si tu
cuenta de Unicus tiene un solo país activo, el SDK lo selecciona
automáticamente. Si la cuenta tiene varios países activos y tu app debe
permitir que el usuario elija, usa primero `prepareEnrollmentVerify(...)`.

{% code overflow="wrap" %}
```swift
func startWithCountrySelection(from viewController: UIViewController) async throws {
    let prepared = try await UnicusSdk.shared.prepareEnrollmentVerify(
        document: UnicusDocument(type: .id, externalDatabaseRefId: "123456789")
    )

    let selectedCountry: String? = prepared.requiresCountrySelection
        ? await showYourCountryPicker(prepared.countries)
        : prepared.defaultCountry?.code

    let result = try await UnicusSdk.shared.start(
        prepared.toRequest(country: selectedCountry),
        from: viewController
    )

    // Maneja aquí el UnicusVerificationResult.
}
```
{% endcode %}

`prepareEnrollmentVerify(...)` crea el id de transacción de Unicus, recolecta
opcionalmente la ubicación y devuelve los países activos. No presenta la
pantalla nativa de verificación. Llama a `start(...)` con
`prepared.toRequest(...)` después de que tu app haya seleccionado el país. El
valor del país debe ser un código ISO 3166-1 alfa-2, como `CO`, `US` o `MX`.

## Tipos de documento

Usa el enum que provee el SDK.

| Valor en Swift | Valor en la API | Descripción |
| --- | --- | --- |
| `UnicusDocumentType.id` | `ID` | Documento nacional de identidad |
| `UnicusDocumentType.foreignDocument` | `FD` | Documento extranjero |
| `UnicusDocumentType.passport` | `PP` | Pasaporte |
| `UnicusDocumentType.driverLicense` | `DL` | Licencia de conducción |

## Comportamiento del flujo nativo

El SDK maneja internamente el flujo nativo por defecto. La aplicación del
cliente no requiere ningún campo de flujo.

Unicus revisa el estado actual de la sesión:

| Estado de la sesión | Comportamiento |
| --- | --- |
| El usuario ya está enrolado | El SDK inicia la autenticación facial. |
| El usuario no está enrolado | El SDK inicia el enrolamiento facial y de documento. |

En la integración estándar de iOS, la aplicación solo debe enviar el tipo de
documento y el número de documento.

## Lee el resultado

`start` entrega un `UnicusVerificationResult`.

| Campo | Descripción |
| --- | --- |
| `success` | `true` cuando la verificación se completó exitosamente. |
| `outcome` | Categoría normalizada del resultado: `.success`, `.warning`, `.failed`, `.canceled`, `.error` o `.unknown`. |
| `tid` | Id de transacción de Unicus creado por el SDK. |
| `resultCode` | Código de resultado de la transacción de Unicus, cuando está disponible. Si Unicus ya finalizó la transacción, este valor tiene prioridad sobre el estado de la pantalla nativa. |
| `resultMessage` | Mensaje de resultado legible, cuando está disponible. |
| `status` | Estado de la pantalla nativa para diagnóstico técnico. |
| `sessionError` | `true` solo ante una interrupción técnica, una cancelación o un error nativo/de sesión. Los resultados de negocio deben manejarse con `outcome` y `resultCode`. |

Manejo recomendado del resultado:

{% code overflow="wrap" %}
```swift
switch result.outcome {
case .success:
    // Continúa con el usuario verificado.
    break
case .warning:
    // Continúa o envía a revisión manual según tus reglas de negocio.
    break
case .canceled:
    // Permite que el usuario lo intente de nuevo.
    break
case .failed, .error:
    // Muestra el flujo de falla configurado.
    break
case .unknown:
    // Muestra una ruta de soporte o reintento.
    break
}
```
{% endcode %}

`UnicusResultCode.messageFor(result.resultCode)` devuelve el mensaje estándar de
Unicus para un código de resultado cuando tu app necesita un texto por defecto.

## Escucha los eventos de progreso

Puedes registrar un manejador de eventos antes de iniciar la verificación. Los
eventos se entregan en la main queue.

{% code overflow="wrap" %}
```swift
UnicusSdk.shared.eventHandler = { event in
    print("Unicus event: \(event.name)")
    print("Transaction id: \(event.tid ?? "-")")
    print("Message: \(event.message ?? "-")")
}
```
{% endcode %}

Elimina el manejador cuando se cierre la pantalla:

{% code overflow="wrap" %}
```swift
UnicusSdk.shared.eventHandler = nil
```
{% endcode %}

Nombres de eventos comunes: `locationCollected`, `locationSkipped`,
`sessionPrepared`, `processRequest`, `livenessProcessed`,
`enrollmentProcessed`, `authenticationComplete`, `idScanProcessed`,
`idScanBackRequired`, `idScanUserConfirmation`, `idScanComplete`, `completed`,
`error` y `nativeExit`.

## Personaliza los textos de verificación

El SDK incluye textos por defecto en inglés basados en la configuración actual
del SDK web de Unicus. Si tu aplicación necesita otro idioma o redacción,
entrega textos personalizados (overrides) al configurar Unicus.

La app de ejemplo incluye mapas completos de textos de Unicus en inglés y
español, basados en la configuración de idioma actual del SDK web, en
`example/UnicusSDKExample/SampleUnicusTexts.swift`. Usa ese archivo como punto
de partida para tu propio archivo de idioma. La pantalla de inicio del ejemplo
pide intencionalmente solo el número y el tipo de documento; los textos se
cambian en el código, no a través de la UI de la demo.

{% code overflow="wrap" %}
```swift
UnicusSdk.shared.configure(
    UnicusSdkConfig(
        baseUrl: "<UNICUS_BASE_URL>",
        apiKey: "<UNICUS_CUSTOMER_TOKEN>",
        verificationTextOverrides: SampleUnicusTexts.overrides
    )
)
```
{% endcode %}

Unicus combina tus textos personalizados con los textos por defecto y aplica el
mapa de textos final a las pantallas nativas de verificación antes de abrir la
sesión. El SDK adapta las llaves internas de texto nativo, y los saltos de línea
`<br/>` se convierten en saltos de línea nativos.

Llaves de texto comunes:

| Llave en Swift | Texto en pantalla |
| --- | --- |
| `UnicusVerificationTextKey.actionImReady` | Botón de listo. |
| `UnicusVerificationTextKey.actionContinue` | Botón de continuar. |
| `UnicusVerificationTextKey.actionTryAgain` | Botón de reintentar. |
| `UnicusVerificationTextKey.feedbackCenterFace` | Indicación de alineación del rostro. |
| `UnicusVerificationTextKey.initializingCamera` | Mensaje de inicialización de la cámara. |
| `UnicusVerificationTextKey.idScanTypeSelectionHeader` | Título del escaneo del documento. |
| `UnicusVerificationTextKey.resultFaceScanUploadMessage` | Mensaje de carga del rostro. |

Para las etiquetas avanzadas de confirmación de OCR, coordina el diccionario
`verificationOcrLocalization` con el soporte de Tekbees.

## Logs de API opcionales para pruebas

Durante las pruebas en el ambiente de pruebas (sandbox), los logs de API pueden
ayudar a tu equipo a confirmar las respuestas recibidas de Unicus.

{% hint style="warning" %}
Los logs de API son solo para desarrollo y QA. No habilites logs sensibles en
builds de producción.
{% endhint %}

{% code overflow="wrap" %}
```swift
UnicusSdk.shared.configure(
    UnicusSdkConfig(
        baseUrl: "<UNICUS_BASE_URL>",
        apiKey: "<UNICUS_CUSTOMER_TOKEN>",
        enableApiLogging: true
    )
)

UnicusSdk.shared.apiLogHandler = { entry in
    print(entry.toPrettyJson())
}
```
{% endcode %}

Por defecto, los logs están sanitizados. Los blobs biométricos cifrados, los
payloads de OCR, los datos del documento y los tokens de sesión se ocultan o se
truncan.

## Ejemplo completo con botón

{% code overflow="wrap" expandable="true" %}
```swift
import UIKit
import UnicusSDK

final class VerificationViewController: UIViewController {

    private let verifyButton = UIButton(type: .system)

    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemBackground

        UnicusSdk.shared.configure(
            UnicusSdkConfig(
                baseUrl: "<UNICUS_BASE_URL>",
                apiKey: "<UNICUS_CUSTOMER_TOKEN>"
            )
        )

        verifyButton.setTitle("Verify identity", for: .normal)
        verifyButton.addTarget(self, action: #selector(startVerification), for: .touchUpInside)
        verifyButton.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(verifyButton)
        NSLayoutConstraint.activate([
            verifyButton.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            verifyButton.centerYAnchor.constraint(equalTo: view.centerYAnchor)
        ])
    }

    @objc private func startVerification() {
        verifyButton.isEnabled = false
        verifyButton.setTitle("Verifying...", for: .normal)

        UnicusSdk.shared.start(
            .enrollmentVerify(
                document: UnicusDocument(type: .id, externalDatabaseRefId: "123456789")
            ),
            from: self
        ) { [weak self] result in
            guard let self = self else { return }
            self.verifyButton.isEnabled = true
            self.verifyButton.setTitle("Verify identity", for: .normal)

            let message: String
            switch result {
            case .success(let verification):
                message = verification.success
                    ? "Identity verified successfully"
                    : (verification.resultMessage ?? "Identity could not be verified")
            case .failure(let error):
                message = error.message
            }

            let alert = UIAlertController(title: "Unicus", message: message, preferredStyle: .alert)
            alert.addAction(UIAlertAction(title: "OK", style: .default))
            self.present(alert, animated: true)
        }
    }
}
```
{% endcode %}

Las apps SwiftUI pueden llamar a la misma API desde un view model usando
`async/await`; el ejemplo incluido muestra este patrón.

## Apariencia

El SDK aplica automáticamente la apariencia de la empresa configurada en Unicus
y devuelta por el endpoint de sesión.

| Campo | Comportamiento |
| --- | --- |
| `backgroundColor` | Color de fondo de la pantalla nativa de verificación. |
| `windowColor` | Color principal de la verificación nativa. |
| `buttonColor` | Color de acento de botones, progreso, marco y OCR. |
| `textColor` | Color del texto de botones e indicaciones. |
| `logo` | Logo de la empresa. |

iOS usa la URL remota del logo devuelta por Unicus cuando la URL apunta a una
imagen rasterizada, como PNG o JPG. Si no se puede descargar el logo, el SDK
muestra el logo de Unicus incluido en el paquete.

## Lista de verificación de pruebas

Usa un dispositivo físico para la validación final.

1. Compila la app para el simulador para confirmar que los frameworks están
   vinculados.
2. Ejecuta la app en un iPhone físico con build firmado y acepta el permiso de
   cámara.
3. Prueba un documento válido con tipo de documento `ID`, `FD`, `PP` o `DL`.
4. Prueba el permiso de ubicación aceptado y rechazado. Ambos caminos deben
   continuar.
5. Si la cuenta tiene un solo país activo, confirma que el flujo continúa sin
   mostrar un selector de país.
6. Si la cuenta tiene varios países activos, confirma que tu app muestra un
   selector de país antes de presentar la pantalla nativa de verificación.
7. Confirma que se abre la pantalla nativa de verificación.
8. Confirma que los colores y el logo de la empresa se muestran como se espera.
9. Confirma que tu app recibe un `UnicusVerificationResult`.
10. Confirma que la transacción aparece en el portal administrativo de Unicus.

Comandos útiles:

{% code overflow="wrap" %}
```bash
xcrun xctrace list devices
xcodebuild -scheme YourApp -destination 'id=<DEVICE_ID>' build
```
{% endcode %}

En iPhones físicos, usa siempre un build firmado desde Xcode. Los builds para
simulador son solo verificaciones de compilación.

## Solución de problemas

| Problema | Qué revisar |
| --- | --- |
| Código de error `not_configured` | Confirma que `UnicusSdk.shared.configure(...)` se ejecuta antes de `start(...)`. |
| Código de error `session_active` | Todavía se está ejecutando una verificación anterior. Espera a que termine antes de iniciar otra. |
| Código de error `no_view_controller` | Pasa el view controller que presenta en `start(_:from:)` o asegúrate de que exista una key window con un root view controller. |
| `dyld: Library not loaded` al iniciar | Confirma que cada `.xcframework` de `sdk/Frameworks` esté configurado como **Embed & Sign** en el target de la app. |
| Se rechaza el permiso de cámara | Confirma que `NSCameraUsageDescription` esté presente en `Info.plist`. |
| Se rechaza el permiso de ubicación | Esto no detiene la transacción. El SDK continúa sin datos de ubicación. |
| No aparece el selector de país | Confirma que la cuenta del cliente tenga más de un país activo en Unicus. Con un solo país activo, el SDK lo selecciona automáticamente. |
| No se devuelve un id de transacción | Confirma `baseUrl`, `apiKey`, el tipo de documento y el número de documento. |
| La verificación nativa no se abre en un build Debug en dispositivo | Confirma que la fase *Run Script* que intercambia el framework de desarrollo de Unicus se ejecuta después de *Embed Frameworks*. |
| La verificación nativa no se abre | Confirma que estás ejecutando en un dispositivo físico compatible con acceso a la cámara. |
| No aparecen los colores de la empresa | Confirma que `/get-restart-session` devuelve `windowColor`, `buttonColor` y `textColor`. |
| No aparece el logo | Confirma que `/get-restart-session` devuelve `logo` como una URL HTTPS de PNG o JPG. Las URL SVG no son válidas para el logo móvil nativo. |

## Soporte

Al contactar a soporte, incluye:

1. URL del ambiente.
2. Plataforma y versión de la app.
3. Modelo del dispositivo y versión del sistema operativo.
4. Id de transacción de Unicus `tid`, si se creó.
5. Código de resultado y mensaje de resultado, si están disponibles.
6. Una breve descripción del paso en el que ocurrió el problema.
