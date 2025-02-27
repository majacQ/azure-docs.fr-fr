---
title: Modes d’orchestration pour les groupes de machines virtuelles identiques dans Azure
description: Découvrez comment utiliser les modes d’orchestration Flexible et Uniform pour les groupes de machines virtuelles identiques dans Azure.
author: fitzgeraldsteele
ms.author: fisteele
ms.topic: conceptual
ms.service: virtual-machine-scale-sets
ms.date: 08/05/2021
ms.reviewer: jushiman
ms.custom: mimckitt, devx-track-azurecli, vmss-flex, devx-track-azurepowershell
ms.openlocfilehash: 834fcd1732435d99d63d16efc3d867ba58fe4844
ms.sourcegitcommit: 106f5c9fa5c6d3498dd1cfe63181a7ed4125ae6d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/02/2021
ms.locfileid: "131074571"
---
# <a name="orchestration-modes-for-virtual-machine-scale-sets-in-azure"></a>Modes d’orchestration pour les groupes de machines virtuelles identiques dans Azure


**S’applique à :** :heavy_check_mark: Machines virtuelles Linux :heavy_check_mark: Machines virtuelles Windows :heavy_check_mark: Groupes identiques flexibles :heavy_check_mark: Groupes identiques uniformes

Les groupes de machines virtuelles identiques fournissent un regroupement logique des machines virtuelles gérées par la plateforme. Avec les groupes identiques, vous créez un modèle de configuration de machine virtuelle, ajoutez ou supprimez automatiquement des instances supplémentaires en fonction de la charge de l’UC ou de la mémoire, puis effectuez une mise à niveau automatique vers la dernière version du système d’exploitation. Traditionnellement, les groupes identiques vous permettent de créer des machines virtuelles à l’aide d’un modèle de configuration de machine virtuelle fourni au moment de la création d’un groupe identique, et le groupe identique peut uniquement gérer des machines virtuelles qui sont créées implicitement en fonction du modèle de configuration. 

Les modes d’orchestration de groupe identique vous permettent de mieux contrôler la façon dont les instances de machine virtuelle sont gérées par le groupe identique.

> [!IMPORTANT]
> Le mode d’orchestration est défini lorsque vous créez le groupe identique et ne peut pas être modifié ou mis à jour ultérieurement.


## <a name="scale-sets-with-uniform-orchestration"></a>Groupes identiques avec orchestration Uniform
Ce mode est optimisé pour les charges de travail sans état à grande échelle avec des instances identiques.

Les groupes de machines virtuelles identiques (VMSS, virtual machine scale set) avec orchestration Uniform utilisent un modèle ou un profil de machine virtuelle pour effectuer un scale-up jusqu’à la capacité souhaitée. Même s’il est possible de gérer ou personnaliser des instances de machine virtuelle individuelles, le mode Uniform utilise des instances de machine virtuelle identiques. Les instances de machine virtuelle Uniform individuelles sont exposées par le biais des commandes d’API de machine virtuelle VMSS. Les instances individuelles ne sont pas compatibles avec les commandes d’API de machine virtuelle Azure IaaS standard, les fonctionnalités de gestion Azure telles que les autorisations RBAC d’étiquetage des ressources d’Azure Resource Manager et les services Sauvegarde Azure et Azure Site Recovery. L’orchestration Uniform garantit une haute disponibilité du domaine d’erreur quand elle est configurée avec moins de 100 instances. L’orchestration Uniform est en disponibilité générale et prend en charge une gamme complète de fonctionnalités de gestion et d’orchestration des groupes identiques, y compris la mise à l’échelle automatique basée sur des métriques, la protection des instances et les mises à niveau automatiques de système d’exploitation.


## <a name="scale-sets-with-flexible-orchestration"></a>Groupes identiques avec orchestration Flexible
Obtenez une haute disponibilité à grande échelle avec un ou plusieurs types de machine virtuelle.

