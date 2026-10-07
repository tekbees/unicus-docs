---
description: Política de versionado y notas de versión de Unicus Web SDK 5.0.
---

# Versionado

{% hint style="info" %}
[English version](../versioning.md)
{% endhint %}

{% hint style="warning" %}
**Próximamente.** Web SDK 5.0 aún no está disponible en producción. Tekbees
anunciará la fecha de lanzamiento. En esa fecha Web SDK 4.x deja de funcionar y toda
integración ejecuta 5.0, incluidas las páginas que todavía cargan la URL actual del script.
Mientras tanto, solicita a Tekbees acceso al ambiente de pruebas (sandbox) para prepararte.
{% endhint %}

El script se publica bajo una ruta de versión mayor:

| Ruta | Recibe |
| --- | --- |
| `https://unicusbtn.idunicus.com/v5/sdkButton.js` | Cada actualización 5.x compatible (corrección de errores, nuevos atributos opcionales, nuevos eventos). Los navegadores pueden conservar la copia anterior durante unos minutos; las páginas toman la actualización en una carga posterior sin que debas hacer nada. |
| `https://unicusbtn.idunicus.com/sdkButton.js` | La dirección de las integraciones 4.x. Desde la fecha de lanzamiento de 5.0 sirve el mismo script que `/v5/`, por lo que las páginas existentes pasan a 5.0 sin cambios. |
| `https://unicusbtn.idunicus.com/v6/…` (futuro) | Cambios incompatibles. Se anuncian con anticipación; la ruta anterior sigue funcionando durante la transición. |

`customElements.get('unicus-btn').version` devuelve la versión exacta cargada
(inclúyela cuando contactes a [Soporte](support.md)).

## 5.0.0

* Nueva aplicación web: flujos modulares compuestos en el portal administrativo
  (consentimiento, información, prueba de vida, documento con validaciones en servidor, comparación facial,
  firma, OTP, formulario, verificación de edad).
* Inicio liviano; motor biométrico precargado en segundo plano y almacenado en caché;
  diseñado para redes móviles de bajo ancho de banda.
* Traspaso al celular (hand-off) desde el computador por QR, WhatsApp o SMS, con un espejo en vivo del
  progreso en el celular y el resultado final en el computador.
* Tokens de traspaso de un solo uso en el fragmento de la URL; el id de la transacción nunca
  viaja en un enlace.
* Rediseño del botón: una sola pieza con la marca Unicus, colores de marca aplicados
  automáticamente y recordados, atributos `label`, `size` y `radius`, estados visibles, clic durante la carga respetado, nueva
  transacción después de cada flujo finalizado o abandonado.
* Eventos: payloads `stepProgress` por paso, `resultCode` en `finished` y
  `error`, `2002` *no configurado*, `2013` *en revisión manual*. Los eventos se propagan (bubble) y
  son `composed`, así que un listener en `document` los recibe.
* Textos por flujo en español e inglés; selector de idioma en las pantallas
  de los pasos; atributo `label` en el botón.
* Seguridad: sesiones de corta duración ligadas a una transacción, mensajería con verificación de origen,
  permisos estrictos del iframe, sin filtración del referrer.
* Contrato público (atributos, eventos, `transactionId`) compatible con 4.x.
* Reemplaza a 4.x en toda integración en la fecha de lanzamiento; la URL del script 4.x
  sirve 5.0 desde entonces.
