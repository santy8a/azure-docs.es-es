---
title: Ejecución de un runbook en Azure Automation
description: Describe los detalles de cómo se procesa un runbook en Azure Automation.
services: automation
ms.subservice: process-automation
ms.date: 04/14/2020
ms.topic: conceptual
ms.openlocfilehash: 274e621408deaf2aafcc2600d1097efe684055fc
ms.sourcegitcommit: b9d4b8ace55818fcb8e3aa58d193c03c7f6aa4f1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2020
ms.locfileid: "82582471"
---
# <a name="runbook-execution-in-azure-automation"></a>Ejecución de un runbook en Azure Automation

La automatización de procesos en Azure Automation permite crear y administrar PowerShell, el flujo de trabajo de PowerShell y los runbooks gráficos. Para ver los detalles, consulte [Runbooks de Azure Automation](automation-runbook-types.md). 

Automation ejecuta los runbooks según la lógica definida dentro de ellos. Si se interrumpe un runbook, se reinicia desde el principio. Este comportamiento requiere que escriba runbooks que admitan el reinicio si se producen problemas transitorios.

Al iniciar un runbook en Azure Automation se crea un trabajo, que es una instancia de ejecución única del runbook. Cada trabajo accede a los recursos de Azure mediante una conexión a la suscripción a Azure. El trabajo solo accede a los recursos de su centro de datos si estos recursos están accesibles desde la nube pública.

Azure Automation asigna un rol de trabajo para ejecutar cada trabajo durante la ejecución de un runbook. Aunque los trabajadores los comparten varias cuentas de Azure, los trabajos de diferentes cuentas de Automation están aislados entre sí. No puede controlar qué servicios de rol de trabajo solicita su trabajo.

Al ver la lista de runbooks en Azure Portal, se muestra el estado de cada trabajo iniciado para cada runbook. Azure Automation almacena los registros de trabajo durante un máximo de 30 días.

