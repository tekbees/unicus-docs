---
description: >-
  Integra la verificación de identidad de Unicus en aplicaciones Android nativas
  (Kotlin o Java) usando solo el SDK de Unicus.
---

# INTEGRACIÓN ANDROID

El SDK de Unicus para Android permite que tu aplicación Android nativa ejecute la
verificación de identidad. Tu app integra **Unicus** únicamente. El SDK crea la
transacción de Unicus, obtiene la configuración de la sesión, aplica la marca de
la empresa, abre las pantallas nativas de verificación, procesa los datos
biométricos cifrados a través de Unicus y devuelve el resultado final.

{% hint style="info" %}
El SDK de Unicus para Android incluye todo lo necesario para la verificación de
identidad: captura con la cámara, prueba de vida, escaneo de documentos, cifrado
y las llamadas a la API de Unicus. No agregues otras librerías biométricas o de
captura de documentos para este flujo, y no reenvíes resultados de actividades ni
resultados de permisos: el SDK de Unicus los gestiona internamente.
{% endhint %}

## Lo que Unicus te entregará

Antes de iniciar la integración, solicita los siguientes valores a tu
administrador de Unicus o al equipo de soporte de Tekbees.

| Valor | Descripción | Ejemplo |
| --- | --- | --- |
| `baseUrl` | URL del ambiente de la API de Unicus. | `https://alpha.idunicus.com:8080` |
| `apiKey` | Customer Token generado para tu empresa. | `<UNICUS_CUSTOMER_TOKEN>` |

Las llaves internas de Unicus (el device id de sesión de Tekbees y la llave del
motor de verificación de Unicus) están embebidas dentro del SDK de Unicus para
Android. La app del cliente no debe solicitar, almacenar ni enviar esas llaves
internas.

La app del cliente usa `apiKey` solo para su Customer Token de Unicus. La creación
de la transacción usa ese customer token a través de `X-Customer-ID`. La consulta
de países activos usa el mismo customer token a través de `X-Device-ID`, porque
ese endpoint lo usa para resolver los países habilitados del cliente. Las llamadas
de sesión como `/get-restart-session` y `/sdk-execution-keys` usan el device id
de sesión de Tekbees embebido en el SDK a través de `X-Device-ID`.

{% hint style="warning" %}
Usa los valores del ambiente correcto. Las credenciales del ambiente de pruebas
(sandbox), staging y producción son diferentes. No subas credenciales de
producción a repositorios públicos.
{% endhint %}

## Requisitos

| Plataforma | Requisito |
| --- | --- |
| Android | `minSdk 21` o superior, `compileSdk 36` recomendado |
| Herramientas de build | Android Gradle Plugin `8.x` o superior, Gradle `8.x` o superior, Java `17` |
| Lenguaje | Kotlin `2.x` (también se ofrece una API de callbacks compatible con Java) |
| Librerías | AndroidX. El SDK incluye `appcompat`, `core-ktx` y `kotlinx-coroutines-android` de forma transitiva. |
| Dispositivos | Dispositivo Android físico con cámara para la validación completa |
| Permisos | La cámara es obligatoria. La ubicación es opcional y el SDK continúa si el usuario la rechaza. |

## 1. Revisa el paquete para clientes

Tekbees entrega un paquete para clientes con un nombre como este:

{% code overflow="wrap" %}
```text
unicus_sdk_android_0.1.0_customer_package.zip
```
{% endcode %}

Al descomprimirlo, contiene:

{% code overflow="wrap" %}
```text
unicus_sdk_android_0.1.0_customer_package/
  README.md
  TECHNICAL_INTEGRATION.md
  sdk/
    repo/
      com/tekbees/unicus/unicus-sdk/0.1.0/
      ...
  example/
    settings.gradle.kts
    app/
      build.gradle.kts
      unicus.properties.example
      src/main/kotlin/com/tekbees/unicus/example/
        MainActivity.kt
        UnicusExampleScreen.kt
        SampleUnicusTexts.kt
```
{% endcode %}

