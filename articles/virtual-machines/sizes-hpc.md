---
title: 'Tamaños de las máquinas virtuales de Azure: HPC | Microsoft Docs'
description: Enumera los tamaños diferentes disponibles para las máquinas virtuales de informática de alto rendimiento en Azure. Se proporciona información sobre el número de unidades vCPU, discos de datos y NIC, así como sobre el rendimiento de almacenamiento y el ancho de banda de red para los tamaños de esta serie.
services: virtual-machines
documentationcenter: ''
author: vermagit
manager: gwallace
editor: ''
tags: azure-resource-manager,azure-service-management
ms.assetid: ''
ms.service: virtual-machines
ms.devlang: na
ms.topic: article
ms.workload: infrastructure-services
ms.date: 02/03/2020
ms.author: amverma
ms.reviewer: jonbeck
ms.openlocfilehash: 409fe69d111e2c5aebe0ad0bd38ced10604b5f1b
ms.sourcegitcommit: 11572a869ef8dbec8e7c721bc7744e2859b79962
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/05/2020
ms.locfileid: "82839069"
---
# <a name="high-performance-computing-vm-sizes"></a>Tamaños de VM para informática de alto rendimiento

La máquinas virtuales de la serie H de Azure están diseñada para ofrecer rendimiento de primer nivel, escalabilidad de MPI y rentabilidad para diversas cargas de trabajo de HPC del mundo real.

Las máquinas virtuales de la [serie HBv2](hbv2-series.md) cuentan con la característica Mellanox HDR InfiniBand a 200 GB/s, mientras que las máquinas virtuales de las series HB y HC cuentan con la característica MeliBox EDR InfiniBand a 100 GB/s. Cada uno de estos tipos de máquinas virtuales están conectados en un árbol FAT sin bloqueos para un rendimiento de RDMA optimizado y coherente. Las máquinas virtuales HBv2 admiten el enrutamiento adaptativo y el transporte conectado dinámico (DCT, además de los transportes estándar RC y UD). Estas características mejoran el rendimiento, la escalabilidad y la coherencia de las aplicaciones, y se recomienda totalmente su uso.

Las máquinas virtuales de la [serie HB](hb-series.md) están optimizadas para aplicaciones que funcionan con ancho de banda de la memoria, como la dinámica de fluidos, el análisis explícito de elementos finitos y los modelos de clima. Las máquinas virtuales HB cuentan con 60 núcleos de procesador AMD EPYC 7551, 4 GB de RAM por núcleo de CPU y no tienen hyperthreading. La plataforma AMD EPYC proporciona un ancho de banda de memoria de más de 260 GB/s.

Las máquinas virtuales de la [serie HC](hc-series.md) están optimizadas para aplicaciones basadas en cálculos intensivos, como el análisis implícito de elementos finitos, la dinámica molecular y la química computacional. Las máquinas virtuales HC cuentan con 44 núcleos de procesador Intel Xeon Platinum 8168, 8 GB de RAM por núcleo de CPU y no tienen hyperthreading. La plataforma Intel Xeon Platinum admite el rico ecosistema de herramientas de software de Intel, como la biblioteca Intel Math Kernel Library.

Las máquinas virtuales de la [serie H](h-series.md) están optimizadas para aplicaciones basadas en frecuencias altas de CPU o requisitos de gran cantidad de memoria por núcleo. Las máquinas virtuales de la serie H cuentan con 8 o 16 núcleos de procesador Intel Xeon E5 2667 v3, 7 o 14 GB de RAM por núcleo de CPU y no tienen hyperthreading. Las máquinas de la serie H cuentan con una interconexión Mellanox FDR InfiniBand de 56 Gb/s en una configuración de árbol de FAT sin bloqueo para proporcionar un rendimiento de RDMA consistente. Las máquinas virtuales de la serie H admiten Intel MPI 5.x y MS-MPI.

