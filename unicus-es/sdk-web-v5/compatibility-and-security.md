---
description: >-
  Requisitos de navegador, Content Security Policy, permisos y modelo de
  seguridad de Web SDK 5.0. Léelo antes de la salida a producción.
---

# Compatibilidad y seguridad

## Requisitos

| Requisito | Por qué |
| --- | --- |
| HTTPS | Los navegadores solo exponen la cámara a orígenes seguros, y los enlaces de traspaso al celular (hand-off) son solo HTTPS. Se acepta `http://localhost` para desarrollo. |
| JavaScript y custom elements | El botón es un web component. |
| Sin registro de dominio | Tu página puede servirse desde cualquier dominio. El Customer Token identifica a tu empresa; no necesitas enviar el dominio de tu sitio a Tekbees. |
| Iframe de terceros permitido | El flujo se ejecuta en un iframe del dominio de Unicus. |
| Permiso de cámara | Pasos de prueba de vida y de documento, en el celular o tableta que los ejecuta. Antes de abrir la cámara, la aplicación web solicita al navegador la ubicación, que se registra con la transacción; el usuario puede rechazarla y el flujo continúa sin ella. |
| Conexión estable durante la captura | Las cargas son pequeñas (aproximadamente 1.5 MB por enrolamiento), pero deben completarse. |

## Navegadores soportados

| Plataforma | Navegadores |
| --- | --- |
| Android | Chrome y Samsung Internet, versión mayor actual y anterior. |
| iOS / iPadOS | Safari 15 o posterior. Otros navegadores de iOS usan el mismo motor, pero pueden manejar los permisos de cámara de forma diferente; valida con Safari. |
| Escritorio | Versiones actuales de Chrome, Edge, Firefox y Safari. Un computador de escritorio o portátil nunca ejecuta los pasos de cámara, aunque tenga cámara web: ejecuta los pasos anteriores al primer paso de cámara y traspasa el resto a un celular (QR, WhatsApp o SMS). |
| Navegadores integrados en apps (webviews) | Soportados cuando la app anfitriona otorga permiso de cámara al webview. Facebook, Instagram y algunos navegadores integrados de apps bancarias no lo hacen; se le indica al usuario que abra el enlace en el navegador del sistema. |

## Content Security Policy

Si tu página define una CSP, permite:

| Directiva | Valor |
| --- | --- |
| `script-src` | `https://unicusbtn.idunicus.com` |
| `frame-src` (o `child-src`) | `https://id.idunicus.com` |
| `connect-src` | `https://unicusapi.idunicus.com` (creación de la transacción desde el botón) |

{% code overflow="wrap" %}
```
Content-Security-Policy: script-src 'self' https://unicusbtn.idunicus.com; frame-src https://id.idunicus.com; connect-src 'self' https://unicusapi.idunicus.com
```
{% endcode %}

No se necesita ninguna entrada `img-src` ni `style-src`: el botón dibuja su
marca como elementos SVG y se aplica estilos con una constructable stylesheet,
que una `style-src` estricta no bloquea. Los navegadores sin constructable
stylesheets (por ejemplo, Safari anterior a 16.4) recurren a un elemento
`<style>` dentro del botón, que necesita `style-src 'unsafe-inline'`; sin ella,
el botón sigue funcionando, pero allí se muestra sin estilos. El botón crea sus
elementos solo con llamadas DOM, por lo que también funciona en páginas que
exigen Trusted Types.

Los ambientes de pruebas (sandbox) usan otros dominios proporcionados por
Tekbees. El iframe de verificación tiene su propia política; tu página solo
necesita las entradas anteriores.

Si tu página define `Referrer-Policy: no-referrer`, la verificación sigue
funcionando: conoce el origen de tu página mediante un handshake con
verificación de origen con el botón.

## Permissions Policy

El botón crea el iframe con
`allow="camera; microphone; geolocation; fullscreen"`, lo que otorga estas
funciones únicamente al origen de Unicus. Si tu sitio envía un encabezado
`Permissions-Policy`, este no debe negarlas al origen de Unicus, por ejemplo:

```
Permissions-Policy: camera=(self "https://id.idunicus.com"), geolocation=(self "https://id.idunicus.com")
```

## Modelo de seguridad

* **Ningún secreto en el navegador.** El Customer Token es un identificador
  público de tu empresa. Las transacciones se autorizan mediante una sesión de
  corta duración emitida por Unicus y vinculada a una transacción; nunca está en
  una URL.
* **Ningún id de transacción en los enlaces.** Los enlaces de QR, SMS y WhatsApp
  llevan un token de un solo uso en el fragmento de la URL, que los navegadores
  nunca envían a los servidores. La aplicación web no envía `Referer`.
* **Mensajería con verificación de origen.** La aplicación web solo se comunica
  con la página que la embebió (origen conocido durante el handshake) y el botón
  solo acepta mensajes del iframe que creó.
* **Autoridad del servidor.** El orden de los pasos, su finalización y los
  resultados los decide Unicus, no el navegador.
* **Los datos biométricos** nunca llegan a la página del cliente. Los eventos
  solo llevan códigos de resultado y metadatos. Las imágenes y plantillas las
  procesa Unicus según el acuerdo de tratamiento de datos de tu empresa.
* **Abandono.** Cerrar o recargar la verificación nunca cancela la
  transacción: el avance queda en Unicus, así que el usuario puede retomarla
  (una recarga continúa en el primer paso pendiente; en un computador, el
  usuario puede enviar un enlace nuevo al celular). Solo una sesión de cámara
  cancelada por el usuario en el dispositivo que la ejecuta termina la
  transacción como cancelada (`2041`); un computador que solo refleja el
  celular no puede cancelar. Una transacción que nadie retoma expira.
* **Almacenamiento local.** El botón guarda los colores de la empresa en
  `localStorage` (clave `unicus-btn:brand:<Customer Token>`) para pintar el
  color de marca en el primer frame de visitas posteriores. No se almacenan
  datos personales.

## Consumo de datos

| Elemento | Tamaño | Cuándo |
| --- | --- | --- |
| Script del botón | unos pocos KB | Una vez por carga de página; el navegador lo guarda en caché. |
| Pantallas de verificación | decenas de KB | Cuando se abre el flujo; en caché. |
| Motor biométrico | varios MB | Una vez por dispositivo; se descarga en segundo plano durante las primeras pantallas y queda en caché para transacciones posteriores. |
| Cargas | aproximadamente 0.5 MB por selfie y por lado del documento | Durante el flujo. |

## Lista de verificación previa a producción

1. Página servida por HTTPS.
2. `https://unicusbtn.idunicus.com/v5/sdkButton.js` carga; el botón cambia de
   neutro al color de tu marca.
3. `OnUnicus:loaded` llega con un `tid`.
4. Un enrolamiento completo desde un celular y otro desde un computador con
   traspaso al celular.
5. Una verificación de una persona ya enrolada.
6. `OnUnicus:finished` llega a la página y la transacción aparece en el portal
   y en tu webhook o en `query-transaction`.
7. Ruta de error: renderiza el botón con `clientid="ID:"` (sin número) y
   confirma que tu página muestra un mensaje recuperable en `OnUnicus:error`.
8. Cierra la verificación antes de terminar y confirma que tu página maneja
   `OnUnicus:exit`; haz clic de nuevo y confirma que llega un nuevo `tid` en
   `OnUnicus:loaded`.
