---
description: >-
  La configuración de tu empresa en el portal: datos y logo, apariencia,
  biometría, integración, seguridad, y países y documentos.
---

# Configuración de la compañía

**Configuración → Compañía** reúne la configuración de tu empresa en cinco
secciones. Quien no tiene permiso para editarla la ve en modo de solo lectura.

## General

* **Datos de la compañía:** el nombre (es el que ve la persona en WhatsApp y
  SMS) y la identificación tributaria. Para cambiarlos, escríbele a
  [Soporte](../sdk-web-v5/support.md).
* **Logo:** PNG o JPG de hasta 2 MB. Aparece en la verificación, en los
  correos y en el portal.

## Apariencia

* **Apariencia de la captura:** el color de tu marca, el color del texto sobre
  ese color y el de los botones de la cámara. La vista previa muestra el
  inicio, la captura del rostro y la del documento, y te avisa si el contraste
  no se lee bien. Estos colores se aplican al botón y a la verificación (ver
  [Referencia del botón](../sdk-web-v5/button-reference.md)).
* **Tu portal** (administradores): el color del portal para tu equipo y, si
  Tekbees te la asignó, la dirección propia de tu portal para copiarla y
  compartirla.

## Biometría

**Nivel de coincidencia rostro/documento:** qué tan parecido debe ser el
rostro a la foto del documento para aprobar, desde **Ágil** hasta **Máximo**.
Un nivel más alto rechaza más suplantaciones y también más intentos legítimos
con fotos de mala calidad. El portal simula cómo habría funcionado cada nivel
con tus enrolamientos recientes antes de que guardes el cambio.

## Integración

Las credenciales (ID de la compañía y API keys), el webhook y su firma, y las
conexiones con otros servicios. Ver [Integración](integration.md).

## Seguridad

Solo para administradores:

* **Acceso solo desde las IPs de la compañía:** el portal solo acepta inicios
  de sesión desde las direcciones o rangos (CIDR) que agregues. Usa **Usar mi
  IP actual** para no quedarte por fuera; el portal no te deja guardar una
  lista que bloquearía tu propia sesión.
* **Aprobación de una segunda persona:** publicar un flujo y crear una API key
  de producción necesitan que otro administrador lo apruebe (ver
  [Aprobaciones](users-roles-and-approvals.md#aprobaciones)). Requiere al menos
  dos administradores activos.
* **Código por correo para todos:** además de la contraseña, todos los
  usuarios de tu empresa escriben un código que llega a su correo al iniciar
  sesión.

## Países y documentos

**Configuración → Países y documentos** muestra los países y los tipos de
documento que acepta tu empresa en las verificaciones. Los países activos
también definen a qué números de celular se pueden enviar SMS y WhatsApp. Para
agregar un país o un documento, escríbele a [Soporte](../sdk-web-v5/support.md).
