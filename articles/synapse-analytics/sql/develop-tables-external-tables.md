---
title: Uso de tablas externas con Synapse SQL
description: Lectura o escritura de archivos de datos con Synapse SQL
services: synapse-analytics
author: julieMSFT
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: ''
ms.date: 04/15/2020
ms.author: jrasnick
ms.reviewer: jrasnick
ms.openlocfilehash: 4d13d15fe950c89687acfca355d4ed183756536a
ms.sourcegitcommit: b80aafd2c71d7366838811e92bd234ddbab507b6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2020
ms.locfileid: "81419979"
---
# <a name="use-external-tables-with-synapse-sql"></a>Uso de tablas externas con Synapse SQL

Una tabla externa apunta a datos ubicados en Hadoop, Azure Storage Blob o Azure Data Lake Storage. Las tablas externas se usan para leer datos de archivos de Azure Storage o escribir datos en ellos. Con Synapse SQL, se pueden usar tablas externas para leer y escribir datos en un grupo de SQL o en SQL a petición (versión preliminar).

## <a name="external-tables-in-sql-pool"></a>Tablas externas en un grupo de SQL

En un grupo de SQL, se puede usar una tabla externa para:

- Consultar Azure Blob Storage y Azure Data Lake Gen2 con instrucciones Transact-SQL.
- Importar y almacenar datos de Azure Blob Storage y Azure Data Lake Storage en un grupo de SQL.