| Ruta | Propósito |
| --- | --- |
| `README.md` | Guía de inicio rápido del paquete. |
| `TECHNICAL_INTEGRATION.md` | Referencia técnica para el equipo de desarrollo del cliente. |
| `sdk/repo/` | Repositorio Maven local con el SDK cerrado de Unicus (`com.tekbees.unicus:unicus-sdk`) y sus dependencias internas. |
| `example/` | Proyecto ejecutable de Android Studio (Jetpack Compose) ya configurado para usar el SDK incluido. |
| `example/app/src/main/kotlin/.../SampleUnicusTexts.kt` | Mapas de idioma/textos editables que usan las llaves públicas `Unicus_`. |

El paquete contiene el SDK cerrado de Unicus con todos sus componentes internos,
recursos y un ejemplo ejecutable. Las aplicaciones del cliente integran solo
Unicus.

## 2. Ejecuta el ejemplo incluido

Antes de modificar tu propia app, ejecuta el ejemplo incluido para confirmar el
ambiente, el dispositivo, los permisos y el customer token.

{% code overflow="wrap" %}
```bash
cd unicus_sdk_android_0.1.0_customer_package/example
cp app/unicus.properties.example app/unicus.properties
```
{% endcode %}

Edita `app/unicus.properties` con los valores entregados por Tekbees:

{% code overflow="wrap" %}
```properties
UNICUS_BASE_URL=<UNICUS_BASE_URL>
UNICUS_API_KEY=<UNICUS_CUSTOMER_TOKEN>
```
{% endcode %}

Luego abre la carpeta `example` en Android Studio y ejecuta la configuración
`app` en un dispositivo físico, o usa la línea de comandos:

{% code overflow="wrap" %}
```bash
./gradlew :app:installDebug
```
{% endcode %}

La pantalla del ejemplo solo solicita:

1. Número de documento.
2. Tipo de documento: `ID`, `FD`, `PP` o `DL`.

El ejemplo también puede mostrar la solicitud de permiso de ubicación de Android.
Si el usuario no comparte su ubicación, la transacción continúa sin datos de
ubicación. Si tu cuenta de Unicus tiene más de un país activo, el ejemplo muestra
un selector de país después de crear el id de la transacción. Si solo hay un país
activo, el SDK lo selecciona automáticamente y continúa.

El SDK crea la transacción, lee la configuración de sesión del cliente, aplica el
tema y los textos, abre la experiencia nativa de verificación, procesa las
solicitudes cifradas y devuelve el resultado. El ejemplo también muestra las
respuestas sanitizadas de la API de Unicus recibidas durante el flujo.

Para una verificación solo de compilación:

{% code overflow="wrap" %}
```bash
./gradlew :app:assembleDebug
```
{% endcode %}

Usa un dispositivo Android físico con cámara para la verificación completa de
punta a punta.

## 3. Agrega la dependencia

Para tu propia app, copia `sdk/repo` del paquete para clientes en el repositorio
de tu aplicación, por ejemplo:

{% code overflow="wrap" %}
```text
your_android_app/
  vendor/
    unicus/
      repo/
```
{% endcode %}

Registra el repositorio Maven local en `settings.gradle.kts`:

{% code overflow="wrap" %}
```kotlin
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
        maven {
            url = uri(rootDir.resolve("vendor/unicus/repo"))
        }
    }
}
```
{% endcode %}

Luego agrega la dependencia al módulo de tu aplicación (`app/build.gradle.kts`):

{% code overflow="wrap" %}
```kotlin
dependencies {
    implementation("com.tekbees.unicus:unicus-sdk:0.1.0")
}
```
{% endcode %}

El SDK de Unicus requiere compatibilidad con Java 17 en el módulo de la
aplicación:

{% code overflow="wrap" %}
```kotlin
android {
    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }
}
```
{% endcode %}

Sincroniza el proyecto. Las dependencias internas del SDK de Unicus se resuelven
automáticamente desde el mismo repositorio local; no las declares tú mismo y no
solicites llaves internas de Tekbees.

## 4. Configura Android