Avec l’orchestration Flexible, Azure offre une expérience unifiée sur tout l’écosystème de machines virtuelles Azure. L’orchestration Flexible garantit une haute disponibilité (jusqu’à 1 000 machines virtuelles) en répartissant les machines virtuelles entre différents domaines d’erreur dans une région ou dans une zone de disponibilité. Vous pouvez ainsi effectuer un scale-out de votre application tout en conservant l’isolation du domaine d’erreur, essentielle pour exécuter des charges de travail avec état ou basées sur un quorum, y compris :
- Charges de travail basées sur un quorum
- Bases de données open source
- Applications avec état
- Services nécessitant une haute disponibilité à grande échelle
- Services cherchant à combiner différents types de machines virtuelles ou à tirer parti à la fois de machines virtuelles spot et à la demande
- Applications d’un groupe à haute disponibilité existant


## <a name="what-has-changed-with-flexible-orchestration-mode"></a>Quels changements le mode d’orchestration Flexible apporte-t-il ?
L’un des principaux avantages de l’orchestration Flexible est qu’elle fournit des fonctionnalités d’orchestration sur des machines virtuelles Azure IaaS standard et non sur des machines virtuelles enfants de groupe identique. Vous pouvez donc utiliser toutes les API de machine virtuelle standard pour la gestion d’instances avec orchestration Flexible, plutôt que les API de machine virtuelle VMSS que vous utilisez avec l’orchestration Uniform. La gestion des instances avec une orchestration Flexible et celle avec une orchestration Uniform diffèrent sur plusieurs points. D’une manière générale, nous vous recommandons d’utiliser les API de machine virtuelle Azure IaaS standard dans la mesure du possible. Cette section met en avant des exemples de bonnes pratiques pour la gestion des instances de machine virtuelle avec une orchestration Flexible.

### <a name="scale-out-with-standard-azure-virtual-machines"></a>Effectuer un scale-out avec des machines virtuelles Azure standard
Les groupes de machines virtuelles identiques en mode d’orchestration Flexible gèrent les machines virtuelles Azure standard. Vous avez un contrôle total sur le cycle de vie des machines virtuelles, ainsi que sur les interfaces réseau et les disques, grâce aux commandes et aux API Azure standard. Les machines virtuelles créées avec le mode d’orchestration Uniform sont exposées et gérées via les commandes de l’API de machine virtuelle du groupe de machines virtuelles identiques. Les instances individuelles ne sont pas compatibles avec les commandes d’API de machine virtuelle Azure IaaS standard, les fonctionnalités de gestion Azure telles que les autorisations RBAC d’étiquetage des ressources d’Azure Resource Manager et les services Sauvegarde Azure et Azure Site Recovery.

### <a name="assign-fault-domain-during-vm-creation"></a>Attribuer un domaine d’erreur au moment de la création de la machine virtuelle
Vous pouvez choisir le nombre de domaines d’erreur pour le groupe identique avec orchestration Flexible. Par défaut, quand vous ajoutez une machine virtuelle à un groupe identique Flexible, Azure répartit uniformément les instances entre les domaines d’erreur. Il est recommandé de laisser Azure attribuer le domaine d’erreur. Cependant, pour des scénarios avancés ou de dépannage, vous pouvez remplacer ce comportement par défaut et spécifier le domaine d’erreur auquel appartiendra l’instance.

```azurecli-interactive
az vm create –vmss "myVMSS"  –-platform_fault_domain 1
```

### <a name="instance-naming"></a>Nommage d’instance
Quand vous créez une machine virtuelle et l’ajoutez à un groupe identique Flexible, vous disposez d’un contrôle total sur les noms d’instance dans le cadre des règles de convention de nommage Azure. Quand des machines virtuelles sont ajoutées automatiquement au groupe identique par le biais d’une mise à l’échelle automatique, vous spécifiez un préfixe et Azure ajoute un numéro unique à la fin du nom.

