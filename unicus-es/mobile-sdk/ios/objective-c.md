---
description: >-
  Usa el SDK de Unicus para iOS desde Objective-C con la fachada UnicusSdkObjC:
  configurar, iniciar, reanudar, cancelar y leer el resultado.
---

# Objective-C

Las apps escritas en Objective-C usan `UnicusSdkObjC`, una fachada sobre el SDK
de Swift. Cubre el camino habitual: configurar, iniciar una verificación para un
documento, reanudar por `tid`, cancelar y borrar los datos de reanudación. La
instalación es la misma que para Swift ([Instalación](installation.md)).

Los pasos de UI personalizados, los eventos de progreso y los logs de la API
solo están disponibles en Swift. Si los necesitas, agrega a tu target un archivo
Swift pequeño que use `UnicusSdk` directamente.

## Importar

{% code overflow="wrap" %}
```objc
@import UnicusSDK;   // módulos activados (valor por defecto de Xcode)
```
{% endcode %}

## Configurar

{% code overflow="wrap" %}
```objc
NSError *error = [UnicusSdkObjC.shared configureWithApiKey:@"<CUSTOMER_TOKEN>" environment:@"dev"];
if (error) {
    // Nombre de ambiente desconocido: no se configuró nada.
    NSLog(@"Unicus: %@", error.userInfo[UnicusSdkObjC.errorCodeKey]);   // invalid_configuration
}
```
{% endcode %}

`environment` acepta `dev`, `staging` o `production` (en mayúsculas o
minúsculas). Igual que en Swift, hoy solo está disponible `dev`; los demás hacen
que `start` falle con `environment_not_available`.

## Iniciar una verificación

{% code overflow="wrap" %}
```objc
[UnicusSdkObjC.shared startWithDocumentType:@"ID"
                             documentNumber:documentNumber
                                  presenter:self
                                 completion:^(UnicusResultObjC *result, NSError *error) {
    if (error) {
        // La verificación no se pudo ejecutar.
        NSString *code = error.userInfo[UnicusSdkObjC.errorCodeKey];
        NSLog(@"Unicus error %@ (%ld)", code, (long)error.code);
        return;
    }
    if (result.success) {
        // Verificada: confirma result.tid en tu backend.
    } else if (result.resumable) {
        // 2003: inicia de nuevo con el mismo documento, o reanuda con result.tid.
    } else if (result.retryable) {
        // Falla temporal: ofrece intentar de nuevo.
    } else {
        // No verificada: result.outcome, result.resultCode.
    }
}];
```
{% endcode %}

* `documentType`: `ID`, `FD`, `PP` o `DL`. Otro valor termina con el código de
  error `invalid_request`.
* `presenter` puede ser `nil`: el SDK usa el view controller superior.
* Exactamente uno de `result` / `error` no es nil. El completion se ejecuta en
  la cola principal.

## Reanudar por `tid`

{% code overflow="wrap" %}
```objc
[UnicusSdkObjC.shared startExistingTransaction:savedTid
                                     presenter:self
                                    completion:^(UnicusResultObjC *result, NSError *error) {
    // mismo manejo que arriba
}];
```
{% endcode %}

La reanudación automática con el mismo documento funciona igual que en Swift
(consulta [Resultados y reanudación](../results-and-resuming.md)).

## Cancelar y cerrar sesión

{% code overflow="wrap" %}
```objc
[UnicusSdkObjC.shared cancel];            // cancela la verificación activa
[UnicusSdkObjC.shared clearResumeData];   // al cerrar sesión
```
{% endcode %}

## `UnicusResultObjC`

| Propiedad | Tipo | Significado |
| --- | --- | --- |
| `success` | `BOOL` | Identidad verificada. |
| `outcome` | `NSString` | `success`, `warning`, `failed`, `canceled`, `error`, `resumable`, `unknown`. |
| `resultCode` | `NSNumber *` (nullable) | Código de resultado de Unicus. Consulta [Códigos de resultado](../result-codes.md). |
| `resultMessage` | `NSString *` (nullable) | Mensaje del código, para logs. |
| `tid` | `NSString *` (nullable) | Id de la transacción. |
| `resumable` | `BOOL` | 2003: la transacción está abierta. |
| `retryable` | `BOOL` | Falla temporal: inicia de nuevo. |
| `dictionary` | `NSDictionary *` | Todos los campos del resultado de Swift (`steps`, `flowId`, `rejectionReason`…). |

## Errores (`NSError`)

| Parte | Valor |
| --- | --- |
| `domain` | `com.tekbees.unicus.sdk` (`UnicusSdkObjC.errorDomain`) |
| `code` | `resultCode` de Unicus cuando el servidor envió uno; si no, `-1`. |
| `userInfo[UnicusSdkObjC.errorCodeKey]` | Código de error estable (`not_configured`, `network_error`, `flow_not_supported`…). Decide según este valor. |
| `userInfo[UnicusSdkObjC.httpStatusKey]` | Estado HTTP, cuando existe. |
| `userInfo[@"UnicusResultMessage"]` | `resultMessage` de Unicus, cuando existe. |
| `localizedDescription` | Texto para desarrolladores; no lo muestres al usuario final. |

Todos los códigos: [Errores y solución de problemas](../errors-and-troubleshooting.md).

## Otros miembros

| Miembro | Uso |
| --- | --- |
| `UnicusSdkObjC.shared.version` | Versión del SDK. |
| `UnicusSdkObjC.shared.isSessionActive` | `YES` mientras una verificación se prepara o se muestra. |
| `configureWithApiKey:baseUrl:` | Avanzado: URL base explícita de la API. Prefiere `configureWithApiKey:environment:`. |
