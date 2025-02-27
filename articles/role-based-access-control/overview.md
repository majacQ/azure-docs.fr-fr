---
title: Qu’est-ce que le contrôle d’accès en fonction du rôle Azure (RBAC Azure) ?
description: Découvrez une vue d’ensemble du contrôle d’accès en fonction du rôle Azure (RBAC Azure). Utilisez les attributions de rôles pour contrôler l’accès aux ressources Azure.
services: active-directory
author: rolyon
manager: mtillman
ms.service: role-based-access-control
ms.topic: overview
ms.workload: identity
ms.date: 05/17/2021
ms.author: rolyon
ms.custom: contperf-fy21q1, azuread-video-2020
ms.openlocfilehash: 3b731b83264802884bba01ed3c32db6152d0a4f9
ms.sourcegitcommit: 87de14fe9fdee75ea64f30ebb516cf7edad0cf87
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/01/2021
ms.locfileid: "129356122"
---
# <a name="what-is-azure-role-based-access-control-azure-rbac"></a>Qu’est-ce que le contrôle d’accès en fonction du rôle Azure (RBAC Azure) ?

Il est vital pour toute organisation qui utilise le cloud de pouvoir gérer les accès aux ressources situées dans cloud. Le contrôle d’accès basé sur un rôle Azure (RBAC Azure) permet de gérer les utilisateurs ayant accès aux ressources Azure, les modes d’utilisation des ressources par ces derniers et les zones auxquelles ils ont accès.

Le contrôle RBAC Azure est un système d’autorisation basé sur [Azure Resource Manager](../azure-resource-manager/management/overview.md) qui propose une gestion affinée des accès des ressources Azure.

Cette vidéo fournit une vue d’ensemble rapide du RBAC Azure.