### <a name="query-instances-for-power-state"></a>Instances de requête pour l’état d’alimentation
La méthode recommandée consiste à utiliser Azure Resource Graph pour interroger toutes les machines virtuelles d’un groupe de machines virtuelles identiques. Azure Resource Graph fournit des fonctionnalités de requête efficaces pour les ressources Azure à grande échelle, sur les différents abonnements.

```
| where type =~ 'Microsoft.Compute/virtualMachines'
| where properties.virtualMachineScaleSet contains "demo"
| extend powerState = properties.extended.instanceView.powerState.code
| project name, resourceGroup, location, powerState
| order by resourceGroup desc, name desc
```

L’interrogation des ressources avec [Azure Resource Graph](../governance/resource-graph/overview.md) représente un moyen pratique et efficace d’interroger des ressources Azure et de réduire les appels d’API au fournisseur de ressources. Azure Resource Graph est un cache cohérent à terme. Il se peut que les ressources nouvelles ou mises à jour n’y soient pas reflétées pendant une période allant jusqu’à 60 secondes. Vous pouvez :
- Lister les machines virtuelles d’un groupe de ressources ou d’un abonnement.
- Utiliser l’option de développement pour récupérer la vue d’instance (attribution de domaine d’erreur, états d’alimentation et de provisionnement) pour toutes les machines virtuelles de votre abonnement.
- Utiliser les commandes et l’API de machine virtuelle Get pour obtenir un modèle et une vue d’instance pour une seule instance.

### <a name="scale-sets-vm-batch-operations"></a>Opérations de traitement par lots des machines virtuelles de groupe identique
Utilisez les commandes de machine virtuelle standard pour démarrer, arrêter, redémarrer et supprimer des instances, plutôt que les API de machine virtuelle VMSS. Les opérations de traitement par lots de machine virtuelle VMSS (tout démarrer, tout arrêter, tout réinitialiser, etc.) ne sont pas utilisées avec le mode d’orchestration Flexible.

### <a name="monitor-application-health"></a>Superviser l’intégrité de l’application
Avec la supervision du fonctionnement de l’application, votre application fournit à Azure une pulsation pour déterminer si votre application est saine ou non. Azure peut remplacer automatiquement les instances de machine virtuelle non saines. Pour les instances de groupe identique Flexibles, vous devez installer et configurer l’extension Intégrité de l’application sur la machine virtuelle. Pour les instances de groupe identique Uniform, vous pouvez utiliser l’extension Intégrité de l’application ou mesurer l’intégrité avec une sonde d’intégrité personnalisée Azure Load Balancer.

### <a name="list-scale-sets-vm-api-changes"></a>Modifications de l’API de machine virtuelle qui liste les instances d’un groupe identique
VMSS (Virtual Machine Scale Sets) vous permet de lister les instances appartenant au groupe identique. Avec l’orchestration Flexible, la commande de machine virtuelle VMSS de liste retourne une liste d’ID de machines virtuelles d’un groupe identique. Vous pouvez ensuite appeler les commandes de machine virtuelle VMSS GET pour obtenir plus d’informations sur la façon dont le groupe identique fonctionne avec l’instance de machine virtuelle. Pour obtenir toutes les informations de la machine virtuelle, utilisez les commandes de machine virtuelle GET standard ou [Azure Resource Graph](../governance/resource-graph/overview.md).

### <a name="retrieve-boot-diagnostics-data"></a>Récupérer les données de diagnostics de démarrage
Utilisez les API et commandes de machine virtuelle standard pour récupérer les captures d’écran et données de diagnostic de démarrage d’instance. Les commandes et API de diagnostic de démarrage de machine virtuelle VMSS ne sont pas utilisées avec les instances en mode d’orchestration Flexible.

### <a name="vm-extensions"></a>Extensions de machine virtuelle
Utilisez les extensions ciblant les machines virtuelles standard plutôt que celles ciblant les instances en mode d’orchestration Uniform.


## <a name="a-comparison-of-flexible-uniform-and-availability-sets"></a>Comparaison du mode d’orchestration Flexible, du mode d’orchestration Uniform et des groupes à haute disponibilité
Le tableau suivant compare le mode d’orchestration Flexible, le mode d’orchestration Uniform et les groupes à haute disponibilité selon leurs fonctionnalités.

