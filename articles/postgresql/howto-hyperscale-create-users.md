---
title: Creación de usuarios de Hiperescala (Citus) en Azure Database for PostgreSQL
description: En este artículo se describe cómo puede crear nuevas cuentas de usuario para interactuar con Azure Database for PostgreSQL - Hiperescala (Citus).
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: conceptual
ms.date: 1/8/2019
ms.openlocfilehash: 684116f92544e61a892b3653f8539f9f8f03e0c9
ms.sourcegitcommit: b9d4b8ace55818fcb8e3aa58d193c03c7f6aa4f1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2020
ms.locfileid: "82584087"
---
# <a name="create-users-in-azure-database-for-postgresql---hyperscale-citus"></a>Crear usuarios en Azure Database for PostgreSQL - Hiperescala (Citus)

> [!NOTE]
> El término "usuarios" hace referencia a los usuarios de un grupo de servidores de Hiperescala (Citus). Para obtener información sobre los usuarios de la suscripción Azure y sus privilegios, visite el artículo [sobre el control de acceso basado en el rol (RBAC) de Azure](../role-based-access-control/built-in-roles.md) o revise [cómo personalizar los roles](../role-based-access-control/custom-roles.md).

## <a name="the-server-admin-account"></a>La cuenta de administrador del servidor

El motor de PostgreSQL usa [roles](https://www.postgresql.org/docs/current/sql-createrole.html) para controlar el acceso a los objetos de base de datos y un grupo de servidores de Hiperescala (Citus) recién creado incluye varios roles predefinidos:

* [Roles predeterminados de PostgreSQL](https://www.postgresql.org/docs/current/default-roles.html)
* `azure_pg_admin`
* `postgres`
* `citus`

Dado que Hiperescala es un servicio PaaS administrado, solo Microsoft puede iniciar sesión con el rol de superusuario `postgres`. En el caso de acceso de administrador limitado, Hiperescala proporciona el rol `citus`.

Permisos para el rol de `citus`:

* Leer todas las variables de configuración, incluso las variables que normalmente solo son visibles para los superusuarios.
* Leer todas las vistas de pg\_stat\_\* y usar varias extensiones relacionadas con estadísticas, incluso las vistas o extensiones que normalmente solo son visibles para los superusuarios.
* Ejecutar funciones de supervisión que puedan tener bloqueos de uso compartido de acceso en tablas, potencialmente durante mucho tiempo.
* [Crear extensiones de PostgreSQL](concepts-hyperscale-extensions.md) (porque el rol es miembro de `azure_pg_admin`).

En particular, el rol `citus` tiene algunas restricciones:

* No puede crear roles.
* No puede crear bases de datos.

## <a name="how-to-create-additional-user-roles"></a>Cómo crear roles de usuario adicionales

Como se ha mencionado, la cuenta de administrador `citus` carece de permiso para crear usuarios adicionales. Para agregar un usuario, use la interfaz de Azure Portal.

1. Vaya a la página de **Roles** del grupo Servidor de hiperescala y haga clic en **+ Agregar**:

   ![Página de roles](media/howto-hyperscale-create-users/1-role-page.png)

2. Introduzca el nombre y la contraseña del rol. Haga clic en **Save**(Guardar).

   ![Agregar rol](media/howto-hyperscale-create-users/2-add-user-fields.png)

El usuario se creará en el nodo coordinador del grupo de servidores y se propagará a todos los nodos de trabajo. Los roles creados a través de Azure Portal tienen el atributo `LOGIN`, lo que significa que son verdaderos usuarios que pueden iniciar sesión en la base de datos.

## <a name="how-to-modify-privileges-for-user-role"></a>Cómo modificar los privilegios para el rol de usuario

Los nuevos roles se usan comúnmente para proporcionar acceso a la base de datos con privilegios restringidos. Para modificar los privilegios de usuario, use los comandos estándar de PostgreSQL con una herramienta como PgAdmin o psql. (Consulte [conectarse con psql](quickstart-create-hyperscale-portal.md#connect-to-the-database-using-psql) en el inicio rápido de la Hiperescala (Citus).)

Por ejemplo, para permitir que `db_user` lea `mytable`, conceda el permiso:

```sql
GRANT SELECT ON mytable TO db_user;
```

Hiperescala (Citus) propaga las instrucciones GRANT de tabla única a través de todo el clúster y las aplica a todos los nodos de trabajo. Sin embargo, las instrucciones GRANT que abarcan todo el sistema (por ejemplo, todas las tablas de un esquema) deben ejecutarse en cada nodo de fecha.  Use la función auxiliar `run_command_on_workers()`:

```sql
-- applies to the coordinator node
GRANT SELECT ON ALL TABLES IN SCHEMA public TO db_user;

-- make it apply to workers as well
SELECT run_command_on_workers(
  'GRANT SELECT ON ALL TABLES IN SCHEMA public TO db_user;'
);
```

## <a name="how-to-delete-a-user-role-or-change-their-password"></a>Cómo eliminar un rol de usuario o cambiar su contraseña

Para actualizar un usuario, visite la página **Roles** del grupo de servidores de Hiperescala y haga clic en los puntos suspensivos **...**  junto al usuario. Los puntos suspensivos abrirán un menú para eliminar el usuario o restablecer su contraseña.

   ![Editar un rol](media/howto-hyperscale-create-users/edit-role.png)

El rol `citus` tiene privilegios y no se puede eliminar.

## <a name="next-steps"></a>Pasos siguientes

Abra el firewall para las direcciones IP de las máquinas de los nuevos usuarios para permitirles conectarse: [Creación y administración de las reglas de firewall en Hiperescala (Citus) con Azure Portal](howto-hyperscale-manage-firewall-using-portal.md).

Para más información sobre la gestión de cuentas de usuario de la base de datos, consulte la documentación del producto PostgreSQL:

* [Roles y privilegios de la base de datos](https://www.postgresql.org/docs/current/static/user-manag.html)
* [Sintaxis GRANT](https://www.postgresql.org/docs/current/static/sql-grant.html)
* [Privilegios](https://www.postgresql.org/docs/current/static/ddl-priv.html)