>[!VIDEO https://www.youtube.com/embed/Dzhm-garKBM]

## <a name="what-can-i-do-with-azure-rbac"></a>Comment utiliser le contrôle RBAC Azure ?

Voici quelques exemples de ce que vous pouvez faire avec le contrôle RBAC Azure :

- Autoriser un utilisateur à gérer des machines virtuelles dans un abonnement et un autre utilisateur à gérer des réseaux virtuels
- Permettre à un groupe d’administrateurs de base de données de gérer des bases de données SQL dans un abonnement
- Permettre à un utilisateur à gérer toutes les ressources dans un groupe de ressources, telles que des machines virtuelles, des sites Web et des sous-réseaux
- Permettre à une application d’accéder à toutes les ressources d’un groupe de ressources

## <a name="how-azure-rbac-works"></a>Fonctionnement du contrôle RBAC Azure

Les attributions de rôles Azure vous permettent de contrôler l’accès aux ressources avec RBAC Azure. Il s’agit d’un concept clé dont la compréhension est essentielle, à savoir la façon dont les autorisations sont appliquées. Une attribution de rôle se compose de trois éléments : un principal de sécurité, une définition de rôle et une étendue.

### <a name="security-principal"></a>Principal de sécurité

Un *principal de sécurité* est un objet qui représente un utilisateur, un groupe, un principal de service ou une identité managée demandant l’accès à des ressources Azure. Vous pouvez attribuer un rôle à l’un de ces principaux de sécurité.

![Schéma présentant les principaux types de sécurité pour l’attribution de rôle.](./media/shared/rbac-security-principal.png)

### <a name="role-definition"></a>Définition de rôle

Une *définition de rôle* est une collection d’autorisations. Elle est généralement simplement appelée *rôle*. Une définition de rôle liste les actions qui peuvent être effectuées, comme lire, écrire et supprimer. Les rôles peuvent être de haut niveau, comme propriétaire, ou spécifiques, comme lecteur de machines virtuelles.

![Schéma présentant un exemple de définition de rôle pour une attribution de rôle](./media/shared/rbac-role-definition.png)

Azure inclut plusieurs [rôles intégrés](built-in-roles.md) que vous pouvez utiliser. Par exemple, le rôle de [contributeur de machine virtuelle](built-in-roles.md#virtual-machine-contributor) permet à l’utilisateur de créer et gérer des machines virtuelles. Si les rôles intégrés ne répondent pas aux besoins spécifiques de votre organisation, vous pouvez créer vos propres [rôles personnalisés Azure](custom-roles.md).

Cette vidéo fournit une vue d’ensemble rapide des rôles intégrés et des rôles personnalisés.

>[!VIDEO https://www.youtube.com/embed/I1mefHptRgo]

Azure propose des actions de données qui vous permettent d’accorder l’accès aux données au sein d’un objet. Par exemple, si un utilisateur dispose d’un accès en lecture aux données d’un compte de stockage, il peut lire les objets blob ou les messages de ce compte de stockage.

Pour plus d’informations, consultez [Comprendre les définitions de rôle Azure](role-definitions.md).

### <a name="scope"></a>Étendue

*Étendue* représente l’ensemble des ressources auxquelles l’accès s’applique. Lorsque vous attribuez un rôle, vous pouvez restreindre les actions autorisées en définissant une étendue. Cette possibilité s’avère utile si vous voulez par exemple attribuer le rôle de [contributeur de site web](built-in-roles.md#website-contributor) à quelqu’un, mais seulement pour un groupe de ressources.

Dans Azure, vous pouvez spécifier une étendue à quatre niveaux : [groupe d’administration](../governance/management-groups/overview.md), abonnement, [groupe de ressources](../azure-resource-manager/management/overview.md#resource-groups) ou ressource. Les étendues sont structurées dans une relation parent-enfant. Vous pouvez attribuer des rôles à n’importe de ces niveaux d’étendue.

![Schéma présentant les niveaux d’étendue d’une attribution de rôle.](./media/shared/rbac-scope.png)

Pour plus d’informations sur l’étendue, consultez [Comprendre l’étendue](scope-overview.md).

### <a name="role-assignments"></a>Affectations de rôles

Une *attribution de rôle* est le processus d’attachement d’une définition de rôle à un utilisateur, un groupe, un principal de service ou une identité managée au niveau d’une étendue spécifique pour accorder des accès. La création d’une attribution de rôle permet d’accorder un accès, qui peut être révoqué par la suppression d’une attribution de rôle.

Le diagramme suivant montre un exemple d’attribution de rôle. Dans cet exemple, le rôle de [contributeur](built-in-roles.md#contributor) a été attribué au groupe Marketing pour le groupe de ressources pharma-sales. Cela signifie que les utilisateurs du groupe Marketing peuvent créer ou gérer n’importe quelle ressource Azure dans le groupe de ressources pharma-sales. Les utilisateurs Marketing n’ont pas accès aux ressources en dehors du groupe de ressources pharma-sales, sauf si elles font partie d’une autre attribution de rôle.

![Schéma présentant comment le principal de sécurité, la définition de rôle et l’étendue créent une attribution de rôle.](./media/overview/rbac-overview.png)

Vous pouvez attribuer des rôles en utilisant le portail Azure, Azure PowerShell, Azure CLI, des SDK Azure ou des API REST.

Pour plus d’informations, consultez [Étapes pour attribuer un rôle Azure](role-assignments-steps.md).

## <a name="groups"></a>Groupes

Les attributions de rôle sont transitives pour les groupes, ce qui signifie que si un utilisateur est membre d’un groupe et que ce groupe est membre d’un autre groupe auquel un rôle a été attribué, l’utilisateur a les autorisations définies dans l’attribution de rôle.

![Diagramme montrant comment les attributions de rôles sont transitives pour les groupes.](./media/overview/rbac-groups-transitive.png)

## <a name="multiple-role-assignments"></a>Attributions de rôles multiples

Que se passe-t-il si plusieurs attributions de rôles se chevauchent ? Le contrôle RBAC Azure étant un modèle additif, vos autorisations effectives correspondent à la somme de vos attributions de rôles. Prenons l’exemple suivant où un utilisateur reçoit le rôle Contributeur pour l’étendue de l’abonnement et le rôle Lecteur pour un groupe de ressources. La somme des autorisations du rôle Contributeur et des autorisations du rôle Lecteur correspond effectivement au rôle Contributeur pour l’abonnement. Ainsi, dans ce cas, l’attribution du rôle Lecteur n’a aucun impact.

![Schéma présentant le chevauchement de plusieurs attributions de rôles.](./media/overview/rbac-multiple-roles.png)

## <a name="deny-assignments"></a>Affectations de refus

Le contrôle RBAC Azure, qui était jusqu’à maintenant exclusivement un modèle d’autorisation sans possibilité de refus, prend désormais en charge des affectations de refus dans une certaine mesure. À l’instar d’une attribution de rôle, une *affectation de refus* attache un ensemble d’actions de refus à un utilisateur, un groupe, un principal de service ou une identité managée, sur une étendue spécifique, afin de refuser l’accès. Une attribution de rôle définit un ensemble d’actions *autorisées*, tandis qu’une affectation de refus définit un ensemble d’actions *non autorisées*. En d’autres termes, les affectations de refus empêchent les utilisateurs d’effectuer des actions spécifiées, même si une attribution de rôle leur accorde l’accès. Les affectations de refus ont priorité sur les attributions de rôles.

Pour plus d’informations, consultez [Comprendre les affectations de refus Azure](deny-assignments.md).

## <a name="how-azure-rbac-determines-if-a-user-has-access-to-a-resource"></a>Comment le contrôle RBAC Azure détermine si un utilisateur a accès à une ressource

Voici les principales étapes suivies par le contrôle RBAC Azure pour déterminer si vous avez accès à une ressource. Ces étapes s’appliquent aux services Azure Resource Manager ou de plan de données intégrés à Azure RBAC. Cela s’avère utile pour comprendre si vous tentez de résoudre un problème d’accès.

1. Un utilisateur (ou principal de service) se procure un jeton pour Azure Resource Manager.

    Le jeton inclut les appartenances de l’utilisateur à des groupes (notamment les appartenances à des groupes transitifs).

1. L’utilisateur effectue un appel d’API REST vers Azure Resource Manager avec le jeton joint.

1. Azure Resource Manager récupère toutes les attributions de rôle et les affectations de refus qui s’appliquent à la ressource sur laquelle l’action est entreprise.

1. Si c’est le cas, l’accès est bloqué. Dans le cas contraire, l’évaluation se poursuit.

1. Azure Resource Manager restreint les attributions de rôles qui s’appliquent à cet utilisateur ou à son groupe, et détermine les rôles dont l’utilisateur dispose pour cette ressource.

1. Azure Resource Manager détermine si l’action contenue dans l’appel d’API est incluse dans les rôles dont l’utilisateur dispose pour cette ressource. Si les rôles incluent `Actions` un caractère générique (`*`), les autorisations effectives sont calculées en soustrayant `NotActions` de l’autorisation `Actions`. De même, la même soustraction est effectuée pour toutes les actions de données.

    `Actions - NotActions = Effective management permissions`

    `DataActions - NotDataActions = Effective data permissions`

1. Si l’utilisateur n’a aucun rôle avec l’action appropriée dans l’étendue demandée, l’accès n’est pas autorisé. Dans le cas contraire, toutes les conditions sont évaluées.

1. Si l’attribution de rôle comprend des conditions, elles sont évaluées. Sinon, l’accès est accordé.

1. Si les conditions sont remplies, l’accès est autorisé. Sinon, l’accès n’est pas accordé.

Le schéma suivant est un résumé de la logique d’évaluation.

![Organigramme de la logique d’évaluation pour déterminer l’accès à une ressource.](./media/overview/evaluation-logic.png)

## <a name="license-requirements"></a>Conditions de licence :

[!INCLUDE [Azure AD free license](../../includes/active-directory-free-license.md)]

## <a name="next-steps"></a>Étapes suivantes

- [Attribuer des rôles Azure à l’aide du portail Azure](role-assignments-portal.md)
- [Comprendre les différents rôles](rbac-and-directory-admin-roles.md)
- [Framework d’adoption du cloud : Gestion de l’accès aux ressources dans Azure](/azure/cloud-adoption-framework/govern/resource-consistency/resource-access-management)
