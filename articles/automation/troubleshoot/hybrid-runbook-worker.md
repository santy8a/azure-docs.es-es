---
title: Solución de problemas de Hybrid Runbook Worker de Azure Automation
description: En este artículo se proporciona información para solucionar problemas de Hybrid Runbook Worker de Azure Automation.
services: automation
ms.service: automation
ms.subservice: ''
author: mgoedtel
ms.author: magoedte
ms.date: 11/25/2019
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: 6d734c910cc966cfd83f1e1c7f9cbd728643fbc4
ms.sourcegitcommit: 11572a869ef8dbec8e7c721bc7744e2859b79962
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/05/2020
ms.locfileid: "82836519"
---
# <a name="troubleshoot-hybrid-runbook-workers"></a>Solución de problemas de Hybrid Runbook Worker

En este artículo se proporciona información sobre cómo solucionar problemas con las instancias de Hybrid Runbook Worker de Azure Automation.

>[!NOTE]
>Este artículo se ha actualizado para usar el nuevo módulo Az de Azure PowerShell. Aún puede usar el módulo de AzureRM que continuará recibiendo correcciones de errores hasta diciembre de 2020 como mínimo. Para más información acerca del nuevo módulo Az y la compatibilidad con AzureRM, consulte [Introducing the new Azure PowerShell Az module](https://docs.microsoft.com/powershell/azure/new-azureps-module-az?view=azps-3.5.0) (Presentación del nuevo módulo Az de Azure PowerShell). Para obtener instrucciones sobre la instalación del módulo Az en Hybrid Runbook Worker, consulte [Instalación del módulo de Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-az-ps?view=azps-3.5.0). Puede actualizar los módulos de su cuenta de Azure Automation a la última versión mediante [Actualización de módulos de Azure PowerShell en Azure Automation](../automation-update-azure-modules.md).

## <a name="general"></a>General

Hybrid Runbook Worker depende de un agente para comunicarse con su cuenta de Azure Automation para registrar el trabajo, recibir trabajos de runbook e informar del estado. Para Windows, este agente es el agente de Log Analytics para Windows. Para Linux, es el agente de Log Analytics para Linux.

### <a name="scenario-runbook-execution-fails"></a><a name="runbook-execution-fails"></a>Escenario: Error en la ejecución de un runbook

#### <a name="issue"></a>Problema

Se produce un error en la ejecución del runbook y recibe el mensaje de error siguiente:

```error
"The job action 'Activate' cannot be run, because the process stopped unexpectedly. The job action was attempted three times."
```

El runbook se suspende poco después de intentar ejecutarse tres veces. Hay condiciones que pueden interrumpir la finalización del runbook. El mensaje de error relacionado podría no incluir ninguna información adicional.

#### <a name="cause"></a>Causa

Las posibles causas son las siguientes:

* Los runbooks no se pueden autenticar con recursos locales.
* Hybrid Worker está detrás de un firewall o proxy.
* El equipo configurado para ejecutar Hybrid Runbook Worker no cumple los requisitos mínimos de hardware.

#### <a name="resolution"></a>Solución

Compruebe que el equipo tenga acceso saliente a * **.azure-automation.net** en el puerto 443.

Los equipos que ejecutan Hybrid Runbook Worker deben cumplir los requisitos mínimos de hardware antes de que el trabajo esté configurado para hospedar esta característica. Los runbooks y el proceso en segundo plano que usan pueden provocar un uso excesivo del sistema, así como tiempos de espera y retrasos del trabajo de los runbooks.

Confirme que el equipo que ejecutará la característica Hybrid Runbook Worker cumple los requisitos mínimos de hardware. Si es así, supervise el uso de la CPU y memoria para determinar las posibles correlaciones entre el rendimiento de los procesos de Hybrid Runbook Worker y Windows. Cualquier presión de CPU o memoria puede indicar la necesidad de actualizar los recursos. También puede seleccionar otro recurso de proceso que pueda admitir los requisitos mínimos y escalarse cuando las demandas de carga de trabajo indiquen que es necesario un aumento.

Compruebe el registro de eventos **Microsoft-SMA** en busca de un evento que corresponda a la descripción `Win32 Process Exited with code [4294967295]`. La causa de este error es que no ha configurado la autenticación en sus runbooks o no ha especificado las credenciales de ejecución para el grupo de Hybrid Runbook Worker. Para confirmar que ha configurado correctamente la autenticación para sus runbooks, revise los permisos de runbook en [Ejecución de runbooks en Hybrid Runbook Worker](../automation-hrw-run-runbooks.md).

### <a name="scenario-event-15011-in-the-hybrid-runbook-worker"></a><a name="cannot-connect-signalr"></a>Escenario: evento 15011 en la instancia de Hybrid Runbook Worker

#### <a name="issue"></a>Problema

Hybrid Runbook Worker recibe el evento 15011, lo que indica que el resultado de una consulta no es válido. El error siguiente aparece cuando el rol de trabajo intenta abrir una conexión con el [servidor de SignalR](https://docs.microsoft.com/aspnet/core/signalr/introduction?view=aspnetcore-3.1).

```error
[AccountId={c7d22bd3-47b2-4144-bf88-97940102f6ca}]
[Uri=https://cc-jobruntimedata-prod-su1.azure-automation.net/notifications/hub][Exception=System.TimeoutException: Transport timed out trying to connect
   at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at JobRuntimeData.NotificationsClient.JobRuntimeDataServiceSignalRClient.<Start>d__45.MoveNext()
```

#### <a name="cause"></a>Causa

Hybrid Runbook Worker no se configuró correctamente para la solución de implementación automatizada. Esta solución contiene una parte que conecta la máquina virtual al área de trabajo de Log Analytics. El script de PowerShell busca el área de trabajo en la suscripción con el nombre proporcionado. En este caso, el área de trabajo de Log Analytics está en una suscripción distinta. El script no puede encontrar el área de trabajo e intenta crear una, pero el nombre ya está ocupado. Como resultado, se produce un error en la implementación.

#### <a name="resolution"></a>Solución

Tiene dos opciones para resolver este problema:

* Modifique el script de PowerShell para buscar el área de trabajo de Log Analytics en otra suscripción. Esta es una buena solución si planea implementar muchas máquinas de Hybrid Runbook Worker en el futuro.

* Configure manualmente el equipo de trabajo para que se ejecute en un espacio aislado de Orchestrator. Luego, ejecute un runbook creado en la cuenta de Azure Automation en el rol de trabajo para probar la funcionalidad.

### <a name="scenario-windows-azure-vms-automatically-dropped-from-a-hybrid-worker-group"></a><a name="vm-automatically-dropped"></a>Escenario: las máquinas virtuales de Windows Azure se quitan automáticamente de un grupo de Hybrid Worker

#### <a name="issue"></a>Problema

No puede ver Hybrid Runbook Worker ni las máquinas virtuales cuando el equipo de trabajo ha estado apagado durante mucho tiempo.

#### <a name="cause"></a>Causa

La máquina de Hybrid Runbook Worker no ha hecho ping a Azure Automation durante más de 30 días. Como resultado, Automation ha purgado el grupo de Hybrid Runbook Worker o el grupo de trabajo del sistema. 

#### <a name="resolution"></a>Solución

Inicie la máquina de trabajo y vuelva a registrarla con Azure Automation. Para obtener instrucciones sobre cómo instalar el entorno de runbook y conectarse a Azure Automation, consulte [Implementación de Hybrid Runbook Worker en Windows](../automation-windows-hrw-install.md).

### <a name="scenario-no-certificate-was-found-in-the-certificate-store-on-the-hybrid-runbook-worker"></a><a name="no-cert-found"></a>Escenario: No se encontró ningún certificado en el almacén de certificados de la instancia de Hybrid Runbook Worker

#### <a name="issue"></a>Problema

Se produce un error en un runbook que se ejecutan en Hybrid Runbook Worker con el mensaje de error siguiente:

```error
Connect-AzAccount : No certificate was found in the certificate store with thumbprint 0000000000000000000000000000000000000000
At line:3 char:1
+ Connect-AzAccount -ServicePrincipal -Tenant $Conn.TenantID -Appl ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : CloseError: (:) [Connect-AzAccount], ArgumentException
    + FullyQualifiedErrorId : Microsoft.Azure.Commands.Profile.ConnectAzAccountCommand
```
#### <a name="cause"></a>Causa

Este error se produce cuando se intenta usar una [cuenta de ejecución](../manage-runas-account.md) en un runbook que se ejecuta en una instancia de Hybrid Runbook Worker en la que no existe el certificado de la cuenta de ejecución. Las instancias de Hybrid Runbook Worker no tienen el recurso de certificado localmente de forma predeterminada. La cuenta de ejecución requiere que este recurso funcione correctamente.

#### <a name="resolution"></a>Solución

Si su instancia de Hybrid Runbook Worker es una VM de Azure, puede [usar la autenticación de runbooks con identidades administradas](../automation-hrw-run-runbooks.md#runbook-auth-managed-identities) en su lugar. Este escenario simplifica la autenticación, lo que le permite autenticar los recursos de Azure mediante la identidad administrada de la VM de Azure en lugar de la cuenta de ejecución. Cuando la instancia de Hybrid Runbook Worker es una máquina local, deberá instalar el certificado de cuenta de ejecución en la máquina. Para obtener información sobre cómo instalar el certificado, consulte los pasos para ejecutar el runbook de PowerShell **Export-RunAsCertificateToHybridWorker** en [Ejecución de runbooks en Hybrid Runbook Worker](../automation-hrw-run-runbooks.md).

### <a name="scenario-error-403-during-registration-of-a-hybrid-runbook-worker"></a><a name="error-403-on-registration"></a>Escenario: error 403 durante el registro de una instancia de Hybrid Runbook Worker

#### <a name="issue"></a>Problema

Se produce un error en la fase inicial de registro del trabajo y recibe el siguiente error (403):

```error
"Forbidden: You don't have permission to access / on this server."
```

#### <a name="cause"></a>Causa

Los siguientes problemas son posibles causas:

* Hay un identificador de área de trabajo o una clave de área de trabajo (principal) mal escritos en la configuración del agente. 
* Hybrid Runbook Worker no puede descargar la configuración, lo que ocasiona un error de vinculación de la cuenta. Cuando Azure habilita las soluciones, solo en determinadas regiones permite vincular un área de trabajo de Log Analytics y una cuenta de Automation. También es posible que se haya establecido una fecha o una hora incorrectas en el equipo. Si la hora es 15 minutos anterior o posterior a la hora actual, se produce un error en la incorporación.

#### <a name="resolution"></a>Solución

##### <a name="mistyped-workspace-id-or-key"></a>Identificador o clave de área de trabajo mal escritos
Para comprobar si el identificador o la clave del área de trabajo del agente están escritos de manera incorrecta, consulte el artículo sobre cómo [agregar o quitar un área de trabajo: agente de Windows](../../azure-monitor/platform/agent-manage.md#windows-agent) para el agente de Windows, o bien el artículo sobre cómo [agregar o quitar un área de trabajo: agente de Linux](../../azure-monitor/platform/agent-manage.md#linux-agent) para el agente de Linux. Asegúrese de seleccionar la cadena completa de Azure Portal y cópiela y péguela con cuidado.

##### <a name="configuration-not-downloaded"></a>Configuración no descargada

Su área de trabajo de Log Analytics y su cuenta de Automation deben estar en una región vinculada. Para obtener una lista de las regiones admitidas, consulte [Asignaciones de áreas de trabajo de Log Analytics y Azure Automation](../how-to/region-mappings.md).

También es posible que necesite actualizar la fecha o la zona horaria del equipo. Si selecciona un intervalo de tiempo personalizado, asegúrese de que esté en formato UTC, que puede diferir del de la zona horaria local.

## <a name="linux"></a>Linux

Hybrid Runbook Worker de Linux depende del [agente de Log Analytics para Linux](../../azure-monitor/platform/log-analytics-agent.md) a fin de comunicarse con su cuenta de Automation para registrar el trabajo, recibir trabajos de runbook e informar del estado. Si se produce un error de registro del trabajo, estas son algunas de las causas posibles:

### <a name="scenario-linux-hybrid-runbook-worker-receives-prompt-for-a-password-when-signing-a-runbook"></a><a name="prompt-for-password"></a>Escenario: Una instancia de Hybrid Runbook Worker de Linux recibe una solicitud de contraseña al firmar un runbook

#### <a name="issue"></a>Problema

La ejecución del comando `sudo` para una instancia de Hybrid Runbook Worker de Linux recupera una solicitud inesperada de contraseña.

#### <a name="cause"></a>Causa

La cuenta **nxautomationuser** del agente de Log Analytics para Linux no está configurada correctamente en el archivo **sudoers**. La instancia de Hybrid Runbook Worker necesita la configuración adecuada de los permisos de la cuenta y otros datos para que pueda firmar runbooks en el trabajo de runbook de Linux.

#### <a name="resolution"></a>Solución

* Asegúrese de que la instancia de Hybrid Runbook Worker tiene el archivo ejecutable de GnuPG (GPG) en la máquina.

* Compruebe la configuración de la cuenta **nxautomationuser** en el archivo **sudoers**. Consulte [Ejecución de runbooks en Hybrid Runbook Worker](../automation-hrw-run-runbooks.md).

### <a name="scenario-the-log-analytics-agent-for-linux-isnt-running"></a><a name="oms-agent-not-running"></a>Escenario: el agente de Log Analytics para Linux no se está ejecutando

#### <a name="issue"></a>Problema

El agente de Log Analytics para Linux no está en ejecución.

#### <a name="cause"></a>Causa

Si el agente no se está ejecutando, se impide que Hybrid Runbook Worker de Linux se comunique con Azure Automation. Son varios los motivos por los que el agente podría no estar ejecutándose.

#### <a name="resolution"></a>Solución

 Escriba el comando `ps -ef | grep python` para comprobar si el agente está en ejecución. Debería ver una salida similar a la siguiente. Python procesa la cuenta de usuario **nxautomation**. Si la solución Update Management o Azure Automation no está habilitada, no se ejecuta ninguno de los procesos siguientes.

```bash
nxautom+   8567      1  0 14:45 ?        00:00:00 python /opt/microsoft/omsconfig/modules/nxOMSAutomationWorker/DSCResources/MSFT_nxOMSAutomationWorkerResource/automationworker/worker/main.py /var/opt/microsoft/omsagent/state/automationworker/oms.conf rworkspace:<workspaceId> <Linux hybrid worker version>
nxautom+   8593      1  0 14:45 ?        00:00:02 python /opt/microsoft/omsconfig/modules/nxOMSAutomationWorker/DSCResources/MSFT_nxOMSAutomationWorkerResource/automationworker/worker/hybridworker.py /var/opt/microsoft/omsagent/state/automationworker/worker.conf managed rworkspace:<workspaceId> rversion:<Linux hybrid worker version>
nxautom+   8595      1  0 14:45 ?        00:00:02 python /opt/microsoft/omsconfig/modules/nxOMSAutomationWorker/DSCResources/MSFT_nxOMSAutomationWorkerResource/automationworker/worker/hybridworker.py /var/opt/microsoft/omsagent/<workspaceId>/state/automationworker/diy/worker.conf managed rworkspace:<workspaceId> rversion:<Linux hybrid worker version>
```

En la siguiente lista, se muestran los procesos que se inician para un Hybrid Runbook Worker de Linux. Todos se encuentran en el directorio /var/opt/microsoft/omsagent/state/automationworker/.

* **oms.conf**: proceso del administrador de trabajos. Se inicia directamente desde DSC.
* **worker.conf**: proceso de Hybrid Worker registrado automáticamente. Lo inicia el administrador de trabajos. Este proceso lo usa Update Management y es transparente para el usuario. Este proceso no está presente si la solución Update Management no está habilitada en el equipo.
* **diy/worker.conf**: proceso de Hybrid Worker de implementación manual. El proceso de DIY Hybrid Worker se usa para ejecutar runbooks de usuario en Hybrid Runbook Worker. Difiere del proceso de Hybrid Worker registrado automáticamente solo en el detalle clave de que usa otra configuración. Este proceso no existe si la solución Azure Automation está deshabilitada y el rol de Hybrid Worker de Linux de implementación manual no está registrado.

Si el agente no se está ejecutando, ejecute el siguiente comando para iniciar el servicio: `sudo /opt/microsoft/omsagent/bin/service_control restart`.

### <a name="scenario-the-specified-class-doesnt-exist"></a><a name="class-does-not-exist"></a>Escenario: La clase especificada no existe

Si ve el mensaje de error `The specified class does not exist..` en **/var/opt/microsoft/omsconfig/omsconfig.log**, es necesario actualizar el agente de Log Analytics para Linux. Ejecute el siguiente comando para volver a instalar el agente.

```bash
wget https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/installer/scripts/onboard_agent.sh && sh onboard_agent.sh -w <WorkspaceID> -s <WorkspaceKey>
```

## <a name="windows"></a>Windows

Hybrid Runbook Worker de Windows depende del [agente de Log Analytics para Windows ](../../azure-monitor/platform/log-analytics-agent.md) a fin de comunicarse con su cuenta de Automation para registrar el trabajo, recibir trabajos de runbook e informar del estado. Si se produce un error en el registro del trabajo, en esta sección se incluyen algunas razones posibles.

### <a name="scenario-the-log-analytics-agent-for-windows-isnt-running"></a><a name="mma-not-running"></a>Escenario: el agente de Log Analytics para Windows no está en ejecución

#### <a name="issue"></a>Problema

`healthservice` no se está ejecutando en la máquina de Hybrid Runbook Worker.

#### <a name="cause"></a>Causa

Si el servicio Log Analytics para Windows no se está ejecutando, Hybrid Runbook Worker no se puede comunicar con Azure Automation.

#### <a name="resolution"></a>Solución

Para comprobar que el agente se está ejecutando, escriba el comando siguiente en PowerShell: `Get-Service healthservice`. Si se detiene el servicio, escriba el siguiente comando de PowerShell para iniciar el servicio: `Start-Service healthservice`.

### <a name="scenario-event-4502-in-the-operations-manager-log"></a><a name="event-4502"></a>Escenario: evento 4502 del registro de Operations Manager

#### <a name="issue"></a>Problema

En el registro de eventos **Registro de aplicaciones y servicios\Operations Manager**, aparece el evento 4502 y un mensaje de evento que contiene `Microsoft.EnterpriseManagement.HealthService.AzureAutomation.HybridAgent` con la descripción siguiente:<br>`The certificate presented by the service \<wsid\>.oms.opinsights.azure.com was not issued by a certificate authority used for Microsoft services. Please contact your network administrator to see if they are running a proxy that intercepts TLS/SSL communication.`

#### <a name="cause"></a>Causa

Este problema se puede deber a que el firewall de red o de proxy está bloqueando la comunicación con Microsoft Azure. Compruebe que el equipo tiene acceso saliente a * **.azure-automation.net** en el puerto 443.

#### <a name="resolution"></a>Solución

Los registros se almacenan localmente en cada Hybrid Worker en C:\ProgramData\Microsoft\System Center\Orchestrator\7.2\SMA\Sandboxes. Puede comprobar si hay algún evento de advertencia o de error en los registros de eventos **Registro de aplicaciones y servicios\Microsoft-SMA\Operations** y **Registro de aplicaciones y servicios\Operations Manager**. Estos registros indican una conectividad u otro tipo de problema que afecta a la incorporación del rol a Azure Automation, o bien a un problema detectado durante las operaciones normales. Para obtener ayuda adicional para solucionar problemas relacionados con el agente de Log Analytics, consulte [Procedimientos para solucionar problemas relacionados con el agente de Log Analytics para Windows](../../azure-monitor/platform/agent-windows-troubleshoot.md).

Los roles de Hybrid Worker envían [salida y mensajes de los runbooks](../automation-runbook-output-and-messages.md) a Azure Automation de la misma manera en que los trabajos de runbook que se ejecutan en la nube envían salida y mensajes. Puede habilitar los flujos Detallado y Progreso del mismo modo que haría para los runbooks.

### <a name="scenario-orchestratorsandboxexe-cant-connect-to-office-365-through-proxy"></a><a name="no-orchestrator-sandbox-connect-O365"></a>Escenario: Orchestrator.sandbox.exe no se puede conectar a Office 365 mediante proxy

#### <a name="issue"></a>Problema

Un script que se ejecuta en una instancia de Hybrid Runbook Worker de Windows no puede conectarse como se esperaba a Office 365 en un espacio aislado de Orchestrator. El script usa [Connect-MsolService](https://docs.microsoft.com/powershell/module/msonline/connect-msolservice?view=azureadps-1.0) para la conexión. 

Si ajusta **Orchestrator.Sandbox.exe.config** para establecer el proxy y la lista de omisión, el espacio aislado sigue sin conectarse correctamente. Sin embargo, un archivo **Powershell_ise.exe.config** con la misma configuración de proxy y lista de omisión parece funcionar como se espera. Los registros de Service Management Automation (SMA) y los registros de PowerShell no ofrecen ninguna información relacionada con el proxy.

#### <a name="cause"></a>Causa

La conexión a Servicios de federación de Active Directory (AD FS) del servidor no puede omitir el proxy. Recuerde que un espacio aislado de PowerShell se ejecuta como el usuario registrado. Sin embargo, un espacio aislado de Orchestrator está muy personalizado y podría pasar por alto la configuración de **Orchestrator.Sandbox.exe.config**. Este archivo tiene código especial para controlar la configuración del proxy del agente de Log Analytics, pero no para controlar otras opciones de proxy personalizadas. 

#### <a name="resolution"></a>Solución

Puede resolver el problema del espacio aislado de Orchestrator mediante la migración del script para que use los módulos de Azure Active Directory en lugar del módulo MSOnline para los cmdlets de PowerShell. Para más información, consulte [Migración de Orchestrator a Azure Automation (beta)](https://docs.microsoft.com/azure/automation/automation-orchestrator-migration).

Si desea seguir usando los cmdlets del módulo MSOnline, cambie el script para que use [Invoke-Command](https://docs.microsoft.com/powershell/module/microsoft.powershell.core/invoke-command?view=powershell-7). Especifique valores de los parámetros `ComputerName` y `Credential`. 

```powershell
$Credential = Get-AutomationPSCredential -Name MyProxyAccessibleCredential
Invoke-Command -ComputerName $env:COMPUTERNAME -Credential $Credential 
{ Connect-MsolService … }
```

Este cambio de código inicia una sesión de PowerShell completamente nueva en el contexto de las credenciales especificadas. Debe permitir que el tráfico fluya a través de un servidor proxy que autentique al usuario activo.

>[!NOTE]
>Esta solución hace que sea innecesario manipular el archivo de configuración del espacio aislado. Incluso si consigue que el archivo de configuración funcione con el script, el archivo se borrará cada vez que se actualice el agente de Hybrid Runbook Worker.

### <a name="scenario-hybrid-runbook-worker-not-reporting"></a><a name="corrupt-cache"></a>Escenario: Hybrid Runbook Worker sin informes

#### <a name="issue"></a>Problema

El equipo de Hybrid Runbook Worker se está ejecutando, pero no se muestran los datos de latido para el equipo en el área de trabajo.

En la consulta de ejemplo siguiente se muestran los equipos en un área de trabajo y su último latido:

```loganalytics
// Last heartbeat of each computer
Heartbeat
| summarize arg_max(TimeGenerated, *) by Computer
```

#### <a name="cause"></a>Causa

Este problema se puede deber a una caché dañada en el Hybrid Runbook Worker.

#### <a name="resolution"></a>Solución

Para resolver este problema, inicie sesión en Hybrid Runbook Worker y ejecute el script siguiente. Este script detiene el agente de Log Analytics, quita su caché y reinicia el servicio. Esta acción obliga a Hybrid Runbook Worker a volver a descargar su configuración de Azure Automation.

```powershell
Stop-Service -Name HealthService

Remove-Item -Path 'C:\Program Files\Microsoft Monitoring Agent\Agent\Health Service State' -Recurse

Start-Service -Name HealthService
```

### <a name="scenario-you-cant-add-a-hybrid-runbook-worker"></a><a name="already-registered"></a>Escenario: No puede agregar una instancia de Hybrid Runbook Worker.

#### <a name="issue"></a>Problema

Recibe el siguiente mensaje cuando intenta agregar una instancia de Hybrid Runbook Worker mediante el cmdlet `Add-HybridRunbookWorker`:

```error
Machine is already registered
```

#### <a name="cause"></a>Causa

Este problema puede surgir si el equipo ya está registrado con una cuenta de Automation diferente o si intenta volver a agregar la instancia de Hybrid Runbook Worker después de quitarla de un equipo.

#### <a name="resolution"></a>Solución

Para resolver este problema, quite la clave del Registro siguiente, reinicie `HealthService` y pruebe nuevamente el cmdlet `Add-HybridRunbookWorker`.

`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\HybridRunbookWorker`

## <a name="next-steps"></a>Pasos siguientes

Si su problema no aparece aquí o no puede resolverlo, intente obtener ayuda adicional mediante uno de los siguientes canales:

* Obtenga respuestas de expertos de Azure en los [foros de Azure](https://azure.microsoft.com/support/forums/).
* Póngase en contacto con [@AzureSupport](https://twitter.com/azuresupport), la cuenta oficial de Microsoft Azure para mejorar la experiencia del cliente. El Soporte técnico de Azure pone en contacto a la comunidad de Azure con respuestas, soporte técnico y expertos.
* Registrar un incidente de soporte técnico de Azure. Vaya al [sitio de Soporte técnico de Azure](https://azure.microsoft.com/support/options/) y seleccione **Obtener soporte técnico**.