El manifest del SDK de Unicus ya declara los permisos requeridos y las actividades
internas que necesita. Se combinan automáticamente con tu app:

{% code overflow="wrap" %}
```xml
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
```
{% endcode %}

Si tu aplicación no debe solicitar la ubicación para este flujo, elimina los
permisos de ubicación en tu `AndroidManifest.xml` y el SDK continúa sin datos de
ubicación:

{% code overflow="wrap" %}
```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" tools:node="remove" />
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" tools:node="remove" />
</manifest>
```
{% endcode %}

No se requieren cambios en la `Activity`. El SDK abre la verificación desde su
propia actividad anfitriona transparente, por lo que tu app no sobrescribe
`onActivityResult` ni `onRequestPermissionsResult`.

Si tu build de release usa R8/ProGuard, no se necesitan reglas adicionales: el SDK
incluye reglas de consumidor que conservan la API pública de Unicus y sus
componentes internos.

## 5. Configura el SDK

Configura la instancia compartida `UnicusSdk` en la parte de tu app que gestiona
el flujo de verificación.

{% code overflow="wrap" %}
```kotlin
import com.tekbees.unicus.sdk.UnicusSdk
import com.tekbees.unicus.sdk.UnicusSdkConfig

fun configureUnicus() {
    UnicusSdk.shared.configure(
        UnicusSdkConfig(
            baseUrl = "<UNICUS_BASE_URL>",
            apiKey = "<UNICUS_CUSTOMER_TOKEN>",
            collectLocationOnStart = true
        )
    )
}
```
{% endcode %}

Java:

{% code overflow="wrap" %}
```java
UnicusSdk.getShared().configure(
    new UnicusSdkConfig.Builder("<UNICUS_BASE_URL>", "<UNICUS_CUSTOMER_TOKEN>")
        .collectLocationOnStart(true)
        .build()
);
```
{% endcode %}

Llama a `configureUnicus()` antes de iniciar la primera verificación. Un lugar
habitual es después de que el usuario llega a la pantalla donde puede comenzar la
verificación de identidad. `collectLocationOnStart` es opcional y su valor por
defecto es `true`. Configúralo en `false` solo cuando tu app no deba solicitar la
ubicación para este flujo.

## 6. Inicia una verificación

Envía el tipo de documento y el número de documento del usuario a `start`, junto
con la `Activity` que está actualmente en pantalla. Todos los callbacks se
entregan en el hilo principal.

{% code overflow="wrap" %}
```kotlin
import com.tekbees.unicus.sdk.UnicusCallback
import com.tekbees.unicus.sdk.model.UnicusDocument
import com.tekbees.unicus.sdk.model.UnicusDocumentType
import com.tekbees.unicus.sdk.model.UnicusSdkException
import com.tekbees.unicus.sdk.model.UnicusVerificationRequest
import com.tekbees.unicus.sdk.model.UnicusVerificationResult

fun startUnicusVerification(activity: Activity) {
    UnicusSdk.shared.start(
        activity,
        UnicusVerificationRequest.enrollmentVerify(
            document = UnicusDocument(
                type = UnicusDocumentType.ID,
                externalDatabaseRefId = "123456789"
            )
        ),
        object : UnicusCallback<UnicusVerificationResult> {
            override fun onSuccess(value: UnicusVerificationResult) {
                if (value.success) {
                    // La verificación de identidad fue exitosa.
                } else {
                    // Muestra una ruta de reintento, rechazo o soporte según tu flujo de negocio.
                }
            }

            override fun onError(error: UnicusSdkException) {
                // No se pudo iniciar la verificación (error de configuración, de red o de sesión).
            }
        }
    )
}
```
{% endcode %}

Con corrutinas de Kotlin, cada operación tiene una variante `suspend` que lanza
`UnicusSdkException` en caso de falla:

