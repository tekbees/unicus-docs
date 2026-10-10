---
description: >-
  Instala el SDK Android de Unicus: contenido del paquete, configuración de
  Gradle, permisos, versiones mínimas y R8 / ProGuard.
---

# Instalación

## Contenido del paquete

Tekbees entrega el SDK como un paquete ZIP:

{% code overflow="wrap" %}
```text
unicus_sdk_android_<versión>_customer_package/
  README.md                    Inicio rápido del paquete.
  TECHNICAL_INTEGRATION.md     Referencia técnica.
  sdk/repo/                    Repositorio Maven local:
    com/tekbees/unicus/unicus-sdk/<versión>/     el SDK de Unicus (AAR)
    <motor biométrico>/                          su motor biométrico embebido
  example/                     Proyecto de Android Studio que usa el binario del SDK.
```
{% endcode %}

El repositorio contiene todo lo que el SDK necesita. Tu app declara **una**
dependencia, `com.tekbees.unicus:unicus-sdk`; el motor biométrico y las demás
librerías llegan de forma transitiva.

{% hint style="info" %}
Un repositorio Maven alojado estará disponible próximamente. Cuando se publique
solo cambiarás la entrada `maven { }` local por la URL que te dé Tekbees; la
línea de la dependencia no cambia.
{% endhint %}

## Configuración de Gradle

1. Copia `sdk/repo` en tu proyecto, por ejemplo en `vendor/unicus/repo`, y
   súbelo al control de versiones (o guárdalo en tu repositorio interno de
   artefactos).
2. Registra el repositorio en `settings.gradle.kts`:

   {% code overflow="wrap" %}
   ```kotlin
   dependencyResolutionManagement {
       repositories {
           google()
           mavenCentral()
           maven { url = uri(rootDir.resolve("vendor/unicus/repo")) }
       }
   }
   ```
   {% endcode %}

   Con Groovy (`settings.gradle`):

   {% code overflow="wrap" %}
   ```groovy
   dependencyResolutionManagement {
       repositories {
           google()
           mavenCentral()
           maven { url = uri("${rootDir}/vendor/unicus/repo") }
       }
   }
   ```
   {% endcode %}

3. Agrega la dependencia al módulo de la app:

   {% code overflow="wrap" %}
   ```kotlin
   dependencies {
       implementation("com.tekbees.unicus:unicus-sdk:0.1.0")
   }
   ```
   {% endcode %}

4. Sincroniza Gradle y compila. Para comprobar la instalación, ejecuta el
   proyecto `example/` del paquete en un dispositivo
   (`cp app/unicus.properties.example app/unicus.properties`, completa tu
   Customer Token y luego `./gradlew :app:installDebug`).

El SDK trae `kotlinx-coroutines-android`, `appcompat`, `core-ktx`,
`androidx.webkit` y `androidx.browser`. Si tu app fija versiones anteriores de
estas librerías, Gradle resuelve la más reciente.

## Versiones mínimas

| Elemento | Mínimo |
| --- | --- |
| `minSdk` | 21 |
| `compileSdk` | 34 |
| Android Gradle Plugin | 8.x (9.x soportado) |
| Kotlin Gradle plugin | 2.0, o sin Kotlin (las apps solo Java están soportadas) |
| Toolchain Java | 17 |
| Android System WebView | Chromium 90, para flujos con pantallas en modo WebView |

## Permisos

| Permiso | Quién lo declara | Notas |
| --- | --- | --- |
| `CAMERA` | El manifiesto del SDK (se incorpora automáticamente). | Se pide al usuario cuando se abre la cámara. No gestionas el resultado. |
| `INTERNET` | El manifiesto del SDK. | |
| `ACCESS_FINE_LOCATION` / `ACCESS_COARSE_LOCATION` | **Tu app**, solo si quieres la ubicación. | Opcional. El SDK no los declara. |

La ubicación es un metadato opcional de la transacción. Cuando tu app declara
uno de los permisos de ubicación y `collectLocationOnStart` es `true` (valor
por defecto), el SDK la pide al usuario y la envía con la transacción. Si el
usuario la niega, la app no la declara o el dispositivo no tiene ubicación, la
verificación continúa sin ella. Si declaras la ubicación, infórmala en el
formulario *Seguridad de los datos* de Play Console (ubicación aproximada /
precisa, recopilada, no compartida).

{% code overflow="wrap" %}
```xml
<!-- AndroidManifest.xml de tu app, solo si quieres la ubicación -->
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
```
{% endcode %}

El SDK registra sus propias actividades (contenedor de la cámara, pantallas del
flujo, solicitud de permisos). No agregas nada más a tu manifiesto y no
sobrescribes `onActivityResult` ni `onRequestPermissionsResult`.

## R8 / ProGuard

No necesitas reglas. El AAR trae sus reglas de consumidor: conservan la API
pública, las actividades del SDK y el motor biométrico. Minimiza tu build de
release como siempre.

Si tu app usa herramientas RASP o de blindaje (app shielding) que exigen una
lista de permitidos, consulta
[Compatibilidad y seguridad](../compatibility-and-security.md); las
actividades que debes permitir son
`com.tekbees.unicus.sdk.internal.UnicusVerificationActivity` y
`com.tekbees.unicus.sdk.internal.UnicusFlowWebActivity`.

## Actualizar el SDK

Reemplaza la carpeta `vendor/unicus/repo` por la del paquete nuevo y cambia la
versión en la línea de la dependencia. Revisa la página
[Versiones](../versioning.md) para conocer los cambios entre versiones.

Siguiente: [Configuración](configuration.md).
