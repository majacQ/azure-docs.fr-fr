---
title: Disques Ultra pour machines virtuelles - Disques managés Azure
description: En savoir plus sur les disques Ultra pour machines virtuelles Azure
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 08/17/2021
ms.author: rogarana
ms.subservice: disks
ms.custom: references_regions, devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: 2f1c56070a2b86945494bf0a8625f223d017f326
ms.sourcegitcommit: 37cc33d25f2daea40b6158a8a56b08641bca0a43
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2021
ms.locfileid: "130070815"
---
# <a name="using-azure-ultra-disks"></a>Utilisation de disques Ultra Azure

**S’applique à :** :heavy_check_mark: Machines virtuelles Linux :heavy_check_mark: Machines virtuelles Windows :heavy_check_mark: Groupes identiques flexibles :heavy_check_mark: Groupes identiques uniformes

Cet article explique comment déployer et utiliser un disque Ultra ; pour obtenir des informations conceptuelles sur les disques Ultra, reportez-vous à [Quels sont les types de disque disponibles dans Azure ?](disks-types.md#ultra-disks).

Les disques Ultra Azure permettent un stockage de disque à haut débit avec un nombre élevé d’IOPS et une faible latence homogène pour les machines virtuelles Azure IaaS. Cette nouvelle offre fournit des performances optimales aux mêmes niveaux de disponibilité que nos offres de disques existantes. Entre autres avantages, les disques Ultra permettent de changer dynamiquement les performances des disques SSD en fonction de vos charges de travail, sans avoir à redémarrer les machines virtuelles. Les disques Ultra sont adaptés aux charges de travail qui consomment beaucoup de données, par exemple SAP HANA, les bases de données de niveau supérieur et les charges de travail avec un grand nombre de transactions.

## <a name="ga-scope-and-limitations"></a>Étendue et limitations de la version en disponibilité générale

[!INCLUDE [managed-disks-ultra-disks-GA-scope-and-limitations](../../includes/managed-disks-ultra-disks-GA-scope-and-limitations.md)]

## <a name="determine-vm-size-and-region-availability"></a>Déterminer la taille de la machine virtuelle et la disponibilité des régions

### <a name="vms-using-availability-zones"></a>Machines virtuelles utilisant des zones de disponibilité

Pour tirer parti des disques Ultra, vous devez déterminer la zone de disponibilité dans laquelle vous vous trouvez. Toutes les régions ne prennent pas en charge toutes les tailles de machine virtuelle avec des disques Ultra. Pour déterminer si votre région, votre zone et votre taille de machine virtuelle prend en charge les disques Ultra, exécutez l’une des commandes suivantes, assurez-vous d’abord de remplacer les valeurs **Région**, **vmSize** et **Abonnement** :

#### <a name="cli"></a>Interface de ligne de commande

```azurecli
subscription="<yourSubID>"
# example value is southeastasia
region="<yourLocation>"
# example value is Standard_E64s_v3
vmSize="<yourVMSize>"

az vm list-skus --resource-type virtualMachines  --location $region --query "[?name=='$vmSize'].locationInfo[0].zoneDetails[0].Name" --subscription $subscription
```

#### <a name="powershell"></a>PowerShell

```powershell
$region = "southeastasia"
$vmSize = "Standard_E64s_v3"
$sku = (Get-AzComputeResourceSku | where {$_.Locations.Contains($region) -and ($_.Name -eq $vmSize) -and $_.LocationInfo[0].ZoneDetails.Count -gt 0})
if($sku){$sku[0].LocationInfo[0].ZoneDetails} Else {Write-host "$vmSize is not supported with Ultra Disk in $region region"}
```

La réponse aura un format similaire au suivant, où X correspond à la zone à utiliser pour le déploiement dans la région que vous avez choisie. X peut être 1, 2 ou 3.

Conservez la valeur **Zones**, qui représente votre zone de disponibilité. Vous en aurez besoin pour déployer un disque Ultra.

|ResourceType  |Nom  |Emplacement  |Zones  |Restriction  |Fonctionnalité  |Valeur  |
|---------|---------|---------|---------|---------|---------|---------|
|disks     |UltraSSD_LRS         |eastus2         |X         |         |         |         |

> [!NOTE]
> En l’absence de réponse de la commande, la taille de machine virtuelle sélectionnée n’est pas prise en charge avec les disques Ultra dans la région sélectionnée.

Maintenant que vous savez quelle zone déployer, suivez les étapes de déploiement décrites dans cet article pour déployer une machine virtuelle avec un disque Ultra attaché ou attacher un disque Ultra à une machine virtuelle existante.

### <a name="vms-with-no-redundancy-options"></a>Machines virtuelles sans options de redondance

Pour le moment, dans certaines régions, les disques Ultra doivent être déployés sans aucune option de redondance. Cela dit, ces régions ne comprennent pas nécessairement toutes les tailles de disque qui prennent en charge les disques Ultra. Pour déterminer les tailles de disque qui prennent en charge les disques Ultra, vous pouvez utiliser l'un ou l'autre des extraits de code suivants. Veillez d’abord à remplacer les valeurs `vmSize` et `subscription` :

```azurecli
subscription="<yourSubID>"
region="westus"
# example value is Standard_E64s_v3
vmSize="<yourVMSize>"

az vm list-skus --resource-type virtualMachines  --location $region --query "[?name=='$vmSize'].capabilities" --subscription $subscription
```

```azurepowershell
$region = "westus"
$vmSize = "Standard_E64s_v3"
(Get-AzComputeResourceSku | where {$_.Locations.Contains($region) -and ($_.Name -eq $vmSize) })[0].Capabilities
```

La réponse se présente comme suit ; `UltraSSDAvailable   True` indique si la taille de machine virtuelle prend en charge les disques Ultra dans cette région.

```
Name                                         Value
----                                         -----
MaxResourceVolumeMB                          884736
OSVhdSizeMB                                  1047552
vCPUs                                        64
HyperVGenerations                            V1,V2
MemoryGB                                     432
MaxDataDiskCount                             32
LowPriorityCapable                           True
PremiumIO                                    True
VMDeploymentTypes                            IaaS
vCPUsAvailable                               64
ACUs                                         160
vCPUsPerCore                                 2
CombinedTempDiskAndCachedIOPS                128000
CombinedTempDiskAndCachedReadBytesPerSecond  1073741824
CombinedTempDiskAndCachedWriteBytesPerSecond 1073741824
CachedDiskBytes                              1717986918400
UncachedDiskIOPS                             80000
UncachedDiskBytesPerSecond                   1258291200
EphemeralOSDiskSupported                     True
AcceleratedNetworkingEnabled                 True
RdmaEnabled                                  False
MaxNetworkInterfaces                         8
UltraSSDAvailable                            True
```

## <a name="deploy-an-ultra-disk-using-azure-resource-manager"></a>Déployer un disque Ultra à l’aide d’Azure Resource Manager

Tout d’abord, déterminez la taille de la machine virtuelle à déployer. Pour obtenir la liste des tailles de machines virtuelles prises en charge, consultez la section [Étendue et limitations de la version en disponibilité générale](#ga-scope-and-limitations).

Pour créer une machine virtuelle avec plusieurs disques Ultra, consultez l’exemple [Créer une machine virtuelle avec plusieurs disques Ultra](https://aka.ms/ultradiskArmTemplate).

Si vous avez l’intention d’utiliser votre propre modèle, assurez-vous que la valeur **apiVersion** pour `Microsoft.Compute/virtualMachines` et `Microsoft.Compute/Disks` est définie sur `2018-06-01` (ou version ultérieure).

Définissez la référence SKU du disque sur **UltraSSD_LRS**, puis définissez la capacité du disque, le nombre d’IOPS, la zone de disponibilité et le début (en Mbits/s) pour créer votre disque Ultra.

Une fois que la machine virtuelle est provisionnée, vous pouvez partitionner et formater les disques de données, puis les configurer pour vos charges de travail.


## <a name="deploy-an-ultra-disk"></a>Déployer un disque Ultra

# <a name="portal"></a>[Portail](#tab/azure-portal)

Cette section couvre le déploiement d'une machine virtuelle équipée d'un disque Ultra comme disque de données. Elle suppose que vous maîtrisez le déploiement d'une machine virtuelle ; si ce n'est pas le cas, consultez notre section [Démarrage rapide : Création d’une machine virtuelle Windows dans le portail Azure](./windows/quick-create-portal.md).

1. Connectez-vous au [portail Azure](https://portal.azure.com/), puis recherchez une machine virtuelle à déployer.
1. Assurez-vous de choisir une [taille et une région de machine virtuelle prises en charge](#ga-scope-and-limitations).
1. Sélectionnez **Zone de disponibilité** dans **Options de disponibilité**.
1. Remplissez les entrées restantes avec les sélections de votre choix.
1. Sélectionnez **Disques**.

    :::image type="content" source="media/virtual-machines-disks-getting-started-ultra-ssd/new-ultra-vm-create.png" alt-text="Capture d’écran du panneau De base du flux de création de machine virtuelle." lightbox="media/virtual-machines-disks-getting-started-ultra-ssd/new-ultra-vm-create.png":::

1. Dans le panneau Disques, sélectionnez **Oui** pour **Activer la compatibilité avec les disques Ultra**.
1. Sélectionnez **Créer un disque et l'attacher** pour attacher un disque Ultra maintenant.

    :::image type="content" source="media/virtual-machines-disks-getting-started-ultra-ssd/new-ultra-vm-disk-enable.png" alt-text="Capture d'écran du panneau Disque du flux de création d'une machine virtuelle, dans lequel Ultra est activé et Créer et attacher un nouveau disque est en surbrillance." :::

1. Dans le panneau **Créer un disque**, entrez un nom, puis sélectionnez **Modifier la taille**.

    :::image type="content" source="media/virtual-machines-disks-getting-started-ultra-ssd/new-ultra-create-disk.png" alt-text="Capture d'écran du panneau Créer un disque, avec l'option Modifier la taille en surbrillance.":::


1. Modifiez la **Référence SKU du disque** en **Disque Ultra**.
1. Remplacez les valeurs des champs **Taille de disque personnalisée (Gio)** , **IOPS de disque** et **Débit de disque** par celles de votre choix.
1. Sélectionnez **OK** dans les deux panneaux.

    :::image type="content" source="media/virtual-machines-disks-getting-started-ultra-ssd/new-select-ultra-disk-size.png" alt-text="Capture d'écran du panneau Sélectionner une taille de disque, dans lequel le type de stockage Disque Ultra est sélectionné tandis que d'autres valeurs sont en surbrillance.":::

1. Poursuivez le déploiement de la machine virtuelle comme si vous déployiez une machine virtuelle standard.

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Tout d’abord, déterminez la taille de la machine virtuelle à déployer. Pour obtenir la liste des tailles de machines virtuelles prises en charge, consultez la section [Étendue et limitations de la version en disponibilité générale](#ga-scope-and-limitations).

Pour attacher un disque Ultra, vous devez créer une machine virtuelle capable d’utiliser des disques Ultra.

Remplacez ou définissez les variables **$vmname**, **$rgname**, **$diskname**, **$location**, **$password**, **$user** par votre propres valeurs. Définissez **$zone** sur la valeur de votre zone de disponibilité, que vous avez obtenue grâce à la procédure décrite au [début de cet article](#determine-vm-size-and-region-availability). Ensuite, exécutez la commande CLI suivante pour créer une machine virtuelle capable de gérer les disques SSD Ultra :

```azurecli-interactive
az disk create --subscription $subscription -n $diskname -g $rgname --size-gb 1024 --location $location --sku UltraSSD_LRS --disk-iops-read-write 8192 --disk-mbps-read-write 400
az vm create --subscription $subscription -n $vmname -g $rgname --image Win2016Datacenter --ultra-ssd-enabled true --zone $zone --authentication-type password --admin-password $password --admin-username $user --size Standard_D4s_v3 --location $location --attach-data-disks $diskname
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Tout d’abord, déterminez la taille de la machine virtuelle à déployer. Pour obtenir la liste des tailles de machines virtuelles prises en charge, consultez la section [Étendue et limitations de la version en disponibilité générale](#ga-scope-and-limitations).

Pour pouvoir utiliser des disques Ultra, vous devez créer une machine virtuelle capable d’utiliser ce type de disque. Remplacez ou définissez les variables **$resourcegroup** et **$vmName** par vos propres valeurs. Définissez **$zone** sur la valeur de votre zone de disponibilité, que vous avez obtenue grâce à la procédure décrite au [début de cet article](#determine-vm-size-and-region-availability). Ensuite, exécutez la commande [New-AzVm](/powershell/module/az.compute/new-azvm) suivante pour créer une machine virtuelle capable de gérer les disques SSD Ultra :

```powershell
New-AzVm `
    -ResourceGroupName $resourcegroup `
    -Name $vmName `
    -Location "eastus2" `
    -Image "Win2016Datacenter" `
    -EnableUltraSSD `
    -size "Standard_D4s_v3" `
    -zone $zone
```

### <a name="create-and-attach-the-disk"></a>Créer et attacher le disque

Une fois votre machine virtuelle déployée, vous pouvez y créer et y attacher un disque dur à l'aide du script suivant :

```powershell
# Set parameters and select subscription
$subscription = "<yourSubscriptionID>"
$resourceGroup = "<yourResourceGroup>"
$vmName = "<yourVMName>"
$diskName = "<yourDiskName>"
$lun = 1
Connect-AzAccount -SubscriptionId $subscription

# Create the disk
$diskconfig = New-AzDiskConfig `
-Location 'EastUS2' `
-DiskSizeGB 8 `
-DiskIOPSReadWrite 1000 `
-DiskMBpsReadWrite 100 `
-AccountType UltraSSD_LRS `
-CreateOption Empty `
-zone $zone;

New-AzDisk `
-ResourceGroupName $resourceGroup `
-DiskName $diskName `
-Disk $diskconfig;

# add disk to VM
$vm = Get-AzVM -ResourceGroupName $resourceGroup -Name $vmName
$disk = Get-AzDisk -ResourceGroupName $resourceGroup -Name $diskName
$vm = Add-AzVMDataDisk -VM $vm -Name $diskName -CreateOption Attach -ManagedDiskId $disk.Id -Lun $lun
Update-AzVM -VM $vm -ResourceGroupName $resourceGroup
```

---

## <a name="deploy-an-ultra-disk---512-byte-sector-size"></a>Déploiement d’un Disque Ultra d’une taille de secteur de 512 octets

# <a name="portal"></a>[Portail](#tab/azure-portal)

1. Connectez-vous au [portail Azure](https://portal.azure.com/), puis recherchez et sélectionnez **Disques**.
1. Sélectionnez **+ Nouveau** pour créer un disque.
1. Sélectionnez une région qui prend en charge les disques Ultra, sélectionnez une zone de disponibilité, puis renseignez le reste des valeurs comme vous le souhaitez.
1. Sélectionnez **Changer la taille**.

    :::image type="content" source="media/virtual-machines-disks-getting-started-ultra-ssd/create-managed-disk-basics-workflow.png" alt-text="Capture d’écran du panneau Créer un disque, avec la région, la zone de disponibilité et le changement de taille en surbrillance.":::

1. Pour **Référence SKU de disque**, sélectionnez **Disque Ultra**, entrez les valeurs des performances souhaitées, puis sélectionnez **OK**.

    :::image type="content" source="media/virtual-machines-disks-getting-started-ultra-ssd/select-disk-size-ultra.png" alt-text="Capture d’écran de la création d’un disque Ultra.":::

1. Dans le panneau **De base**, sélectionnez l’onglet **Avancé**.
1. Sélectionnez **512** pour **Taille de secteur logique**, puis sélectionnez **Vérifier + créer**.

    :::image type="content" source="media/virtual-machines-disks-getting-started-ultra-ssd/select-different-sector-size-ultra.png" alt-text="Capture d’écran du sélecteur pour la modification de la taille de secteur logique de disque Ultra en 512.":::

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Tout d’abord, déterminez la taille de la machine virtuelle à déployer. Pour obtenir la liste des tailles de machines virtuelles prises en charge, consultez la section [Étendue et limitations de la version en disponibilité générale](#ga-scope-and-limitations).

Pour attacher un disque Ultra, vous devez créer une machine virtuelle capable d’utiliser des disques Ultra.

Remplacez ou définissez les variables **$vmname**, **$rgname**, **$diskname**, **$location**, **$password**, **$user** par votre propres valeurs. Définissez **$zone** sur la valeur de votre zone de disponibilité, que vous avez obtenue grâce à la procédure décrite au [début de cet article](#determine-vm-size-and-region-availability). Exécutez ensuite la commande CLI suivante pour créer une machine virtuelle avec un Disque Ultra d’une taille de secteur de 512 octets :

```azurecli
#create an ultra disk with 512 sector size
az disk create --subscription $subscription -n $diskname -g $rgname --size-gb 1024 --location $location --sku UltraSSD_LRS --disk-iops-read-write 8192 --disk-mbps-read-write 400 --logical-sector-size 512
az vm create --subscription $subscription -n $vmname -g $rgname --image Win2016Datacenter --ultra-ssd-enabled true --zone $zone --authentication-type password --admin-password $password --admin-username $user --size Standard_D4s_v3 --location $location --attach-data-disks $diskname
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Tout d’abord, déterminez la taille de la machine virtuelle à déployer. Pour obtenir la liste des tailles de machines virtuelles prises en charge, consultez la section [Étendue et limitations de la version en disponibilité générale](#ga-scope-and-limitations).

Pour pouvoir utiliser des disques Ultra, vous devez créer une machine virtuelle capable d’utiliser ce type de disque. Remplacez ou définissez les variables **$resourcegroup** et **$vmName** par vos propres valeurs. Définissez **$zone** sur la valeur de votre zone de disponibilité, que vous avez obtenue grâce à la procédure décrite au [début de cet article](#determine-vm-size-and-region-availability). Ensuite, exécutez la commande [New-AzVm](/powershell/module/az.compute/new-azvm) suivante pour créer une machine virtuelle capable de gérer les disques SSD Ultra :

```powershell
New-AzVm `
    -ResourceGroupName $resourcegroup `
    -Name $vmName `
    -Location "eastus2" `
    -Image "Win2016Datacenter" `
    -EnableUltraSSD `
    -size "Standard_D4s_v3" `
    -zone $zone
```

Pour créer et attacher un Disque Ultra d’une taille de secteur de 512 octets, vous pouvez utiliser le script suivant :

```powershell
# Set parameters and select subscription
$subscription = "<yourSubscriptionID>"
$resourceGroup = "<yourResourceGroup>"
$vmName = "<yourVMName>"
$diskName = "<yourDiskName>"
$lun = 1
Connect-AzAccount -SubscriptionId $subscription

# Create the disk
$diskconfig = New-AzDiskConfig `
-Location 'EastUS2' `
-DiskSizeGB 8 `
-DiskIOPSReadWrite 1000 `
-DiskMBpsReadWrite 100 `
-LogicalSectorSize 512 `
-AccountType UltraSSD_LRS `
-CreateOption Empty `
-zone $zone;

New-AzDisk `
-ResourceGroupName $resourceGroup `
-DiskName $diskName `
-Disk $diskconfig;

# add disk to VM
$vm = Get-AzVM -ResourceGroupName $resourceGroup -Name $vmName
$disk = Get-AzDisk -ResourceGroupName $resourceGroup -Name $diskName
$vm = Add-AzVMDataDisk -VM $vm -Name $diskName -CreateOption Attach -ManagedDiskId $disk.Id -Lun $lun
Update-AzVM -VM $vm -ResourceGroupName $resourceGroup
```
---
## <a name="attach-an-ultra-disk"></a>Attacher un disque Ultra

# <a name="portal"></a>[Portail](#tab/azure-portal)

Si votre machine virtuelle se trouve dans une région/zone de disponibilité capable d’utiliser les disques Ultra, vous pouvez utiliser des disques Ultra sans avoir à créer de machine virtuelle. En activant des disques Ultra sur votre machine virtuelle existante, puis en les attachant comme des disques de données. Pour activer la compatibilité des disques Ultra, vous devez arrêter la machine virtuelle. Une fois la machine virtuelle arrêtée, vous pouvez activer la compatibilité, puis redémarrer la machine virtuelle. Une fois la compatibilité activée, vous pouvez attacher un disque Ultra :

1. Accédez à votre machine virtuelle, arrêtez-la et attendez qu'elle soit libérée.
1. Une fois votre machine virtuelle libérée, sélectionnez **Disques**.
1. Sélectionnez **Paramètres supplémentaires**.

    :::image type="content" source="media/virtual-machines-disks-getting-started-ultra-ssd/new-ultra-disk-additional-settings.png" alt-text="Capture d’écran du panneau Disque, avec les paramètres supplémentaires en surbrillance.":::

1. Sélectionnez **Oui** pour **Activer la compatibilité avec les disques Ultra**.

    :::image type="content" source="media/virtual-machines-disks-getting-started-ultra-ssd/enable-ultra-disks-existing-vm.png" alt-text="Capture d'écran d'Activer la compatibilité avec les disques Ultra.":::

1. Sélectionnez **Enregistrer**.
1. Sélectionnez **Créer et attacher un nouveau disque**, puis entrez un nom pour votre nouveau disque.
1. Pour **Type de stockage**, sélectionnez **Disque Ultra**.
1. Remplacez les valeurs des champs **Taille (Gio)** , **Opérations d’E/S par seconde maximales** et **Débit maximal** par les valeurs de votre choix.
1. De retour dans le panneau de votre disque, sélectionnez **Enregistrer**.

    :::image type="content" source="media/virtual-machines-disks-getting-started-ultra-ssd/new-create-ultra-disk-existing-vm.png" alt-text="Capture d’écran du panneau Disque, montrant l’ajout d’un nouveau disque Ultra.":::

1. Redémarrez votre machine virtuelle.

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Si votre machine virtuelle se trouve dans une région/zone de disponibilité capable d’utiliser les disques Ultra, vous pouvez utiliser des disques Ultra sans avoir à créer de machine virtuelle.

### <a name="enable-ultra-disk-compatibility-on-an-existing-vm---cli"></a>Activer la compatibilité des disques Ultra sur une machine virtuelle existante - Interface CLI

Si votre machine virtuelle répond aux exigences décrites dans [Étendue et limitations de la version en disponibilité générale](#ga-scope-and-limitations) et qu’elle se trouve dans la [zone appropriée pour votre compte](#determine-vm-size-and-region-availability), vous pouvez activer la compatibilité des disques Ultra sur votre machine virtuelle.

Pour activer la compatibilité des disques Ultra, vous devez arrêter la machine virtuelle. Une fois la machine virtuelle arrêtée, vous pouvez activer la compatibilité, puis redémarrer la machine virtuelle. Une fois la compatibilité activée, vous pouvez attacher un disque Ultra :

```azurecli
az vm deallocate -n $vmName -g $rgName
az vm update -n $vmName -g $rgName --ultra-ssd-enabled true
az vm start -n $vmName -g $rgName
```

### <a name="create-an-ultra-disk---cli"></a>Créer un disque Ultra - Interface CLI

Maintenant que vous disposez d’une machine virtuelle capable d’attacher des disques Ultra, vous pouvez créer un disque Ultra et l’attacher à cette machine.

```azurecli-interactive
location="eastus2"
subscription="xxx"
rgname="ultraRG"
diskname="ssd1"
vmname="ultravm1"
zone=123

#create an ultra disk
az disk create `
--subscription $subscription `
-n $diskname `
-g $rgname `
--size-gb 4 `
--location $location `
--zone $zone `
--sku UltraSSD_LRS `
--disk-iops-read-write 1000 `
--disk-mbps-read-write 50
```

### <a name="attach-the-disk---cli"></a>Attacher le disque - Interface CLI

```azurecli
rgName="<yourResourceGroupName>"
vmName="<yourVMName>"
diskName="<yourDiskName>"
subscriptionId="<yourSubscriptionID>"

az vm disk attach -g $rgName --vm-name $vmName --disk $diskName --subscription $subscriptionId
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Si votre machine virtuelle se trouve dans une région/zone de disponibilité capable d’utiliser les disques Ultra, vous pouvez utiliser des disques Ultra sans avoir à créer de machine virtuelle.

### <a name="enable-ultra-disk-compatibility-on-an-existing-vm---powershell"></a>Activer la compatibilité des disques Ultra sur une machine virtuelle existante - PowerShell

Si votre machine virtuelle répond aux exigences décrites dans [Étendue et limitations de la version en disponibilité générale](#ga-scope-and-limitations) et qu’elle se trouve dans la [zone appropriée pour votre compte](#determine-vm-size-and-region-availability), vous pouvez activer la compatibilité des disques Ultra sur votre machine virtuelle.

Pour activer la compatibilité des disques Ultra, vous devez arrêter la machine virtuelle. Une fois la machine virtuelle arrêtée, vous pouvez activer la compatibilité, puis redémarrer la machine virtuelle. Une fois la compatibilité activée, vous pouvez attacher un disque Ultra :

```azurepowershell
#Stop the VM
Stop-AzVM -Name $vmName -ResourceGroupName $rgName
#Enable ultra disk compatibility
$vm1 = Get-AzVM -name $vmName -ResourceGroupName $rgName
Update-AzVM -ResourceGroupName $rgName -VM $vm1 -UltraSSDEnabled $True
#Start the VM
Start-AzVM -Name $vmName -ResourceGroupName $rgName
```

### <a name="create-and-attach-an-ultra-disk---powershell"></a>Créer et attacher un disque Ultra - PowerShell

Maintenant que vous disposez d’une machine virtuelle capable d’utiliser des disques Ultra, vous pouvez créer un disque Ultra et l’attacher à cette machine :

```powershell
# Set parameters and select subscription
$subscription = "<yourSubscriptionID>"
$resourceGroup = "<yourResourceGroup>"
$vmName = "<yourVMName>"
$diskName = "<yourDiskName>"
$lun = 1
Connect-AzAccount -SubscriptionId $subscription

# Create the disk
$diskconfig = New-AzDiskConfig `
-Location 'EastUS2' `
-DiskSizeGB 8 `
-DiskIOPSReadWrite 1000 `
-DiskMBpsReadWrite 100 `
-AccountType UltraSSD_LRS `
-CreateOption Empty `
-zone $zone;

New-AzDisk `
-ResourceGroupName $resourceGroup `
-DiskName $diskName `
-Disk $diskconfig;

# add disk to VM
$vm = Get-AzVM -ResourceGroupName $resourceGroup -Name $vmName
$disk = Get-AzDisk -ResourceGroupName $resourceGroup -Name $diskName
$vm = Add-AzVMDataDisk -VM $vm -Name $diskName -CreateOption Attach -ManagedDiskId $disk.Id -Lun $lun
Update-AzVM -VM $vm -ResourceGroupName $resourceGroup
```

---
## <a name="adjust-the-performance-of-an-ultra-disk"></a>Ajuster les performances d'un disque Ultra

# <a name="portal"></a>[Portail](#tab/azure-portal)

Les disques Ultra offrent une capacité unique qui vous permet d’ajuster leurs performances. Vous pouvez effectuer ces réglages depuis le portail Azure, sur les disques eux-mêmes.

1. Accédez à votre machine virtuelle et sélectionnez **Disques**.
1. Sélectionnez le disque Ultra dont vous souhaitez modifier les performances.

    :::image type="content" source="media/virtual-machines-disks-getting-started-ultra-ssd/select-ultra-disk-to-modify.png" alt-text="Capture d'écran du panneau Disques de votre machine virtuelle, dans lequel Disque Ultra est en surbrillance.":::

1. Sélectionnez **Paille + performances**, puis apportez vos modifications.
1. Sélectionnez **Enregistrer**.

    :::image type="content" source="media/virtual-machines-disks-getting-started-ultra-ssd/modify-ultra-disk-performance.png" alt-text="Capture d'écran du panneau Configuration de votre disque Ultra dans lequel la taille du disque, les e/s par seconde et le débit sont en surbrillance, de même que l'option Enregistrer.":::

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Les disques Ultra offrent une capacité unique qui vous permet d’ajuster leurs performances. La commande suivante illustre l’utilisation de cette fonctionnalité :

```azurecli-interactive
az disk update `
--subscription $subscription `
--resource-group $rgname `
--name $diskName `
--set diskIopsReadWrite=80000 `
--set diskMbpsReadWrite=800
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

## <a name="adjust-the-performance-of-an-ultra-disk-using-powershell"></a>Ajuster les performances d’un disque Ultra à l’aide de PowerShell

Les disques Ultra présentent une capacité unique, qui vous permet d’ajuster leurs performances. L’exemple de commande suivant ajuste les performances sans nécessiter le détachement du disque :

```powershell
$diskupdateconfig = New-AzDiskUpdateConfig -DiskMBpsReadWrite 2000
Update-AzDisk -ResourceGroupName $resourceGroup -DiskName $diskName -DiskUpdate $diskupdateconfig
```
---

## <a name="next-steps"></a>Étapes suivantes

- [Utiliser des disques Ultra Azure dans Azure Kubernetes Service (préversion)](../aks/use-ultra-disks.md).
- [Migrer le disque de stockage des journaux vers un disque Ultra](../azure-sql/virtual-machines/windows/storage-migrate-to-ultradisk.md).