{% code overflow="wrap" %}
```kotlin
import com.tekbees.unicus.sdk.start

lifecycleScope.launch {
    try {
        val result = UnicusSdk.shared.start(activity, request)
        // Gestiona aquí el UnicusVerificationResult.
    } catch (error: UnicusSdkException) {
        // error.code y error.message describen la falla.
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
`/get-restart-session` en la integración estándar de Android. Tampoco debe pedir
al usuario ni al desarrollador de la aplicación que seleccione un valor de
proceso.

La secuencia estándar es:

1. Tu app llama a `UnicusSdk.shared.start(...)`.
2. El SDK llama a `/start-mobile-transaction` y recibe un nuevo id de transacción
   `tid`.
3. El SDK solicita la ubicación a la plataforma cuando está habilitada. Si el
   usuario rechaza el permiso o el dispositivo no puede proporcionar la ubicación,
   el SDK continúa.
4. El SDK llama a `/company-countries`, lee los países activos del cliente y
   selecciona automáticamente el único país activo cuando aplica.
5. El SDK llama a `/get-restart-session` usando el `tid` y el país seleccionado
   cuando hay uno disponible.
6. El SDK aplica los colores, el logo y los textos de verificación de Unicus de la
   empresa.
7. El SDK abre la pantalla nativa de verificación.
8. El SDK envía los datos de verificación cifrados a Unicus.
9. Tu app recibe un `UnicusVerificationResult`.

Solo puede ejecutarse una verificación a la vez. Llamar a `start` mientras otra
verificación está activa falla con el código de error `session_active`.

## Selección de país

Para la mayoría de las integraciones, basta con llamar a `start(...)`. Si tu
cuenta de Unicus tiene un solo país activo, el SDK lo selecciona automáticamente.
Si la cuenta tiene varios países activos y tu app debe permitir que el usuario
elija, usa primero `prepareEnrollmentVerify(...)`.

{% code overflow="wrap" %}
```kotlin
import com.tekbees.unicus.sdk.prepareEnrollmentVerify
import com.tekbees.unicus.sdk.start

