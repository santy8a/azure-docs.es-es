---
title: Sincronización del repositorio de GitHub con App Configuration
description: Uso de Acciones de GitHub para actualizar automáticamente la instancia de App Configuration cuando se actualiza el repositorio de GitHub.
author: lisaguthrie
ms.author: lcozzens
ms.date: 02/20/2020
ms.topic: conceptual
ms.service: azure-app-configuration
ms.openlocfilehash: 602ccddf97938022df3c5903b573608558fe5d35
ms.sourcegitcommit: 3c318f6c2a46e0d062a725d88cc8eb2d3fa2f96a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/02/2020
ms.locfileid: "80585478"
---
# <a name="sync-your-github-repository-to-app-configuration"></a>Sincronización del repositorio de GitHub con App Configuration

Los equipos que quieran seguir usando sus procedimientos de control de código fuente existentes pueden usar Acciones de GitHub para sincronizar automáticamente su repositorio de GitHub con el almacén de App Configuration. Esto le permite realizar cambios en los archivos de configuración como haría normalmente y, al mismo tiempo, obtener ventajas adicionales de App Configuration como: <br>
&nbsp;&nbsp;&nbsp;&nbsp;• Configuración centralizada fuera del código <br>
&nbsp;&nbsp;&nbsp;&nbsp;• Actualizaciones de la configuración sin volver a implementar toda la aplicación <br>
&nbsp;&nbsp;&nbsp;&nbsp;• Integración con servicios como Azure App Service y Functions 