### <a name="basic-setup"></a>Configuration de base
| Fonctionnalité | Prise en charge par l’orchestration Flexible pour les groupes identiques | Prise en charge par l’orchestration Uniform pour les groupes identiques | Prise en charge par les groupes à haute disponibilité |
|---|---|---|---|
| Type de machine virtuelle  | Machine virtuelle Azure IaaS standard (Microsoft.compute/virtualmachines)  | Machines virtuelles propres aux groupes identiques (Microsoft.compute/virtualmachinescalesets/virtualmachines)  | Machine virtuelle Azure IaaS standard (Microsoft.compute/virtualmachines) |
| Nombre maximal d’instances (avec garanties de domaine d’erreur)  | 1 000  | 100  | 200 |
| Références SKU prises en charge  | Série D, série E, série F, série A, série B, Intel, AMD ; les références SKU de spécialité (G, H, L, M, N) ne sont pas prises en charge | Toutes les références SKU  | Toutes les références SKU |
| Contrôle total sur la machine virtuelle, les cartes réseau et les disques  | Oui  | Contrôle limité avec l’API de machine virtuelle de groupes de machines virtuelles identiques (VMSS)  | Oui  |
| Autorisations RBAC nécessaires  | Écriture sur des VMSS de calcul, écriture sur des machines virtuelles de calcul, réseau | Écriture sur des VMSS de calcul  | N/A |
| Mise en réseau accélérée  | Oui  | Oui  | Oui |
| Instances spot et tarifs   | Oui, vous pouvez avoir des instances spot et des instances à priorité normale  | Oui, les instances doivent être toutes des instances spot ou toutes des instances normales  | Non, des instances à priorité normale uniquement |
| Différents systèmes d’exploitation  | Oui, Linux et Windows peuvent résider dans le même groupe identique Flexible  | Non, les instances ont le même système d’exploitation  | Oui, Linux et Windows peuvent résider dans le même groupe identique Flexible |
| Types de disques  | Disques managés uniquement, tous les types de stockage  | Disques managés et non managés, tous les types de stockage  | Disques managés et non managés, Ultradisk non pris en charge |
| Accélérateur d’écriture   | Non  | Oui  | Oui |
| Groupes de placement de proximité   | Oui, lisez la [Documentation sur les groupes de placement de proximité](../virtual-machine-scale-sets/proximity-placement-groups.md) | Oui, lisez la [Documentation sur les groupes de placement de proximité](../virtual-machine-scale-sets/proximity-placement-groups.md) | Oui |
| Hôtes dédiés Azure   | Non  | Oui  | Oui |
| Identité managée  | Identité affectée par l’utilisateur uniquement  | Affectée par le système ou par l’utilisateur  | N/A (peut spécifier une identité managée pour chaque instance) |
| Ajout d’une machine virtuelle existante à un groupe/suppression du groupe  | Non  | Non  | Non |
| Service Fabric  | Non  | Oui  | Non |
| Azure Kubernetes Service (AKS) / AKE  | Non  | Oui  | Non |
| UserData  | Partielle. UserData peut être spécifié pour chaque machine virtuelle | Oui  | UserData peut être spécifié pour chaque machine virtuelle |