suspend fun startWithCountrySelection(activity: Activity) {
    val prepared = UnicusSdk.shared.prepareEnrollmentVerify(
        activity = activity,
        document = UnicusDocument(
            type = UnicusDocumentType.ID,
            externalDatabaseRefId = "123456789"
        )
    )

    val selectedCountry: String? = if (prepared.requiresCountrySelection) {
        showYourCountryPicker(prepared.countries)
    } else {
        prepared.defaultCountry?.code
    }

    val result = UnicusSdk.shared.start(
        activity,
        prepared.toRequest(country = selectedCountry)
    )

    // Gestiona aquí el UnicusVerificationResult.
}
```
{% endcode %}

`prepareEnrollmentVerify(...)` crea el id de transacción de Unicus, opcionalmente
recolecta la ubicación y devuelve los países activos. No abre la pantalla nativa
de verificación. Llama a `start(...)` con `prepared.toRequest(...)` después de que
tu app haya seleccionado el país. El valor del país debe ser un código ISO 3166-1
alfa-2, como `CO`, `US` o `MX`.

## Tipos de documento

Usa el enum que ofrece el SDK.

| Valor en Kotlin | Valor en la API | Descripción |
| --- | --- | --- |
| `UnicusDocumentType.ID` | `ID` | Documento nacional de identidad |
| `UnicusDocumentType.FOREIGN_DOCUMENT` | `FD` | Documento extranjero |
| `UnicusDocumentType.PASSPORT` | `PP` | Pasaporte |
| `UnicusDocumentType.DRIVER_LICENSE` | `DL` | Licencia de conducción |

## Comportamiento del flujo nativo

El flujo nativo por defecto lo gestiona internamente el SDK. No se requiere ningún
campo de flujo en la aplicación del cliente.

Unicus revisa el estado actual de la sesión:

| Estado de la sesión | Comportamiento |
| --- | --- |
| El usuario ya está enrolado | El SDK inicia la autenticación facial. |
| El usuario no está enrolado | El SDK inicia el enrolamiento facial y de documento. |

En la integración estándar de Android, la aplicación solo debe proporcionar el
tipo de documento y el número de documento.

## Lee el resultado

`start` entrega un `UnicusVerificationResult`.

| Campo | Descripción |
| --- | --- |
| `success` | `true` cuando la verificación se completó exitosamente. |
| `outcome` | Categoría normalizada del resultado: `SUCCESS`, `WARNING`, `FAILED`, `CANCELED`, `ERROR` o `UNKNOWN`. |
| `tid` | Id de la transacción de Unicus creada por el SDK. |
| `resultCode` | Código de resultado de la transacción de Unicus, cuando está disponible. Si Unicus ya finalizó la transacción, este valor tiene prioridad sobre el estado de la pantalla nativa. |
| `resultMessage` | Mensaje de resultado legible, cuando está disponible. |
| `status` | Estado de la pantalla nativa para diagnóstico técnico. |
| `sessionError` | `true` solo ante una interrupción técnica, una cancelación o un error nativo/de sesión. Los resultados de negocio deben gestionarse a través de `outcome` y `resultCode`. |

Manejo recomendado del resultado:

{% code overflow="wrap" %}
```kotlin
when (result.outcome) {
    UnicusVerificationOutcome.SUCCESS -> {
        // Continúa con el usuario verificado.
    }
    UnicusVerificationOutcome.WARNING -> {
        // Continúa o envía a revisión manual según tus reglas de negocio.
    }
    UnicusVerificationOutcome.CANCELED -> {
        // Permite que el usuario reintente.
    }
    UnicusVerificationOutcome.FAILED,
    UnicusVerificationOutcome.ERROR -> {
        // Muestra el flujo de falla configurado.
    }
    UnicusVerificationOutcome.UNKNOWN -> {
        // Muestra una ruta de soporte o de reintento.
    }
}
```
{% endcode %}

`UnicusResultCode.messageFor(result.resultCode)` devuelve el mensaje estándar de
Unicus para un código de resultado cuando tu app necesita un texto por defecto.

## Escucha los eventos de progreso

Puedes registrar un listener de eventos antes de iniciar la verificación. Los
eventos se entregan en el hilo principal.

{% code overflow="wrap" %}
```kotlin
UnicusSdk.shared.setEventListener { event ->
    Log.d("Unicus", "Unicus event: ${event.name}")
    Log.d("Unicus", "Transaction id: ${event.tid}")
    Log.d("Unicus", "Message: ${event.message}")
}
```
{% endcode %}

Elimina el listener cuando se destruya la pantalla:

{% code overflow="wrap" %}
```kotlin
UnicusSdk.shared.setEventListener(null)
```
{% endcode %}

Nombres de eventos comunes: `locationCollected`, `locationSkipped`,
`sessionPrepared`, `processRequest`, `livenessProcessed`, `enrollmentProcessed`,
`authenticationComplete`, `idScanProcessed`, `idScanBackRequired`,
`idScanUserConfirmation`, `idScanComplete`, `completed`, `error` y
`nativeExit`.

## Personaliza los textos de verificación

El SDK incluye textos por defecto en inglés basados en la configuración actual del
SDK web de Unicus. Si tu aplicación necesita otro idioma o redacción, proporciona
textos personalizados al configurar Unicus.

La app de ejemplo incluye mapas de textos de Unicus completos en inglés y español,
basados en la configuración de idioma actual del SDK web, en
`example/app/src/main/kotlin/com/tekbees/unicus/example/SampleUnicusTexts.kt`.
Usa ese archivo como punto de partida para tu propio archivo de idioma. La
pantalla de inicio del ejemplo solo solicita intencionalmente el número y el tipo
de documento; los textos se cambian en el código, no desde la interfaz de la demo.

{% code overflow="wrap" %}
```kotlin
UnicusSdk.shared.configure(
    UnicusSdkConfig(
        baseUrl = "<UNICUS_BASE_URL>",
        apiKey = "<UNICUS_CUSTOMER_TOKEN>",
        verificationTextOverrides = SampleUnicusTexts.overrides
    )
)
```
{% endcode %}

Unicus combina tus textos personalizados con los textos por defecto y aplica el
mapa de textos final a las pantallas nativas de verificación antes de abrir la
sesión. El SDK adapta las llaves internas de texto nativas, y los saltos de línea
`<br/>` se convierten en saltos de línea nativos.

Llaves de texto comunes:

| Llave en Kotlin | Texto en pantalla |
| --- | --- |
| `UnicusVerificationTextKey.ACTION_IM_READY` | Botón de listo. |
| `UnicusVerificationTextKey.ACTION_CONTINUE` | Botón de continuar. |
| `UnicusVerificationTextKey.ACTION_TRY_AGAIN` | Botón de reintentar. |
| `UnicusVerificationTextKey.FEEDBACK_CENTER_FACE` | Indicación de alineación del rostro. |
| `UnicusVerificationTextKey.INITIALIZING_CAMERA` | Mensaje de inicialización de la cámara. |
| `UnicusVerificationTextKey.ID_SCAN_TYPE_SELECTION_HEADER` | Título del escaneo de documento. |
| `UnicusVerificationTextKey.RESULT_FACE_SCAN_UPLOAD_MESSAGE` | Mensaje de carga del rostro. |

Para etiquetas avanzadas de confirmación de OCR, coordina el diccionario
`verificationOcrLocalization` con el soporte de Tekbees.

## Logs opcionales de la API para pruebas

Durante las pruebas en el ambiente de pruebas (sandbox), los logs de la API pueden
ayudar a tu equipo a confirmar las respuestas recibidas de Unicus.

{% hint style="warning" %}
Los logs de la API son solo para desarrollo y QA. No habilites logs sensibles en
builds de producción.
{% endhint %}

{% code overflow="wrap" %}
```kotlin
UnicusSdk.shared.configure(
    UnicusSdkConfig(
        baseUrl = "<UNICUS_BASE_URL>",
        apiKey = "<UNICUS_CUSTOMER_TOKEN>",
        enableApiLogging = true
    )
)

