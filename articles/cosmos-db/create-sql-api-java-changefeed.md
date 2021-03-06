---
title: 'Tutorial: ejemplo de aplicación de SQL API de Async Java de un extremo a otro con fuente de cambios'
description: Este tutorial le guía a través de una sencilla aplicación de SQL API de Java que inserta documentos en un contenedor de Azure Cosmos DB, mientras mantiene una vista materializada del contenedor mediante la fuente de cambios.
author: anfeldma
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: java
ms.topic: tutorial
ms.date: 04/01/2020
ms.author: anfeldma
ms.openlocfilehash: 5eab523dde2a13a85b0c8ff5bcbb3ecb5912e78e
ms.sourcegitcommit: 3c318f6c2a46e0d062a725d88cc8eb2d3fa2f96a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/02/2020
ms.locfileid: "80587222"
---
# <a name="tutorial---an-end-to-end-async-java-sql-api-application-sample-with-change-feed"></a>Tutorial: ejemplo de aplicación de SQL API de Async Java de un extremo a otro con fuente de cambios

Este tutorial le guía a través de una sencilla aplicación de SQL API de Java que inserta documentos en un contenedor de Azure Cosmos DB, mientras mantiene una vista materializada del contenedor mediante la fuente de cambios.

## <a name="prerequisites"></a>Prerrequisitos

* PC

* URI y clave de la cuenta de Azure Cosmos DB

* Maven

* Java 8

## <a name="background"></a>Información previa

La fuente de cambios de Azure Cosmos DB proporciona una interfaz orientada a eventos para desencadenar acciones en respuesta a la inserción de documentos. Esto tiene muchas utilidades. Por ejemplo, en aplicaciones en las que se realizan gran cantidad de operaciones de lectura y escritura, la fuente de cambios se usa principalmente para crear una **vista materializada** en tiempo real de un contenedor durante la ingesta de documentos. El contenedor de la vista materializada contendrá los mismos datos pero particionados para facilitar las lecturas, lo que garantizará la eficacia de las operaciones de lectura y escritura de la aplicación.

La biblioteca de procesadores de fuente de cambios integrada en el SDK se ocupa en gran medida del trabajo de administración de eventos de la fuente de cambios. Esta biblioteca es lo suficientemente eficaz como para distribuir eventos de la fuente de cambios entre varios trabajos, si así se desea. Todo lo que tiene que hacer es proporcionar una devolución de llamada a la biblioteca de fuentes de cambios.

En este sencillo ejemplo se muestra cómo la biblioteca de procesadores de fuente de cambios con un solo trabajo crea y elimina documentos de una vista materializada.

## <a name="setup"></a>Configurar

Si todavía no lo ha hecho, clone el repositorio de ejemplo de la aplicación:

```bash
git clone https://github.com/Azure-Samples/azure-cosmos-java-sql-app-example.git
```

> Puede completar este inicio rápido con el SDK de Java 4.0 o el SDK de Java 3.7.0. **Si desea usar el SDK de Java 3.7.0, en el terminal, escriba ```git checkout SDK3.7.0```** . De lo contrario, manténgase en la rama ```master```, que usa el SDK 4.0 de Java de forma predeterminada.

Abra un terminal en el directorio del repositorio. Ejecute lo siguiente para compilar la aplicación:

```bash
mvn clean package
```

## <a name="walkthrough"></a>Tutorial

1. Como primer requisito, debe tener una cuenta de Azure Cosmos DB. Abra **Azure Portal** en el explorador, vaya a la cuenta de Azure Cosmos DB y, en el panel izquierdo, vaya a **Explorador de datos**.

    ![Cuenta de Azure Cosmos DB](media/create-sql-api-java-changefeed/cosmos_account_empty.JPG)

1. Ejecute la aplicación en el terminal con los siguientes comandos:

    ```bash
    mvn exec:java -Dexec.mainClass="com.azure.cosmos.workedappexample.SampleGroceryStore" -DACCOUNT_HOST="your-account-uri" -DACCOUNT_KEY="your-account-key" -Dexec.cleanupDaemonThreads=false
    ```

