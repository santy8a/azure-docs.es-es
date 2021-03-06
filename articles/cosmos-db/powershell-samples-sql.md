---
title: 'Ejemplos de Azure PowerShell para Azure Cosmos DB: SQL (Core) API'
description: Obtenga los ejemplos de Azure PowerShell para realizar varias tareas comunes en cuentas de SQL API de Azure Cosmos DB.
author: markjbrown
ms.service: cosmos-db
ms.topic: sample
ms.date: 03/26/2020
ms.author: mjbrown
ms.openlocfilehash: efc0ff8e6c198071d3906a0e7e999510198f73bf
ms.sourcegitcommit: 07d62796de0d1f9c0fa14bfcc425f852fdb08fb1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/27/2020
ms.locfileid: "80366193"
---
# <a name="azure-powershell-samples-for-azure-cosmos-db---sql-core-api"></a>Ejemplos de Azure PowerShell para Azure Cosmos DB: SQL (Core) API

En la tabla siguiente se incluyen vínculos a scripts de Azure PowerShell usados habitualmente para la API Azure Cosmos DB for SQL (Core). Si desea bifurcar estos ejemplos de PowerShell para Cosmos DB desde nuestro repositorio de GitHub, visite los [ejemplos de PowerShell para Cosmos DB en GitHub](https://github.com/Azure/azure-docs-powershell-samples/tree/master/cosmosdb).

Para ver más ejemplos de PowerShell para Cosmos DB para SQL (Core) API y documentación al respecto, consulte [Administración de recursos de SQL API de Azure Cosmos DB mediante PowerShell](manage-with-powershell.md). Para ver ejemplos de PowerShell para Cosmos DB para otras API consulte, [Cassandra API](powershell-samples-cassandra.md), [MongoDB API](powershell-samples-mongodb.md), [Gremlin API](powershell-samples-gremlin.md) y [Table API](powershell-samples-table.md).

> [!NOTE]
> En los ejemplos, se usan los cmdlets de administración de [Az.CosmosDB](https://docs.microsoft.com/powershell/module/az.cosmosdb). Tenga en cuenta que los cmdlets de `Az.CosmosDB` están todavía en versión preliminar y pueden cambiar antes de que estén disponibles. Compruebe periódicamente si hay actualizaciones para `Az.CosmosDB`.

| | |
|---|---|
|[Crear una cuenta, base de datos y contenedor](scripts/powershell/sql/ps-sql-create.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Cree una cuenta, una base de datos y un contenedor de Azure Cosmos DB. |
|[Creación de un contenedor con una clave de partición de gran tamaño](scripts/powershell/sql/ps-sql-container-create-large-partition-key.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Cree un contenedor con una clave de partición de gran tamaño. |
|[Enumerar u obtener las bases de datos o contenedores](scripts/powershell/sql/ps-sql-list-get.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Enumere u obtenga las bases de datos o contenedores. |
|[Obtener RU/s](scripts/powershell/sql/ps-sql-ru-get.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Obtenga RU/s para una base de datos o contenedor. |
|[Actualizar RU/s](scripts/powershell/sql/ps-sql-ru-update.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Actualice RU/s para una base de datos o contenedor. |
|[Creación de un contenedor sin directiva de índice](scripts/powershell/sql/ps-sql-container-create-index-none.md?toc=%2fpowershell%2fmodule%2ftoc.json) | Cree un contenedor de Azure Cosmos con directiva de índice desactivada.|
|[Actualización de una cuenta](scripts/powershell/common/ps-account-update.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Actualice el nivel de coherencia predeterminado de una cuenta de Cosmos DB. |
|[Actualización de las regiones de una cuenta](scripts/powershell/common/ps-account-update-region.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Actualice las regiones de una cuenta de Cosmos DB. |
|[Cambio de la prioridad de la conmutación por error o desencadenamiento de la conmutación por error](scripts/powershell/common/ps-account-failover-priority-update.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Cambie la prioridad de la conmutación por error regional de una cuenta de Azure Cosmos o desencadene una conmutación por error manual. |
|[Claves de cuenta o cadenas de conexión](scripts/powershell/common/ps-account-keys-connection-strings.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Obtenga las claves principales y secundarias, las cadenas de conexión o vuelva a generar una clave de cuenta de una cuenta de Azure Cosmos DB. |
|[Creación de una cuenta de Cosmos con firewall de IP](scripts/powershell/common/ps-account-firewall-create.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Cree una cuenta de Azure Cosmos DB con firewall de IP habilitado. |
|||
