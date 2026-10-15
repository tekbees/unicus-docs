---
description: >-
  Invita a tu equipo, define qué puede hacer cada rol, aprueba cambios
  sensibles y consulta el registro de auditoría.
---

# Usuarios, roles y aprobaciones

## Usuarios

**Administración → Usuarios** lista a las personas de tu empresa con su rol,
su estado (activo, pendiente o inactivo) y su último acceso.

* **Crear usuario:** correo, nombres, apellidos y rol. La persona recibe una
  invitación por correo y crea su propia contraseña (el enlace vence en 72
  horas). Un correo solo puede pertenecer a una cuenta de Unicus; el de un
  usuario eliminado de tu empresa se puede volver a usar.
* **Editar:** sus datos, su rol y su acceso. En **Seguridad** puedes enviarle
  el correo para restablecer la contraseña (el enlace vence en 24 horas y
  sirve una vez).
* **Eliminar:** la persona pierde el acceso. El administrador principal de la
  empresa no se puede editar ni eliminar, y nadie puede eliminarse a sí mismo.

Solo el administrador de la compañía da el rol de administrador o edita a un
administrador. Desactivar a un usuario o cambiarle el rol cierra sus sesiones
de inmediato.

**Correo de inicio de sesión:** nadie cambia el suyo propio. El de un
administrador lo cambia otro administrador o Tekbees, y el del administrador
principal, solo Tekbees. Al cambiarlo se cierran todas las sesiones de la
persona y se avisa a la dirección anterior.

## Roles

**Administración → Roles** define qué puede hacer cada grupo de personas. Cada
rol tiene permisos por servicio (enrolamiento, verificación, prueba de vida,
datos personales, reportes, Dashboard y analítica, Auditoría, flujos,
usuarios, roles, Unicus Link…) con las acciones **ver, crear, editar y
eliminar** que admite cada servicio.

Cada empresa tiene cuatro roles **predeterminados**, iguales en todas las
compañías:

| Rol | Qué puede hacer |
| --- | --- |
| **Operador** | Consulta transacciones y reportes, envía Unicus Link. |
| **Revisor de fraude** | Lo del Operador, más los datos personales y la decisión de duplicados. |
| **Consulta** | Solo lectura de transacciones, reportes y Dashboard. |
| **Integrador** | Flujos, configuración e integración (sin API keys). |

Los roles predeterminados no se editan ni se eliminan: si necesitas una
variante, usa **Duplicar** y ajusta la copia.

* El rol de administrador está protegido.
* Un rol solo puede tener los servicios que tu empresa tiene contratados.
* Solo puedes dar los permisos que tú tienes.
* No puedes modificar tu propio rol.
* Un rol que tienen usuarios no se puede eliminar: primero asígnales otro.
* Algunos nombres están reservados (por ejemplo, los que se leen como el de
  administrador).
* Hay acciones reservadas a los administradores, sin importar el rol (ver
  [Descripción general](overview.md)).

{% hint style="info" %}
Da a cada rol solo lo que necesita. Por ejemplo, un equipo de operación puede
revisar transacciones sin el permiso **Ver datos personales**: verá el nombre,
el número de documento y los resultados, pero no las fotos ni los demás datos
personales.
{% endhint %}

## Aprobaciones

Si tu empresa activó la **aprobación de una segunda persona** (en
[Seguridad](company-settings.md#seguridad)), estas acciones necesitan que otro
administrador las apruebe:

* publicar un flujo;
* crear una API key de producción.

**Administración → Aprobaciones** lista las solicitudes pendientes, aprobadas,
rechazadas y retiradas. Nadie aprueba su propia solicitud; un rechazo lleva un
motivo. Una solicitud aprobada le permite a quien la pidió hacer el cambio una
vez, dentro de las 24 horas siguientes.

## Auditoría

**Administración → Auditoría** registra lo que hacen los usuarios de tu
empresa en el portal: inicios de sesión, cambios en flujos, usuarios, roles,
API keys y configuración, cada copia del ID de la compañía y cada consulta
completa de datos personales. Los registros no se pueden editar ni eliminar.

Filtra por usuario, recurso y acción, y exporta el resultado en CSV.
