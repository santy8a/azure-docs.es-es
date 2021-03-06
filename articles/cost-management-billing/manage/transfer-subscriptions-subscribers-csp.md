---
title: Transferencia de suscripciones de Azure entre suscriptores y CSP
description: Aprenda a transferir suscripciones de Azure entre suscriptores y CSP.
author: bandersmsft
ms.reviewer: dhgandhi
ms.service: cost-management-billing
ms.topic: conceptual
ms.date: 05/04/2020
ms.author: banders
ms.openlocfilehash: b198a781da6e18c363e90d8bd5314dafc563dcca
ms.sourcegitcommit: e0330ef620103256d39ca1426f09dd5bb39cd075
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/05/2020
ms.locfileid: "82791160"
---
# <a name="transfer-azure-subscriptions-between-subscribers-and-csps"></a>Transferencia de suscripciones de Azure entre suscriptores y CSP

En este artículo se brindan los pasos generales que se siguen para transferir suscripciones de Azure entre los partners del Proveedor de soluciones en la nube (CSP) y sus clientes.

## <a name="transfer-ea-subscriptions"></a>Transferencia de suscripciones de EA

Los partners de facturación directa de CSP certificados como [Proveedor de servicios administrados (MSP) experto de Azure](https://partner.microsoft.com/membership/azure-expert-msp) pueden solicitar la transferencia de suscripciones de Azure a sus clientes que tengan un Contrato Enterprise (EA) directo. Solo se permiten las transferencias de suscripciones a los clientes que han aceptado un Contrato de cliente de Microsoft (MCA) y han adquirido un plan de Azure.

Cuando se aprueba la solicitud, el CSP puede entregar una factura combinada a sus clientes. Para obtener más información sobre los CSP que transfieren suscripciones, consulte [Obtención de la propiedad de la facturación de las suscripciones a Azure para la cuenta de MPA](mpa-request-ownership.md).

## <a name="other-subscription-transfers-to-a-csp-partner"></a>Otras transferencias de suscripciones a un partner de CSP

Para transferir cualquier otra suscripción de Azure a un partner de CSP, el suscriptor debe trasladar los recursos de las suscripciones de origen a las suscripciones de CSP. Utilice las siguientes instrucciones para migrar recursos entre suscripciones.

1. Trabaje con su partner de CSP para crear suscripciones de CSP de Azure de destino.
1. Asegúrese de que las suscripciones de CSP de origen y destino están en el mismo inquilino de Azure Active Directory (Azure AD).  
    No se puede cambiar al inquilino de Azure AD por una suscripción de Azure CSP. En su lugar, debe agregar o asociar la suscripción de origen al inquilino de Azure AD de CSP. Para más información, consulte [Asociación o incorporación de una suscripción de Azure al inquilino de Azure Active Directory](../../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md).
    > [!IMPORTANT]
    > - Al asociar una suscripción a otro directorio de Azure AD, los usuarios que tengan roles asignados mediante el [control de acceso basado en rol (RBAC)](../../role-based-access-control/role-assignments-portal.md) pierden el acceso. Los administradores de suscripciones clásicas, incluidos el administrador y los coadministradores del servicio, también pierden el acceso.
    > - También se quitan las asignaciones de directivas de una suscripción cuando dicha suscripción está asociada a un directorio diferente.
1. La cuenta de usuario que use para realizar la transferencia debe tener el acceso de propietario [RBAC](add-change-subscription-administrator.md) en ambas suscripciones.
1. Antes de comenzar, [valide](/rest/api/resources/resources/validatemoveresources) que todos los recursos de Azure puedan moverse de la suscripción de origen a la suscripción de destino.  
    Algunos recursos de Azure no se pueden mover entre suscripciones. Para ver la lista completa de recursos de Azure que se pueden mover, consulte [Compatibilidad con la operación de traslado para recursos](../../azure-resource-manager/management/move-support-resources.md).
    > [!IMPORTANT]
    >  - CSP de Azure solo admite los recursos de Azure Resource Manager. Si alguno de los recursos de Azure en la suscripción de origen se creó mediante el modelo de implementación clásica de Azure, debe migrarlo a [Azure Resource Manager](https://docs.microsoft.com/azure/cloud-solution-provider/migration/ea-payg-to-azure-csp/ea-open-direct-asm-to-arm) antes de la migración. Debe ser partner para ver la página web.

1. Compruebe que todos los servicios de suscripción de origen usen el modelo de Azure Resource Manager. A continuación, transfiera los recursos de la suscripción de origen a la suscripción de destino siguiendo el [traslado de recursos de Azure](../../azure-resource-manager/management/move-resource-group-and-subscription.md).
    > [!IMPORTANT]
    >  - El traslado de recursos de Azure entre suscripciones puede dar lugar a un tiempo de inactividad del servicio, según los recursos de las suscripciones.

## <a name="all-subscription-transfers-from-a-csp-partner"></a>Todas las transferencias de suscripciones desde un partner de CSP

Para transferir cualquier otra suscripción de un partner de CSP a cualquier otra oferta de Azure, el suscriptor debe trasladar los recursos de las suscripciones de origen a las suscripciones de CSP.

1. Cree las suscripciones a Azure de destino.
1. Asegúrese de que las suscripciones de origen y destino están en el mismo inquilino de Azure Active Directory (Azure AD). Para obtener más información sobre el cambio de un inquilino de Azure AD, consulte [Asociación o incorporación de una suscripción de Azure al inquilino de Azure Active Directory](../../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md).
    Tenga en cuenta que el directorio de cambios no es la suscripción de CSP. Por ejemplo, pasa de un CSP a una suscripción de pago por uso y necesita cambiar el directorio de la suscripción de pago por uso para que coincida con el directorio.

    > [!IMPORTANT]
    >  - Al asociar una suscripción a un directorio diferente, los usuarios que tengan roles asignados mediante [RBAC](../../role-based-access-control/role-assignments-portal.md) pierden el acceso. Los administradores de suscripciones clásicas, incluidos el administrador y los coadministradores del servicio, también pierden el acceso.
    >  - También se quitan las asignaciones de directivas de una suscripción cuando dicha suscripción está asociada a un directorio diferente.

1. La cuenta de usuario que use para realizar la transferencia debe tener el acceso de propietario [RBAC](add-change-subscription-administrator.md) en ambas suscripciones.
1. Antes de comenzar, [valide](/rest/api/resources/resources/validatemoveresources) que todos los recursos de Azure puedan moverse de la suscripción de origen a la suscripción de destino.
    > [!IMPORTANT]
    >  - Algunos recursos de Azure no se pueden mover entre suscripciones. Para ver la lista completa de recursos de Azure que se pueden mover, consulte [Compatibilidad con la operación de traslado para recursos](../../azure-resource-manager/management/move-support-resources.md).

1. Transfiera los recursos de la suscripción de origen a la suscripción de destino siguiendo el [traslado de recursos de Azure](../../azure-resource-manager/management/move-resource-group-and-subscription.md).
    > [!IMPORTANT]
    >  - El traslado de recursos de Azure entre suscripciones puede dar lugar a un tiempo de inactividad del servicio, según los recursos de las suscripciones.

## <a name="next-steps"></a>Pasos siguientes
- [Obtención de la propiedad de la facturación de las suscripciones a Azure para la cuenta de MPA](mpa-request-ownership.md)
- Lea sobre la [Administración de suscripciones y cuentas de Azure con facturación de Azure](index.yml).