### <a name="autoscaling-and-instance-orchestration"></a>Mise à l’échelle automatique et orchestration des instances
| Fonctionnalité | Prise en charge par l’orchestration Flexible pour les groupes identiques | Prise en charge par l’orchestration Uniform pour les groupes identiques | Prise en charge par les groupes à haute disponibilité |
|---|---|---|---|
| Liste des machines virtuelles d’un groupe | Oui | Oui | Oui, liste des machines virtuelles d’un groupe à haute disponibilité |
| Mise à l’échelle automatique (manuelle, basée sur des métriques, basée sur une planification) | Oui | Oui | Non |
| Supprimer automatiquement les cartes réseau et les disques lors de la suppression d’instances de machine virtuelle | Oui | Oui | Non |
| Stratégie de mise à niveau (groupes de machines virtuelles identiques) | Non, la stratégie de mise à niveau doit être null ou [] lors de la création | Automatique, propagée, manuelle | N/A |
| Mises à jour automatiques de système d’exploitation (groupes de machines virtuelles identiques) | Non | Oui | N/A |
| Mise à jour corrective de sécurité intégrée (« in-guest ») | Oui | Non | Oui |
| Notifications d’arrêt (groupes de machines virtuelles identiques) | Oui, lisez la [Documentation sur les notifications d’arrêt](../virtual-machine-scale-sets/virtual-machine-scale-sets-terminate-notification.md) | Oui, lisez la [Documentation sur les notifications d’arrêt](../virtual-machine-scale-sets/virtual-machine-scale-sets-terminate-notification.md) | N/A |
| Supervision de l’intégrité de l’application | Extension Intégrité de l’application | Extension Intégrité de l’application ou sonde Azure Load Balancer | Extension Intégrité de l’application |
| Réparation d’instance (groupes de machines virtuelles identiques) | Oui, lisez la [documentation sur la réparation d’instance](../virtual-machine-scale-sets/virtual-machine-scale-sets-automatic-instance-repairs.md) | Oui, lisez la [documentation sur la réparation d’instance](../virtual-machine-scale-sets/virtual-machine-scale-sets-automatic-instance-repairs.md) | N/A |
| Protection des instances | Non, utilisez des [verrous de ressource Azure](../azure-resource-manager/management/lock-resources.md) | Oui | Non |
| Stratégie de scale-in | Non | Oui | Non |
| VMSS - Obtenir une vue d’instance | Non | Oui | N/A |
| Opérations de traitement par lots de machines virtuelles (Tout démarrer, Tout arrêter, supprimer un sous-ensemble, etc.) | Non (peut déclencher des opérations sur chaque instance à l’aide de l’API de machine virtuelle) | Oui | Non |

### <a name="high-availability"></a>Haute disponibilité 

| Fonctionnalité | Prise en charge par l’orchestration Flexible pour les groupes identiques | Prise en charge par l’orchestration Uniform pour les groupes identiques | Prise en charge par les groupes à haute disponibilité |
|---|---|---|---|
| Contrat SLA de disponibilité | 99,95 % pour les instances réparties sur plusieurs domaines d’erreur ; 99,99 % pour les instances réparties sur plusieurs zones | 99,95 % pour les domaines comprenant plus d’un groupe de placement unique ; 99,99 % pour les instances réparties sur plusieurs zones | 99,95 % |
| Zones de disponibilité | Spécifiez des instances appartenant à 1, 2 ou 3 zones de disponibilité | Spécifiez des instances appartenant à 1, 2 ou 3 zones de disponibilité | Non prise en charge |
| Affecter une machine virtuelle à une zone de disponibilité spécifique | Oui | Non | Non |
| Domaine d’erreur – Diffusion maximale (Azure va diffuser les instances au maximum) | Oui | Oui | Non |
| Domaine d’erreur – Diffusion fixe | 2 ou 3 domaines d’erreur (en fonction du nombre maximal de domaines d’erreur par région), 1 pour les déploiements zonaux | 2, 3 ou 5 domaines ; 1 ou 5 pour les déploiements zonaux | 2 ou 3 domaines d’erreur (en fonction du nombre maximal par région) |
| Attribuer une machine virtuelle à un domaine d’erreur spécifique | Oui | Non | Non |
| Mettre à jour les domaines | Déprécié (maintenance de plateforme effectuée domaine d’erreur par domaine d’erreur) | 5 domaines de mise à jour | Jusqu’à 20 domaines de mise à jour |
| Effectuer la maintenance | Déclencher la maintenance sur chaque instance à l’aide de l’API de machine virtuelle | Oui | N/A |

### <a name="networking"></a>Mise en réseau 

