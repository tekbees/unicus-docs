---
description: >-
  Instala el SDK de Unicus para Flutter: contenido del paquete, dependencia en
  pubspec, configuración de Gradle en Android, Podfile e Info.plist en iOS y
  builds Debug en un iPhone físico.
---

# Instalación

## Contenido del paquete

Tekbees entrega el SDK de Flutter como un paquete ZIP. Los repositorios
públicos (pub, Maven, CocoaPods, Swift Package Manager) llegarán pronto.

| Ruta | Contenido |
| --- | --- |
| `sdk/unicus_sdk_flutter/` | El plugin del que depende tu app: el API de Dart (código fuente), los puentes compilados de Android e iOS, los binarios del SDK nativo de Unicus y el motor biométrico. |
| `sdk/unicus_sdk_flutter/android/repo/` | Repositorio Maven local con los binarios de Android. El plugin lo registra en tu build; no lo declaras. |
| `sdk/unicus_sdk_flutter/ios/` | `UnicusSDK.xcframework`, `<motor de verificación de Unicus>.xcframework` (builds de producción y de desarrollo), el manifiesto de privacidad y el helper de Podfile `unicus_facetec_development.rb`. |
| `example/` | App de ejemplo ejecutable. `example/lib/quick_start.dart` es la integración mínima en un solo archivo. |
| `QUICKSTART.md`, `ERRORS.md`, `TECHNICAL_INTEGRATION.md` | Copias sin conexión del inicio rápido, los códigos de error y una referencia técnica. |

El paquete contiene solo binarios: no hay código fuente del plugin ni nativo
que editar, ni nada que cambiar en `MainActivity` o `AppDelegate`.

## 1. Agrega la dependencia

Copia `sdk/unicus_sdk_flutter` en tu repositorio, por ejemplo
`vendor/unicus_sdk_flutter`, y agrégalo a `pubspec.yaml`:

{% code overflow="wrap" %}
```yaml
dependencies:
  unicus_sdk_flutter:
    path: vendor/unicus_sdk_flutter
```
{% endcode %}

{% code overflow="wrap" %}
```bash
flutter pub get
```
{% endcode %}

Versiona la carpeta con tu app (o guárdala en tu repositorio interno de
artefactos) para que todos los builds usen la misma versión. Para actualizar,
reemplaza la carpeta por la del nuevo paquete y ejecuta `flutter pub get` de
nuevo.

## 2. Android

### Versiones de Gradle

| Elemento | Mínimo del SDK | Notas |
| --- | --- | --- |
| `minSdk` | 21 | `flutter.minSdkVersion` de las versiones actuales de Flutter ya lo cumple. |
| `compileSdk` | 34 | `flutter.compileSdkVersion` de las versiones actuales de Flutter ya lo cumple. |
| Android Gradle Plugin | 8.x o 9.x | Con AGP 9, con Kotlin integrado activado o no. **Flutter 3.47 exige AGP 8.11.1 o superior.** |
| Kotlin Gradle Plugin | 2.0 | El SDK apunta a la API/lenguaje Kotlin 2.0. **Flutter 3.47 exige 2.2.20 o superior.** |
| JDK | 17 | Lo exige AGP 8. |

Tu versión de Flutter fija los mínimos reales: usa las versiones que genera
`flutter create` para ella. Para Flutter 3.47, en
`android/settings.gradle.kts`:

{% code overflow="wrap" %}
```kotlin
plugins {
    id("dev.flutter.flutter-plugin-loader") version "1.0.0"
    id("com.android.application") version "8.11.1" apply false
    id("org.jetbrains.kotlin.android") version "2.2.20" apply false
}
```
{% endcode %}

AGP 8.11.1 necesita Gradle 8.13 o superior
(`android/gradle/wrapper/gradle-wrapper.properties`).

En `android/app/build.gradle.kts`, los valores que genera Flutter sirven:

{% code overflow="wrap" %}
```kotlin
android {
    compileSdk = flutter.compileSdkVersion // 34 o superior
    defaultConfig {
        minSdk = flutter.minSdkVersion // 21 o superior
    }
}
```
{% endcode %}

### Permisos

El plugin agrega `CAMERA` e `INTERNET` a tu manifiesto. La ubicación es
opcional y **no** se agrega. Si quieres que Unicus registre dónde inició la
verificación, declárala en `android/app/src/main/AndroidManifest.xml` (e
infórmala en la sección *Seguridad de los datos* de Play Console):

