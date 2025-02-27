---
title: Stratégies recommandées pour les services Azure
description: Décrit comment rechercher et appliquer les stratégies recommandées pour les services Azure, tels que Machines virtuelles Azure.
ms.date: 08/17/2021
ms.topic: conceptual
ms.custom: generated
ms.openlocfilehash: 06f68dec2c2a1ec8fa717dc721916c070a653b10
ms.sourcegitcommit: 5f659d2a9abb92f178103146b38257c864bc8c31
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/17/2021
ms.locfileid: "122534951"
---
# <a name="recommended-policies-for-azure-services"></a>Stratégies recommandées pour les services Azure

Les clients qui débutent avec Azure Policy recherchent souvent des définitions de stratégies communes pour gérer et régir leurs ressources. Les **stratégies recommandées** d’Azure Policy fournissent une liste ciblée des définitions de stratégies courantes avec lesquelles commencer. L’expérience des **stratégies recommandées** pour une ressource prise en charge est incorporée dans l’expérience du portail pour cette ressource.

Pour plus d’informations sur les éléments intégrés Azure Policy supplémentaires, consultez [Définitions intégrées d’Azure Policy](../samples/built-in-policies.md).

## <a name="azure-virtual-machines"></a>Machines virtuelles Azure

Les **stratégies recommandées** pour [machines virtuelles Azure](../../../virtual-machines/index.yml) figurent dans la page **Vue d’ensemble** des machines virtuelles et sous l’onglet **Fonctionnalités**. Dans la carte _Azure Policy_, sélectionnez le texte « Non configuré » ou « N° assigné » pour ouvrir un volet latéral avec les stratégies recommandées. Toute définition de stratégie déjà attribuée à une étendue dont la machine virtuelle est membre est grisée. Sélectionnez les stratégies recommandées à appliquer à cette machine virtuelle et sélectionnez **Attribuer des stratégies** pour créer une attribution pour chacune d’elles.

Lorsqu’une organisation atteint sa maturité en [organisant ses ressources et la hiérarchie des ressources](/azure/cloud-adoption-framework/ready/azure-best-practices/organize-subscriptions), il est recommandé de faire passer ces attributions de stratégie d’un niveau par ressource au niveau de l’abonnement ou du [groupe d’administration](../../management-groups/index.yml).

### <a name="azure-virtual-machines-recommended-policies"></a>Stratégies recommandées des machines virtuelles Azure

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Auditer des machines virtuelles pour lesquelles la reprise d’activité après sinistre n’est pas configurée](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0015ea4d-51ff-4ce3-8d8c-f3f8f0179a56) |Auditez les machines virtuelles configurées sans reprise d’activité. Pour en savoir plus sur la reprise d’activité, consultez [https://aka.ms/asr-doc](../../../site-recovery/index.yml). |auditIfNotExists |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Compute/RecoveryServices_DisasterRecovery_Audit.json) |
|[Faire l’audit des machines virtuelles n’utilisant aucun disque managé](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F06a78e20-9358-41c9-923c-fb736d382a4d) |Cette stratégie fait l’audit des machines virtuelles n’utilisant pas de disque managé |audit |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Compute/VMRequireManagedDisk_Audit.json) |
|[La sauvegarde Azure doit être activée pour les machines virtuelles](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F013e242c-8828-4970-87b3-ab247555486d) |Assurez la protection de vos machines virtuelles Azure en activant la sauvegarde Azure. La Sauvegarde Azure est une solution de protection des données sécurisée et économique pour Azure. |AuditIfNotExists, Désactivé |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Backup/VirtualMachines_EnableAzureBackup_Audit.json) |

## <a name="next-steps"></a>Étapes suivantes

- Consultez des exemples à la page [Exemples Azure Policy](../samples/index.md).
- Consultez la page [Compréhension des effets de Policy](./effects.md).
- Découvrez comment [corriger des ressources non conformes](../how-to/remediate-resources.md).