| Fonctionnalité | Prise en charge par l’orchestration Flexible pour les groupes identiques | Prise en charge par l’orchestration Uniform pour les groupes identiques | Prise en charge par les groupes à haute disponibilité |
|---|---|---|---|
| Connectivité sortante par défaut | Non, doit avoir une [connectivité sortante explicite](../virtual-network/ip-services/default-outbound-access.md) | Oui | Oui |
| Azure Load Balancer - SKU Standard | Oui | Oui | Oui |
| Application Gateway | Oui | Oui | Oui |
| Réseau InfiniBand | Non | Oui, groupe de placement unique seulement | Oui |
| Équilibreur de charge logiciel De base | Non | Oui | Oui |
| Transfert de port réseau | Oui (règles NAT pour chaque instance) | Oui (pool NAT) | Oui (règles NAT pour chaque instance) |

### <a name="backup-and-recovery"></a>Sauvegarde et récupération 

| Fonctionnalité | Prise en charge par l’orchestration Flexible pour les groupes identiques | Prise en charge par l’orchestration Uniform pour les groupes identiques | Prise en charge par les groupes à haute disponibilité |
|---|---|---|---|
| Sauvegarde Azure  | Oui | Non | Oui |
| Azure Site Recovery | Oui (via PowerShell) | Non | Oui |
| Alertes Azure  | Oui | Oui | Oui |
| Insights de machine virtuelle  | Peut être installé sur chaque machine virtuelle | Oui | Oui |


## <a name="get-started-with-flexible-orchestration-mode"></a>Démarrer avec le mode d’orchestration Flexible

Inscrivez-vous et commencez à utiliser le [Mode d’orchestration flexible](..\virtual-machines\flexible-virtual-machine-scale-sets.md) pour vos groupes de machines virtuelles identiques. 


## <a name="frequently-asked-questions"></a>Forum aux questions

- **Quelle capacité de mise à l’échelle l’orchestration Flexible prend-elle en charge ?**

    Vous pouvez ajouter jusqu’à 1 000 machines virtuelles à un groupe identique en mode d’orchestration Flexible.

- **En quoi l’orchestration Flexible est-elle différente des groupes à haute disponibilité ou de l’orchestration Uniform ?**

    | Attribut de disponibilité  | Orchestration Flexible  | Orchestration Uniform  | Groupes à haute disponibilité  |
    |-|-|-|-|
    | Déploiement entre plusieurs zones de disponibilité  | Oui  | Oui  | Non  |
    | Garanties de disponibilité de domaine d’erreur au sein d’une région  | Oui, jusqu’à 1 000 instances peuvent être réparties sur un maximum de 3 domaines d’erreur dans la région. Le nombre maximal de domaines d’erreur varie selon la région  | Oui, jusqu’à 100 instances  | Oui, jusqu’à 200 instances  |
    | Groupes de placement  | Le mode Flexible utilise toujours plusieurs groupes de placement (singlePlacementGroup = false)  | Vous pouvez choisir un groupe de placement unique ou plusieurs groupes de placement | N/A  |
    | Domaines de mise à jour  | Aucun. La maintenance et les mises à jour de l’ordinateur hôte sont effectuées domaine d’erreur par domaine d’erreur  | Jusqu’à 5 domaines de mise à jour  | Jusqu’à 20 domaines de mise à jour  |

- **Quel est le nombre maximal absolu d’instances avec une disponibilité garantie de domaine d’erreur ?**

    | Fonctionnalité  | Prise en charge par l’orchestration Flexible  | Prise en charge par l’orchestration Uniform (disponibilité générale)  | Prise en charge par les groupes à haute disponibilité (disponibilité générale)  |
    |-|-|-|-|
    | Nombre maximal d’instances (avec garantie de disponibilité de domaine d’erreur)  | 1 000  | 3000  | 200  |


## <a name="next-steps"></a>Étapes suivantes
> [!div class="nextstepaction"]
> [Découvrez comment déployer votre application sur un groupe identique.](virtual-machine-scale-sets-deploy-app.md)