Si se usa en combinación con la instrucción [CREATE TABLE AS SELECT](../sql-data-warehouse/sql-data-warehouse-develop-ctas.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json), al realizar la selección de una tabla externa importa los datos en una tabla de un grupo de SQL. Además de para la [instrucción COPY](/sql/t-sql/statements/copy-into-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest), las tablas externas son útiles para cargar datos. Si desea ver un tutorial de carga, consulte el artículo en el que se explica el [uso de PolyBase para cargar datos de Azure Blob Storage](../sql-data-warehouse/load-data-from-azure-blob-storage-using-polybase.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

## <a name="external-tables-in-sql-on-demand-preview"></a>Tablas externas en SQL a petición (versión preliminar)

En el caso de SQL a petición, usará una tabla externa para:

- Consultar datos de Azure Blob Storage y Azure Data Lake Storage con instrucciones Transact-SQL.
- Almacenar los resultados de consultas de SQL a petición en archivos de Azure Blob Storage o Azure Data Lake Storage mediante [CETAS](develop-tables-cetas.md).

Puede crear tablas externas mediante SQL a petición. Para ello debe seguir estos pasos:

1. CREATE EXTERNAL DATA SOURCE
2. CREATE EXTERNAL FILE FORMAT
3. CREATE EXTERNAL TABLE

## <a name="create-external-data-source"></a>CREATE EXTERNAL DATA SOURCE

Los orígenes de datos externos se usan para conectarse a las cuentas de almacenamiento. La documentación completa se describe [aquí](/sql/t-sql/statements/create-external-data-source-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest).

## <a name="syntax-for-create-external-data-source"></a>Sintaxis de CREATE EXTERNAL DATA SOURCE

```syntaxsql
CREATE EXTERNAL DATA SOURCE <data_source_name>
WITH
(    LOCATION         = '<prefix>://<path>' )
[;]
```

## <a name="arguments-for-create-external-data-source"></a>Argumentos de CREATE EXTERNAL DATA SOURCE

data_source_name: especifica el nombre definido por el usuario para el origen de datos. El nombre debe ser único en la base de datos.

LOCATION = `'<prefix>://<path>'`: proporciona el protocolo de conectividad y la ruta de acceso al origen de datos externo. La ruta de acceso puede incluir un contenedor, cuya forma sería `'<prefix>://<path>/container'`, y una carpeta, con la forma de `'<prefix>://<path>/container/folder'`.

| Origen de datos externo        | Prefijo de ubicación | Ruta de acceso de ubicación                                         |
| --------------------------- | --------------- | ----------------------------------------------------- |
| Azure Blob Storage          | `wasb[s]`       | `<container>@<storage_account>.blob.core.windows.net` |
| Azure Data Lake Store Gen1 | `adl`           | `<storage_account>.azuredatalake.net`                 |
| Azure Data Lake Store Gen2 | `abfs[s]`       | `<container>@<storage_account>.dfs.core.windows.net`  |

## <a name="example-for-create-external-data-source"></a>Ejemplo de CREATE EXTERNAL DATA SOURCE

En el ejemplo siguiente se crea un origen de datos externo para Azure Data Lake Gen2 que apunta al conjunto de datos de Nueva York:

```sql
CREATE EXTERNAL DATA SOURCE AzureDataLakeStore
WITH
  -- Please note the abfss endpoint when your account has secure transfer enabled
  ( LOCATION = 'abfss://newyorktaxidataset.azuredatalakestore.net' ,
    CREDENTIAL = ADLS_credential ,
    TYPE = HADOOP
  ) ;
```

## <a name="create-external-file-format"></a>CREATE EXTERNAL FILE FORMAT

Crea un objeto de formato de archivo externo que define los datos externos almacenados en Azure Blob Storage o Azure Data Lake Store. La creación de un formato de archivo externo es un requisito previo para crear una tabla externa. La documentación completa se encuentra [aquí](/sql/t-sql/statements/create-external-file-format-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest).

Al crear un formato de archivo externo, se especifica el diseño real de los datos a los que hace referencia una tabla externa.

## <a name="syntax-for-create-external-file-format"></a>Sintaxis de CREATE EXTERNAL FILE FORMAT

```syntaxsql
-- Create an external file format for PARQUET files.  
CREATE EXTERNAL FILE FORMAT file_format_name  
WITH (  
    FORMAT_TYPE = PARQUET  
    [ , DATA_COMPRESSION = {  
        'org.apache.hadoop.io.compress.SnappyCodec'  
      | 'org.apache.hadoop.io.compress.GzipCodec'      }  
    ]);  

--Create an external file format for DELIMITED TEXT files
CREATE EXTERNAL FILE FORMAT file_format_name  
WITH (  
    FORMAT_TYPE = DELIMITEDTEXT  
    [ , FORMAT_OPTIONS ( <format_options> [ ,...n  ] ) ]  
    );  

<format_options> ::=  
{  
    FIELD_TERMINATOR = field_terminator  
    | STRING_DELIMITER = string_delimiter
    | First_Row = integer
    | USE_TYPE_DEFAULT = { TRUE | FALSE }
    | Encoding = {'UTF8' | 'UTF16'}
}
```

## <a name="arguments-for-create-external-file-format"></a>Argumentos para CREATE EXTERNAL FILE FORMAT

file_format_name: especifica un nombre para el formato de archivo externo.

FORMAT_TYPE = [ PARQUET | DELIMITEDTEXT]: especifica el formato de los datos externos.

- PARQUET: especifica un formato Parquet.
- DELIMITEDTEXT: especifica un formato de texto con delimitadores de columna, también denominados terminadores de campo.

FIELD_TERMINATOR = *field_terminator*: solo se aplica a archivos de texto delimitado. El terminador de campo especifica uno o varios caracteres que marcan el final de cada campo (columna) en el archivo de texto delimitado. El valor predeterminado es el carácter de barra vertical ("|").

Ejemplos:

- FIELD_TERMINATOR = '|'
- FIELD_TERMINATOR = ' '
- FIELD_TERMINATOR = ꞌ\tꞌ

STRING_DELIMITER = *string_delimiter*: especifica el terminador de campo de los datos de tipo cadena en el archivo de texto delimitado. El delimitador de cadena tiene una longitud de uno o más caracteres y se escribe entre comillas simples. El valor predeterminado es la cadena vacía ("").

Ejemplos:

- STRING_DELIMITER = '"'
- STRING_DELIMITER = '*'
- STRING_DELIMITER = ꞌ,ꞌ

FIRST_ROW = *First_row_int*: especifica el número de fila que se lee primero y se aplica a todos los archivos. Si establece el valor en dos hará que se omita la primera fila de cada archivo (fila de encabezado) al cargar los datos. Las filas se omiten en función de la existencia de terminadores de fila (/r/n, /r, /n).

USE_TYPE_DEFAULT = { TRUE | **FALSE** }: especifica cómo administrar valores que faltan en archivos de texto delimitado al recuperar datos del archivo de texto.

TRUE: si va a recuperar datos del archivo de texto, almacene todos los valores que falten mediante el tipo de datos del valor predeterminado para la columna correspondiente en la definición de la tabla externa. Por ejemplo, reemplace un valor que falta con:

- 0 si la columna se define como una columna numérica. Las columnas decimales no se admiten y provocarán un error.
- Cadena vacía ("") si la columna es una columna de cadena.
- 1900-01-01 si la columna es una columna de fecha.

FALSE: almacena todos los valores que faltan como NULL. Los valores NULL que se almacenan mediante la palabra NULL en el archivo de texto delimitado se importan como la cadena 'NULL'.

Encoding = {' UTF8 ' | ' UTF16 '}: SQL a petición puede leer archivos de texto delimitados con codificación UTF8 y UTF16.

DATA_COMPRESSION = *data_compression_method*: este argumento especifica el método de compresión de datos para los datos externos. Cuando se lee de tablas externas, se omite. Solo se usa al escribir en tablas externas mediante [CETAS](develop-tables-cetas.md).

El tipo de formato PARQUET admite estos métodos de compresión:

- DATA_COMPRESSION = 'org.apache.hadoop.io.compress.GzipCodec'
- DATA_COMPRESSION = 'org.apache.hadoop.io.compress.SnappyCodec'

## <a name="example-for-create-external-file-format"></a>Ejemplo de CREATE EXTERNAL FILE FORMAT

En el ejemplo siguiente se crea un formato de archivo externo para los archivos del censo:

```sql
CREATE EXTERNAL FILE FORMAT census_file_format
WITH
(  
    FORMAT_TYPE = PARQUET,
    DATA_COMPRESSION = 'org.apache.hadoop.io.compress.SnappyCodec'
)
```

## <a name="create-external-table"></a>CREATE EXTERNAL TABLE

El comando CREATE EXTERNAL TABLE crea una tabla externa para Synapse SQL para acceder a los datos almacenados en Azure Blob Storage o Azure Data Lake Storage. 

## <a name="syntax-for-create-external-table"></a>Sintaxis de CREATE EXTERNAL TABLE

```syntaxsql
CREATE EXTERNAL TABLE { database_name.schema_name.table_name | schema_name.table_name | table_name }
    ( <column_definition> [ ,...n ] )  
    WITH (
        LOCATION = 'folder_or_filepath',  
        DATA_SOURCE = external_data_source_name,  
        FILE_FORMAT = external_file_format_name
    )  
[;]  

<column_definition> ::=
column_name <data_type>
    [ COLLATE collation_name ]
```

## <a name="arguments-create-external-table"></a>Argumento de CREATE EXTERNAL TABLE

*{ database_name.schema_name.table_name | schema_name.table_name | table_name }*

Nombre de entre una y tres partes de la tabla que se va a crear. En el caso de una tabla externa, SQL a petición almacena solo los metadatos de la tabla. En SQL a petición no se mueven ni se almacenan datos reales.

<column_definition>, ...*n* ]

