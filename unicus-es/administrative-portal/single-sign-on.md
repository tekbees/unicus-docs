---
description: >-
  Tu equipo entra al portal con la cuenta de la empresa (Microsoft Entra ID,
  Google Workspace u Okta) y sus usuarios se crean y desactivan solos (SCIM).
---

# Inicio de sesión único (SSO)

Con el inicio de sesión único, tu equipo entra al portal con la cuenta de la
empresa en lugar de una contraseña de Unicus. Tu proveedor de identidad (Entra
ID, Google Workspace u Okta) confirma quién es cada persona y aplica sus propias
reglas: doble factor, bloqueo de cuentas, horarios.

Lo configura un **administrador de tu empresa** en
**Configuración → Compañía → Seguridad**, junto con quien administra el
proveedor de identidad. Se usa el estándar OpenID Connect (OIDC).

{% hint style="info" %}
El SSO no crea usuarios por sí solo: solo entran las personas que ya tienen
usuario en tu empresa en Unicus. Créalas en [Usuarios](users-roles-and-approvals.md)
o deja que tu proveedor las cree con el
[aprovisionamiento automático (SCIM)](#aprovisionamiento-automatico-scim).
{% endhint %}

## Antes de empezar

* El **dominio** o los dominios de correo de tu empresa (por ejemplo
  `tuempresa.com`). Un dominio solo puede estar en una empresa de Unicus, y no se
  aceptan dominios de correo gratuito (gmail.com, outlook.com…).
* La **URL de retorno** que muestra el portal en **Seguridad → Inicio de sesión
  único (SSO)**, por ejemplo `https://app.idunicus.com/sso/callback`. Si tu
  equipo entra por la dirección propia de tu empresa
  (`tuempresa.idunicus.com`), registra también
  `https://tuempresa.idunicus.com/sso/callback`.
* Acceso de administrador a tu proveedor de identidad.

## Microsoft Entra ID (Azure AD)

1. En el [centro de administración de Entra](https://entra.microsoft.com), ve a
   **Identidad → Aplicaciones → Registros de aplicaciones → Nuevo registro**.
   * **Tipos de cuenta:** solo las cuentas de este directorio (un solo
     inquilino).
   * **URI de redirección:** plataforma **Web**, con la URL de retorno del
     portal.
2. En **Certificados y secretos**, crea un **secreto de cliente** y copia su
   valor (solo se muestra una vez).
3. En **Configuración de token**, agrega la notificación opcional **email** al
   token de ID. Si un usuario no tiene correo en Entra, Unicus usa su nombre de
   inicio de sesión (UPN) cuando tiene forma de correo.
4. Copia de **Información general** el **Id. de aplicación (cliente)** y el
   **Id. de directorio (inquilino)**.
5. En el portal, en **Seguridad → Inicio de sesión único (SSO)**:
   * **Emisor (issuer):** `https://login.microsoftonline.com/<id-de-inquilino>/v2.0`
   * **Client ID** y **client secret:** los de los pasos 2 y 4.
   * **Dominios:** los de tu empresa.

Unicus solo acepta tokens de ese inquilino; una dirección `common` u
`organizations` no se puede guardar.

## Google Workspace

1. En la [consola de Google Cloud](https://console.cloud.google.com), con un
   proyecto de tu organización, abre **APIs y servicios → Pantalla de
   consentimiento de OAuth** y elige el tipo **Interno**: así solo entran
   cuentas de tu Workspace.
2. En **Credenciales → Crear credenciales → ID de cliente de OAuth**, elige
   **Aplicación web** y agrega la URL de retorno del portal en **URI de
   redireccionamiento autorizados**.
3. Copia el **ID de cliente** y el **secreto del cliente**.
4. En el portal:
   * **Emisor (issuer):** `https://accounts.google.com`
   * **Client ID** y **client secret:** los del paso 3.
   * **Dominios:** los de tu Workspace.

Unicus comprueba que la cuenta pertenezca al Workspace de ese dominio: una cuenta
personal de Google creada con un correo de la empresa no entra.

{% hint style="warning" %}
Google Workspace no envía usuarios por SCIM a aplicaciones como Unicus. Con
Google, crea los usuarios en el portal; después entran con su cuenta de Google.
{% endhint %}

## Okta

1. En la consola de Okta, ve a **Applications → Create App Integration**, elige
   **OIDC - OpenID Connect** y **Web Application**.
2. **Sign-in redirect URIs:** la URL de retorno del portal. **Grant type:**
   Authorization Code.
3. Asigna la aplicación a las personas o grupos que deben entrar a Unicus.
4. Copia el **Client ID** y el **Client secret**.
5. En el portal:
   * **Emisor (issuer):** `https://<tu-organizacion>.okta.com` (o
     `https://<tu-organizacion>.okta.com/oauth2/default` si usas el servidor de
     autorización por defecto).
   * **Client ID**, **client secret** y **dominios**.

## Guardar y probar

Al guardar, el portal lee la configuración pública de tu proveedor en la URL del
emisor; si no la encuentra, te lo indica y no guarda nada. El secreto se guarda
cifrado y no se vuelve a mostrar: para cambiarlo, escribe uno nuevo; si lo dejas
vacío se conserva el actual.

Para probarlo, cierra sesión y en la pantalla de inicio escribe un correo de tu
dominio y elige **Entrar con SSO corporativo**.

### Hacerlo obligatorio

Activa **Obligatorio para estos dominios** cuando todo tu equipo ya entre con
SSO. Desde ese momento, la contraseña de Unicus deja de servir para los correos
de esos dominios y el portal les indica que usen el inicio de sesión
corporativo. Antes de activarlo, confirma que al menos un administrador entra
bien con SSO.

## Aprovisionamiento automático (SCIM)

Con SCIM, tu proveedor de identidad crea en Unicus a las personas que le
asignes y las desactiva cuando dejan la empresa o les quitas el acceso. Funciona
con Microsoft Entra ID y con Okta.

1. En el portal, en **Seguridad → Aprovisionamiento automático (SCIM)**, copia
   la **URL base de SCIM** y crea un **token** con un nombre que lo identifique
   (por ejemplo «Entra ID producción»). Cópialo: solo se muestra una vez.
2. En tu proveedor:
   * **Entra ID:** **Aplicaciones empresariales →** tu aplicación **→
     Aprovisionamiento →** modo **Automático**. **URL del inquilino:** la URL
     base de SCIM; **token secreto:** el token. Prueba la conexión y activa el
     aprovisionamiento.
   * **Okta:** en la aplicación, **General → Provisioning: SCIM**; en
     **Provisioning**, la URL base de SCIM como **SCIM connector base URL**,
     **Unique identifier field:** `userName`, autenticación **HTTP Header** con el
     token. Activa **Create Users**, **Update User Attributes** y
     **Deactivate Users**.

Lo que hace Unicus con cada cambio:

* **Alta:** crea el usuario en tu empresa con el rol **Operador** y sin
  contraseña (entra con SSO). Cambia su rol en [Usuarios](users-roles-and-approvals.md)
  si necesita otro.
* **Cambios de nombre o correo:** los aplica.
* **Baja o desactivación:** desactiva el usuario y cierra sus sesiones; nunca
  lo borra, y lo reactiva si vuelves a asignarlo.
* El **administrador principal** de tu empresa nunca se modifica por SCIM.

Revoca un token desde la misma sección si deja de usarse o crees que se
expuso; tu proveedor dejará de poder sincronizar hasta que le pongas uno nuevo.

## Si algo no funciona

| Mensaje en el portal | Qué revisar |
| --- | --- |
| No pudimos leer la configuración del proveedor | La URL del emisor: debe ser exacta, con `https://` y sin barra al final de más. |
| No se aceptan dominios de correo gratuito | Escribe el dominio de tu empresa, no uno de correo gratuito (gmail.com, outlook.com…). |
| El dominio ya está en uso | Ese dominio está configurado en otra empresa de Unicus: escríbenos. |
| Tu correo no está autorizado | El correo que entrega tu proveedor no es de los dominios configurados, o (Google) la cuenta no es de tu Workspace. |
| No hay un usuario con ese correo | La persona no tiene usuario activo en tu empresa en Unicus: créalo o asígnalo por SCIM. |
| Se canceló el inicio de sesión | La persona canceló en el proveedor, o tu proveedor no le dio acceso a la aplicación. |
| Tu empresa usa inicio de sesión corporativo | El SSO es obligatorio para ese dominio: usa **Entrar con SSO corporativo**. |