En el siguiente diagrama se muestra el ciclo de vida de un trabajo de runbook para [runbooks de PowerShell](automation-runbook-types.md#powershell-runbooks), [runbooks de flujo de trabajo de PowerShell](automation-runbook-types.md#powershell-workflow-runbooks) y [runbooks gráficos](automation-runbook-types.md#graphical-runbooks).

![Estados de trabajo: flujo de trabajo de PowerShell](./media/automation-runbook-execution/job-statuses.png)

[!INCLUDE [GDPR-related guidance](../../includes/gdpr-dsr-and-stp-note.md)]

>[!NOTE]
>Este artículo se ha actualizado para usar el nuevo módulo Az de Azure PowerShell. Aún puede usar el módulo de AzureRM que continuará recibiendo correcciones de errores hasta diciembre de 2020 como mínimo. Para más información acerca del nuevo módulo Az y la compatibilidad con AzureRM, consulte [Introducing the new Azure PowerShell Az module](https://docs.microsoft.com/powershell/azure/new-azureps-module-az?view=azps-3.5.0) (Presentación del nuevo módulo Az de Azure PowerShell). Para obtener instrucciones sobre la instalación del módulo Az en Hybrid Runbook Worker, consulte [Instalación del módulo de Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-az-ps?view=azps-3.5.0). Puede actualizar los módulos de su cuenta de Automation a la versión más reciente mediante [Actualización de módulos de Azure PowerShell en Azure Automation](automation-update-azure-modules.md).

## <a name="where-to-run-your-runbooks"></a>Dónde ejecutar los runbooks

Los runbooks de Azure Automation puede ejecutarse en un espacio aislado de Azure o una instancia de [Hybrid Runbook Worker](automation-hybrid-runbook-worker.md). Cuando los runbooks están diseñados para autenticarse y ejecutarse en recursos de Azure, se ejecutan en un espacio aislado de Azure, que es un entorno compartido que pueden usar varios trabajos. Los trabajos con el mismo espacio aislado están sujetos a las limitaciones de recursos de dicho espacio.

>[!NOTE]
>El entorno de espacio aislado de Azure no admite operaciones interactivas. Impide el acceso a todos los servidores COM fuera de proceso. También requiere el uso de archivos MOF locales para runbooks que realizan llamadas a Win32.

Puede usar una instancia de Hybrid Runbook Worker para ejecutar runbooks directamente en el equipo que hospeda el rol y en los recursos locales del entorno. Azure Automation almacena y administra los runbooks y después los entrega a uno o más equipos asignados.

En la tabla siguiente se enumeran algunas tareas de ejecución de runbooks con el entorno de ejecución recomendado que se muestra para cada una.

|Tarea|Recomendación|Notas|
|---|---|---|
|Integración con recursos de Azure|Espacio aislado de Azure|Hospedado en Azure, la autenticación es más sencilla. Si usa una instancia de Hybrid Runbook Worker en una VM de Azure, puede usar [identidades administradas para recursos de Azure](automation-hrw-run-runbooks.md#managed-identities-for-azure-resources).|
|Obtención del rendimiento óptimo para administrar recursos de Azure|Espacio aislado de Azure|El script se ejecuta en el mismo entorno, que tiene menos latencia.|
|Minimizar los costos operativos|Espacio aislado de Azure|No hay sobrecarga de proceso y no hay necesidad de una VM.|
|Ejecución de script de ejecución prolongada|Hybrid Runbook Worker|Los espacios aislados de Azure tienen [límites de recursos](../azure-resource-manager/management/azure-subscription-service-limits.md#automation-limits).|
|Interacción con servicios locales|Hybrid Runbook Worker|Puede acceder directamente al equipo host, a los recursos de otros entornos en la nube o en el entorno local. |
|Requerimiento de software de terceros y archivos ejecutables|Hybrid Runbook Worker|Usted administra el sistema operativo y puede instalar el software.|
|Supervisar un archivo o carpeta con un runbook|Hybrid Runbook Worker|Use una [tarea del Monitor](automation-watchers-tutorial.md) en una instancia de Hybrid Runbook Worker.|
|Ejecución de un script con un uso intensivo de recursos|Hybrid Runbook Worker| Los espacios aislados de Azure tienen [límites de recursos](../azure-resource-manager/management/azure-subscription-service-limits.md#automation-limits).|
|Uso de módulos con requisitos específicos| Hybrid Runbook Worker|Ejemplos:</br> WinSCP: dependencia de winscp.exe </br> Administración de IIS: dependencia de la habilitación o administración de IIS.|
|Instalación de un módulo con un instalador|Hybrid Runbook Worker|Los módulos para espacios aislados deben admitir copias.|
|Uso de runbooks o módulos que requieren .NET Framework con una versión distinta de 4.7.2|Hybrid Runbook Worker|Los espacios aislados de Automation son compatibles con .NET Framework 4.7.2 y no se admite la actualización a una versión diferente.|
|Ejecución de scripts que requieren elevación|Hybrid Runbook Worker|Los espacios aislados no permiten elevación. Con una instancia de Hybrid Runbook Worker, puede desactivar UAC y usar [Invoke-Command](https://docs.microsoft.com/powershell/module/microsoft.powershell.core/invoke-command?view=powershell-7) al ejecutar el comando que requiere elevación.|
|Ejecución de scripts que requieren acceso a Instrumental de administración de Windows (WMI)|Hybrid Runbook Worker|Los trabajos que se ejecutan en espacios aislados en la nube no pueden acceder al proveedor de WMI. |

## <a name="using-modules-in-your-runbooks"></a>Uso de módulos en los runbooks

Azure Automation admite varios módulos predeterminados, incluidos los módulos AzureRM (AzureRM.Automation) y un módulo que contiene varios cmdlets internos. También se admiten módulos instalables, incluidos los módulos Az (Az.Automation), que actualmente se usan actualmente en lugar de los módulos AzureRM. Para más información sobre los módulos que están disponibles para sus runbooks y configuraciones de DSC, consulte [Administración de módulos en Azure Automation](shared-resources/modules.md).

## <a name="creating-resources"></a>Creación de recursos

Si el runbook crea un recurso, el script debe comprobar que el recurso ya exista antes de intentar crearlo. Este es un ejemplo básico.

```powershell
$vmName = "WindowsVM1"
$resourceGroupName = "myResourceGroup"
$myCred = Get-AutomationPSCredential "MyCredential"
$vmExists = Get-AzResource -Name $vmName -ResourceGroupName $resourceGroupName

if(!$vmExists)
    {
    Write-Output "VM $vmName does not exist, creating"
    New-AzVM -Name $vmName -ResourceGroupName $resourceGroupName -Credential $myCred
    }
else
    {
    Write-Output "VM $vmName already exists, skipping"
    }
```

## <a name="using-self-signed-certificates"></a>Uso de certificados autofirmados

Para usar certificados autofirmados en sus runbooks, consulte [Creación de un certificado nuevo](https://docs.microsoft.com/azure/automation/shared-resources/certificates#creating-a-new-certificate).

## <a name="handling-jobs"></a>Control de trabajos

Puede reutilizar el entorno de ejecución para los trabajos de la misma cuenta de Automation. Un runbook individual puede tener muchos trabajos que se ejecutan al mismo tiempo. Cuantos más trabajos se ejecuten al mismo tiempo, más a menudo se podrán enviar al mismo espacio aislado.

Los trabajos que se ejecutan en el mismo proceso de espacio aislado pueden afectarse entre sí. Un ejemplo es la ejecución del cmdlet [Disconnect-AzAccount](https://docs.microsoft.com/powershell/module/az.accounts/disconnect-azaccount?view=azps-3.7.0). La ejecución de este cmdlet desconecta cada trabajo de runbook en el proceso de espacio aislado compartido.

Es posible que los trabajos de PowerShell iniciados desde un runbook que se ejecuta en un espacio aislado de Azure no se puedan ejecutar en [modo de lenguaje de PowerShell](/powershell/module/microsoft.powershell.core/about/about_language_modes) completo. Para más información sobre cómo interactuar con los trabajos de Azure Automation, consulte [Recuperación del estado del trabajo mediante PowerShell](#retrieving-job-status-using-powershell).

### <a name="job-statuses"></a> Estados del trabajo

En la tabla siguiente se describen los estados posibles para un trabajo.

| Status | Descripción |
|:--- |:--- |
| Completed |El trabajo se completó correctamente. |
| Con error |Un runbook de flujo de trabajo de PowerShell o gráfico no se pudo compilar. Un runbook de script de PowerShell no se pudo iniciar o el trabajo tuvo una excepción. Consulte [Tipos de runbooks de Azure Automation](automation-runbook-types.md).|
| Error, esperando recursos |Se produjo un error en el trabajo porque ha alcanzado el límite de [distribución equilibrada](#fair-share) tres veces y se ha iniciado en el mismo punto de control o en el inicio del runbook cada vez. |
| En cola |El trabajo espera que los recursos en un trabajo de Automation estén disponibles para que se pueda iniciar. |
| Iniciando |El trabajo se ha asignado a un trabajador y el sistema lo está iniciando. |
| Reanudando |El sistema está reanudando el trabajo después de que se suspendió. |
| En ejecución |El trabajo se está ejecutando. |
| En ejecución, esperando recursos |El trabajo se ha descargado porque ha alcanzado el límite de distribución equilibrada. Se reanudará en breve desde su último punto de control. |
| Detenido |El trabajo lo detuvo el usuario antes de completarse. |
| Deteniéndose |El sistema está deteniendo el trabajo. |
| Suspended |Solo se aplica a [runbooks de flujo de trabajo de PowerShell y gráficos](automation-runbook-types.md). El trabajo lo ha suspendido el usuario, el sistema o un comando en el runbook. Si un runbook no tiene un punto de control, se inicia desde el principio. Si lo tiene, puede volver a empezar y reanudarse desde su último punto de comprobación. El sistema solo suspende el runbook cuando se produce una excepción. De forma predeterminada, la variable `ErrorActionPreference` se establece en Continue, lo que indica que el trabajo sigue ejecutándose tras un error. Si la variable de preferencia se establece en Stop, se suspende el trabajo tras un error.  |
| Suspendiendo |Solo se aplica a [Runbooks de flujo de trabajo de PowerShell y gráficos](automation-runbook-types.md). El sistema intenta suspender el trabajo a petición del usuario. El runbook debe alcanzar su siguiente punto de control antes de que se pueda suspender. Si ya ha pasado su último punto de control, se completa antes de que se pueda suspender. |

### <a name="viewing-job-status-from-the-azure-portal"></a>Visualización de un estado de trabajo desde el portal de Azure

Puede ver un resumen del estado de todos los trabajos del runbook o profundizar en los detalles de un trabajo específico del runbook en Azure Portal. También puede configurar la integración con el área de trabajo de Log Analytics para reenviar flujos de trabajos y el estado del trabajo del runbook. Para más información sobre la integración con los registros de Azure Monitor, consulte [Reenvío del estado del trabajo y flujos de trabajo de Automation a los registros de Azure Monitor](automation-manage-send-joblogs-log-analytics.md).

A la derecha de la cuenta de Automation seleccionada, puede ver un resumen de todos los trabajos de runbook en el icono **Estadísticas de trabajo**.

![Icono Estadísticas de trabajo](./media/automation-runbook-execution/automation-account-job-status-summary.png)

Este icono muestra un contador y una representación gráfica del estado de trabajo de cada trabajo ejecutado.

Al hacer clic en el icono aparece la página Trabajos, que incluye una lista resumida de todos los trabajos ejecutados. En esta página se muestra el estado, el nombre del runbook, la hora de inicio y la hora de finalización de cada trabajo.

![Página Trabajos de la cuenta de Automation](./media/automation-runbook-execution/automation-account-jobs-status-blade.png)

Para filtrar la lista de trabajos, puede seleccionar **Filtrar trabajos**. Filtre por un runbook específico, estado de trabajo o una opción de la lista desplegable y proporcione el intervalo de tiempo para la búsqueda.

![Filtrado por estado del trabajo](./media/automation-runbook-execution/automation-account-jobs-filter.png)

Como alternativa, puede ver los detalles resumidos de los trabajos de un runbook concreto; para ello, selecciónelo en la página Runbooks de la cuenta de Automation y, después, seleccione el icono **Trabajos**. Esta acción presenta la página Trabajos. Desde aquí, puede hacer clic en el registro del trabajo para ver sus detalles y resultados.

![Página Trabajos de la cuenta de Automation](./media/automation-runbook-execution/automation-runbook-job-summary-blade.png)

### <a name="viewing-the-job-summary"></a>Visualización del resumen del trabajo

El resumen del trabajo descrito anteriormente permite ver una lista de todos los trabajos que se han creado para un runbook determinado y su estado más reciente. Para ver información detallada y la salida de un trabajo, haga clic en su nombre en la lista. La vista detallada del trabajo incluye los valores para los parámetros de runbook que se proporcionaron con ese trabajo.

Puede utilizar los pasos siguientes para ver los trabajos de un runbook.

1. En Azure Portal, seleccione **Automation** y después elija el nombre de una cuenta de Automation.
2. Del centro de conectividad, seleccione **Runbooks** en **Automatización de procesos**.
3. En la página Runbooks, seleccione un runbook de la lista.
3. En la página del runbook seleccionado, haga clic en el icono **Trabajos**.
4. Haga clic en uno de los trabajos de la lista y vea sus detalles y la salida en la página de detalles del trabajo.

### <a name="retrieving-job-status-using-powershell"></a>Recuperación del estado del trabajo con PowerShell

Puede utilizar el cmdlet [Get-AzAutomationJob](https://docs.microsoft.com/powershell/module/Az.Automation/Get-AzAutomationJob?view=azps-3.7.0) para recuperar los trabajos creados para un runbook y los detalles de un trabajo determinado. Si inicia un runbook con PowerShell mediante `Start-AzAutomationRunbook`, se devuelve el trabajo resultante. Utilice [Get-AzAutomationJobOutput](https://docs.microsoft.com/powershell/module/Az.Automation/Get-AzAutomationJobOutput?view=azps-3.5.0) para recuperar la salida del trabajo.

En el ejemplo siguiente se recupera el último trabajo para un runbook de ejemplo y se muestra su estado, los valores proporcionados para los parámetros del runbook y la salida del trabajo.

```azurepowershell-interactive
$job = (Get-AzAutomationJob –AutomationAccountName "MyAutomationAccount" `
–RunbookName "Test-Runbook" -ResourceGroupName "ResourceGroup01" | sort LastModifiedDate –desc)[0]
$job.Status
$job.JobParameters
Get-AzAutomationJobOutput -ResourceGroupName "ResourceGroup01" `
–AutomationAccountName "MyAutomationAcct" -Id $job.JobId –Stream Output
```

En el ejemplo siguiente se recupera la salida de un trabajo específico y se devuelve cada registro. Si hay una excepción para uno de los registros, el script escribe la excepción en lugar del valor. Este comportamiento resulta útil, ya que las excepciones pueden proporcionar información adicional que podría no registrarse normalmente durante la salida.

```azurepowershell-interactive
$output = Get-AzAutomationJobOutput -AutomationAccountName <AutomationAccountName> -Id <jobID> -ResourceGroupName <ResourceGroupName> -Stream "Any"
foreach($item in $output)
{
    $fullRecord = Get-AzAutomationJobOutputRecord -AutomationAccountName <AutomationAccountName> -ResourceGroupName <ResourceGroupName> -JobId <jobID> -Id $item.StreamRecordId
    if ($fullRecord.Type -eq "Error")
    {
        $fullRecord.Value.Exception
    }
    else
    {
    $fullRecord.Value
    }
}
```

## <a name="getting-details-from-the-activity-log"></a>Obtención de detalles del registro de actividad

Puede recuperar detalles del runbook, como la persona o la cuenta que ha iniciado el runbook, del registro de actividad para la cuenta de Automation. En el siguiente ejemplo de PowerShell se proporciona el último usuario que ha ejecutado el runbook especificado.

```powershell-interactive
$SubID = "00000000-0000-0000-0000-000000000000"
$AutomationResourceGroupName = "MyResourceGroup"
$AutomationAccountName = "MyAutomationAccount"
$RunbookName = "MyRunbook"
$StartTime = (Get-Date).AddDays(-1)
$JobActivityLogs = Get-AzLog -ResourceGroupName $AutomationResourceGroupName -StartTime $StartTime `
                                | Where-Object {$_.Authorization.Action -eq "Microsoft.Automation/automationAccounts/jobs/write"}

$JobInfo = @{}
foreach ($log in $JobActivityLogs)
{
    # Get job resource
    $JobResource = Get-AzResource -ResourceId $log.ResourceId

    if ($JobInfo[$log.SubmissionTimestamp] -eq $null -and $JobResource.Properties.runbook.name -eq $RunbookName)
    {
        # Get runbook
        $Runbook = Get-AzAutomationJob -ResourceGroupName $AutomationResourceGroupName -AutomationAccountName $AutomationAccountName `
                                            -Id $JobResource.Properties.jobId | ? {$_.RunbookName -eq $RunbookName}

        # Add job information to hashtable
        $JobInfo.Add($log.SubmissionTimestamp, @($Runbook.RunbookName,$Log.Caller, $JobResource.Properties.jobId))
    }
}
$JobInfo.GetEnumerator() | sort key -Descending | Select-Object -First 1
```

## <a name="tracking-progress"></a>Seguimiento del progreso

Se recomienda crear los runbooks para que tengan una naturaleza modular, con una lógica que se pueda reutilizar y reiniciar fácilmente. Una buena manera de garantizar que la lógica de un runbook se ejecuta correctamente si se han producido problemas es realizar el seguimiento del progreso del runbook. Es posible hacer un seguimiento del progreso del runbook mediante un origen externo, como una cuenta de almacenamiento, una base de datos o archivos compartidos. Puede crear lógica en el runbook para comprobar primero el estado de la última acción que se realizó. Luego, en función de los resultados de la comprobación, la lógica puede omitir o continuar tareas específicas del runbook.

## <a name="preventing-concurrent-jobs"></a>Evitación de trabajos simultáneos

Algunos runbooks se comportan de forma extraña si se ejecutan en varios trabajos al mismo tiempo. En este caso, es importante que un runbook implemente la lógica para establecer si ya hay un trabajo en ejecución. Este es un ejemplo básico.

```powershell
# Authenticate to Azure
$connection = Get-AutomationConnection -Name AzureRunAsConnection
Connect-AzAccount -ServicePrincipal -Tenant $connection.TenantID `
-ApplicationId $connection.ApplicationID -CertificateThumbprint $connection.CertificateThumbprint

$AzContext = Select-AzSubscription -SubscriptionId $connection.SubscriptionID

# Check for already running or new runbooks
$runbookName = "<RunbookName>"
$rgName = "<ResourceGroupName>"
$aaName = "<AutomationAccountName>"
$jobs = Get-AzAutomationJob -ResourceGroupName $rgName -AutomationAccountName $aaName -RunbookName $runbookName -AzContext $AzureContext

# Check to see if it is already running
$runningCount = ($jobs | ? {$_.Status -eq "Running"}).count

If (($jobs.status -contains "Running" -And $runningCount -gt 1 ) -Or ($jobs.Status -eq "New")) {
    # Exit code
    Write-Output "Runbook is already running"
    Exit 1
} else {
    # Insert Your code here
}
```

## <a name="handling-exceptions"></a>Control de excepciones

En esta sección se describen algunas maneras de controlar excepciones o problemas intermitentes en los runbooks. Un ejemplo es una excepción de WebSocket. El control de excepciones correcto evita que los errores de red transitorios provoquen un error en los runbooks. 

### <a name="erroractionpreference"></a>ErrorActionPreference

La variable [ErrorActionPreference](/powershell/module/microsoft.powershell.core/about/about_preference_variables#erroractionpreference) determina la manera en que PowerShell responde a un error de no terminación. Los errores de terminación siempre terminan y no se ven afectados por `ErrorActionPreference`.

Cuando el runbook usa `ErrorActionPreference`, un error normalmente de no terminación, como `PathNotFound` del cmdlet [Get-ChildItem](https://docs.microsoft.com/powershell/module/microsoft.powershell.management/get-childitem?view=powershell-7), impide que el runbook se complete. En el siguiente ejemplo se muestra el uso de `ErrorActionPreference`. El comando final [Write-Output](https://docs.microsoft.com/powershell/module/microsoft.powershell.utility/write-output?view=powershell-7) nunca se ejecuta, a medida que se detiene el script.

```powershell-interactive
$ErrorActionPreference = 'Stop'
Get-ChildItem -path nofile.txt
Write-Output "This message will not show"
```

### <a name="try-catch-finally"></a>Try Catch Finally

[Try Catch Finally](/powershell/module/microsoft.powershell.core/about/about_try_catch_finally) se usa en los scripts de PowerShell para controlar los errores de terminación. El script puede usar este mecanismo para detectar excepciones específicas o excepciones generales. La instrucción `catch` se debe usar para realizar el seguimiento de los errores o para intentar controlarlos. En el ejemplo siguiente se intenta descargar un archivo que no existe. Detecta la excepción `System.Net.WebException` y devuelve el último valor para cualquier otra excepción.

```powershell-interactive
try
{
   $wc = new-object System.Net.WebClient
   $wc.DownloadFile("http://www.contoso.com/MyDoc.doc")
}
catch [System.Net.WebException]
{
    "Unable to download MyDoc.doc from http://www.contoso.com."
}
catch
{
    "An error occurred that could not be resolved."
}
```

### <a name="throw"></a>Throw

[Throw](/powershell/module/microsoft.powershell.core/about/about_throw) se puede usar para generar un error de terminación. Este mecanismo puede ser útil al definir su propia lógica en un runbook. Si el script cumple un criterio que debe detenerlo, puede usar la instrucción `throw` para la detención. En el ejemplo siguiente se utiliza esta instrucción para mostrar un parámetro de función necesario.

```powershell-interactive
function Get-ContosoFiles
{
  param ($path = $(throw "The Path parameter is required."))
  Get-ChildItem -Path $path\*.txt -recurse
}
```

## <a name="handling-errors"></a>Control de errores

Los runbooks deben ser capaces de controlar los errores. PowerShell tiene dos tipos de errores, de terminación y de no terminación. Los errores de terminación detienen la ejecución del runbook cuando se producen. El runbook se detiene con un estado de trabajo de Error.

Los errores de no terminación permiten que un script continúe después de que sucedan. Un ejemplo de error de no terminación es aquel donde un runbook usa el cmdlet `Get-ChildItem` con una ruta de acceso que no existe. PowerShell se ve que la ruta de acceso no existe, genera un error y pasa a la carpeta siguiente. En este caso, el error no establece el estado del trabajo de runbook en Con error y el trabajo podría incluso completarse. Para forzar a un runbook a detenerse ante un error de no terminación, puede usar `ErrorAction Stop` en el cmdlet.

## <a name="using-executables-or-calling-processes"></a>Uso de archivos ejecutables o llamada a procesos

Los runbooks que se ejecutan en espacios aislados de Azure no admiten la llamada a procesos, por ejemplo, ejecutables (archivos **.exe**) ni subprocesos. El motivo es que un espacio aislado de Azure es un proceso compartido que se ejecuta en un contenedor que podría no poder acceder a todas las API subyacentes. En escenarios donde se requiere software de terceros o llamadas a subprocesos, debe ejecutar un runbook en una instancia de [Hybrid Runbook Worker](automation-hybrid-runbook-worker.md).

## <a name="accessing-device-and-application-characteristics"></a>Acceso a características de dispositivos y aplicaciones

Los trabajos de runbook que se ejecutan en espacios aislados de Azure no pueden acceder a las características de ningún dispositivo o aplicación. La API más común que se usa para consultar las métricas de rendimiento en Windows es WMI, donde algunas de las métricas comunes son la memoria y el uso de CPU. Sin embargo, no importa qué API se use, ya que los trabajos que se ejecutan en la nube no pueden acceder a la implementación de Microsoft de Web-Based Enterprise Management (WBEM). Esta plataforma se basa en el Modelo de información común (CIM), que proporciona los estándares del sector para definir las características de dispositivos y aplicaciones.

## <a name="working-with-webhooks"></a>Uso de webhooks

Los servicios externos, como Azure DevOps Services y GitHub, pueden iniciar un runbook en Azure Automation. Para realizar este tipo de inicio, el servicio usa un [webhook](automation-webhooks.md) a través de una única solicitud HTTP. El uso de un webhook permite iniciar runbooks sin necesidad de implementar una solución completa de Azure Automation. 

## <a name="supporting-time-dependent-scripts"></a>Compatibilidad con scripts dependientes del tiempo

Los runbooks deben ser sólidos y capaces de controlar los errores transitorios que pueden provocar que se reinicien o fallen. Si se produce un error en un runbook, Azure Automation vuelve a intentarlo.

Si el runbook se ejecuta normalmente dentro de una restricción de tiempo, haga que el script implemente la lógica para comprobar el tiempo de ejecución. Esta comprobación garantiza la ejecución de operaciones, como el inicio, el apagado o el escalado horizontal, solo durante horas específicas.

> [!NOTE]
> La hora local en el proceso de espacio aislado de Azure se establece en UTC. Debe tenerse en cuenta este aspecto en los cálculos de fecha y hora de los runbooks.

## <a name="working-with-multiple-subscriptions"></a>Trabajo con varias suscripciones

Para tratar con varias suscripciones, el runbook debe usar el cmdlet [Disable-AzContextAutosave](https://docs.microsoft.com/powershell/module/Az.Accounts/Disable-AzContextAutosave?view=azps-3.5.0). Este cmdlet garantiza que no se recupere el contexto de autenticación desde otro runbook que se ejecute en el mismo espacio aislado. El runbook también usa el parámetro `AzContext` en los cmdlets del módulo Az y le pasa el contexto adecuado.

```powershell
# Ensures that you do not inherit an AzContext in your runbook
Disable-AzContextAutosave –Scope Process

$Conn = Get-AutomationConnection -Name AzureRunAsConnection
Connect-AzAccount -ServicePrincipal `
-Tenant $Conn.TenantID `
-ApplicationId $Conn.ApplicationID `
-CertificateThumbprint $Conn.CertificateThumbprint

$context = Get-AzContext

$ChildRunbookName = 'ChildRunbookDemo'
$AutomationAccountName = 'myAutomationAccount'
$ResourceGroupName = 'myResourceGroup'

Start-AzAutomationRunbook `
    -ResourceGroupName $ResourceGroupName `
    -AutomationAccountName $AutomationAccountName `
    -Name $ChildRunbookName `
    -DefaultProfile $context
```

## <a name="sharing-resources-among-runbooks"></a><a name="fair-share"></a>Uso compartido de recursos entre runbooks

Para compartir recursos entre todos los runbooks de la nube, Azure Automation descarga o detiene temporalmente todos los trabajos que lleven más de tres horas en ejecución. Los trabajos de [runbooks de PowerShell](automation-runbook-types.md#powershell-runbooks) y [runbooks de Python](automation-runbook-types.md#python-runbooks) se detienen y no se reinician, y el estado del trabajo cambia a Detenido.

En el caso de tareas de larga duración, se recomienda usar una instancia de Hybrid Runbook Worker. Las instancias de Hybrid Runbook Worker no están limitadas por la distribución equilibrada y no tienen una limitación del tiempo que se puede ejecutar un runbook. Los demás [límites](../azure-resource-manager/management/azure-subscription-service-limits.md#automation-limits) de trabajo se aplican a los espacios aislados de Azure y a Hybrid Runbook Workers. Aunque las instancias de Hybrid Runbook Worker no están restringidas por el límite de distribución equilibrada de tres horas, debe desarrollar los runbooks para que se ejecutan en los roles de trabajo que admitan reinicios si se producen problemas inesperados en la infraestructura local.

Otra opción es optimizar un runbook mediante runbooks secundarios. Por ejemplo, el runbook podría recorrer en bucle la misma función en varios recursos, como una operación de base de datos en varias bases de datos. Puede mover esta función a un [runbook secundario](automation-child-runbooks.md) y hace que el runbook lo llame mediante [Start-AzAutomationRunbook](https://docs.microsoft.com/powershell/module/az.automation/start-azautomationrunbook?view=azps-3.7.0). Los runbooks secundarios se ejecutan en paralelo en procesos independientes.

El uso de runbooks secundarios reduce la cantidad total de tiempo que tarda en completarse el runbook primario. El runbook puede usar el cmdlet [Get-AzAutomationJob](https://docs.microsoft.com/powershell/module/az.automation/get-azautomationjob?view=azps-3.7.0) para comprobar el estado del trabajo de un runbook secundario si aún hay más operaciones después de que se complete el secundario.

## <a name="next-steps"></a>Pasos siguientes

* Para averiguar cómo trabajar con un runbook, consulte [Administración de runbooks en Azure Automation](manage-runbooks.md).
* Para obtener más información sobre los métodos que se pueden usar para iniciar un runbook en Azure Automation, consulte [Inicio de un runbook en Azure Automation](automation-starting-a-runbook.md).
* Para obtener más información sobre PowerShell, incluidos los módulos de referencia de lenguaje y aprendizaje, consulte la [documentación de PowerShell](https://docs.microsoft.com/powershell/scripting/overview).
* Para obtener una referencia de los cmdlets de PowerShell, consulte [Az.Automation](https://docs.microsoft.com/powershell/module/az.automation/?view=azps-3.7.0#automation
).