CREATE EXTERNAL TABLE admite la capacidad de configurar el nombre de columna, tipo de datos, nulabilidad e intercalación. No se puede usar DEFAULT CONSTRAINT en tablas externas.

>[!IMPORTANT]
>Las definiciones de columna, incluidos los tipos de datos y el número de columnas, deben coincidir con los datos de los archivos externos. Si hay algún error de coincidencia, se rechazarán las filas de archivo al consultar los datos reales.

Al leer archivos con formato Parquet, solo puede especificar las columnas que desea leer y omitir el resto.

LOCATION = '*folder_or_filepath*'

Especifica la carpeta o la ruta de acceso del archivo y el nombre de archivo de los datos reales en Azure Blob Storage. La ubicación empieza desde la carpeta raíz. La carpeta raíz es la ubicación de datos especificada en el origen de datos externo.

Si especifica un valor de LOCATION de una carpeta, una consulta de SQL a petición seleccionará la tabla externa y recuperará los archivos de la carpeta.

> [!NOTE]
> A diferencia de Hadoop y PolyBase, SQL a petición no devuelve subcarpetas. Devuelve los archivos cuyo nombre comienza por un carácter de subrayado (_) o un punto (.).

En este ejemplo, si LOCATION='/webdata/', una consulta de SQL a petición devolverá filas de mydata.txt y _hidden.txt. No devolverá mydata2. txt y mydata3. txt porque se encuentran en una subcarpeta.

