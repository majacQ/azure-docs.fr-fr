---
title: Configurer la géoréplication pour les instances Azure Cache pour Redis Premium
description: Découvrez comment répliquer vos instances Azure Cache pour Redis Premium dans des régions Azure
author: curib
ms.service: cache
ms.topic: conceptual
ms.date: 02/08/2021
ms.author: cauribeg
ms.openlocfilehash: 989284bd10fc5d452a738d027c693a15f7871b9b
ms.sourcegitcommit: 216b6c593baa354b36b6f20a67b87956d2231c4c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/11/2021
ms.locfileid: "129729914"
---
# <a name="configure-geo-replication-for-premium-azure-cache-for-redis-instances"></a>Configurer la géoréplication pour les instances Azure Cache pour Redis Premium

Dans cet article, vous allez apprendre à configurer un cache Azure géorépliqué à l’aide du portail Azure.

La géoréplication relie deux instances Azure Cache pour Redis Premium et crée une relation de réplication des données. Ces instances de cache se trouvent généralement dans des régions Azure différentes, même si cela n’est pas nécessaire. Une instance joue le rôle d’instance principale et l’autre d’instance secondaire. L’instance principale gère les demandes de lecture et d’écriture et propage les modifications à l’instance secondaire. Ce processus se poursuit jusqu’à ce que le lien entre les deux instances soit supprimé.

> [!NOTE]
> La géoréplication a été conçue comme une solution de reprise d’activité.
>
>

## <a name="geo-replication-prerequisites"></a>Conditions préalables à la géoréplication

Pour configurer la géoréplication entre deux caches, les conditions préalables suivantes doivent être remplies :

