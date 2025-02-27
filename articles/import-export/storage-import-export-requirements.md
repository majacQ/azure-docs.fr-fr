---
title: Exigences concernant le service Azure Import/Export | Microsoft Docs
description: L’objectif est ici de comprendre la configuration matérielle et logicielle requise pour le service Azure Import/Export.
author: alkohli
services: storage
ms.service: storage
ms.topic: conceptual
ms.date: 04/28/2021
ms.author: alkohli
ms.subservice: common
ms.openlocfilehash: e766b2aba1aef47b16b4351c7852bb2f457475fa
ms.sourcegitcommit: 34feb2a5bdba1351d9fc375c46e62aa40bbd5a1f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/10/2021
ms.locfileid: "111887597"
---
# <a name="azure-importexport-system-requirements"></a>Configuration système requise du service Azure Import/Export

Cet article décrit les principales exigences de votre service Azure Import/Export. Nous vous recommandons de lire attentivement les informations suivantes avant d’utiliser le service Import/Export, puis d’y revenir par la suite si nécessaire.

## <a name="supported-operating-systems"></a>Systèmes d’exploitation pris en charge

Pour préparer les disques durs à l’aide de l’outil WAImportExport, un certain nombre de **systèmes d’exploitation 64 bits autorisant le chiffrement de lecteur BitLocker** sont pris en charge.


|Plateforme |Version |
|---------|---------|
|Windows     | Windows 7 Entreprise, Windows 7 Édition intégrale <br> Windows 8 Pro, Windows 8 Entreprise, Windows 8.1 Professionnel, Windows 8.1 Entreprise <br> Windows 10        |
|Windows Server     |Windows Server 2008 R2 <br> Windows Server 2012 et Windows Server 2012 R2         |

## <a name="other-required-software-for-windows-client"></a>Autres logiciels nécessaires pour le client Windows

|Plateforme |Version |
|---------|---------|
|.NET Framework    | 4.5.1       |
| BitLocker        |  _          |


## <a name="supported-storage-accounts"></a>Comptes de stockage pris en charge

Le service Azure Import/Export prend en charge les types de comptes de stockage suivants :

- Comptes de stockage v2 à usage général standard (recommandés pour la plupart des scénarios)
- Comptes de stockage d’objets blob
- Comptes de stockage à usage général v1 (déploiements Classic ou Azure Resource Manager)

> [!IMPORTANT]
> La prise en charge du protocole NFS (Network File System) 3.0 dans le Stockage Blob Azure n’est pas prise en charge avec Azure Import/Export.

Pour plus d’informations sur les comptes de stockage, consultez [Vue d’ensemble des comptes de stockage Azure](../storage/common/storage-account-overview.md).

Chaque tâche peut servir à transférer des données vers ou à partir d'un seul compte de stockage. Autrement dit, une même tâche d’importation/exportation ne peut pas englober plusieurs comptes de stockage. Pour plus d'informations sur la création d'un compte de stockage, consultez la page [Création d'un compte de stockage](../storage/common/storage-account-create.md).

> [!IMPORTANT]
> Pour les comptes de stockage sur lesquels la fonctionnalité [Points de terminaison de service de réseau virtuel](../virtual-network/virtual-network-service-endpoints-overview.md) a été activée, utilisez le paramètre **Autoriser les services Microsoft approuvés…** pour [activer le service Import/Export](../storage/common/storage-network-security.md) afin d’importer/exporter des données vers ou depuis Azure.

## <a name="supported-storage-types"></a>Types de stockage pris en charge

Les types de stockage suivants sont pris en charge avec le service Azure Import/Export.


|Travail  |Service de stockage |Prise en charge  |Non prise en charge  |
|---------|---------|---------|---------|
|Importer     |  Stockage Blob Azure <br><br> Stockage Azure Files       | Objets blob de blocs et de pages pris en charge <br><br> Fichiers pris en charge          |
|Exporter     |   Stockage Blob Azure       | Objets blob de blocs, de pages et d’ajout pris en charge         | Fichiers Azure non pris en charge<br>Exportation à partir du niveau Archive non prise en charge|


## <a name="supported-hardware"></a>Matériel pris en charge

Dans le cadre du service Azure Import/Export, vous avez besoin de disques pris en charge pour copier les données.

### <a name="supported-disks"></a>Disques pris en charge

Les types de disques suivants sont pris en charge avec le service Azure Import/Export.


|Type de disque  |Size  |Prise en charge |
|---------|---------|---------|
|SSD    |   2,5"      |SATA III          |
|HDD     |  2,5"<br>3,5"       |SATA II, SATA III         |

Les types de disque suivants ne sont pas pris en charge :

- USB.
- Disque dur externe avec adaptateur USB intégré.
- Disques situés à l’intérieur du boîtier d’un disque dur externe.

Un travail d’importation/exportation peut avoir à lui seul :

- Un maximum de 10 disques HHD/SSD
- Un mélange de disques HDD/SSD de n’importe quelle taille

Il est possible de répartir un grand nombre de disques entre plusieurs tâches, et il n’existe aucune limite quant au nombre de tâches pouvant être créées. Dans le cas des tâches d’importation, seul le premier volume de données du lecteur est traité. Il doit être formaté avec NTFS.

Au moment de préparer les disques durs et de copier les données avec l’outil WAImportExport, vous pouvez utiliser les adaptateurs USB externes. La plupart des adaptateurs USB 3.0 ou version ultérieure du commerce doivent fonctionner.

## <a name="next-steps"></a>Étapes suivantes

* [Transfert de données avec l’utilitaire de ligne de commande AzCopy](../storage/common/storage-use-azcopy-v10.md)