1. Presione Entrar cuando vea lo siguiente:

    ```bash
    Press enter to create the grocery store inventory system...
    ```

    A continuación, vuelva al Explorador de datos de Azure Portal en el explorador. Verá que se ha agregado la base de datos **GroceryStoreDatabase** con tres contenedores vacíos: 

    * **InventoryContainer**: el registro de inventario de nuestra tienda de comestibles de ejemplo, con particiones en el elemento ```id```, que es un UUID.
    * **InventoryContainer-pktype**: una vista materializada del registro de inventario, optimizada para consultas sobre el elemento ```type```.
    * **InventoryContainer-leases**: un contenedor de concesiones siempre es necesario para la fuente de cambios; las concesiones realizan un seguimiento del progreso de la aplicación en la lectura de la fuente de cambios.


    ![Contenedores vacíos](media/create-sql-api-java-changefeed/cosmos_account_resources_lease_empty.JPG)


1. En el terminal, ahora debería aparecer un mensaje:

    ```bash
    Press enter to start creating the materialized view...
    ```

    Presione Entrar. Ahora, el siguiente bloque de código ejecutará e inicializará el procesador de fuente de cambios en otro subproceso: 


    **SDK de Java 4.0**
    ```java
    changeFeedProcessorInstance = getChangeFeedProcessor("SampleHost_1", feedContainer, leaseContainer);
    changeFeedProcessorInstance.start()
        .subscribeOn(Schedulers.elastic())
        .doOnSuccess(aVoid -> {
            isProcessorRunning.set(true);
        })
        .subscribe();

    while (!isProcessorRunning.get()); //Wait for Change Feed processor start
    ```

    **SDK de Java 3.7.0**
    ```java
    changeFeedProcessorInstance = getChangeFeedProcessor("SampleHost_1", feedContainer, leaseContainer);
    changeFeedProcessorInstance.start()
        .subscribeOn(Schedulers.elastic())
        .doOnSuccess(aVoid -> {
            isProcessorRunning.set(true);
        })
        .subscribe();

    while (!isProcessorRunning.get()); //Wait for Change Feed processor start    
    ```

    ```"SampleHost_1"``` es el nombre del trabajo del procesador de fuente de cambios. ```changeFeedProcessorInstance.start()``` es lo que realmente inicia el procesador de fuente de cambios.

    Vuelva al Explorador de datos de Azure Portal en el explorador. En el contenedor **InventoryContainer-leases**, haga clic en **Elementos** para ver su contenido. Verá que el procesador de fuente de cambios ha rellenado el contenedor de concesiones; es decir, el procesador ha asignado una concesión al trabajo ```SampleHost_1``` en algunas particiones del contenedor **InventoryContainer**.

    ![Leases](media/create-sql-api-java-changefeed/cosmos_leases.JPG)

1. Presione Entrar de nuevo en el terminal. Esto desencadenará 10 documentos que se insertarán en **InventoryContainer**. Cada inserción de documento aparece en la fuente de cambios como JSON; el siguiente código de devolución de llamada controla estos eventos mediante el reflejo los documentos JSON en una vista materializada:

    **SDK de Java 4.0**
    ```java
    public static ChangeFeedProcessor getChangeFeedProcessor(String hostName, CosmosAsyncContainer feedContainer, CosmosAsyncContainer leaseContainer) {
        ChangeFeedProcessorOptions cfOptions = new ChangeFeedProcessorOptions();
        cfOptions.setFeedPollDelay(Duration.ofMillis(100));
        cfOptions.setStartFromBeginning(true);
        return ChangeFeedProcessor.changeFeedProcessorBuilder()
            .setOptions(cfOptions)
            .setHostName(hostName)
            .setFeedContainer(feedContainer)
            .setLeaseContainer(leaseContainer)
            .setHandleChanges((List<JsonNode> docs) -> {
                for (JsonNode document : docs) {
                        //Duplicate each document update from the feed container into the materialized view container
                        updateInventoryTypeMaterializedView(document);
                }

            })
            .build();
    }

    private static void updateInventoryTypeMaterializedView(JsonNode document) {
        typeContainer.upsertItem(document).subscribe();
    }
    ```

    **SDK de Java 3.7.0**
    ```java
    public static ChangeFeedProcessor getChangeFeedProcessor(String hostName, CosmosContainer feedContainer, CosmosContainer leaseContainer) {
        ChangeFeedProcessorOptions cfOptions = new ChangeFeedProcessorOptions();
        cfOptions.feedPollDelay(Duration.ofMillis(100));
        cfOptions.startFromBeginning(true);
        return ChangeFeedProcessor.Builder()
            .options(cfOptions)
            .hostName(hostName)
            .feedContainer(feedContainer)
            .leaseContainer(leaseContainer)
            .handleChanges((List<CosmosItemProperties> docs) -> {
                for (CosmosItemProperties document : docs) {
                        //Duplicate each document update from the feed container into the materialized view container
                        updateInventoryTypeMaterializedView(document);
                }

            })
            .build();
    }

    private static void updateInventoryTypeMaterializedView(CosmosItemProperties document) {
        typeContainer.upsertItem(document).subscribe();
    }    
    ```

