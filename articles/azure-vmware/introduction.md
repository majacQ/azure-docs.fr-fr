---
title: Introduction
description: Découvrez les fonctionnalités et les avantages d’Azure VMware Solution pour déployer et gérer des charges de travail VMware dans Azure. Le Contrat de niveau de service d’Azure VMware Solution garantit que les outils de gestion Azure VMware (vCenter Server et NSX Manager) seront disponibles au moins 99,9 % du temps.
ms.topic: overview
ms.date: 04/20/2021
ms.openlocfilehash: 10362ef88f572a40771fa6ab3e61b04220c79249
ms.sourcegitcommit: 0046757af1da267fc2f0e88617c633524883795f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/13/2021
ms.locfileid: "122070718"
---
# <a name="what-is-azure-vmware-solution"></a>Présentation d’Azure VMware Solution

Azure VMware Solution fournit des clouds privés qui contiennent des clusters vSphere, créés à partir d’une infrastructure Azure complète dédiée. Le déploiement initial doit comprendre au minimum trois hôtes. Toutefois, vous pouvez ajouter d’autres hôtes, un après l’autre, jusqu’à un maximum de 16 hôtes par cluster. Tous les clouds privés provisionnés disposent de vCenter Server, vSAN, vSphere et NSX-T. Ainsi, vous pouvez effectuer la migration de charges de travail à partir de vos environnements locaux, déployer de nouvelles machines virtuelles et consommer des services Azure à partir de vos clouds privés. Par ailleurs, les outils de gestion Azure VMware Solution (vCenter Server et NSX Manager) sont disponibles au moins 99,9 % du temps. Pour plus d’informations, consultez [Contrat SLA Azure VMware Solution](https://aka.ms/avs/sla).

Azure VMware Solution est une solution validée par VMware, qui teste et valide constamment les améliorations et les mises à niveau. Microsoft gère les logiciels et l’infrastructure de cloud privé. Cela vous permet de vous concentrer sur le développement et l’exécution des charges de travail dans vos clouds privés.

Le diagramme illustre la contiguïté entre les clouds privés et les réseaux virtuels dans Azure, les services Azure et les environnements locaux. L’accès réseau des clouds privés vers les services Azure ou les réseaux virtuels fournit une intégration pilotée par SLA des points de terminaison de service Azure. ExpressRoute Global Reach connecte votre environnement local à votre cloud privé Azure VMware Solution.
 

:::image type="content" source="media/adjacency-overview-drawing-final.png" alt-text="Diagramme de la contiguïté du cloud privé Azure VMware Solution avec Azure et les environnements locaux." border="false":::

## <a name="hosts-clusters-and-private-clouds"></a>Hôtes, clusters et clouds privés

Les clusters et clouds privés Azure VMware Solution reposent sur un hôte d’infrastructure Azure nu et hyper-convergé. Les hôtes haut de gamme ont une RAM de 576 Go et deux processeurs Intel 18 cœurs de 2,3 GHz. Par ailleurs, ils bénéficient en outre de deux groupes de disques vSAN avec un niveau de capacité brut vSAN de 15,36 To (SSD) et un niveau de cache vSAN de 3,2 To (NVMe).

Vous pouvez déployer de nouveaux clouds privés par le biais du portail Azure ou d’Azure CLI.


## <a name="networking"></a>Mise en réseau

[!INCLUDE [avs-networking-description](includes/azure-vmware-solution-networking-description.md)]

Pour plus d’informations, consultez [Concepts relatifs aux réseaux](concepts-networking.md).

## <a name="access-and-security"></a>Accès et sécurité

Pour une sécurité renforcée, les clouds privés Azure VMware Solution utilisent le contrôle d’accès en fonction du rôle vSphere. Vous pouvez intégrer les fonctionnalités LDAP SSO vSphere à Azure Active Directory. Pour plus d’informations, consultez [Concepts relatifs à l’accès et à l’identité](concepts-identity.md).  

Par défaut, le chiffrement des données au repos vSAN est activé et il est utilisé pour garantir la sécurité du magasin de données vSAN. Pour plus d’informations, consultez [Concepts relatifs au stockage](concepts-storage.md).

## <a name="host-and-software-lifecycle-maintenance"></a>Maintenance du cycle de vie des hôtes et des logiciels

Les mises à niveau régulières du cloud privé Azure VMware Solution et des logiciels VMware garantissent la sécurité, la stabilité et l’exécution des ensembles de fonctionnalités les plus récents dans vos clouds privés. Pour plus d’informations, consultez [Maintenance de l’hôte et gestion du cycle de vie](concepts-private-clouds-clusters.md#host-maintenance-and-lifecycle-management).

## <a name="monitoring-your-private-cloud"></a>Supervision de votre cloud privé

Après le déploiement d’Azure VMware Solution dans votre abonnement, les [journaux Azure Monitor](../azure-monitor/overview.md) sont générés automatiquement. 

Dans votre cloud privé, vous pouvez :
- Collecter les journaux sur chacune de vos machines virtuelles
- [Télécharger et installer l’agent MMA](../azure-monitor/agents/log-analytics-agent.md#installation-options) sur les machines virtuelles Linux et Windows
- Activer l’[extension de diagnostic Azure](../azure-monitor/agents/diagnostics-extension-overview.md)
- [Créer et exécuter de nouvelles requêtes](../azure-monitor/logs/data-platform-logs.md#log-queries)
- Exécuter les mêmes requêtes que celles que vous exécutez habituellement sur vos machines virtuelles

Dans Azure VMware Solution, les modèles de supervision sont similaires à ceux des machines virtuelles Azure sur la plateforme IaaS. Pour obtenir des informations supplémentaires et des procédures, consultez [Supervision de machines virtuelles Azure avec Azure Monitor](../azure-monitor/vm/monitor-vm-azure.md).

## <a name="customer-communication"></a>Communication du client
[!INCLUDE [customer-communications](includes/customer-communications.md)]

## <a name="next-steps"></a>Étapes suivantes

L’étape suivante consiste à se familiariser avec les [concepts clés des clouds privés et des clusters](concepts-private-clouds-clusters.md).

<!-- LINKS - external -->

<!-- LINKS - internal -->
[concepts-private-clouds-clusters]: ./concepts-private-clouds-clusters.md