![Datos recursivos para tablas externas](./media/develop-tables-external-tables/folder-traversal.png)

DATA_SOURCE = *external_data_source_name*: especifica el nombre del origen de datos externo que contiene la ubicación de los datos externos. Para crear un origen de datos externo, use[CREATE EXTERNAL DATA SOURCE](#create-external-data-source).

FILE_FORMAT = *external_file_format_name*: especifica el nombre del objeto de formato de archivo externo que almacena el tipo de archivo y el método de compresión de los datos externos. Para crear un formato de archivo externo, use [CREATE EXTERNAL FILE FORMAT](#create-external-file-format).

## <a name="permissions-create-external-table"></a>Permisos de CREATE EXTERNAL TABLE

Para realizar la selección en una tabla externa, se necesitan las credenciales adecuadas con permisos de lectura y de lista.

## <a name="example-create-external-table"></a>Ejemplo de CREATE EXTERNAL TABLE

En el ejemplo siguiente se crea una tabla externa. Devuelve la primera fila:

```sql
CREATE EXTERNAL TABLE census_external_table
(
    decennialTime varchar(20),
    stateName varchar(100),
    countyName varchar(100),
    population int,
    race varchar(50),
    sex    varchar(10),
    minAge int,
    maxAge int
)  
WITH (
    LOCATION = '/parquet/',
    DATA_SOURCE = population_ds,  
    FILE_FORMAT = census_file_format
)
GO

SELECT TOP 1 * FROM census_external_table
```

## <a name="create-and-query-external-tables-from-a-file-in-azure-data-lake"></a>Creación y consulta de tablas externas a partir de un archivo en Azure Data Lake

Mediante las funcionalidades de exploración de Data Lake ya se puede crear y consultar una tabla externa mediante un grupo de SQL o SQL a petición con un solo clic con el botón derecho en el archivo.

## <a name="prerequisites"></a>Prerrequisitos

- Debe acceder al área de trabajo al menos con el rol de acceso basado en ARM Colaborador de datos de blobs de almacenamiento a la cuenta de ADLS Gen2

- Debe tener al menos [permisos para crear](/sql/t-sql/statements/create-external-table-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest#permissions-2) y consultar tablas externas en el grupo de SQL o SQL a petición

- El servicio vinculado asociado a la cuenta de ADLS Gen2 **debe tener acceso al archivo**. Por ejemplo, si el mecanismo de autenticación del servicio vinculado es Identidad administrada, la identidad administrada del área de trabajo debe tener, al menos, permiso de lectura de blob de almacenamiento en la cuenta de almacenamiento

En el panel Data (Datos), seleccione el archivo desde el que desea crear la tabla externa:
> [!div class="mx-imgBorder"]
>![externaltable1](./media/develop-tables-external-tables/external-table-1.png)

Se abrirá una ventana de diálogo. Seleccione un grupo de SQL o SQL a petición, asigne un nombre a la tabla y seleccione Abrir script:

> [!div class="mx-imgBorder"]
>![externaltable2](./media/develop-tables-external-tables/external-table-2.png)

El script SQL se genera automáticamente e infiere el esquema del archivo:
> [!div class="mx-imgBorder"]
>![externaltable3](./media/develop-tables-external-tables/external-table-3.png)

Ejecute el script. El script ejecutará automáticamente una instrucción Select Top 100 *:
> [!div class="mx-imgBorder"]
>![externaltable4](./media/develop-tables-external-tables/external-table-4.png)

Ahora se crea la tabla externa, para una futura exploración del contenido de esta tabla externa el usuario puede consultarla directamente desde el panel Data (Datos):
> [!div class="mx-imgBorder"]
>![externaltable5](./media/develop-tables-external-tables/external-table-5.png)

## <a name="next-steps"></a>Pasos siguientes

Consulte el artículo [CETAS](develop-tables-cetas.md) para obtener información sobre cómo guardar los resultados de una consulta en una tabla externa en Azure Storage. También puede iniciar la consulta de [tablas de Spark](develop-storage-files-spark-tables.md).
