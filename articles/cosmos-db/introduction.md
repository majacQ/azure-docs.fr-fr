---
title: Présentation d’Azure Cosmos DB
description: En savoir plus sur Azure Cosmos DB. Cette base de données multi-modèle distribuée à l’échelle mondiale est conçue pour offrir une faible latence, une scalabilité élastique, une haute disponibilité et une prise en charge native des données NoSQL.
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.topic: overview
ms.date: 08/26/2021
ms.openlocfilehash: ffd59ebcac1779c43222ea9a006888edd634bafa
ms.sourcegitcommit: 03f0db2e8d91219cf88852c1e500ae86552d8249
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/27/2021
ms.locfileid: "123035884"
---
# <a name="welcome-to-azure-cosmos-db"></a>Bienvenue dans Azure Cosmos DB
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Les applications actuelles doivent être hautement réactives et toujours en ligne. Pour obtenir une faible latence et une haute disponibilité, les instances de ces applications doivent être déployées dans des centres de données qui sont proches des utilisateurs. Les applications doivent répondre en temps réel à des changements importants d’utilisation aux heures de pointe, pour stocker des volumes croissants de données et pour rendre ces données disponibles aux utilisateurs en quelques millisecondes.

Azure Cosmos DB est une base de données NoSQL complètement managée pour le développement d’applications modernes. Les temps de réponse inférieurs à dix millisecondes et la scalabilité automatique et instantanée garantissent une vitesse, quelle que soit l’échelle. La continuité de l’activité est garantie par la disponibilité [assurée par un contrat SLA](https://azure.microsoft.com/support/legal/sla/cosmos-db) et une sécurité de qualité professionnelle. Le développement d’applications est plus rapide et plus productif grâce à la distribution de données multirégion clé en main dans le monde entier, aux API open source et aux kits SDK pour les langages les plus courants. En tant que service complètement managé, Azure Cosmos DB prend en charge l’administration de bases de données grâce à une gestion, des mises à jour et des mises à jour correctives automatiques. Il traite également la gestion de la capacité avec des options économiques de mise à l’échelle automatique et serverless qui répondent aux besoins de l’application pour faire correspondre la capacité à la demande.

Vous pouvez [essayer gratuitement Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/) sans abonnement Azure, sans frais ni engagement, ou utiliser le [niveau gratuit d’Azure Cosmos DB](free-tier.md) pour obtenir un compte avec les 1000 premières RU/s (unités de requête par seconde) et les 25 premiers Go de stockage gratuits.

> [!div class="nextstepaction"]
> [Essayer gratuitement Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/)

> [!TIP]
> Pour en savoir plus sur Azure Cosmos DB, rejoignez-nous tous les jeudis à 13H00 Heure du Pacifique sur Azure Cosmos DB Live TV. Consultez le [programme des sessions à venir et les épisodes précédents](https://gotcosmos.com/tv).

:::image type="content" source="./media/introduction/azure-cosmos-db.png" alt-text="Azure Cosmos DB est une base de données NoSQL complètement managée pour le développement d’applications modernes." border="false":::

## <a name="key-benefits"></a>Principaux avantages

### <a name="guaranteed-speed-at-any-scale"></a>Vitesse garantie quelle que soit l’échelle

Bénéficiez d’une vitesse et d’un débit [assurés par un contrat SLA](https://azure.microsoft.com/support/legal/sla/cosmos-db), d’un accès global rapide et d’une élasticité instantanée inégalés.

- L’accès en temps réel avec des latences de lecture et d’écriture rapides dans le monde entier, ainsi que le débit et la cohérence sont tous assurés par des [contrats SLA](https://azure.microsoft.com/support/legal/sla/cosmos-db)
- Les écritures multirégions et la distribution de données dans n’importe quelle région Azure d’un simple clic sur un bouton.
- Mettez à l’échelle le débit et le stockage de façon indépendante et flexible dans les régions Azure, même pendant les pics de trafic imprévisibles, pour une mise à l’échelle illimitée dans le monde entier.

### <a name="simplified-application-development"></a>Développement d’applications simplifié

Créez rapidement des API open source, plusieurs SDK, des données sans schéma et une analytique sans ETL sur les données opérationnelles.

- Étroite intégration aux principaux services Azure utilisés dans le développement d’applications modernes (cloud natives), notamment Azure Functions, IoT Hub, AKS (Azure Kubernetes Service), App Service et plus encore.
- Choisissez entre plusieurs API de base de données, notamment l’API Core (SQL) native, l’API pour MongoDB, l’API Cassandra, l’API Gremlin et l’API Table.
- Créez des applications sur l’API Core (SQL) en utilisant les langages de votre choix avec les SDK pour .NET, Java, Node.js et Python. Ou utilisez les pilotes de votre choix pour toutes les autres API de base de données.
- Exécutez une analytique sans ETL sur les données opérationnelles en quasi-temps réel stockées dans Azure Cosmos DB avec Azure Synapse Analytics.
- Le flux de modification permet de suivre et de gérer facilement les changements apportés aux conteneurs de base de données et de créer des événements déclenchés avec Azure Functions.
- Le service sans schéma d’Azure Cosmos DB indexe automatiquement toutes vos données, quel que soit le modèle de données, pour fournir des requêtes extrêmement rapides.

### <a name="mission-critical-ready"></a>Intégration stratégique immédiate

Garantissez la continuité de l’activité, la disponibilité à 99,999 % et la sécurité de niveau entreprise pour chaque application.

- Azure Cosmos DB offre une suite complète de [contrats SLA](https://azure.microsoft.com/support/legal/sla/cosmos-db) avec notamment une haute disponibilité inégalée dans le monde entier.
- Distribuez facilement les données dans n’importe quelle région Azure avec la réplication automatique des données. Profitez de l’absence de temps d’arrêt avec les écritures multirégions ou un objectif de point de récupération (RPO) égal à 0 quand vous utilisez une cohérence forte.
- Profitez d’un chiffrement au repos de qualité professionnelle avec des clés autogérées.
- Le contrôle d’accès en fonction du rôle Azure assure la sécurité de vos données et offre un contrôle précis.

### <a name="fully-managed-and-cost-effective"></a>Complètement managé et économique

Gestion de bases de données de bout en bout, avec une mise à l’échelle serverless et automatique répondant aux besoins de votre application et au coût total de possession (TCO)

- Service de base de données complètement managé. Maintenance, mises à jour correctives et mises à jour automatiques, ce qui fait gagner du temps et de l’argent aux développeurs.
- Options économiques pour les charges de travail imprévisibles ou sporadiques de toute taille ou échelle, ce qui permet aux développeurs de démarrer facilement sans avoir à planifier ou à gérer la capacité.
- Le modèle serverless offre aux charges de travail imprévisibles un service automatique et réactif pour gérer les pics de trafic à la demande.
- Le débit provisionné en mode de mise à l’échelle met automatiquement et instantanément à l’échelle la capacité des charges de travail imprévisibles, tout en respectant les [contrats SLA](https://azure.microsoft.com/support/legal/sla/cosmos-db).

## <a name="solutions-that-benefit-from-azure-cosmos-db"></a>Les solutions qui profitent des avantages de Azure Cosmos DB

Toutes les [applications web, mobiles, IoT ou de jeux](use-cases.md) qui ont besoin de traiter des volumes importants de données, de lectures et d’écritures à une [échelle mondiale](distribute-data-globally.md) avec des temps de réponse proches du temps réel pour une grande variété de données peuvent profiter des garanties de Cosmos DB en termes de [haute disponibilité](https://azure.microsoft.com/support/legal/sla/cosmos-db/), de haut débit, de faible latence et de cohérence ajustable. En savoir plus sur la façon dont Azure Cosmos DB peut s’appliquer à l’[IoT et la télématique](use-cases.md#iot-and-telematics), à la [vente au détail et au marketing](use-cases.md#retail-and-marketing), aux [jeux](use-cases.md#gaming) et aux [applications web et mobiles](use-cases.md#web-and-mobile-applications)

## <a name="next-steps"></a>Étapes suivantes

Bien démarrer avec Azure Cosmos DB grâce à l’un de nos guides de démarrage rapide :

- Apprendre [à choisir une API](choose-api.md) dans Azure Cosmos DB
- [Prise en main de l’API SQL Azure Cosmos DB](create-sql-api-dotnet.md)
- [Bien démarrer avec l’API Azure Cosmos DB pour MongoDB](mongodb/create-mongodb-nodejs.md)
- [Prise en main de l’API Cassandra Azure Cosmos DB](cassandra/manage-data-dotnet.md)
- [Prise en main de l’API Gremlin Azure Cosmos DB](create-graph-dotnet.md)
- [Prise en main de l’API Table Azure Cosmos DB](table/create-table-dotnet.md)
- [Livre blanc sur le développement d’applications de nouvelle génération avec Azure Cosmos DB](https://azure.microsoft.com/resources/microsoft-azure-cosmos-db-flexible-reliable-cloud-nosql-at-any-scale/)
- Vous tentez d’effectuer une planification de la capacité pour une migration vers Azure Cosmos DB ?
    - Si vous ne connaissez que le nombre de vCores et de serveurs présents dans votre cluster de bases de données existantes, lisez l’article sur l’[estimation des unités de requête à l’aide de vCores ou de processeurs virtuels](convert-vcore-to-request-unit.md) 
    - Si vous connaissez les taux de requêtes typiques de votre charge de travail de base de données actuelle, lisez la section concernant l’[estimation des unités de requête à l’aide du planificateur de capacité Azure Cosmos DB](estimate-ru-with-capacity-planner.md)

> [!div class="nextstepaction"]
> [Essayez gratuitement Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/)
