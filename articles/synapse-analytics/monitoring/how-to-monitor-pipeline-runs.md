---
title: Supervisión de ejecuciones de canalización en Azure Synapse Studio (versión preliminar)
description: Use Azure Synapse Studio para supervisar las ejecuciones de canalización de su área de trabajo.
services: synapse-analytics
author: matt1883
ms.service: synapse-analytics
ms.topic: how-to
ms.date: 04/15/2020
ms.author: mahi
ms.reviewer: mahi
ms.openlocfilehash: 712dc62e29229f03dec12afdf18edbf55667dbdf
ms.sourcegitcommit: b80aafd2c71d7366838811e92bd234ddbab507b6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2020
ms.locfileid: "81427312"
---
# <a name="use-azure-synapse-studio-to-monitor-your-workspace-pipeline-runs"></a>Uso de Azure Synapse Studio para supervisar las ejecuciones de canalización del área de trabajo

Con Azure Synapse Analytics, puede crear canalizaciones complejas que pueden automatizar y orquestar el movimiento de datos, la transformación de datos y las actividades de proceso dentro de su solución. Puede crear y supervisar estas canalizaciones mediante Azure Synapse Studio (versión preliminar).

En este artículo se explica cómo supervisar las ejecuciones de canalización, lo que le permite vigilar el estado más reciente, los problemas y el progreso de sus canalizaciones.

## <a name="access-the-list-of-pipeline-runs"></a>Acceso a la lista de ejecuciones de canalización

Para ver la lista de ejecuciones de canalización en su área de trabajo, primero [abra Azure Synapse Studio](https://web.azuresynapse.net/) y seleccione el área de trabajo.

![Inicio de sesión en el área de trabajo](./media/common/login-workspace.png)

Una vez que haya abierto el área de trabajo, seleccione la sección **Supervisar** a la izquierda.

![Selección del centro Supervisar](./media/common/left-nav.png)

Seleccione **Ejecuciones de canalización** para ver la lista de ejecuciones de canalización.

![Selección de ejecuciones de canalización](./media/how-to-monitor-pipeline-runs/monitor-hub-nav-pipelineruns.png)

## <a name="filtering-your-pipeline-runs"></a>Filtrado de ejecuciones de canalización

Puede filtrar la lista de ejecuciones de canalización a las que le interesan. Los filtros en la parte superior de la pantalla le permiten especificar un campo por el que desea filtrar.

Por ejemplo, puede filtrar la vista para ver solo las ejecuciones de canalización para la canalización denominada "holiday":

![Botón de filtro](./media/common/filter-button.png)

![Filtro de ejemplo](./media/how-to-monitor-pipeline-runs/filter-example.png)

## <a name="viewing-details-about-a-specific-pipeline-run"></a>Visualización de detalles sobre una ejecución de una canalización específica

Para ver los detalles sobre la ejecución de canalización, seleccione la ejecución de la canalización. A continuación, vea las ejecuciones de actividad asociadas a la ejecución de la canalización. Si la canalización todavía está en ejecución, puede supervisar el progreso. 
  
## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre la supervisión de aplicaciones, consulte el artículo [Supervisión de aplicaciones de Apache Spark](how-to-monitor-spark-applications.md). 