UnicusSdk.shared.setApiLogListener { entry ->
    Log.d("Unicus API", entry.toPrettyJson())
}
```
{% endcode %}

Por defecto, los logs están sanitizados. Los blobs biométricos cifrados, los
payloads de OCR, los datos del documento y los tokens de sesión se ocultan o se
truncan.

## Ejemplo completo con botón

{% code overflow="wrap" expandable="true" %}
```kotlin
import android.os.Bundle
import android.widget.Button
import android.widget.Toast
import androidx.appcompat.app.AppCompatActivity
import androidx.lifecycle.lifecycleScope
import com.tekbees.unicus.sdk.UnicusSdk
import com.tekbees.unicus.sdk.UnicusSdkConfig
import com.tekbees.unicus.sdk.model.UnicusDocument
import com.tekbees.unicus.sdk.model.UnicusDocumentType
import com.tekbees.unicus.sdk.model.UnicusSdkException
import com.tekbees.unicus.sdk.model.UnicusVerificationRequest
import com.tekbees.unicus.sdk.start
import kotlinx.coroutines.launch

class VerificationActivity : AppCompatActivity() {

    private lateinit var verifyButton: Button

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_verification)
        verifyButton = findViewById(R.id.verify_button)

        UnicusSdk.shared.configure(
            UnicusSdkConfig(
                baseUrl = "<UNICUS_BASE_URL>",
                apiKey = "<UNICUS_CUSTOMER_TOKEN>"
            )
        )

        verifyButton.setOnClickListener { startVerification() }
    }

    private fun startVerification() {
        verifyButton.isEnabled = false
        verifyButton.text = "Verifying..."

        lifecycleScope.launch {
            try {
                val result = UnicusSdk.shared.start(
                    this@VerificationActivity,
                    UnicusVerificationRequest.enrollmentVerify(
                        document = UnicusDocument(
                            type = UnicusDocumentType.ID,
                            externalDatabaseRefId = "123456789"
                        )
                    )
                )
                val message = if (result.success) {
                    "Identity verified successfully"
                } else {
                    result.resultMessage ?: "Identity could not be verified"
                }
                Toast.makeText(this@VerificationActivity, message, Toast.LENGTH_LONG).show()
            } catch (error: UnicusSdkException) {
                Toast.makeText(this@VerificationActivity, error.message, Toast.LENGTH_LONG).show()
            } finally {
                verifyButton.isEnabled = true
                verifyButton.text = "Verify identity"
            }
        }
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
| `textColor` | Color del texto de botones y de las indicaciones. |
| `logo` | Logo de la empresa. |

Android requiere que el logo sea un recurso drawable nativo, por lo que Android
aplica los colores automáticamente y muestra el logo de Unicus por defecto. Para
mostrar tu propio logo dentro de la pantalla nativa de verificación, agrega un
drawable a tu app y registra su nombre antes de iniciar:

{% code overflow="wrap" %}
```kotlin
UnicusSdk.shared.setAndroidLogoResourceName("my_company_logo")
```
{% endcode %}

Usa un PNG con fondo transparente, de aproximadamente 720x266 píxeles, ubicado en
`res/drawable-nodpi/`.

## Lista de verificación de pruebas

Usa un dispositivo físico para la validación final.

1. Sincroniza el proyecto en Android Studio.
2. Ejecuta la app en un dispositivo Android físico y acepta el permiso de cámara.
3. Prueba un documento válido con tipo de documento `ID`, `FD`, `PP` o `DL`.
4. Prueba el permiso de ubicación aceptado y rechazado. Ambos caminos deben
   continuar.
5. Si la cuenta tiene un país activo, confirma que el flujo continúa sin mostrar
   un selector de país.
6. Si la cuenta tiene varios países activos, confirma que tu app muestra un
   selector de país antes de abrir la pantalla nativa de verificación.
7. Confirma que se abre la pantalla nativa de verificación.
8. Confirma que los colores y el logo de la empresa se muestran como se espera.
9. Confirma que tu app recibe un `UnicusVerificationResult`.
10. Confirma que la transacción aparece en el portal administrativo de Unicus.

Comandos útiles:

{% code overflow="wrap" %}
```bash
./gradlew :app:assembleDebug
./gradlew :app:installDebug
adb logcat -s UnicusSDK
```
{% endcode %}

## Solución de problemas

| Problema | Qué revisar |
| --- | --- |
| Código de error `not_configured` | Confirma que `UnicusSdk.shared.configure(...)` se ejecuta antes de `start(...)`. |
| Código de error `session_active` | Una verificación anterior todavía está en curso. Espera su callback antes de iniciar de nuevo. |
| Gradle no puede resolver `com.tekbees.unicus:unicus-sdk` | Confirma que la entrada `maven { url = uri(...) }` apunta a la carpeta `repo` copiada y que `sdk/repo` se copió por completo. |
| Se rechaza el permiso de cámara | La pantalla nativa muestra su propia guía sobre el permiso de cámara. Confirma que el permiso `CAMERA` no se eliminó del manifest combinado. |
| Se rechaza el permiso de ubicación | Esto no detiene la transacción. El SDK continúa sin datos de ubicación. |
| No aparece el selector de país | Confirma que la cuenta del cliente tiene más de un país activo en Unicus. Con un solo país activo, el SDK lo selecciona automáticamente. |
| No se devuelve un id de transacción | Confirma `baseUrl`, `apiKey`, el tipo de documento y el número de documento. |
| La verificación nativa no se abre | Confirma que estás ejecutando en un dispositivo físico compatible con acceso a la cámara y que la `Activity` enviada a `start` está en pantalla. |
| No aparecen los colores de la empresa | Confirma que `/get-restart-session` devuelve `windowColor`, `buttonColor` y `textColor`. |
| No aparece el logo de la empresa | Android solo muestra recursos drawable. Registra uno con `setAndroidLogoResourceName(...)`. |

## Soporte

Al contactar a soporte, incluye:

1. URL del ambiente.
2. Plataforma y versión de la app.
3. Modelo del dispositivo y versión del sistema operativo.
4. Id de la transacción de Unicus `tid`, si se creó.
5. Código de resultado y mensaje de resultado, si están disponibles.
6. Una breve descripción del paso en el que ocurrió el problema.
