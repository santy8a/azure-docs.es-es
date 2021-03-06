---
title: 'Tutorial: Configuración de Azure Databricks SCIM Connector para el aprovisionamiento automático de usuarios con Azure Active Directory | Microsoft Docs'
description: Aprenda a aprovisionar y cancelar el aprovisionamiento de forma automática de las cuentas de usuario de Azure AD a Azure Databricks SCIM Connector.
services: active-directory
documentationcenter: ''
author: Zhchia
writer: Zhchia
manager: beatrizd
ms.assetid: b16eaf27-4bd1-4509-be2c-9a4f66b6badc
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/15/2020
ms.author: Zhchia
ms.openlocfilehash: fe1260982edc877c049716bd74f1bb3e90d33b0f
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/27/2020
ms.locfileid: "77370522"
---
# <a name="tutorial-configure-azure-databricks-scim-connector-for-automatic-user-provisioning"></a>Tutorial: Configuración de Azure Databricks SCIM Connector para el aprovisionamiento de usuarios automático

En este tutorial, se describen los pasos que debe realizar en Azure Databricks SCIM Connector y Azure Active Directory (Azure AD) para configurar el aprovisionamiento automático de usuarios. Cuando se configura, Azure AD aprovisiona y cancela el aprovisionamiento de usuarios y grupos de manera automática en [Azure Databricks SCIM Connector](https://databricks.com/) mediante el servicio de aprovisionamiento de Azure AD. Para obtener información importante acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../manage-apps/user-provisioning.md). 


## <a name="capabilities-supported"></a>Funcionalidades admitidas
> [!div class="checklist"]
> * Crear usuarios en Azure Databricks SCIM Connector
> * Quitar usuarios en Azure Databricks SCIM Connector cuando ya no requieran acceso
> * Mantener los atributos de usuario sincronizados entre Azure AD y Azure Databricks SCIM Connector
> * Aprovisionar grupos y pertenencias a grupos en Azure Databricks SCIM Connector

## <a name="prerequisites"></a>Prerrequisitos

En el escenario descrito en este tutorial se supone que ya cuenta con los requisitos previos siguientes:

* [Un inquilino de Azure AD](https://docs.microsoft.com/azure/active-directory/develop/quickstart-create-new-tenant) 
* Una cuenta de usuario en Azure AD con [permiso](https://docs.microsoft.com/azure/active-directory/users-groups-roles/directory-assign-admin-roles) para configurar el aprovisionamiento (por ejemplo, Administrador de aplicaciones, Administrador de aplicaciones en la nube, Propietario de la aplicación o Administrador global). 
* Una cuenta de Azure Databricks con permisos de administrador.

## <a name="step-1-plan-your-provisioning-deployment"></a>Paso 1. Planeación de la implementación de aprovisionamiento
1. Obtenga información sobre [cómo funciona el servicio de aprovisionamiento](https://docs.microsoft.com/azure/active-directory/manage-apps/user-provisioning).
2. Determine quién estará en el [ámbito de aprovisionamiento](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts).
3. Determine qué datos [asignar entre Azure AD y Azure Databricks SCIM Connector](https://docs.microsoft.com/azure/active-directory/manage-apps/customize-application-attributes). 

## <a name="step-2-configure-azure-databricks-scim-connector-to-support-provisioning-with-azure-ad"></a>Paso 2. Configuración de Azure Databricks SCIM Connector para admitir el aprovisionamiento con Azure AD

1. Para configurar el aprovisionamiento de SCIM de Azure Databricks, agréguelo como un recurso en el inquilino de Azure Active Directory y configúrelo con los valores siguientes.

    ![Configuración de Azure Databricks](./media/azure-databricks-scim-provisioning-connector-provisioning-tutorial/setup.png)

2. Para generar un token de acceso personal en Azure Databricks consulte [este artículo](https://docs.microsoft.com/azure/databricks/dev-tools/api/latest/authentication#token-management).

3. Copie el **token**. Este valor se escribirá en el campo Token secreto de la pestaña Aprovisionamiento de la aplicación Azure Databricks SCIM Connector en Azure Portal.

## <a name="step-3-add-azure-databricks-scim-connector-from-the-azure-ad-application-gallery"></a>Paso 3. Agregación de Azure Databricks SCIM Connector desde la galería de aplicaciones de Azure AD

Agregue Azure Databricks conector de SCIM desde la galería de aplicaciones de Azure AD para empezar a administrar el aprovisionamiento para Azure Databricks SCIM Connector. Si había configurado previamente Azure Databricks SCIM Connector para el inicio de sesión único, puede usar la misma aplicación. Sin embargo, se recomienda que cree una aplicación independiente al probar la integración inicialmente. Puede encontrar más información sobre cómo agregar una aplicación desde la galería [aquí](https://docs.microsoft.com/azure/active-directory/manage-apps/add-gallery-app). 

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Paso 4. Determinar quién estará en el ámbito de aprovisionamiento 

El servicio de aprovisionamiento de Azure AD le permite definir quién se aprovisionará, en función de la asignación a la aplicación y de los atributos del usuario o grupo. Si elige el ámbito del que se aprovisionará en la aplicación en función de la asignación, puede usar los pasos [siguientes](../manage-apps/assign-user-or-group-access-portal.md) para asignar usuarios y grupos a la aplicación. Si elige el ámbito del que se aprovisionará en función únicamente de los atributos del usuario o grupo, puede usar un filtro de ámbito, tal como se describe [aquí](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts). 

* Al asignar usuarios y grupos a Azure Databricks SCIM Connector, debe seleccionar un rol que no sea **Acceso predeterminado**. Los usuarios con el rol de acceso predeterminado se excluyen del aprovisionamiento y se marcarán como no autorizados en los registros de aprovisionamiento. Si el único rol disponible en la aplicación es el rol de acceso predeterminado, puede [actualizar el manifiesto de aplicación](https://docs.microsoft.com/azure/active-directory/develop/howto-add-app-roles-in-azure-ad-apps) para agregar roles adicionales. 

* Empiece por algo pequeño. Pruebe con un pequeño conjunto de usuarios y grupos antes de implementarlo en todos. Cuando el ámbito del aprovisionamiento se define en los usuarios y grupos asignados, puede controlarlo asignando uno o dos usuarios o grupos a la aplicación. Cuando el ámbito se establece en todos los usuarios y grupos, puede especificar un [filtro de ámbito basado en atributos](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts). 


## <a name="step-5-configure-automatic-user-provisioning-to-azure-databricks-scim-connector"></a>Paso 5. Configuración del aprovisionamiento de usuarios automático con Azure Databricks SCIM Connector 

Esta sección le guía por los pasos necesarios para configurar el servicio de aprovisionamiento de Azure AD a fin de crear, actualizar y deshabilitar usuarios o grupos en TestApp en función de las asignaciones de grupos o usuarios de Azure AD.

> [!NOTE]
> Para más información sobre el punto de conexión de SCIM de Azure Databricks, consulte [este documento](https://docs.databricks.com/dev-tools/api/latest/scim.html
).

### <a name="to-configure-automatic-user-provisioning-for-azure-databricks-scim-connector-in-azure-ad"></a>Para configurar el aprovisionamiento de usuarios automático para Azure Databricks SCIM Connector en Azure AD:

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Aplicaciones empresariales** y luego **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Azure Databricks SCIM Connector**.

    ![Vínculo de Azure Databricks SCIM Connector en la lista de aplicaciones](common/all-applications.png)

3. Seleccione la pestaña **Aprovisionamiento**.

    ![Pestaña Aprovisionamiento](common/provisioning.png)

4. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Pestaña Aprovisionamiento](common/provisioning-automatic.png)

5. En la sección **Credenciales de administrador**, escriba el valor de punto de conexión de SCIM en **URL de inquilino**. La dirección URL del inquilino debe tener el formato `https://<region>.azuredatabricks.net/api/2.0/preview/scim`, donde se puede encontrar la **región** en la dirección URL de la página principal de Azure Databricks. Por ejemplo, un punto de conexión de SCIM para la región **westus** será `https://westus.azuredatabricks.net/api/2.0/preview/scim`. Escriba el valor del token recuperado anteriormente en **Token secreto**. Haga clic en **Probar conexión** para asegurarse de que Azure AD puede conectarse a Azure Databricks SCIM Connector. Si la conexión no se establece, asegúrese de que la cuenta de Azure Databricks SCIM Connector tenga permisos de administrador y pruebe otra vez.

    ![Aprovisionamiento](./media/azure-databricks-scim-provisioning-connector-provisioning-tutorial/provisioning.png)

6. En el campo **Correo electrónico de notificación**, escriba la dirección de correo electrónico de una persona o grupo que deba recibir las notificaciones de error de aprovisionamiento y active la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Correo electrónico de notificación](common/provisioning-notification-email.png)

7. Seleccione **Guardar**.

8. En la sección **Asignaciones**, seleccione **Synchronize Azure Active Directory Users to Azure Databricks SCIM Connector** (Sincronizar usuarios de Azure Active Directory con Azure Databricks SCIM Connector).

9. Revise los atributos de usuario que se sincronizan entre Azure AD y Azure Databricks SCIM Connector en la sección **Asignación de atributos**. Los atributos seleccionados como propiedades de **Coincidencia** se usan para buscar coincidencias con las cuentas de usuario de Azure Databricks SCIM Connector con el objetivo de realizar operaciones de actualización. Si decide cambiar el [atributo de destino coincidente](https://docs.microsoft.com/azure/active-directory/manage-apps/customize-application-attributes), deberá asegurarse de que la API de Azure Databricks SCIM Connector admite el filtrado de usuarios basado en ese atributo. Seleccione el botón **Guardar** para confirmar los cambios.

   |Atributo|Tipo|
   |---|---|
   |userName|String|
   |DisplayName|String|
   |active|Boolean|

10. En la sección **Asignaciones**, seleccione **Synchronize Azure Active Directory Groups to Azure Databricks SCIM Connector** (Sincronizar grupos de Azure Active Directory con Azure Databricks SCIM Connector).

11. Revise los atributos de grupo que se sincronizan entre Azure AD y Azure Databricks SCIM Connector en la sección **Asignación de atributos**. Los atributos seleccionados como propiedades de **Coincidencia** se usan para buscar coincidencias con los grupos de Azure Databricks SCIM Connector con el objetivo de realizar operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

     |Atributo|Tipo|
     |---|---|
     |DisplayName|String|
     |members|Referencia|

11. En la sección **Asignaciones**, seleccione **Synchronize Azure Active Directory Groups to Azure Databricks SCIM Connector** (Sincronizar grupos de Azure Active Directory con Azure Databricks SCIM Connector).

12. Para habilitar el servicio de aprovisionamiento de Azure AD para Azure Databricks SCIM Connector, cambie el **Estado de aprovisionamiento** a **Activado** en la sección **Configuración**.

    ![Estado de aprovisionamiento activado](common/provisioning-toggle-on.png)

13. Defina los usuarios o grupos que quiere que se aprovisionen en Azure Databricks SCIM Connector, para ello, elija los valores que quiera en **Ámbito**, en la sección **Configuración**.

    ![Ámbito del aprovisionamiento](common/provisioning-scope.png)

14. Cuando esté listo para realizar el aprovisionamiento, haga clic en **Guardar**.

    ![Guardar la configuración de aprovisionamiento](common/provisioning-configuration-save.png)

Esta operación inicia el ciclo de sincronización inicial de todos los usuarios y grupos definidos en **Ámbito** en la sección **Configuración**. El ciclo de sincronización inicial tarda más tiempo en realizarse que los ciclos posteriores, que se producen aproximadamente cada 40 minutos si el servicio de aprovisionamiento de Azure AD está ejecutándose. 

## <a name="step-6-monitor-your-deployment"></a>Paso 6. Supervisión de la implementación
Una vez configurado el aprovisionamiento, use los recursos siguientes para supervisar la implementación:

* Use los [registros de aprovisionamiento](https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-provisioning-logs) para determinar qué usuarios se han aprovisionado correctamente o sin éxito.
* Consulte la [barra de progreso](https://docs.microsoft.com/azure/active-directory/manage-apps/application-provisioning-when-will-provisioning-finish-specific-user) para ver el estado del ciclo de aprovisionamiento y cuánto falta para que finalice.
* Si la configuración de aprovisionamiento parece estar en mal estado, la aplicación pasará a estar en cuarentena. Más información sobre los estados de cuarentena [aquí](https://docs.microsoft.com/azure/active-directory/manage-apps/application-provisioning-quarantine-status).  

## <a name="troubleshooting-tips"></a>Sugerencias de solución de problemas
* Databricks siempre convierten sus valores de nombre de usuario en minúsculas cuando se guardan en el directorio, independientemente del uso de mayúsculas que se les envíe mediante SCIM.
* Actualmente, las solicitudes GET en la API de SCIM de Azure Databricks para los usuarios distinguen entre mayúsculas y minúsculas, por lo que si consultamos USER@contoso.com, se mostrarán 0 resultados debido a que se almacena como user@contoso.com.

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../manage-apps/configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../manage-apps/check-status-user-account-provisioning.md)