Un [flujo de trabajo](https://help.github.com/articles/about-github-actions#workflow) de Acciones de GitHub define un proceso automatizado en un repositorio de GitHub. La acción *Sincronización de Azure App Configuration* desencadena actualizaciones en una instancia de configuración de App Configuration cuando se realizan cambios en el repositorio de origen. Asimismo, usa un archivo YAML (.yml) que se encuentra en la ruta de acceso `/.github/workflows/` del repositorio para definir los pasos y los parámetros. Puede desencadenar actualizaciones de configuración al insertar, revisar o bifurcar archivos de configuración de la aplicación, de la misma forma que lo hace con el código de la aplicación.

La [documentación](https://help.github.com/actions/automating-your-workflow-with-github-actions/configuring-a-workflow) de GitHub proporciona una vista detallada de los flujos de trabajo y las acciones de GitHub. 

## <a name="enable-github-actions-in-your-repository"></a>Habilitación de Acciones de GitHub en el repositorio
Para empezar a usar esta acción de GitHub, vaya al repositorio y seleccione la pestaña **Acciones**. Haga clic en **New workflow** (Nuevo flujo de trabajo) y, luego, en **Set up a workflow yourself** (Configurar un flujo de trabajo por su cuenta). Por último, busque en Marketplace "Azure App Configuration Sync".
> [!div class="mx-imgBorder"]
> ![Selección de la pestaña Acción](media/find-github-action.png)

> [!div class="mx-imgBorder"]
> ![Selección de la acción Azure App Configuration Sync](media/app-configuration-sync-action.png)

## <a name="sync-configuration-files-after-a-push"></a>Sincronización de los archivos de configuración después de una inserción
Esta acción sincroniza los archivos de Azure App Configuration cuando se inserta un cambio en `appsettings.json`. Cuando un desarrollador inserta un cambio en `appsettings.json`, la acción App Configuration Sync actualiza la instancia de App Configuration con los nuevos valores.

La primera sección de este flujo de trabajo especifica que la acción se desencadena *en* una *inserción* que contiene `appsettings.json` para la bifurcación *maestra*. En la segunda sección se enumeran los trabajos que se ejecutan una vez que se desencadena la acción. La acción desprotege los archivos pertinentes y actualiza la instancia de App Configuration con la cadena de conexión almacenada como un secreto en el repositorio.  Para más información sobre el uso de secretos en GitHub, consulte este [artículo de GitHub](https://help.github.com/actions/automating-your-workflow-with-github-actions/creating-and-using-encrypted-secrets) sobre la creación y el uso de secretos cifrados.

```json
on: 
  push: 
    branches: 
      - 'master' 
    paths: 
      - 'appsettings.json' 
 
jobs: 
  syncconfig: 
    runs-on: ubuntu-latest 
    steps: 
      # checkout done so that files in the repo can be read by the sync 
      - uses: actions/checkout@v1 
      - uses: azure/appconfiguration-sync@v1 
        with: 
          configurationFile: 'appsettings.json' 
          format: 'json' 
          # Replace <ConnectionString> with the name of the secret in your                        
          # repository 
          connectionString: ${{ secrets.<ConnectionString> }} 
          separator: ':' 
```

## <a name="use-a-dynamic-label-on-sync"></a>Uso de una etiqueta dinámica en la sincronización
La acción anterior actualiza la instancia de App Configuration cada vez que se actualiza `appsettings.json`. Esta acción inserta una etiqueta dinámica en cada sincronización, lo que garantiza que cada una puede identificarse de forma única y permite que los cambios en el código se asignen a cambios en la configuración.

La primera sección de este flujo de trabajo especifica que la acción se desencadena *en* una *inserción* que contiene `appsettings.json` para la bifurcación *maestra*. En la segunda sección se ejecuta un trabajo que crea una etiqueta única para la actualización de la configuración según el hash de confirmación. Después, el trabajo actualiza la instancia de App Configuration con los nuevos valores y la etiqueta única para esta actualización.

```json
on: 
  push: 
    branches: 
      - 'master' 
    paths: 
      - 'appsettings.json' 
 
jobs: 
  syncconfig: 
    runs-on: ubuntu-latest 
    steps: 
      # Creates a label based on the branch name and the first 8 characters          
      # of the commit hash 
      - id: determine_label 
        run: echo ::set-output name=LABEL::"${GITHUB_REF#refs/*/}/${GITHUB_SHA:0:8}" 
      # checkout done so that files in the repo can be read by the sync 
      - uses: actions/checkout@v1 
      - uses: azure/appconfiguration-sync@v1 
        with: 
          configurationFile: 'appsettings.json' 
          format: 'json' 
          # Replace <ConnectionString> with the name of the secret in your 
          # repository 
          connectionString: ${{ secrets.<ConnectionString> }}  
          separator: ':' 
          label: ${{ steps.determine_label.outputs.LABEL }} 
```

## <a name="use-strict-sync"></a>Uso de una sincronización estricta
Cuando el modo estricto está habilitado, la sincronización garantiza que la instancia de App Configuration coincida exactamente con el archivo de configuración para el prefijo y la etiqueta especificados. Se eliminan los pares clave-valor con el mismo prefijo y etiqueta que no están en el archivo de configuración. 
 
Si el modo estricto no está habilitado, la sincronización solo establecerá los pares de clave-valor del archivo de configuración. No se eliminará ningún par clave-valor. 

```json
on: 
  push: 
    branches: 
      - 'master' 
    paths: 
      - 'appsettings.json' 
 
jobs: 
  syncconfig: 
    runs-on: ubuntu-latest 
    steps: 
      # checkout done so that files in the repo can be read by the sync 
      - uses: actions/checkout@v1 
      - uses: azure/appconfiguration-sync@v1 
        with: 
          configurationFile: 'appsettings.json' 
          format: 'json' 
          # Replace <ConnectionString> with the name of the secret in your 
          # repository 
          connectionString: ${{ secrets.<ConnectionString> }}  
          separator: ':' 
          label: 'Label' 
          prefix: 'Prefix:' 
          strict: true 
```

## <a name="use-max-depth-to-limit-github-action"></a>Uso de la profundidad máxima para limitar la acción de GitHub
El comportamiento predeterminado de los atributos JSON anidados es aplanar todo el objeto.  El código JSON siguiente define este par clave-valor:

| Clave | Value |
| --- | --- |
| Object:Inner:InnerKey | InnerValue |

```json
{ "Object": 
    { "Inner":
        {
        "InnerKey": "InnerValue"
        }
    }
}
```

Si el objeto anidado está pensado para ser el valor insertado en la instancia de configuración, puede usar el valor de *profundidad* para detener el acoplamiento con la profundidad adecuada. 

```json
on: 
  push: 
    branches: 
      - 'master' 
    paths: 
      - 'appsettings.json' 
 
jobs: 
  syncconfig: 
    runs-on: ubuntu-latest 
    steps: 
      # checkout done so that files in the repo can be read by the sync 
      - uses: actions/checkout@v1 
      - uses: azure/appconfiguration-sync@v1 
        with: 
          configurationFile: 'appsettings.json' 
          format: 'json' 
          # Replace <ConnectionString> with the name of the secret in your 
          # repository 
          connectionString: ${{ secrets.<ConnectionString> }}  
          separator: ':' 
          depth: 2 
```

Dada una profundidad de 2, el ejemplo anterior devuelve ahora el siguiente par clave-valor:

| Clave | Value |
| --- | --- |
| Object:Inner | {"InnerKey":"InnerValue"} |

## <a name="understand-action-inputs"></a>Descripción de las entradas de acción
Los parámetros de entrada especifican los datos usados por la acción durante el tiempo de ejecución.  La tabla siguiente contiene los parámetros de entrada aceptados por la sincronización de App Configuration y los valores esperados para cada uno.  Para más información sobre las entradas de acción para Acciones de GitHub, consulte la [documentación](https://help.github.com/actions/automating-your-workflow-with-github-actions/metadata-syntax-for-github-actions#inputs) de GitHub.

> [!Note]
> Los identificadores de entradas no distinguen mayúsculas de minúsculas.


| Nombre de entrada | ¿Necesario? | Value |
|----|----|----|
| configurationFile | Sí | Ruta de acceso relativa al archivo de configuración en el repositorio.  Se admiten patrones Glob y pueden incluir varios archivos. |
| format | Sí | Formato del archivo de configuración.  Los formatos válidos son: Propiedades JSON, YAML. |
| connectionString | Sí | Cadena de conexión para la instancia de App Configuration. La cadena de conexión debe almacenarse como un secreto en el repositorio de GitHub y solo se debe usar el nombre de secreto en el flujo de trabajo. |
| separador | Sí | Separador utilizado al aplanar el archivo de configuración en pares de clave y valor.  Los valores válidos son: , ; : - _ __ / |
| prefix | No | Prefijo que se va a agregar al inicio de las claves. |
| etiqueta | No | Etiqueta que se usa al establecer pares de clave-valor. Si no se especifica, se usa una etiqueta null. |
| strict | No | Valor booleano que determina si está habilitado el modo estricto. El valor predeterminado es false. |
| depth | No | Profundidad máxima para aplanar el archivo de configuración.  La profundidad debe ser un número positivo.  El valor predeterminado no tendrá una profundidad máxima. |
| etiquetas | No | Especifica el conjunto de etiquetas en los pares de clave-valor.  El formato esperado es un formato de cadena de un objeto JSON con la siguiente forma: { [propertyName: string]: string; } Cada propiedad nombre-valor se convierte en una etiqueta. |

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido sobre la acción de GitHub de sincronización de App Configuration y del modo en que se puede usar para automatizar las actualizaciones de la instancia de App Configuration. Para información sobre cómo reacciona Azure App Configuration a los cambios en los pares de clave-valor, continúe con el siguiente [artículo](./concept-app-configuration-event.md).
