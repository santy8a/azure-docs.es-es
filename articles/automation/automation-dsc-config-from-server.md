---
title: 'Creación de configuraciones desde de servidores existentes: Azure Automation'
description: Aprenda a crear configuraciones desde servidores existentes para Azure Automation.
keywords: dsc,powershell,configuration,setup
services: automation
ms.service: automation
ms.subservice: dsc
author: mgreenegit
ms.author: migreene
ms.date: 08/08/2019
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: dff9b8f52207a38cf7eaddefa178aff262ddc546
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2020
ms.locfileid: "80585543"
---
# <a name="create-configurations-from-existing-servers"></a>Creación de configuraciones desde servidores existentes

> Se aplica a: Windows PowerShell 5.1

Crear configuraciones desde servidores existentes puede ser una tarea complicada.
Puede que no quiera *toda* la configuración, solo la que le preocupa.
Incluso entonces, necesita saber en qué orden se debe aplicar la configuración para que se aplique correctamente.

> [!NOTE]
> En este artículo se hace referencia a una solución que se mantiene en la comunidad de código abierto.
> La compatibilidad solo está disponible en forma de colaboración en GitHub, no con Microsoft.

## <a name="community-project-reversedsc"></a>Proyecto de la comunidad: ReverseDSC

Se ha creado una solución mantenida por la comunidad llamada [ReverseDSC](https://github.com/microsoft/reversedsc) para que funcione en este área a partir de SharePoint.

La solución se basa en el [recurso SharePointDSC](https://github.com/powershell/sharepointdsc) y la amplía para coordinar la [recopilación de información](https://github.com/Microsoft/sharepointDSC.reverse#how-to-use) de los servidores de SharePoint existentes.
La versión más reciente tiene varios [modos de extracción](https://github.com/Microsoft/SharePointDSC.Reverse/wiki/Extraction-Modes) para determinar el nivel de información que se va a incluir.

El resultado de usar la solución es la generación de [datos de configuración](https://github.com/Microsoft/sharepointDSC.reverse#configuration-data) que se usarán con los scripts de configuración de SharePointDSC.

Una vez que se han generado los archivos de datos, puede usarlos con [scripts de configuración de DSC](/powershell/scripting/dsc/overview/overview) para generar archivos MOF y [cargarlos en Azure Automation](/azure/automation/tutorial-configure-servers-desired-state#create-and-upload-a-configuration-to-azure-automation).
A continuación, registre los servidores desde una [ubicación local](/azure/automation/automation-dsc-onboarding#onboarding-physicalvirtual-windows-machines-on-premises-or-in-a-cloud-other-than-azure-including-aws-ec2-instances) o [en Azure](/azure/automation/automation-dsc-onboarding#onboarding-azure-vms) para extraer las configuraciones.

Para probar ReverseDSC, visite la [Galería de PowerShell](https://www.powershellgallery.com/packages/ReverseDSC/) y descargue la solución o haga clic en "Project Site" (Sitio del proyecto) para ver la [documentación](https://github.com/Microsoft/sharepointDSC.reverse).

## <a name="next-steps"></a>Pasos siguientes

- [Información general sobre la configuración de estado deseado de Windows PowerShell](/powershell/scripting/dsc/overview/overview)
- [Recursos de DSC](/powershell/scripting/dsc/resources/resources)
- [Configuración del administrador de configuración local](/powershell/scripting/dsc/managing-nodes/metaconfig)
