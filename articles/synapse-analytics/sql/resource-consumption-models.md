---
title: Consommation de ressources Synapse SQL
description: Découvrez les modèles de consommation Synapse SQL dans Azure Synapse Analytics.
services: synapse analytics
author: vvasic-msft
ms.service: synapse-analytics
ms.subservice: sql
ms.topic: overview
ms.date: 04/15/2020
ms.author: vvasic
ms.reviewer: jrasnick
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: c135b0e1e1e5c1b8c57b4b8ba7af06d165f00ac8
ms.sourcegitcommit: 6c6b8ba688a7cc699b68615c92adb550fbd0610f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/13/2021
ms.locfileid: "121862336"
---
# <a name="synapse-sql-resource-consumption"></a>Consommation de ressources Synapse SQL

Cet article décrit les modèles de consommation des ressources de Synapse SQL.

## <a name="serverless-sql-pool"></a>Pool SQL serverless

Le pool SQL serverless est un service de paiement à la requête qui ne nécessite pas de choisir la taille appropriée. Le système s’ajuste automatiquement en fonction de vos besoins, ce qui vous libère de la gestion de votre infrastructure et du choix à faire quant à la taille adaptée pour votre solution.

## <a name="dedicated-sql-pool---data-warehouse-units-dwus-and-compute-data-warehouse-units-cdwus"></a>Pool SQL dédié - Unités DWU (Data Warehouse Unit) et cDWU (compute Data Warehouse Unit)

Suggestions pour choisir le nombre idéal de Data Warehouse Units (DWU) en vue d’optimiser les coûts et les performances, et indications pour changer le nombre d’unités.

### <a name="data-warehouse-units"></a>Data Warehouse Units

Un pool Synapse SQL représente une collection de ressources analytiques en cours de provisionnement. Les ressources analytiques sont définies comme une combinaison d’UC, de mémoire et d’E/S. Ces trois ressources sont regroupées dans des unités d’échelle de calcul appelées Data Warehouse Units (DWU). Une DWU représente une mesure abstraite et standardisée des ressources de calcul et de performances. Un changement de votre niveau de service modifie le nombre de DWU disponibles pour le système. À son tour, cette modification ajuste les performances et le coût de votre système.

Pour un meilleur niveau de performance, vous pouvez augmenter le nombre de DWU. Pour un niveau de performance inférieur, vous pouvez diminuer le nombre de DWU. Les coûts de stockage et de calcul sont facturés séparément. Ainsi, la modification des DWU n’a pas d’effet sur les coûts de stockage.

Les performances des DWU sont basées sur les métriques de charge de travail d’entrepôt de données suivantes :

- À quelle vitesse une requête d’entreposage de données standard peut-t-elle analyser un grand nombre de lignes avant d’effectuer une agrégation complexe ? C’est une opération très gourmande en E/S et en UC.
- À quelle vitesse un entrepôt de données peut-il traiter des données provenant du stockage d'objets blob Azure ou d’Azure Data Lake ? C’est une opération très gourmande en réseau et en UC.
- Rapidité avec laquelle la commande T-SQL [`CREATE TABLE AS SELECT`](/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse?view=azure-sqldw-latest&preserve-view=true) peut copier une table. Cette opération implique la lecture des données sur le système de stockage, leur distribution entre les nœuds de l’appliance et la réécriture dans le système de stockage. Cette opération est très gourmande en UC, E/S et réseau.

Augmentation du nombre de DWU :

- Modification linéaire des performances du système pour les analyses, les agrégations et les instructions CTAS
- Augmentation du nombre de processus de lecture et d’écriture pour les opérations de chargement PolyBase
- Augmentation du nombre maximal de requêtes simultanées et d’emplacements de concurrence

### <a name="service-level-objective"></a>Objectif de niveau de service

L’Objectif de niveau de service (SLO) est le paramètre d’extensibilité qui détermine le niveau de coût et de performance de votre entrepôt de données. Les niveaux de service pour Gen2 sont mesurés en unités cDWU (compute Data Warehouse Unit), par exemple DW2000C. Les niveaux de service Gen1 sont mesurés en unités DWU (Data Warehouse Unit), par exemple DW2000.

L’Objectif de niveau de service (SLO) est le paramètre d’extensibilité qui détermine le niveau de coût et de performance de votre entrepôt de données. Les niveaux de service pour le pool SQL dédié Gen2 sont mesurés en unités DWU (Data Warehouse Unit), par exemple DW2000c.

> [!NOTE]
> Azure Synapse Analytics Gen2 a récemment ajouté des fonctionnalités de mise à l’échelle supplémentaires pour prendre en charge les niveaux de calcul inférieurs de 100 cDWU. Les entrepôts de données Gen1 existants qui ont besoin des niveaux de calcul inférieurs peuvent désormais être mis à niveau vers Gen2 dans les régions actuellement disponibles sans aucun coût supplémentaire.  Si votre région n'est pas encore prise en charge, vous pouvez procéder à une mise à niveau vers une région qui l'est. Pour plus d’informations, consultez [Mettre à niveau vers Gen2](../sql-data-warehouse/upgrade-to-latest-generation.md?context=/azure/synapse-analytics/context/context).

