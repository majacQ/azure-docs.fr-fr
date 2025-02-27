---
title: Exemples Azure CLI pour Azure Cosmos DB | Microsoft Docs
description: Cet article répertorie plusieurs exemples de code Azure CLI disponibles pour l’interaction avec Azure Cosmos DB. Affichez des exemples CLI spécifiques aux API.
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: sample
ms.date: 09/17/2021
ms.author: mjbrown
ms.custom: devx-track-azurecli, seo-azure-cli
keywords: cosmos db, exemples azure cli, exemples de code azure cli, exemples de scripts azure cli
ms.openlocfilehash: 4529b51ff5109bfa6b8814b23e7e82844e7fb95c
ms.sourcegitcommit: f6e2ea5571e35b9ed3a79a22485eba4d20ae36cc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/24/2021
ms.locfileid: "128567715"
---
# <a name="azure-cli-samples-for-azure-cosmos-db-core-sql-api"></a>Exemples Azure CLI pour l’API (SQL) Core Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](../includes/appliesto-sql-api.md)]

Le tableau suivant comprend des liens vers des exemples de scripts Azure CLI pour Azure Cosmos DB. Utilisez les liens sur la droite pour accéder aux exemples spécifiques aux API. Les exemples communs sont les mêmes pour toutes les API. Des pages d’informations de référence pour toutes les commandes Azure Cosmos DB CLI sont disponibles dans les [Informations de référence sur Azure CLI](/cli/azure/cosmosdb). Vous trouverez également les exemples de scripts CLI Azure Cosmos DB dans le [dépôt GitHub CLI Azure Cosmos DB](https://github.com/Azure-Samples/azure-cli-samples/tree/master/cosmosdb).

Ces exemples nécessitent Azure CLI version 2.12.1 ou ultérieure. Exécutez `az --version` pour trouver la version. Si vous devez effectuer une installation ou une mise à niveau, consultez [Installer Azure CLI](/cli/azure/install-azure-cli).

Pour obtenir des exemples d’interface Azure CLI destinés à d’autres API, consultez [Exemples d’interface CLI pour Cassandra](../cassandra/cli-samples.md), [Exemples d’interface CLI pour l’API MongoDB](../mongodb/cli-samples.md), [Exemples d’interface CLI pour Gremlin](../graph/cli-samples.md), [Exemples d’interface CLI pour Table](../table/cli-samples.md).

## <a name="common-samples"></a>Exemples communs

Ces exemples s’appliquent à toutes les API Azure Cosmos DB.

|Tâche | Description |
|---|---|
| [Ajouter des régions de basculement](../scripts/cli/common/regions.md?toc=%2fcli%2fazure%2ftoc.json) | Ajoutez une région, changez la priorité de basculement, déclenchez un basculement manuel.|
| [Clés de compte et chaînes de connexion](../scripts/cli/common/keys.md?toc=%2fcli%2fazure%2ftoc.json) | Listez les clés de compte, les clés en lecture seule, regénérez les clés et listez les chaînes de connexion.|
| [Sécuriser avec le pare-feu IP](../scripts/cli/common/ipfirewall.md?toc=%2fcli%2fazure%2ftoc.json)| Créez un compte Cosmos avec le pare-feu IP configuré.|
| [Sécuriser un nouveau compte avec des points de terminaison de service](../scripts/cli/common/service-endpoints.md?toc=%2fcli%2fazure%2ftoc.json)| Créez un compte Cosmos et sécurisez-le avec des points de terminaison de service.|
| [Sécuriser un compte existant avec des points de terminaison de service](../scripts/cli/common/service-endpoints-ignore-missing-vnet.md?toc=%2fcli%2fazure%2ftoc.json)| Mettez à jour un compte Cosmos pour le sécuriser avec des points de terminaison de service quand le sous-réseau est finalement configuré.|
|||

## <a name="core-sql-api-samples"></a>Exemples d’API Core (SQL)

|Tâche | Description |
|---|---|
| [Créer un compte, une base de données et un conteneur Azure Cosmos](../scripts/cli/sql/create.md?toc=%2fcli%2fazure%2ftoc.json)| Crée un compte, une base de données et un conteneur Azure Cosmos DB pour l’API Core (SQL). |
| [Créer un compte, une base de données et un conteneur Azure Cosmos avec mise à l’échelle automatique](../scripts/cli/sql/autoscale.md?toc=%2fcli%2fazure%2ftoc.json)| Crée un compte, une base de données et un conteneur Azure Cosmos DB avec mise à l’échelle automatique pour l’API Core (SQL). |
| [Opérations de débit](../scripts/cli/sql/throughput.md?toc=%2fcli%2fazure%2ftoc.json) | Opérations sur une base de données et un conteneur, comme la lecture du débit, la mise à jour du débit et la migration entre le débit standard et le débit avec mise à l’échelle automatique.|
| [Verrouiller des ressources contre la suppression](../scripts/cli/sql/lock.md?toc=%2fcli%2fazure%2ftoc.json)| Empêchez la suppression de ressources à l’aide de verrous de ressources.|
|||