> [!NOTE]
> Está previsto que las VM A8-A11 se retiren en 3/2021. Para obtener más información, consulte la [guía de migración de HPC](https://azure.microsoft.com/resources/hpc-migration-guide/).

## <a name="rdma-capable-instances"></a>Instancias compatibles con RDMA

La mayoría de los tamaños de VM para HPC (HBv2, HB, HC, H16r, H16mr, A8 y A9) incluye una interfaz de red para la conectividad de acceso directo a memoria remota (RDMA). Los tamaños seleccionados de la [serie N](https://docs.microsoft.com/azure/virtual-machines/nc-series) designados con la letra "r", como las configuraciones de NC24rs (NC24rs_v3, NC24rs_v2 y NC24r) también son compatibles con RDMA. Esta interfaz se agrega a la interfaz de red estándar de Azure disponible en los otros tamaños de máquina virtual.

Esta interfaz permite que las instancias compatibles con RDMA se comuniquen través de una red InfiniBand (IB), que funciona a velocidades HDR en la serie HBv2, a velocidades EDR en las series HB y HC, a velocidades FDR en máquinas virtuales H16r, H16mr y de la serie N compatibles con RDMA, y a velocidades QDR en máquinas virtuales A8 y A9. Estas funcionalidades RDMA pueden mejorar la escalabilidad y el rendimiento de determinadas aplicaciones de la Interfaz de paso de mensajes (MPI). Para más información acerca de la velocidad, consulte los detalles en las tablas de esta página.

> [!NOTE]
> En Azure HPC, hay dos clases de máquinas virtuales en función de si están habilitadas para SR-IOV de InfiniBand. Actualmente, las máquinas virtuales que tienen habilitada la opción de SR-IOV para InfiniBand son: HBv2, HB, HC y NCv3. El resto de las máquinas virtuales compatibles con InfiniBand no están habilitadas para SR-IOV.
> RDMA a través de IB se admite en todas las máquinas virtuales compatibles con RDMA.
> Solo se admite IP sobre IB en máquinas virtuales habilitadas para SR-IOV.

- **Sistema operativo**: Linux es muy compatible con las máquinas virtuales de HPC; comúnmente se usan las distribuciones como CentOS, RHEL, Ubuntu y SUSE. En cuanto a la compatibilidad con Windows, Windows Server 2016 es compatible con todas las máquinas virtuales de la serie HPC. Windows Server 2012 R2 y Windows Server 2012 también se admiten en las máquinas virtuales no habilitadas para SR-IOV.

- **MPI**: los tamaños de máquina virtual habilitados para SR-IOV en Azure (HBv2, HB, HC y NCv3) permiten que se use prácticamente cualquier tipo de MPI con Mellanox OFED.
En las máquinas virtuales que no están habilitadas para SR-IOV, las implementaciones de MPI compatibles usan la interfaz Microsoft Network Direct (ND) para comunicarse entre máquinas virtuales. Por lo tanto, solo se admiten las versiones de Microsoft MPI (MS-MPI) 2012 R2 o posterior e Intel MPI 5.x. Las versiones posteriores (2017 y 2018) de la biblioteca en tiempo de ejecución de MPI pueden ser o no compatibles con los controladores de Azure RDMA.

- **Extensión de VM InfiniBandDriver<Linux|Windows>** : en las máquinas virtuales compatibles con RDMA, agregue la extensión InfiniBandDriver<Linux|Windows> para habilitar InfiniBand. En Linux, la extensión de máquina virtual InfiniBandDriverLinux instala controladores Mellanox OFED (en máquinas virtuales de SR-IOV) para la conectividad RDMA. En Windows, la extensión de máquina virtual InfiniBandDriverWindows instala controladores Windows Network Direct (en máquinas virtuales sin SR-IOV) o controladores Mellanox OFED (en máquinas virtuales con SR-IOV) para la conectividad RDMA.
En algunas implementaciones de las instancias A8 y A9, la extensión HpcVmDrivers se agrega automáticamente. Tenga en cuenta que la extensión de máquina virtual HpcVmDrivers está en desuso, por lo que no se actualizará.
Para agregar la extensión de máquina virtual a una máquina virtual, puede usar los cmdlets de [Azure PowerShell](/powershell/azure/overview). 

  El comando siguiente instala la versión más reciente de la extensión InfiniBandDriverWindows, la versión 1.0, en una máquina virtual compatible con RDMA existente denominada *myVM* implementada en el grupo de recursos denominado *myResourceGroup* de la región *Oeste de EE. UU.* :

  ```powershell
  Set-AzVMExtension -ResourceGroupName "myResourceGroup" -Location "westus" -VMName "myVM" -ExtensionName "InfiniBandDriverWindows" -Publisher "Microsoft.HpcCompute" -Type "InfiniBandDriverWindows" -TypeHandlerVersion "1.0"
  ```

  Como alternativa, se pueden incluir extensiones de máquina virtual en las plantillas de Azure Resource Manager para facilitar la implementación, con el siguiente elemento JSON:

  ```json
  "properties":{
  "publisher": "Microsoft.HpcCompute",
  "type": "InfiniBandDriverWindows",
  "typeHandlerVersion": "1.0",
  } 
  ```

  El comando siguiente instala la versión más reciente de la extensión InfiniBandDriverWindows, la versión 1.0, en todas las VM compatibles con RDMA en un conjunto de escalado de máquinas virtuales denominado *myVMSS* implementado en el grupo de recursos denominado *myResourceGroup*:

  ```powershell
  $VMSS = Get-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myVMSS"
  Add-AzVmssExtension -VirtualMachineScaleSet $VMSS -Name "InfiniBandDriverWindows" -Publisher "Microsoft.HpcCompute" -Type "InfiniBandDriverWindows" -TypeHandlerVersion "1.0"
  Update-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "MyVMSS" -VirtualMachineScaleSet $VMSS
  Update-AzVmssInstance -ResourceGroupName "myResourceGroup" -VMScaleSetName "myVMSS" -InstanceId "*"
  ```

  Para obtener más información, consulte el artículo de [características y extensiones de las máquinas virtuales](./extensions/overview.md). También puede trabajar con las extensiones para las máquinas virtuales implementadas en el [modelo de implementación clásica](https://docs.microsoft.com/previous-versions/azure/virtual-machines/windows/classic/agents-and-extensions-classic).

- **Espacio de direcciones de red RDMA** : la red RDMA en Azure reserva el espacio de direcciones 172.16.0.0/16. Para ejecutar aplicaciones MPI en instancias implementadas en una red virtual Azure, asegúrese de que el espacio de direcciones de la red virtual no se superpone a la red RDMA.

## <a name="cluster-configuration-options"></a>Opciones de configuración del clúster

Azure ofrece varias opciones para crear clústeres de máquinas virtuales de HPC de Windows que se pueden comunicar con la red RDMA, incluidos: 

- **Máquinas virtuales**: implemente las máquinas virtuales de HPC compatibles con RDMA en el mismo conjunto de escalado o de disponibilidad (cuando use el modelo de implementación de Azure Resource Manager). Si usa el modelo de implementación clásica, implemente las máquinas virtuales en el mismo servicio en la nube.

- **Conjuntos de escalado de máquinas virtuales**: en un conjunto de escalado de máquinas virtuales (VMSS), asegúrese de limitar la implementación a un único grupo de selección de ubicación para la comunicación InfiniBand dentro del VMSS. Por ejemplo, en una plantilla de Resource Manager, establezca la propiedad `singlePlacementGroup` en `true`. Tenga en cuenta que el tamaño de VMSS máximo que se puede usar con la propiedad `singlePlacementGroup` en `true` está limitado a 100 máquinas virtuales de forma predeterminada. Si los requisitos de escalado de su trabajo de HPC son superiores a 100 máquinas virtuales en un único inquilino de VMSS, puede solicitar un aumento. Para ello, [realice una solicitud de soporte técnico al cliente en línea](../azure-supportability/how-to-create-azure-support-request.md) sin cargo alguno. El número máximo de máquinas virtuales en un único VMSS se puede aumentar hasta 300. Tenga en cuenta que al implementar máquinas virtuales con conjuntos de disponibilidad, el límite máximo es de 200 máquinas virtuales por conjunto de disponibilidad.

- **MPI entre las máquinas virtuales**: si la característica de RDMA (por ejemplo, para usar la comunicación de MPI) es necesaria entre las máquinas virtuales (VM), asegúrese de que las VM estén en el mismo conjunto de escalado de máquinas virtuales o conjunto de disponibilidad.

- **Azure CycleCloud**: cree un clúster de HPC en [Azure CycleCloud](/azure/cyclecloud/) para ejecutar trabajos MPI.

- **Azure Batch**: cree un grupo de [Azure Batch](/azure/batch/) para ejecutar cargas de trabajo MPI. Para usar instancias de proceso intensivo para ejecutar aplicaciones MPI con Azure Batch, consulte [Uso de tareas de instancias múltiples para ejecutar aplicaciones de la Interfaz de paso de mensajes (MPI) en Azure Batch](../batch/batch-mpi.md).

- **Microsoft HPC Pack**: [HPC Pack](https://docs.microsoft.com/powershell/high-performance-computing/overview) incluye un entorno de tiempo de ejecución para MS-MPI que usa la red RDMA de Azure cuando se implementa en máquinas virtuales Linux compatibles con RDMA. Para obtener ejemplos de implementación, consulte [Configuración de un clúster de RDMA de Linux con HPC Pack para ejecutar aplicaciones MPI](https://docs.microsoft.com/powershell/high-performance-computing/hpcpack-linux-openfoam).

## <a name="deployment-considerations"></a>Consideraciones de la implementación

- **Suscripción de Azure**: para implementar más que algunas instancias de proceso intensivo, considere la posibilidad de usar una suscripción de pago por uso u otras opciones de compra. Si usa una [cuenta gratuita de Azure](https://azure.microsoft.com/free/), solo puede usar un número limitado de núcleos de proceso de Azure.

- **Disponibilidad y precios**: los tamaños de máquina virtual se ofrecen solo en el plan de tarifa estándar. Para ver la disponibilidad en las regiones de Azure, consulte [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/) .

- **Cuota de núcleos**: quizás tenga que aumentar la cuota de núcleos de su suscripción de Azure partiendo del valor predeterminado. La suscripción también podría limitar el número de núcleos que se pueden implementar en ciertas familias de tamaño de máquina virtual, como la serie H. Para solicitar un aumento de cuota, [abra una solicitud de soporte técnico al cliente en línea](../azure-supportability/how-to-create-azure-support-request.md) sin cargo alguno. (Los límites predeterminados pueden variar según la categoría de suscripción).

  > [!NOTE]
  > Si tiene necesidades de capacidad a gran escala, póngase en contacto con el soporte técnico de Azure. Las cuotas de Azure son límites de crédito, no garantías de capacidad. Independientemente de la cuota, solamente se le cobrarán los núcleos que use.
  
- **Red virtual** : no se necesita una [red virtual](https://azure.microsoft.com/documentation/services/virtual-network/) de Azure para usar instancias de proceso intensivo. Sin embargo, para muchas implementaciones necesita al menos una red virtual de Azure basada en la nube o una conexión de sitio a sitio si necesita acceder a recursos locales. Si es necesario, cree una red virtual para implementar las instancias. No se admite la adición de máquinas virtuales de proceso intensivo a las redes virtuales de grupos de afinidad.

- **Cambio de tamaño**: debido a su hardware especializado, solo se puede cambiar el tamaño de las instancias de proceso intensivo dentro de la misma familia de tamaño (serie H o serie A de proceso intensivo). Por ejemplo, una máquina virtual de la serie H solo se puede cambiar de un tamaño de serie H a otro. Además, no se admite el cambio de tamaño de un tamaño que no sea de proceso intensivo a un tamaño que sí lo sea.  


## <a name="other-sizes"></a>Otros tamaños

- [Uso general](sizes-general.md)
- [Proceso optimizado](sizes-compute.md)
- [Memoria optimizada](sizes-memory.md)
- [Almacenamiento optimizado](sizes-storage.md)
- [GPU optimizada](sizes-gpu.md)
- [Generaciones anteriores](sizes-previous-gen.md)

## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información sobre cómo optimizar la aplicación de HPC para Azure y algunos ejemplos en [Cargas de trabajo de HPC](https://docs.microsoft.com/azure/virtual-machines/workloads/hpc/overview). 

- Obtenga más información sobre cómo las [unidades de proceso de Azure (ACU)](acu.md) pueden ayudarlo a comparar el rendimiento en los distintos SKU de Azure.