Dans T-SQL, le paramètre SERVICE_OBJECTIVE détermine le niveau de service et le niveau de performance de votre pool SQL dédié.

```sql
CREATE DATABASE mySQLDW
(Edition = 'Datawarehouse'
 ,SERVICE_OBJECTIVE = 'DW1000c'
)
;
```

### <a name="performance-tiers-and-data-warehouse-units"></a>Niveaux de performances et Data Warehouse Units

Chaque niveau de performances utilise une unité de mesure légèrement différente pour ses DWU. Cette différence est répercutée sur la facture, car l’unité d’échelle s’applique directement à la facturation.

- Les entrepôts de données Gen1 sont mesurés en unités DWU.
- Les entrepôts de données Gen2 sont mesurés en unités cDWU.

Les DWU et les cDWU prennent en charge la mise à l’échelle du calcul (augmentation ou réduction) et la suspension du calcul lorsque vous n’avez pas besoin d’utiliser l’entrepôt de données. Ces opérations se font toutes à la demande. Gen2 utilise un cache sur disque local sur les nœuds de calcul pour améliorer les performances. Lorsque vous mettez à l’échelle ou suspendez le système, le cache est invalidé, et une période de préchauffage du cache est nécessaire pour pouvoir bénéficier de performances optimales.

Lorsque vous augmentez les DWU, vous augmentez de façon linéaire le nombre de ressources de calcul. Gen2 offre les meilleures performances de requête et la plus grande échelle. Les systèmes Gen2 exploitent aussi au maximum le cache.

#### <a name="capacity-limits"></a>Limites de capacité

