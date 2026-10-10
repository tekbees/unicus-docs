---
description: >-
  Agrega el SDK de Unicus para iOS a tu proyecto de Xcode: contenido del
  paquete, Xcode, CocoaPods o Swift Package Manager, permisos y la fase Debug.
---

# Instalación

## Contenido del paquete

Tekbees entrega un ZIP con un nombre como `unicus_sdk_ios_<versión>_customer_package.zip`:

{% code overflow="wrap" %}
```text
unicus_sdk_ios_<versión>_customer_package/
  README.md                  Inicio rápido del paquete.
  QUICKSTART.md              Una página: dependencia, Info.plist, código, resultados.
  ERRORS.md                  Todos los códigos de UnicusSdkError.
  TECHNICAL_INTEGRATION.md   Referencia de integración.
  sdk/
    Frameworks/
      UnicusSDK.xcframework                           SDK de Unicus (dispositivo + simulador).
      <Unicus verification engine>.xcframework        Motor de verificación que usa el SDK.
      <Unicus verification engine>ForDevelopment.xcframework   Solo para compilaciones Debug en dispositivo.
    UnicusSDK.podspec        Integración con CocoaPods.
    Package.swift            Integración con Swift Package Manager.
    swap_facetec_development_framework.sh   Script de la fase de compilación Debug en dispositivo.
  example/                   Proyecto de Xcode ejecutable que usa los binarios.
```
{% endcode %}

Tu app incluye dos frameworks: `UnicusSDK.xcframework` y el framework del motor
de verificación. El framework `ForDevelopment` nunca se incluye directamente
(consulta *Compilaciones Debug en un iPhone físico* más abajo).

{% hint style="info" %}
Los repositorios alojados de Swift Package Manager y CocoaPods llegarán
**próximamente**. Mientras tanto, integra desde la carpeta del paquete como se
muestra abajo. Pasar al repositorio alojado solo cambiará la línea de la
dependencia.
{% endhint %}

## 1. Copia el SDK en tu repositorio

{% code overflow="wrap" %}
```text
TuApp/
  Vendor/
    Unicus/            ← contenido de la carpeta sdk/ del paquete
      Frameworks/
      UnicusSDK.podspec
      Package.swift
      swap_facetec_development_framework.sh
```
{% endcode %}

## 2. Agrega la dependencia (elige una)

{% tabs %}
{% tab title="Xcode (manual)" %}
1. Selecciona el target de tu app → *General* → *Frameworks, Libraries, and
   Embedded Content*.
2. Agrega `Vendor/Unicus/Frameworks/UnicusSDK.xcframework` y el framework del
   motor de verificación (el que **no** tiene `ForDevelopment` en el nombre).
3. Configura ambos como **Embed & Sign**.

Incluir también el framework `ForDevelopment` rompe la compilación.
{% endtab %}

{% tab title="CocoaPods" %}
{% code overflow="wrap" %}
```ruby
platform :ios, '15.0'

target 'TuApp' do
  use_frameworks!
  pod 'UnicusSDK', :path => 'Vendor/Unicus'
end
```
{% endcode %}

Luego ejecuta `pod install` y abre el `.xcworkspace`.
{% endtab %}

{% tab title="Swift Package Manager" %}
1. *File* → *Add Package Dependencies…* → *Add Local…*
2. Elige la carpeta `Vendor/Unicus` (contiene `Package.swift`).
3. Vincula el producto `UnicusSDK` al target de tu app.

El producto trae ambos frameworks binarios.
{% endtab %}
{% endtabs %}

## 3. Info.plist

| Clave | Obligatoria | Uso |
| --- | --- | --- |
| `NSCameraUsageDescription` | Sí | Captura de rostro y documento. Sin ella iOS cierra la app al abrir la cámara. |
| `NSLocationWhenInUseUsageDescription` | No | Ubicación como metadato de la transacción. Sin la clave el SDK omite la ubicación y continúa. |

{% code overflow="wrap" %}
```xml
<key>NSCameraUsageDescription</key>
<string>Usamos la cámara para verificar tu identidad.</string>
<key>NSLocationWhenInUseUsageDescription</key>
<string>Registramos dónde inicia la verificación.</string>
```
{% endcode %}

Si tu `Info.plist` declara `WKAppBoundDomains`, agrega el dominio de la app de
flujos de tu ambiente (DEV: `dev-id.idunicus.com`); de lo contrario `start`
falla con `webview_domain_not_allowed`. Las apps sin esa clave no necesitan nada.

No se requieren cambios en `AppDelegate`, `SceneDelegate` ni en otros archivos
nativos.

## Compilaciones Debug en un iPhone físico

El motor de verificación viene como binario de producción y binario de
desarrollo (el framework cuyo nombre termina en `ForDevelopment`). Las
**compilaciones Debug instaladas en un iPhone físico** deben usar el binario de
desarrollo. El script incluido en el paquete lo reemplaza después del paso de
inclusión; no hace nada en compilaciones Release, en el simulador ni en
compilaciones sin firma.

1. Selecciona el target de tu app → *Build Phases* → **+** → *New Run Script
   Phase*.
2. Muévela **después** de *Embed Frameworks* (con CocoaPods: después de
   *[CP] Embed Pods Frameworks*).
3. Shell `/bin/bash`; desmarca *Based on dependency analysis*.
4. Script (usa el nombre exacto del framework `ForDevelopment` de tu carpeta
   `Frameworks`):

{% code overflow="wrap" %}
```bash
bash "${SRCROOT}/Vendor/Unicus/swap_facetec_development_framework.sh" \
     "${SRCROOT}/Vendor/Unicus/Frameworks/<Unicus verification engine>ForDevelopment.xcframework"
```
{% endcode %}

{% hint style="warning" %}
Si tu proyecto activa *User Script Sandboxing* (`ENABLE_USER_SCRIPT_SANDBOXING`),
la fase no puede modificar el bundle de la app. Configúralo en `No` para el
target de la app, como hace el ejemplo incluido.
{% endhint %}

Las compilaciones Release, TestFlight y App Store conservan el binario de
producción.

## Ejecuta el ejemplo incluido

Antes de tocar tu app, ejecuta el ejemplo para validar el token, el dispositivo
y los permisos:

1. Abre `example/UnicusSDKExample.xcodeproj`.
2. Completa `UNICUS_API_KEY` en
   `example/UnicusSDKExample/Config/Unicus.xcconfig` con tu Customer Token de DEV.
3. Selecciona tu equipo en *Signing & Capabilities* y ejecútalo en un iPhone
   físico.

El botón *Inicio rápido* ejecuta la integración mínima (`QuickStart.swift`). El
resto del ejemplo permite probar todas las opciones y muestra los logs
sanitizados de la API.

## Versiones mínimas

| Elemento | Mínimo |
| --- | --- |
| Deployment target de iOS | 15.0 |
| Xcode | 16 |
| Swift | 5.9 (el modo de lenguaje Swift 6 compila; consulta [Pasos de interfaz propios](custom-ui-steps.md#hilos) para el código del proveedor) |
| Arquitecturas | Dispositivos arm64; simuladores arm64 y x86\_64 (solo compilación) |

No se requiere nada más: ningún permiso adicional para las pantallas del flujo
(usan el WebKit del sistema) y ninguna llave que solicitar. Herramientas de
ofuscación o protección de apps: consulta
[Compatibilidad y seguridad](../compatibility-and-security.md).

## Siguiente paso

[Configuración](configuration.md).
