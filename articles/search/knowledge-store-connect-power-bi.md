---
title: Conexión a un almacén de conocimiento (versión preliminar) con Power BI
titleSuffix: Azure Cognitive Search
description: Conecte un almacén de conocimiento (versión preliminar) de Azure Cognitive Search con Power BI para su análisis y exploración.
author: HeidiSteen
ms.author: heidist
manager: nitinme
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/26/2019
ms.openlocfilehash: 4fd71a7f322cb2672eb485f17e4de2619a7c2d2c
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/28/2020
ms.locfileid: "78270024"
---
# <a name="connect-a-knowledge-store-with-power-bi"></a>Conexión a un almacén de conocimiento con Power BI

> [!IMPORTANT] 
> El almacén de conocimiento está actualmente en versión preliminar pública. La funcionalidad de versión preliminar se ofrece sin un Acuerdo de Nivel de Servicio y no es aconsejable usarla para cargas de trabajo de producción. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). En la [API REST versión 2019-05-06-Preview](search-api-preview.md) se proporcionan características en versión preliminar. Actualmente hay compatibilidad limitada con el portal y no la hay con el SDK de .NET.

En este artículo, aprenderá a conectarse a un almacén de conocimiento y a explorarlo mediante Power Query en la aplicación Power BI Desktop. Puede empezar a trabajar más rápido con plantillas o crear un panel personalizado desde cero.

+ Siga los pasos descritos en [Creación de un almacén de conocimiento en Azure Portal](knowledge-store-create-portal.md) o en [Creación de un almacén de conocimiento de Azure Cognitive Search mediante REST](knowledge-store-create-rest.md) para crear el almacén de conocimiento de ejemplo que se usa en este tutorial. También necesitará el nombre de la cuenta de Azure Storage que usó para crear el almacén de conocimiento, junto con su clave de acceso de Azure Portal.