Chaque serveur SQL (par exemple, myserver.database.windows.net) a un quota de [d’unités de transaction de base de données (DTU)](../../azure-sql/database/service-tiers-dtu.md) qui autorise un nombre spécifique d’unités d’entrepôt de données. Pour plus d’informations, consultez les [limites de capacité de gestion de la charge de travail](../sql-data-warehouse/sql-data-warehouse-service-capacity-limits.md?context=/azure/synapse-analytics/context/context#workload-management).

### <a name="assess-the-number-of-data-warehouse-units-you-need"></a>Évaluer le nombre d’unités DWU (Data Warehouse Unit) dont vous avez besoin

Le nombre idéal de DWU dépend en grande partie de votre charge de travail et de la quantité de données que vous avez chargées dans le système.

Étapes pour rechercher la DWU la mieux adaptée à votre charge de travail :

1. Commencez par sélectionner une DWU plus petite.
2. Surveillez les performances de votre application pendant le test des charges de données dans le système, en observant notamment le nombre de DWU sélectionné.
3. Identifiez des besoins supplémentaires pour les périodes ponctuelles de pics d’activité. Les charges de travail présentant d'importantes variations d'activité doivent être fréquemment mises à l’échelle.

Le pool SQL est un système de Scale-out qui peut fournir des quantités importantes de calcul et lancer des requêtes sur une grande quantité de données. Pour tester ses capacités de mise à l’échelle, surtout avec un nombre élevé de DWU, nous vous recommandons d’effectuer la mise à l’échelle de l’ensemble de données en vous assurant que vous disposez de suffisamment de données pour alimenter les UC. Pour le test de mise à l’échelle, nous recommandons d’utiliser au moins 1 To.

> [!NOTE]
>
> Si les travaux peuvent être fractionnés entre les nœuds de calcul, les performances des requêtes augmentent uniquement avec une parallélisation renforcée. Si vous trouvez que la mise à l’échelle n’altère pas les performances, vous devrez éventuellement modifier la conception de votre table et/ou vos requêtes. Pour des conseils de paramétrage des requêtes, consultez [Gérer les requêtes utilisateur](../overview-terminology.md?bc=%2fazure%2fsynapse-analytics%2fbreadcrumb%2ftoc.json&toc=%2fazure%2fsynapse-analytics%2ftoc.json).

### <a name="permissions"></a>Autorisations

La modification des DWU requiert les autorisations décrites dans [ALTER DATABASE](/sql/t-sql/statements/alter-database-transact-sql?view=azure-sqldw-latest&preserve-view=true).

Les rôles intégrés Azure, comme Contributeur de base de données SQL et Contributeur de SQL Server, peuvent changer les paramètres des DWU.

#### <a name="view-current-dwu-settings"></a>Afficher les paramètres d’unités DWU actuels

Pour afficher le paramètre DWU actuel :

1. Ouvrez l’Explorateur d’objets SQL Server dans Visual Studio.
2. Connectez-vous à la base de données associée au serveur SQL logique.
3. Sélectionnez dans la vue de gestion dynamique sys.database_service_objectives. Voici un exemple :

```sql
SELECT  db.name [Database]
,        ds.edition [Edition]
,        ds.service_objective [Service Objective]
FROM    sys.database_service_objectives   AS ds
JOIN    sys.databases                     AS db ON ds.database_id = db.database_id
;
```

### <a name="change-data-warehouse-units"></a>Modifier les DWU

#### <a name="azure-portal"></a>Portail Azure

Pour modifier les DWU :

1. Ouvrez le [portail Azure](https://portal.azure.com), ouvrez votre base de données, puis sélectionnez **Mettre à l’échelle**.

2. Sous **Mettre à l’échelle**, déplacez le curseur vers la gauche ou vers la droite pour modifier le paramètre DWU.

3. Sélectionnez **Enregistrer**. Un message de confirmation s’affiche. Sélectionnez **Oui** pour confirmer, ou **Non** pour annuler.

#### <a name="powershell"></a>PowerShell

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Pour modifier les DWU, utilisez l’applet de commande PowerShell [Set-AzSqlDatabase](/powershell/module/az.sql/set-azsqldatabase). L'exemple suivant définit l'objectif de niveau de service sur DW1000 pour la base de données MySQLDW hébergée sur le serveur MyServer.

```powershell
Set-AzSqlDatabase -DatabaseName "MySQLDW" -ServerName "MyServer" -RequestedServiceObjectiveName "DW1000c"
```

Pour plus d’informations, consultez [Applets de commande PowerShell pour Azure Synapse Analytics](/powershell/module/az.synapse)

#### <a name="t-sql"></a>T-SQL

Avec T-SQL, vous pouvez afficher les paramètres actuels de DWU, les modifier et consulter la progression.

Pour modifier les unités DWU :

1. Connectez-vous à la base de données MASTER associée à votre serveur.
2. Utilisez l’instruction TSQL [ALTER DATABASE](/sql/t-sql/statements/alter-database-transact-sql?view=azure-sqldw-latest&preserve-view=true). L’exemple suivant définit l'objectif de niveau de service sur DW1000c pour la base de données MySQLDW.

```sql
ALTER DATABASE MySQLDW
MODIFY (SERVICE_OBJECTIVE = 'DW1000c')
;
```

#### <a name="rest-apis"></a>API REST

Pour modifier les DWU, utilisez l’API REST [Créer ou mettre à jour une base de données](/sql/t-sql/statements/alter-database-transact-sql?view=azure-sqldw-latest&preserve-view=true). L’exemple suivant définit l'objectif de niveau de service sur DW1000c pour la base de données MySQLDW hébergée sur le serveur MyServer. Le serveur est un groupe de ressources Azure appelé ResourceGroup1.

```
PUT https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Sql/servers/{server-name}/databases/{database-name}?api-version=2014-04-01-preview HTTP/1.1
Content-Type: application/json; charset=UTF-8

{
    "properties": {
        "requestedServiceObjectiveName": DW1000
    }
}
```

Pour plus d’exemples d’API REST, consultez [API REST pour Azure Synapse Analytics](/rest/api/synapse/?view=azps-6.1.0&preserve-view=true).

### <a name="check-status-of-dwu-changes"></a>Vérifier l’état des modifications de DWU

Les modifications de DWU peuvent prendre plusieurs minutes. Si vous effectuez une mise à l’échelle automatique, envisagez d’implémenter une logique pour vous assurer que certaines opérations sont bien terminées avant de passer à une autre action.

Nous vous conseillons de vérifier l’état de la base de données en différents points de terminaison afin d’implémenter correctement l’automatisation. Si le portail vous informe de la fin d’une opération et de l’état actuel des bases de données, il ne vous permet pas de programmer la vérification de l’état.

Vous ne pouvez pas vérifier l’état de la base de données pour les opérations de scale-out avec le portail Azure.

Pour vérifier l’état des modifications de DWU :

1. Connectez-vous à la base de données MASTER associée à votre serveur.
2. Envoyez la requête suivante pour vérifier l’état de la base de données.

```sql
SELECT    *
FROM      sys.databases
;
```

1. Envoyez la requête suivante pour vérifier l’état de l’opération.

```sql
SELECT    *
FROM      sys.dm_operation_status
WHERE     resource_type_desc = 'Database'
AND       major_resource_id = 'MySQLDW'
;
```

Cette vue de gestion dynamique retourne des informations sur différentes opérations de gestion de votre pool SQL dédié, comme l’opération et l’état de l’opération, qui est IN_PROGRESS ou COMPLETED.

### <a name="the-scaling-workflow"></a>Flux de travail de mise à l’échelle

Lorsque vous démarrez une opération de mise à l’échelle, le système arrête tout d’abord toutes les sessions ouvertes, ce qui annule toutes les transactions en cours pour garantir un état cohérent. La mise à l’échelle intervient uniquement une fois la restauration des transactions effectuée.

- Pour une opération de montée en puissance, le système détache tous les nœuds de calcul, approvisionne le calcul supplémentaire, puis le réassocie à la couche de stockage.
- Pour une opération de descente en puissance, le système détache tous les nœuds de calcul, puis rattache uniquement les nœuds nécessaire à la couche de stockage.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur la gestion des performances, consultez [Classes de ressources pour la gestion de la charge de travail](../sql-data-warehouse/resource-classes-for-workload-management.md?context=/azure/synapse-analytics/context/context) et [Limites de mémoire et de concurrence](../sql-data-warehouse/memory-concurrency-limits.md?context=/azure/synapse-analytics/context/context).