1. Deje que el código se ejecute entre 5 y 10 segundos. A continuación, vuelva al Explorador de datos de Azure Portal y vaya a **InventoryContainer > Elementos**. Debería ver que los elementos se insertan en el contenedor de inventario. Anote la clave de partición (```id```).

    ![Contenedor de la fuente](media/create-sql-api-java-changefeed/cosmos_items.JPG)

1. Ahora, en el Explorador de datos, vaya a **InventoryContainer-pktype > Elementos**. Esta es la vista materializada: los elementos de este contenedor reflejan el contenedor **InventoryContainer**, porque se han insertado mediante programación con la fuente de cambios. Anote la clave de partición (```type```). Esta vista materializada está optimizada para el filtrado de consultas por ```type```, lo que sería ineficaz en **InventoryContainer** porque está particionado en ```id```.

    ![Vista materializada](media/create-sql-api-java-changefeed/cosmos_materializedview2.JPG)

1. Vamos a eliminar un documento de **InventoryContainer** y de **InventoryContainer-pktype** usando simplemente una única llamada de ```upsertItem()```. En primer lugar, observe el Explorador de datos de Azure Portal. Eliminaremos el documento para el que ```/type == "plums"``` aparece en un recuadro rojo a continuación

    ![Vista materializada](media/create-sql-api-java-changefeed/cosmos_materializedview-emph-todelete.JPG)

    Presione Entrar de nuevo para llamar a la función ```deleteDocument()``` en el código de ejemplo. Esta función, que se muestra a continuación, actualiza o inserta una nueva versión del documento con ```/ttl == 5```, que establece el período de vida (TTL) del documento en 5 segundos. 
    
    **SDK de Java 4.0**
    ```java
    public static void deleteDocument() {

        String jsonString =    "{\"id\" : \"" + idToDelete + "\""
                + ","
                + "\"brand\" : \"Jerry's\""
                + ","
                + "\"type\" : \"plums\""
                + ","
                + "\"quantity\" : \"50\""
                + ","
                + "\"ttl\" : 5"
                + "}";

        ObjectMapper mapper = new ObjectMapper();
        JsonNode document = null;

        try {
            document = mapper.readTree(jsonString);
        } catch (Exception e) {
            e.printStackTrace();
        }

        feedContainer.upsertItem(document,new CosmosItemRequestOptions()).block();
    }    
    ```

    **SDK de Java 3.7.0**
    ```java
    public static void deleteDocument() {

        String jsonString =    "{\"id\" : \"" + idToDelete + "\""
                + ","
                + "\"brand\" : \"Jerry's\""
                + ","
                + "\"type\" : \"plums\""
                + ","
                + "\"quantity\" : \"50\""
                + ","
                + "\"ttl\" : 5"
                + "}";

        ObjectMapper mapper = new ObjectMapper();
        JsonNode document = null;

        try {
            document = mapper.readTree(jsonString);
        } catch (Exception e) {
            e.printStackTrace();
        }

        feedContainer.upsertItem(document,new CosmosItemRequestOptions()).block();
    }    
    ```

    La fuente de cambios ```feedPollDelay``` está establecida en 100 ms; por lo tanto, la fuente de cambios responde a esta actualización casi al instante y llama a ```updateInventoryTypeMaterializedView()``` mostrado anteriormente. Esa última llamada de función actualizará o insertará el nuevo documento con un período de vida de 5 segundos en **InventoryContainer-pktype**.

    En consecuencia, después de unos 5 segundos, el documento expirará y se eliminará de ambos contenedores.

    Este procedimiento es necesario porque la fuente de cambios solo emite eventos al insertar o actualizar elementos, no al eliminarlos.

1. Presione Entrar una vez más para cerrar el programa y limpiar sus recursos.