+ [Instalación de Power BI Desktop](https://powerbi.microsoft.com/downloads/)

## <a name="sample-power-bi-template---azure-portal-only"></a>Plantilla de ejemplo de Power BI: solo Azure Portal

Al crear un [almacén de conocimiento con Azure Portal](knowledge-store-create-portal.md), tiene la opción de descargar una [plantilla de Power BI](https://github.com/Azure-Samples/cognitive-search-templates) en la segunda página del **Asistente para la importación de datos**. Esta plantilla proporciona varias visualizaciones, como WordCloud y Network Navigator, para el contenido basado en texto. 

Haga clic en **Obtener plantilla de Power BI** en la página **Agregar conocimientos cognitivos** para recuperar y descargar la plantilla desde su ubicación pública de GitHub. El asistente modifica la plantilla para acomodar la forma de los datos capturados en las proyecciones del almacén de conocimiento especificadas en dicho asistente. Por esta razón, la plantilla que descargue variará cada vez que ejecute el asistente, suponiendo que haya diferentes entradas de datos y selecciones de aptitudes.

![Plantilla de ejemplo de Power BI en Azure Cognitive Search](media/knowledge-store-connect-power-bi/powerbi-sample-template-portal-only.png "Plantilla de ejemplo de Power BI")

> [!NOTE]
> Aunque la plantilla se descarga mientras el asistente está en curso, tendrá que esperar a que el almacén de conocimiento se cree realmente en el almacenamiento de tablas de Azure antes de poder usarlo.

## <a name="connect-with-power-bi"></a>Conexión con Power BI

1. Inicie Power BI Desktop y haga clic en **Obtener datos**.

1. En la ventana **Obtener datos**, seleccione **Azure** y después **Azure Table Storage**.

1. Haga clic en **Conectar**.

1. En **URL o nombre de cuenta**, escriba el nombre de la cuenta de Azure Storage (la dirección URL completa se creará automáticamente).

1. Si se le solicita, escriba la clave de la cuenta de almacenamiento.

1. Seleccione las tablas que contienen los datos de reseñas de hoteles creados en los tutoriales anteriores. 

   + En el tutorial del portal, los nombres de las tablas son *hotelReviewsSsDocument*, *hotelReviewsSsEntities*, *hotelReviewsSsKeyPhrases* y *hotelReviewsSsPages*. 
   
   + En el tutorial de REST, los nombres de las tablas son *hotelReviewsDocument*, *hotelReviewsPages*, *hotelReviewsKeyPhrases* y *hotelReviewsSentiment*.

1. Haga clic en **Cargar**.

1. En la cinta de opciones superior, haga clic en **Editar consultas** para abrir el **Editor de Power Query**.

   ![Abrir Power Query](media/knowledge-store-connect-power-bi/powerbi-edit-queries.png "Abrir Power Query")

1. Seleccione *hotelReviewsSsDocument* y quite las columnas *PartitionKey*, *RowKey* y *Timestamp*. 
   ![Editar tablas](media/knowledge-store-connect-power-bi/powerbi-edit-table.png "Editar tablas")

1. Haga clic en el icono con flechas opuestas en el lado superior derecho de la tabla para expandir el *contenido*. Cuando aparezca la lista de columnas, seleccione todas las columnas y, a continuación, anule la selección de las columnas que comienzan por "metadata". Haga clic en **Aceptar** para mostrar las columnas seleccionadas.

   ![Editar tablas](media/knowledge-store-connect-power-bi/powerbi-expand-content-table.png "Expandir contenido")

1. Para cambiar el tipo de datos de las columnas siguientes, haga clic en el icono ABC-123 situado en la parte superior izquierda de la columna.

   + Para *content.latitude* y *Content.longitude*, seleccione **Número decimal**.
   + En *Content.reviews_date* y *Content.reviews_dateAdded*, seleccione **Fecha y hora**.

   ![Cambiar tipos de datos](media/knowledge-store-connect-power-bi/powerbi-change-type.png "Cambiar tipos de datos")

1. Seleccione *hotelReviewsSsPages* y repita los pasos 9 y 10 para eliminar las columnas y expandir el *contenido*.
1. Cambie el tipo de datos de *Content.SentimentScore* a **Número decimal**.
1. Seleccione *hotelReviewsSsKeyPhrases* y repita los pasos 9 y 10 para eliminar las columnas y expandir el *contenido*. No hay modificaciones de tipo de datos para esta tabla.

1. En la barra de comandos, haga clic en **Cerrar y aplicar**.

1. Haga clic en el icono Modelo en el panel de navegación izquierdo y compruebe que Power BI muestra relaciones entre las tres tablas.

   ![Validar relaciones](media/knowledge-store-connect-power-bi/powerbi-relationships.png "Validar relaciones")

1. Haga doble clic en cada relación y asegúrese de que la **dirección de filtro cruzado** está establecida en **Ambas**.  Esto permite que los objetos visuales se actualicen cuando se aplica un filtro.

1. Haga clic en el icono de informe en el panel de navegación izquierdo para explorar los datos mediante visualizaciones. En el caso de los campos de texto, las tablas y las tarjetas son visualizaciones útiles. Puede elegir campos de cada una de las tres tablas para rellenar la tabla o tarjeta. 

<!-- ## Try with larger data sets

We purposely kept the data set small to avoid charges for a demo walkthrough. For a more realistic experience, you can create and then attach a billable Cognitive Services resource to enable a larger number of transactions against the sentiment analyzer, keyphrase extraction, and language detector skills.

Create new containers in Azure Blob storage and upload each CSV file to its own container. Specify one of these containers in the data source creation step in Import data wizard.

| Description | Link |
|-------------|------|
| Free tier   | [HotelReviews_Free.csv](https://knowledgestoredemo.blob.core.windows.net/hotel-reviews/HotelReviews_Free.csv?st=2019-07-29T17%3A51%3A30Z&se=2021-07-30T17%3A51%3A00Z&sp=rl&sv=2018-03-28&sr=c&sig=LnWLXqFkPNeuuMgnohiz3jfW4ijePeT5m2SiQDdwDaQ%3D) |
| Small (500 Records) | [HotelReviews_Small.csv](https://knowledgestoredemo.blob.core.windows.net/hotel-reviews/HotelReviews_Small.csv?st=2019-07-29T17%3A51%3A30Z&se=2021-07-30T17%3A51%3A00Z&sp=rl&sv=2018-03-28&sr=c&sig=LnWLXqFkPNeuuMgnohiz3jfW4ijePeT5m2SiQDdwDaQ%3D) |
| Medium (6000 Records)| [HotelReviews_Medium.csv](https://knowledgestoredemo.blob.core.windows.net/hotel-reviews/HotelReviews_Medium.csv?st=2019-07-29T17%3A51%3A30Z&se=2021-07-30T17%3A51%3A00Z&sp=rl&sv=2018-03-28&sr=c&sig=LnWLXqFkPNeuuMgnohiz3jfW4ijePeT5m2SiQDdwDaQ%3D)
| Large (Full dataset 35000 Records) | [HotelReviews_Large.csv](https://knowledgestoredemo.blob.core.windows.net/hotel-reviews/HotelReviews_Large.csv?st=2019-07-29T17%3A51%3A30Z&se=2021-07-30T17%3A51%3A00Z&sp=rl&sv=2018-03-28&sr=c&sig=LnWLXqFkPNeuuMgnohiz3jfW4ijePeT5m2SiQDdwDaQ%3D). Be aware that very large data sets are expensive to process. This one costs roughly $1000 U.S dollars.|

In the enrichment step of the wizard, attach a billable [Cognitive Services](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account) resource, created at the *S0* tier, in the same region as Azure Cognitive Search to use larger data sets. 

  ![Create a Cognitive Services resource](media/knowledge-store-connect-power-bi/create-cognitive-service.png "Create a Cognitive Services resource") -->

## <a name="clean-up"></a>Limpieza

Cuando trabaje con su propia suscripción, es una buena idea al final de un proyecto identificar si todavía se necesitan los recursos que ha creado. Los recursos que se dejan en ejecución pueden costarle mucho dinero. Puede eliminar los recursos de forma individual o eliminar el grupo de recursos para eliminar todo el conjunto de recursos.

Puede encontrar y administrar recursos en el portal, mediante el vínculo **Todos los recursos** o **Grupos de recursos** en el panel de navegación izquierdo.

Si está usando un servicio gratuito, recuerde que está limitado a tres índices, indexadores y orígenes de datos. Puede eliminar elementos individuales en el portal para mantenerse por debajo del límite.

## <a name="next-steps"></a>Pasos siguientes

Para obtener información sobre cómo explorar este almacén de conocimiento con el Explorador de Storage, consulte el siguiente artículo.

> [!div class="nextstepaction"]
> [Visualización con el Explorador de Storage](knowledge-store-view-storage-explorer.md)