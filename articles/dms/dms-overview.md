---
title: Qu’est-ce qu’Azure Database Migration Service ?
description: Vue d’ensemble d’Azure Database Migration Service, qui fournit des migrations transparentes de nombreuses sources de base de données vers des plateformes de données Azure.
services: database-migration
author: pochiraju
ms.author: rajpo
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.topic: overview
ms.date: 09/01/2021
ms.openlocfilehash: 68d462a93d891c25602bb305417ddeb64f5f02a1
ms.sourcegitcommit: 851b75d0936bc7c2f8ada72834cb2d15779aeb69
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/31/2021
ms.locfileid: "123317274"
---
# <a name="what-is-azure-database-migration-service"></a>Qu’est-ce qu’Azure Database Migration Service ?

Azure Database Migration Service est un service entièrement géré conçu pour permettre des migrations fluides de diverses sources de base de données vers des plateformes de données Azure avec un temps d’arrêt minime (migrations en ligne).

[!INCLUDE [database-migration-services-sql-mi-sql-vm](../../includes/database-migration-services-sql-mi-sql-vm.md)]

## <a name="migrate-databases-to-azure-with-familiar-tools"></a>Migrer des bases de données vers Azure avec des outils bien connus

Azure Database Migration Service intègre des fonctionnalités de nos services et outils existants. Il fournit aux clients une solution complète hautement disponible. Le service utilise [l’Assistant de migration des données](/sql/dma/dma-overview) pour générer des rapports d’évaluation qui fournissent des recommandations concernant les modifications requises avant d’effectuer une migration. Libre à vous d’effectuer ou non une correction nécessaire. Quand vous êtes prêt à commencer le processus de migration, Azure Database Migration Service effectue toutes les étapes associées. Vous pouvez lancer vos projets de migration et ne plus vous en occuper, car vous savez que le processus s’appuie sur les bonnes pratiques déterminées par Microsoft. 

> [!NOTE]
> L’utilisation d’Azure Database Migration Service pour effectuer une migration en ligne nécessite la création d’une instance basée sur le niveau tarifaire Premium.

## <a name="regional-availability"></a>Disponibilité régionale

Pour obtenir des informations à jour sur la disponibilité régionale d’Azure Database Migration Service, consultez [Produits disponibles par région](https://azure.microsoft.com/global-infrastructure/services/?products=database-migration).

## <a name="pricing"></a>Tarifs

Pour obtenir des informations à jour sur les tarifs d’Azure Database Migration Service, consultez [Tarifs d’Azure Database Migration Service](https://azure.microsoft.com/pricing/details/database-migration/).

## <a name="next-steps"></a>Étapes suivantes

* [État des scénarios de migration pris en charge par Azure Database Migration Service](resource-scenario-status.md).
* [Créer une instance d’Azure Database Migration Service à l’aide du portail Azure](quickstart-create-data-migration-service-portal.md).
* [Migrer SQL Server vers Azure SQL Database](tutorial-sql-server-to-azure-sql.md).
* [Vue d’ensemble de la configuration requise pour l’utilisation d’Azure Database Migration Service](pre-reqs.md).
* [Questions fréquentes (FAQ) sur l’utilisation d’Azure Database Migration Service](faq.yml).
* [Services et outils disponibles pour les scénarios de migration de données](dms-tools-matrix.md).