{% code overflow="wrap" %}
```xml
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
```
{% endcode %}

Sin estos permisos, o si el usuario los niega, la verificación continúa sin
ubicación.

### R8 / ProGuard

No hay nada que agregar. Las reglas de conservación del plugin, del SDK nativo
y del motor biométrico vienen dentro de sus librerías y se aplican
automáticamente a tu build de release.

### WebView

Los flujos con pasos de interfaz (consentimiento, formulario, firma, OTP)
necesitan un *Android System WebView* actualizado (Chromium 90 o superior). Si
no, `start` lanza `webview_unavailable` antes de mostrar nada. El plugin no
depende de `webview_flutter`, así que no hay conflicto de versiones con tu app.

## 3. iOS

### Deployment target y CocoaPods

La parte de iOS se integra con CocoaPods. En `ios/Podfile`:

{% code overflow="wrap" %}
```ruby
platform :ios, '15.0'
```
{% endcode %}

Luego instala los pods (Flutter también lo hace en `flutter run` /
`flutter build ios`):

{% code overflow="wrap" %}
```bash
cd ios
pod install
```
{% endcode %}

{% hint style="info" %}
Swift Package Manager no es compatible en esta versión. El plugin no tiene
`Package.swift`, así que Flutter lo integra con CocoaPods aun con SPM
habilitado (muestra una advertencia). Conserva `ios/Podfile`. Para quitar la
advertencia, deshabilita SPM para tu app en `pubspec.yaml`:

```yaml
flutter:
  config:
    enable-swift-package-manager: false
```
{% endhint %}

### Info.plist

En `ios/Runner/Info.plist`:

{% code overflow="wrap" %}
```xml
<key>NSCameraUsageDescription</key>
<string>Usamos la cámara para verificar tu identidad.</string>
<!-- Opcional: ubicación como metadato de la transacción; sin ella la verificación continúa. -->
<key>NSLocationWhenInUseUsageDescription</key>
<string>Registramos dónde inicia la verificación.</string>
```
{% endcode %}

Si tu app declara `WKAppBoundDomains`, agrega el dominio de las pantallas del
flujo de tu ambiente (lo entrega Tekbees). Si no, `start` lanza
`webview_domain_not_allowed` antes de mostrar nada. Consulta
[Compatibilidad y seguridad](../compatibility-and-security.md).

### Builds Debug en un iPhone físico

El build de producción del motor biométrico no se ejecuta en builds Debug en
un dispositivo (con el depurador conectado). El paquete trae su build de
desarrollo y un helper de Podfile que lo intercambia con una fase de build.
Agrega el helper a `ios/Podfile`:

{% code overflow="wrap" %}
```ruby
require File.expand_path('.symlinks/plugins/unicus_sdk_flutter/ios/unicus_facetec_development.rb', __dir__)

post_install do |installer|
  installer.pods_project.targets.each do |target|
    flutter_additional_ios_build_settings(target)
  end
  unicus_install_facetec_development_swap(File.join(__dir__, 'Runner.xcodeproj'))
end
```
{% endcode %}

Si tu Podfile ya tiene un bloque `post_install`, agrégale solo la línea
`unicus_install_facetec_development_swap`. Luego ejecuta `pod install` (o
`flutter run`).

* La fase solo actúa en **builds Debug firmados para dispositivo**
  (`iphoneos`). Los builds de simulador, Profile, Release y `--no-codesign`
  conservan el framework de producción, así que tu build para la App Store no
  se ve afectado.
* La fase es idempotente: ejecutar `pod install` de nuevo la actualiza.
* Si el target de tu app no se llama `Runner`, pasa su nombre como segundo
  argumento: `unicus_install_facetec_development_swap(path, 'MiApp')`.

## 4. Comprueba la instalación

Ejecuta el ejemplo del paquete, o tu app, en un dispositivo físico:

{% code overflow="wrap" %}
```bash
flutter devices
flutter run -d <DEVICE_ID> --dart-define=UNICUS_API_KEY=<CUSTOMER_TOKEN>
```
{% endcode %}

Los valores de `--dart-define` quedan compilados en el build: úsalo solo para
pruebas locales y carga el Customer Token desde tu propia configuración en tu
app.

Siguiente: [Configuración](configuration.md).
