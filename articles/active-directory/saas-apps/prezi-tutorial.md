---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Prezi | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Prezi.
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: barbkess
ms.assetid: dd6a6d39-f6c4-49ef-b849-12724d5ea3c2
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.topic: tutorial
ms.date: 04/30/2020
ms.author: jeedes
ms.collection: M365-identity-device-management
ms.openlocfilehash: dab850e1919c735f35d2efb9fa373c8cd331c165
ms.sourcegitcommit: 4499035f03e7a8fb40f5cff616eb01753b986278
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/03/2020
ms.locfileid: "82735910"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-prezi"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Prezi

En este tutorial aprenderá a integrar Prezi con Azure Active Directory (Azure AD). Al integrar Prezi con Azure AD, podrá:

* Controlar en Azure AD quién tiene acceso a Prezi.
* Permitir que los usuarios inicien sesión automáticamente en Prezi con sus cuentas de Azure AD.
* Administrar las cuentas desde una ubicación central (Azure Portal).

Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Prezi.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, va a configurar y probar el inicio de sesión único de Azure AD en un entorno de prueba.

* Prezi admite el inicio de sesión único iniciado por **SP e IDP**.
* Prezi admite el aprovisionamiento de usuarios **Just-In-Time**.
* Una vez configurado Prezi, puede aplicar el control de sesión, que protege la filtración y la infiltración de la información confidencial de la organización en tiempo real. El control de sesión procede del acceso condicional. [Aprenda a aplicar el control de sesión con Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-prezi-from-the-gallery"></a>Adición de Prezi desde la galería

Para configurar la integración de Prezi en Azure AD, será preciso agregar Prezi desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Prezi** en el cuadro de búsqueda.
1. Seleccione **Prezi** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.

## <a name="configure-and-test-azure-ad-single-sign-on-for-prezi"></a>Configuración y prueba del inicio de sesión único de Azure AD para Prezi

Configure y pruebe el inicio de sesión único (SSO) de Azure AD con Prezi mediante un usuario de prueba llamado **B.Simon**. Para que el inicio de sesión único funcione, es necesario establecer una relación de vínculo entre un usuario de Azure AD y el usuario relacionado de Prezi.

Para configurar y probar el inicio de sesión único de Azure AD con Prezi, es preciso completar los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** , para permitir que los usuarios puedan utilizar esta característica.
    1. **[Creación de un usuario de prueba de Azure AD](#create-an-azure-ad-test-user)** , para probar el inicio de sesión único de Azure AD con B.Simon.
    1. **[Asignación del usuario de prueba de Azure AD](#assign-the-azure-ad-test-user)** , para habilitar a B.Simon para que use el inicio de sesión único de Azure AD.
1. **[Configuración del inicio de sesión único en Prezi](#configure-prezi-sso)** , para configurar los valores de inicio de sesión único en la aplicación.
    1. **[Creación de un usuario de prueba en Prezi](#create-prezi-test-user)** , para tener un homólogo de B.Simon en Prezi vinculado a la representación del usuario en Azure AD.
1. **[Prueba del inicio de sesión único](#test-sso)** : para comprobar si la configuración funciona.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **Prezi**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección **Configuración básica de SAML**, el usuario no tiene que realizar ningún paso porque la aplicación ya se ha integrado previamente con Azure.

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL: `https://prezi.com/login/sso/`

1. Haga clic en **Save**(Guardar).

1. La aplicación Prezi espera las aserciones de SAML en un formato específico, que requiere que se agreguen asignaciones de atributos personalizados a la configuración de los atributos del token de SAML. La siguiente captura de muestra la lista de atributos predeterminados.

    ![imagen](common/default-attributes.png)

1. Además de lo anterior, la aplicación Prezi espera que se devuelvan algunos atributos más, que se muestran a continuación en la respuesta de SAML. Estos atributos también se rellenan previamente, pero puede revisarlos según sus requisitos.
    
    | Nombre | Atributo de origen|
    | ---------------| --------------- |
    | given_name | user.givenname |
    | family_name | user.surname |

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, busque **Certificado (Base64)** y seleccione **Descargar** para descargarlo y guardarlo en el equipo.

    ![Vínculo de descarga del certificado](common/certificatebase64.png)

1. En la sección **Set up Prezi** (Configurar Prezi), copie las direcciones URL que necesite.

    ![Copiar direcciones URL de configuración](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Creación de un usuario de prueba de Azure AD

En esta sección, va a crear un usuario de prueba llamado B.Simon en Azure Portal.

1. En el panel izquierdo de Azure Portal, seleccione **Azure Active Directory**, **Usuarios** y **Todos los usuarios**.
1. Seleccione **Nuevo usuario** en la parte superior de la pantalla.
1. En las propiedades del **usuario**, siga estos pasos:
   1. En el campo **Nombre**, escriba `B.Simon`.  
   1. En el campo **Nombre de usuario**, escriba username@companydomain.extension. Por ejemplo, `B.Simon@contoso.com`.
   1. Active la casilla **Show password** (Mostrar contraseña) y, después, anote el valor que se muestra en el cuadro **Contraseña**.
   1. Haga clic en **Crear**.

### <a name="assign-the-azure-ad-test-user"></a>Asignación del usuario de prueba de Azure AD

En esta sección, va a permitir que B.Simon acceda a Prezi mediante el inicio de sesión único de Azure.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Prezi**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione **B.Simon** de la lista de usuarios y haga clic en el botón **Seleccionar** de la parte inferior de la pantalla.
1. Si espera que haya un valor de rol en la aserción de SAML, en el cuadro de diálogo **Seleccionar rol**, seleccione en la lista el rol adecuado para el usuario y haga clic en el botón **Seleccionar** en la parte inferior de la pantalla.
1. En el cuadro de diálogo **Agregar asignación**, haga clic en el botón **Asignar**.

## <a name="configure-prezi-sso"></a>Configuración del inicio de sesión único de Prezi

1. En otra ventana del explorador web, inicie sesión en Prezi con su cuenta de equipo y vaya a **[Consola de administración](https://prezi.com/organizations/manage)** .

1. En la **Consola de administración**, haga clic en la pestaña **Configuración**.

    ![Configuración de Prezi](./media/prezi-tutorial/settings-image.png)

1. Vaya a la sección **Inicio de sesión único (SSO)** y use el modificador para habilitar SSO.
    
    ![Configuración de Prezi](./media/prezi-tutorial/single-signon.png)

1. En la sección **Inicio de sesión único (SSO)** , siga estos pasos:

    ![Configuración de Prezi](./media/prezi-tutorial/configuration.png)

    a. En el cuadro de texto **Identifier or Issuer URL** (URL de identificador o emisor), pegue el valor de **Azure AD Identifier** (Identificador de Azure AD) que ha copiado de Azure Portal.

    b. En el cuadro de texto **SAML 2.0 Endpoint(HTTP)** (Punto de conexión SAML 2.0 [HTTP]), pegue el valor de **Dirección URL de inicio de sesión** que ha copiado de Azure Portal.

    c. Abra el **Certificado(Base64)** descargado desde Azure Portal al Bloc de notas y copie el contenido, y, luego, péguelo en el cuadro de texto **Certificado (X509)** .

    d. Haga clic en **Save**(Guardar).

### <a name="create-prezi-test-user"></a>Creación de un usuario de prueba de Prezi

En esta sección, se crea el usuario B.Simon en Prezi. Prezi admite el aprovisionamiento de usuarios Just-In-Time, que está habilitado de forma predeterminada. No hay ningún elemento de acción para usted en esta sección. Si aún no existe un usuario en Prezi, se crea uno después de la autenticación.

## <a name="test-sso"></a>Prueba de SSO 

En esta sección, probará la configuración de inicio de sesión único de Azure AD mediante el Panel de acceso.

Al hacer clic en el icono de Prezi en el panel de acceso, debería iniciar sesión automáticamente en la versión de Prezi para la que configuró el inicio de sesión único. Para más información sobre el Panel de acceso, consulte [Introducción al Panel de acceso](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe Prezi con Azure AD](https://aad.portal.azure.com/)

- [¿Qué es el control de sesiones en Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)

- [Protección de Prezi con controles y visibilidad avanzados](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)