- Les deux caches sont de [niveau Premium](cache-overview.md#service-tiers).
- Les deux caches figurent dans le même abonnement Azure.
- La taille du cache lié secondaire est supérieure ou égale à celle du cache lié principal.
- Les deux caches sont créés et en cours d'exécution.

> [!NOTE]
> Le transfert de données entre les régions Azure est facturé aux [tarifs de bande passante](https://azure.microsoft.com/pricing/details/bandwidth/) standard.

Certaines fonctionnalités ne sont pas prises en charge par la géoréplication :

- La redondance de zone n’est pas prise en charge avec la géoréplication.
- La persistance n'est pas prise en charge par la géoréplication.
- Le clustering est pris en charge s'il est activé pour les deux caches et si ceux-ci possèdent le même nombre de partitions.
- Les caches dans le même réseau virtuel (VNet) sont pris en charge.
- Les caches situés dans des réseaux virtuels différents sont pris en charge avec des mises en garde. Pour plus d’informations, consultez [Puis-je utiliser la géoréplication avec mes caches dans un réseau virtuel ?](#can-i-use-geo-replication-with-my-caches-in-a-vnet)

Une fois la géoréplication configurée, les restrictions suivantes s’appliquent à votre paire de caches liés :

- Le cache lié secondaire est en lecture seule. Il n’est pas possible d’y écrire des données. Si vous choisissez de lire à partir de l’instance géographique secondaire, il est important de noter que, à chaque fois qu’une synchronisation complète des données se produit entre les instances principale et secondaire (lors de la mise à jour de l’instance géographique principale ou secondaire et lors de certains scénarios de redémarrage), l’instance géographique secondaire lève des erreurs (indiquant qu’une synchronisation complète des données est en cours) sur toute opération Redis, jusqu’à la fin de la synchronisation complète des données. Les applications qui lisent depuis une instance géographique secondaire doivent être générées pour revenir à l’instance géographique principale chaque fois que l’instance géographique secondaire génère de telles erreurs.
- Toutes les données présentes dans le cache lié secondaire avant l’ajout du lien sont supprimées. Toutefois, en cas de suppression ultérieure de la géoréplication, les données répliquées restent dans le cache lié secondaire.
- Vous ne pouvez pas procéder à la [mise à l'échelle](cache-how-to-scale.md) d'un seul des deux caches lorsque ceux-ci sont liés.
- Vous ne pouvez pas [modifier le nombre de partitions](cache-how-to-premium-clustering.md) si le clustering est activé pour le cache.
- Vous ne pouvez activer la persistance sur aucun des caches.
- Vous pouvez [exporter](cache-how-to-import-export-data.md#export) à partir de l'un ou l'autre des caches.
- Vous ne pouvez pas [importer](cache-how-to-import-export-data.md#import) dans le cache lié secondaire.
- Vous ne pouvez pas supprimer les caches liés ou le groupe de ressources qui les contient tant qu'ils n'ont pas été dissociés. Pour plus d’informations, consultez [Pourquoi ma tentative de suppression de mon cache lié a-t-elle échoué ?](#why-did-the-operation-fail-when-i-tried-to-delete-my-linked-cache)
- Si les caches se trouvent dans des régions différentes, des frais de sortie de réseau s'appliquent aux données déplacées d'une région à l'autre. Pour plus d’informations, consultez [Combien coûte la réplication de mes données entre des régions Azure ?](#how-much-does-it-cost-to-replicate-my-data-across-azure-regions)
- Il n'y a pas de basculement automatique entre le cache lié principal et le cache lié secondaire. Pour plus d'informations et pour apprendre à basculer une application cliente, consultez [Comment fonctionne le basculement vers le cache lié secondaire ?](#how-does-failing-over-to-the-secondary-linked-cache-work)

## <a name="add-a-geo-replication-link"></a>Ajouter un lien de géoréplication

1. Pour lier deux caches à des fins de géoréplication, cliquez d'abord sur **Géoréplication** dans le menu Ressources du cache que vous souhaitez utiliser comme cache lié principal. Ensuite, dans le panneau **Géoréplication** sur la gauche, cliquez sur **Ajouter une liaison de réplication de cache**.

    :::image type="content" source="media/cache-how-to-geo-replication/cache-geo-location-menu.png" alt-text="Menu de géoréplication du cache":::

1. Dans la liste **Caches compatibles**, cliquez sur le nom du cache secondaire souhaité. Si le cache secondaire ne figure pas dans la liste, vérifiez que les [conditions préalables à la géoréplication](#geo-replication-prerequisites) du cache secondaire sont remplies. Pour filtrer les caches par région, sélectionnez la région dans la carte afin de n’afficher que les caches figurant dans la liste **Caches compatibles**.

    :::image type="content" source="media/cache-how-to-geo-replication/cache-geo-location-select-link.png" alt-text="Sélectionner le cache compatible":::

    Vous pouvez également lancer le processus de liaison ou afficher des détails sur le cache secondaire à l'aide du menu contextuel.

    :::image type="content" source="media/cache-how-to-geo-replication/cache-geo-location-select-link-context-menu.png" alt-text="Menu contextuel de la géoréplication":::

1. Sélectionnez **Lier** pour lier les deux caches et commencer le processus de réplication.

    :::image type="content" source="media/cache-how-to-geo-replication/cache-geo-location-confirm-link.png" alt-text="Lier des caches":::

1. Vous pouvez voir la progression du processus de réplication dans le panneau **Géoréplication** sur la gauche.

    :::image type="content" source="media/cache-how-to-geo-replication/cache-geo-location-linking.png" alt-text="État du lien":::

    Vous pouvez également voir l’état de la liaison sur la gauche, en utilisant la **Vue d’ensemble** pour les caches principal et secondaire.

    :::image type="content" source="media/cache-how-to-geo-replication/cache-geo-location-link-status.png" alt-text="Capture d’écran montrant comment afficher l’état de liaison pour les caches principal et secondaire.":::

    Une fois le processus de réplication terminé, l’**État du lien** devient **Réussi**.

    :::image type="content" source="media/cache-how-to-geo-replication/cache-geo-location-link-successful.png" alt-text="État du cache":::

    Le cache lié principal reste disponible pour une utilisation pendant le processus de liaison. Le cache lié secondaire n'est pas disponible tant que le processus de liaison n'est pas terminé.

## <a name="remove-a-geo-replication-link"></a>Supprimer un lien de géoréplication

1. Pour supprimer la liaison entre deux caches et arrêter la géoréplication, dans le panneau **Géoréplication** sur la gauche, cliquez sur **Dissocier les caches**.

    :::image type="content" source="media/cache-how-to-geo-replication/cache-geo-location-unlink.png" alt-text="Dissocier les caches":::

    Une fois le processus de dissociation terminé, le cache secondaire est disponible tant en lecture qu’en écriture.

>[!NOTE]
>En cas de suppression du lien de géoréplication, les données répliquées à partir du cache lié principal restent dans le cache secondaire.
>
>

## <a name="geo-replication-faq"></a>FAQ concernant la géoréplication

- [Puis-je utiliser la géoréplication avec un cache de niveau Standard ou De base ?](#can-i-use-geo-replication-with-a-standard-or-basic-tier-cache)
- [Mon cache est-il disponible pendant le processus de liaison ou de dissociation ?](#is-my-cache-available-for-use-during-the-linking-or-unlinking-process)
- [Puis-je lier plus de deux caches ?](#can-i-link-more-than-two-caches-together)
- [Puis-je lier deux caches d’abonnements Azure différents ?](#can-i-link-two-caches-from-different-azure-subscriptions)
- [Puis-je lier deux caches de tailles différentes ?](#can-i-link-two-caches-with-different-sizes)
- [Puis-je utiliser la géoréplication quand le clustering est activé ?](#can-i-use-geo-replication-with-clustering-enabled)
- [Puis-je utiliser la géoréplication avec mes caches dans un réseau virtuel ?](#can-i-use-geo-replication-with-my-caches-in-a-vnet)
- [Quelle est la planification de réplication pour la géoréplication Redis ?](#what-is-the-replication-schedule-for-redis-geo-replication)
- [Quelle est la durée de réplication pour la géoréplication ?](#how-long-does-geo-replication-replication-take)
- [Y a-t-il un point de récupération de la réplication garanti ?](#is-the-replication-recovery-point-guaranteed)
- [Puis-je utiliser PowerShell ou Azure CLI pour gérer la géoréplication ?](#can-i-use-powershell-or-azure-cli-to-manage-geo-replication)
- [Combien coûte la réplication de mes données entre des régions Azure ?](#how-much-does-it-cost-to-replicate-my-data-across-azure-regions)
- [Pourquoi ma tentative de suppression de mon cache lié a-t-elle échoué ?](#why-did-the-operation-fail-when-i-tried-to-delete-my-linked-cache)
- [Quelle région dois-je utiliser pour mon cache lié secondaire ?](#what-region-should-i-use-for-my-secondary-linked-cache)
- [Comment fonctionne le basculement vers le cache lié secondaire ?](#how-does-failing-over-to-the-secondary-linked-cache-work)
- [Puis-je configurer un pare-feu avec la géoréplication ?](#can-i-configure-a-firewall-with-geo-replication)

### <a name="can-i-use-geo-replication-with-a-standard-or-basic-tier-cache"></a>Puis-je utiliser la géoréplication avec un cache de niveau Standard ou De base ?

Non, la géoréplication est disponible uniquement pour des caches de niveau Premium.

### <a name="is-my-cache-available-for-use-during-the-linking-or-unlinking-process"></a>Mon cache est-il disponible pendant le processus de liaison ou de dissociation ?

- Lors d'une liaison, le cache lié principal reste disponible jusqu'à la fin du processus.
- Lors d'une liaison, le cache lié secondaire n'est pas disponible tant que le processus n'est pas terminé.
- Lors d'une dissociation, les deux caches restent disponibles jusqu'à la fin du processus.

### <a name="can-i-link-more-than-two-caches-together"></a>Puis-je lier plus de deux caches ?

Non, vous ne pouvez lier que deux caches.

### <a name="can-i-link-two-caches-from-different-azure-subscriptions"></a>Puis-je lier deux caches d’abonnements Azure différents ?

Non, les deux caches doivent figurer dans le même abonnement Azure.

### <a name="can-i-link-two-caches-with-different-sizes"></a>Puis-je lier deux caches de tailles différentes ?

Oui, pour autant que le cache lié secondaire soit plus grand que le cache lié principal.

### <a name="can-i-use-geo-replication-with-clustering-enabled"></a>Puis-je utiliser la géoréplication quand le clustering est activé ?

Oui, pour autant que les deux caches aient le même nombre de partitions.

### <a name="can-i-use-geo-replication-with-my-caches-in-a-vnet"></a>Puis-je utiliser la géoréplication avec mes caches dans un réseau virtuel ?

Oui, la géoréplication de caches dans des réseaux virtuels est prise en charge avec des mises en garde :

- La géoréplication entre caches figurant dans un même réseau virtuel est prise en charge.
- La géoréplication entre caches figurant dans des réseaux virtuels différents est également prise en charge.
  - Si les réseaux virtuels se trouvent dans la même région, vous pouvez les connecter via un [peering de réseaux virtuels](../virtual-network/virtual-network-peering-overview.md) ou une [connexion de passerelle VPN de réseau virtuel à réseau virtuel](../vpn-gateway/vpn-gateway-howto-vnet-vnet-resource-manager-portal.md).
  - Si les réseaux virtuels se trouvent dans des régions différentes, la géoréplication à l’aide du peering de réseau virtuel est prise en charge, mais une machine virtuelle cliente dans le réseau virtuel 1 (région 1) ne pourra pas accéder au cache dans le réseau virtuel 2 (région 2) via son nom DNS en raison d’une contrainte liée aux équilibreurs de charge internes de base. Pour plus d'informations sur les contraintes liées au peering de réseaux virtuels, consultez [Réseau virtuel - Peering - Exigences et contraintes](../virtual-network/virtual-network-manage-peering.md#requirements-and-constraints). Nous vous recommandons d’utiliser une connexion de passerelle VPN de réseau virtuel à réseau virtuel.
  
[Ce modèle Azure](https://azure.microsoft.com/resources/templates/redis-vnet-geo-replication/) vous permet de déployer rapidement deux caches géorépliqués dans un réseau virtuel connecté avec une connexion de passerelle VPN de réseau virtuel à réseau virtuel.

### <a name="what-is-the-replication-schedule-for-redis-geo-replication"></a>Quelle est la planification de réplication pour la géoréplication Redis ?

La réplication est continue et asynchrone. Elle ne se produit pas selon une planification spécifique. Toutes les écritures effectuées sur le cache principal sont instantanément répliquées de façon asynchrone dans le cache secondaire.

### <a name="how-long-does-geo-replication-replication-take"></a>Quelle est la durée de réplication pour la géoréplication ?

La réplication s'effectue en continu de manière incrémentielle et asynchrone. Sa durée est proche de la latence interrégion. Dans certaines circonstances, le cache secondaire peut avoir besoin d'une synchronisation complète des données à partir du cache principal. Dans ce cas, la durée de la réplication dépend d'un certain nombre de facteurs, tels que la charge sur le cache principal, la bande passante réseau disponible et la latence interrégion. Nous avons relevé que la durée de réplication d'une paire géorépliquée complète de 53 Go peut être comprise entre 5 et 10 minutes.

### <a name="is-the-replication-recovery-point-guaranteed"></a>Y a-t-il un point de récupération de la réplication garanti ?

Pour les caches en mode géorépliqué, la persistance est désactivée. Si une paire géorépliquée est dissociée, comme dans le cas d'un basculement initié par le client, les données du cache lié secondaire restent synchronisées jusqu'à ce moment-là. Aucun point de récupération n'est garanti dans de telles situations.

Pour obtenir un point de récupération, [exportez](cache-how-to-import-export-data.md#export) à partir de l'un ou l'autre des caches. Vous pourrez ensuite [importer](cache-how-to-import-export-data.md#import) dans le cache lié principal.

### <a name="can-i-use-powershell-or-azure-cli-to-manage-geo-replication"></a>Puis-je utiliser PowerShell ou Azure CLI pour gérer la géoréplication ?

Oui, la géoréplication peut être gérée à l'aide du portail Azure, de PowerShell ou d'Azure CLI. Pour plus d'informations, consultez la documentation de [PowerShell](/powershell/module/az.rediscache/#redis_cache) ou d'[Azure CLI](/cli/azure/redis/server-link).

### <a name="how-much-does-it-cost-to-replicate-my-data-across-azure-regions"></a>Combien coûte la réplication de mes données entre régions Azure ?

Lorsque vous utilisez la géoréplication, les données du cache lié principal sont répliquées vers le cache lié secondaire. Le transfert de données est gratuit si les deux caches liés se trouvent dans la même région. Si les deux caches liés se trouvent dans des régions différentes, les frais de transfert de données correspondent aux frais de sortie de réseau des données transférées dans l'une ou l'autre des régions. Pour plus d'informations, consultez [Détails de la tarification de la bande passante](https://azure.microsoft.com/pricing/details/bandwidth/).

### <a name="why-did-the-operation-fail-when-i-tried-to-delete-my-linked-cache"></a>Pourquoi ma tentative de suppression de mon cache lié a-t-elle échoué ?

Les caches géorépliqués et leurs groupes de ressources ne peuvent pas être supprimés tant qu'ils sont liés. Vous devez d'abord supprimer le lien de géoréplication. Si vous tentez de supprimer le groupe de ressources contenant l’un des caches liés ou les deux, les autres ressources du groupe sont supprimées, mais le groupe de ressources reste dans l’état `deleting` et les caches liés dans le groupe de ressources restent dans l’état `running`. Pour supprimer complètement le groupe de ressources et les caches liés qu’il contient, dissociez les caches comme décrit dans [Supprimer un lien de géoréplication](#remove-a-geo-replication-link).

### <a name="what-region-should-i-use-for-my-secondary-linked-cache"></a>Quelle région dois-je utiliser pour mon cache lié secondaire ?

En règle générale, votre cache doit se trouver dans la même région Azure que l'application qui y accède. Pour les applications disposant d'une région principale et d'une région de secours distinctes, il est recommandé que vos caches principal et secondaire soient présents dans ces mêmes régions. Pour plus d’informations sur les régions liées, consultez [Meilleures pratiques : régions Azure liées](../best-practices-availability-paired-regions.md).

### <a name="how-does-failing-over-to-the-secondary-linked-cache-work"></a>Comment fonctionne le basculement vers le cache lié secondaire ?

Le basculement automatique entre régions Azure n'est pas pris en charge pour les caches géorépliqués. Dans un scénario de récupération d'urgence, les clients doivent faire apparaître toute la pile d'applications de manière coordonnée dans leur région de sauvegarde. Le fait de laisser des composants d'application individuels décider eux-mêmes à quel moment basculer vers leurs sauvegardes peut avoir un impact négatif sur les performances. 

L'un des principaux avantages de Redis est qu'il s'agit d'un magasin à très faible latence. Si l'application principale du client se trouve dans une région différente de celle de son cache, le temps d'aller-retour supplémentaire aura un impact non négligeable sur les performances. Nous évitons donc les basculements automatiques en raison des problèmes temporaires de disponibilité que cela engendrerait.

Pour lancer un basculement initié par le client, commencez par dissocier les caches. Puis modifiez votre client Redis pour qu'il utilise le point de terminaison de connexion du cache secondaire (précédemment lié). Une fois les deux caches dissociés, le cache secondaire redevient un cache en lecture-écriture normal, et accepte directement les demandes des clients Redis.

### <a name="can-i-configure-a-firewall-with-geo-replication"></a>Puis-je configurer un pare-feu avec la géoréplication ?

Oui, vous pouvez configurer un [pare-feu](./cache-configure.md#firewall) avec la géoréplication. Pour que la géoréplication fonctionne avec le pare-feu, assurez-vous que l’adresse IP du cache secondaire est ajoutée aux règles de pare-feu du cache principal.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les fonctionnalités d’Azure Cache pour Redis.

- [Niveaux de service Azure Cache pour Redis](cache-overview.md#service-tiers)
- [Haute disponibilité pour Azure Cache pour Redis](cache-high-availability.md)
