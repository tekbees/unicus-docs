---
description: >-
  Qué puedes hacer en el portal administrativo de Unicus, cómo está organizado
  y quién ve cada sección.
---

# Descripción general

El portal administrativo es donde tu empresa configura Unicus y consulta los
resultados de las verificaciones. Desde ahí:

* compones los **flujos** que ve la persona (consentimiento, prueba de vida,
  documento, firma…) y los asignas a cada tipo de transacción;
* obtienes las **credenciales** de integración (ID de la compañía, API keys) y
  configuras el **webhook**;
* revisas cada **enrolamiento, verificación y prueba de vida**, con su
  constancia en PDF;
* envías **Unicus Link** a una persona por WhatsApp o SMS;
* administras a tu **equipo**: usuarios y roles, y su acceso con el inicio de sesión de tu empresa (SSO).

## Cómo entrar

Tu empresa recibe acceso cuando Tekbees aprueba su registro (ver
[Registro de tu empresa](sign-up.md)). Inicia sesión en el
[portal administrativo](https://app.idunicus.com/) o en la dirección propia de
tu empresa, si Tekbees te asignó una (por ejemplo `tuempresa.idunicus.com`).
Consulta [Acceso y tu cuenta](access-and-account.md).

{% hint style="info" %}
El portal se usa en español o inglés, en modo claro u oscuro. Cámbialo en el
menú de tu perfil (abajo a la izquierda). Las fechas y horas se muestran en la
zona horaria de tu navegador.
{% endhint %}

## Cómo está organizado

| Grupo del menú | Secciones | Para qué |
| --- | --- | --- |
| — | Dashboard, Analítica | Las cifras de tus verificaciones en el periodo. Ver [Panel de control](dashboard.md). |
| Operación | Enrolamientos, Cola de revisión, Verificaciones, Pruebas de vida, Validación de edad, Reportes | El trabajo diario con las transacciones. Ver [Transacciones y reportes](transactions.md). |
| Herramientas | Unicus Link, Validación de correo, Verificar constancia | Enviar enlaces de verificación, revisar validaciones de correo y comprobar que una constancia es auténtica. |
| Plataforma | Flujos, Webhooks | Qué hace la persona en cada verificación y el registro de entregas de tu webhook. Ver [Flujos de verificación](flows.md) y [Integración](integration.md). |
| Administración | Usuarios, Roles, Auditoría | Quién accede, qué puede hacer y el registro de lo que hizo cada uno. Ver [Usuarios y roles](users-roles-and-approvals.md). |
| Configuración | Compañía, Países y documentos | Marca, biometría, integración y seguridad. Ver [Configuración de la compañía](company-settings.md). |

Con **Ctrl + K** (o **Cmd + K** en Mac) abres la búsqueda rápida: busca una
transacción por su TID, el número de documento o el nombre de la persona.

## Quién ve qué

Cada usuario tiene un **rol**, y cada rol tiene permisos por servicio: ver,
crear, editar y eliminar. Una sección aparece con un candado y el mensaje "No
incluido en tu plan" cuando tu empresa no tiene ese servicio contratado o tu
rol no tiene el permiso; esa sección no se puede abrir.

Algunas acciones son solo para los **administradores** de la empresa, sin
importar los permisos de un rol personalizado: la seguridad de acceso y el
inicio de sesión único (SSO), las API keys, la firma y el registro de entregas del webhook, la
dirección y el color del portal, copiar o regenerar el ID de la compañía, y
asignar el rol de administrador o editar a un administrador.

## Primeros pasos

Mientras tu empresa completa su configuración, el Dashboard muestra la guía
**Primeros pasos**:

1. Personaliza tu marca (logo y colores).
2. Asigna un flujo a cada tipo de transacción que vayas a usar.
3. Conecta tu webhook y activa su firma.
4. Crea una API key para tu backend.
5. Invita a tu equipo.
6. Haz tu primera verificación, por ejemplo con un Unicus Link.

Después sigue la [Lista de integración](../sdk-web-v5/integration-checklist.md)
del Web SDK.
