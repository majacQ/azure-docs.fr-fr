---
title: Azure App Configuration – Bonnes pratiques | Microsoft Docs
description: Découvrez les meilleures pratiques lors de l’utilisation d’Azure App Configuration. Parmi les sujets abordés figurent les regroupements de clés, les compositions clé-valeur, l’amorçage d’App Configuration et plus encore.
services: azure-app-configuration
documentationcenter: ''
author: AlexandraKemperMS
editor: ''
ms.assetid: ''
ms.service: azure-app-configuration
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: alkemper
ms.custom: devx-track-csharp, mvc
ms.openlocfilehash: e75fb11379ccbdff90d1acd1a3bce36b62bd8a1d
ms.sourcegitcommit: 692382974e1ac868a2672b67af2d33e593c91d60
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2021
ms.locfileid: "130250860"
---
# <a name="azure-app-configuration-best-practices"></a>Azure App Configuration – Bonnes pratiques

Cet article décrit les modèles courants et les bonnes pratiques à observer quand vous utilisez Azure App Configuration.

## <a name="key-groupings"></a>Regroupements de clés

App Configuration propose deux options pour organiser les clés :

* Préfixes de clés
* Étiquettes

Vous pouvez regrouper vos clés avec une de ces options ou les deux.

Les *préfixes de clés* constituent la partie initiale des clés. Vous pouvez regrouper logiquement un ensemble de clés en ajoutant le même préfixe à leur nom. Les préfixes peuvent être constitués de plusieurs composants reliés par un délimiteur, par exemple `/`, à la manière d’un chemin d’URL, pour former un espace de noms. De telles hiérarchies sont utiles quand vous stockez des clés pour un grand nombre d’applications et microservices dans un même magasin App Configuration.

Une chose importante à garder à l’esprit est que les clés sont ce à quoi votre code d’application fait référence pour récupérer les valeurs des paramètres correspondants. Les clés ne doivent pas changer, sans quoi vous devez modifier votre code chaque fois que cela se produit.

Les *étiquettes* représentent un attribut de clé. Elles servent à créer des variantes d’une clé. Par exemple, vous pouvez attribuer des étiquettes à plusieurs versions d’une clé. Une version peut être une itération, un environnement ou d’autres informations contextuelles. Votre application peut demander un ensemble de valeurs de clés totalement différent en spécifiant une autre étiquette. De ce fait, toutes les références de clés restent inchangées dans votre code.

## <a name="key-value-compositions"></a>Compositions clé-valeur

App Configuration considère toutes les clés qui y sont stockées comme des entités indépendantes. App Configuration ne tente pas de déduire une quelconque relation entre les clés ou d’hériter des valeurs de clés en fonction de leur hiérarchie. Vous avez la possibilité d’agréger plusieurs ensembles de clés, mais il convient dans ce cas d’utiliser des étiquettes avec un empilement de configuration adéquat dans votre code d’application.

Intéressons-nous à un exemple. Supposez que vous disposez d’un paramètre nommé **Asset1**, dont la valeur peut varier en fonction de l’environnement de développement. Vous créez une clé nommée « Asset1 » avec une étiquette vide et une étiquette nommée « Development ». Dans la première étiquette, vous placez la valeur par défaut de **Asset1**, et vous placez une valeur spécifique pour « Development » dans l’autre.

Dans votre code, vous commencez par récupérer les valeurs de clés sans étiquettes, puis vous récupérez une deuxième fois le même ensemble de valeurs de clés, mais cette fois avec l’étiquette « Development ». Quand vous récupérez les valeurs à la deuxième fois, les valeurs précédentes des clés sont remplacées. Le système de configuration .NET Core vous permet d’« empiler » plusieurs ensembles de données de configuration les uns sur les autres. Si une clé se trouve dans plusieurs ensembles, le dernier ensemble la contenant est utilisé. Avec un framework de programmation moderne comme .NET Core, vous bénéficiez gratuitement de cette fonctionnalité d’empilement si vous utilisez un fournisseur de configuration natif pour accéder à App Configuration. L’extrait de code suivant montre comment vous pouvez implémenter l’empilement dans une application .NET Core :

```csharp
// Augment the ConfigurationBuilder with Azure App Configuration
// Pull the connection string from an environment variable
configBuilder.AddAzureAppConfiguration(options => {
    options.Connect(configuration["connection_string"])
           .Select(KeyFilter.Any, LabelFilter.Null)
           .Select(KeyFilter.Any, "Development");
});
```

La rubrique [Utilisation d’étiquettes pour permettre différentes configurations selon les environnements](./howto-labels-aspnet-core.md) fournit un exemple complet.

## <a name="references-to-external-data"></a>Références à des données externes

App Configuration est conçu pour stocker toutes les données de configuration que vous enregistreriez normalement dans des fichiers de configuration ou des variables d’environnement. Toutefois, certains types de données peuvent être mieux adaptés à d’autres sources. Par exemple, stockez les secrets dans Key Vault, les fichiers dans Stockage Azure, les informations d’appartenance dans les groupes Azure AD ou les listes de clients dans une base de données.

Vous pouvez toujours tirer parti d’App Configuration en enregistrant une référence à des données externes dans une valeur clé. Vous pouvez [utiliser le type de contenu](./concept-key-value.md#use-content-type) pour différencier chaque source de données. Lorsque votre application lit une référence, vous chargez les données à partir de la source référencée. Si vous modifiez l’emplacement de vos données externes, il vous suffit de mettre à jour la référence dans App Configuration au lieu de mettre à jour et de redéployer l’ensemble de votre application.

La fonctionnalité [ Référence Key Vault](use-key-vault-references-dotnet-core.md) d’App Configuration est un exemple de ce cas. Elle permet aux secrets requis pour une application d’être mis à jour en fonction des besoins, tandis que les secrets sous-jacents sont conservés dans Key Vault.

## <a name="app-configuration-bootstrap"></a>Amorçage d’App Configuration

Pour accéder à un magasin App Configuration, vous pouvez utiliser sa chaîne de connexion, qui est disponible sur le portail Azure. Étant donné que les chaînes de connexion contiennent des informations d’identification, elles sont considérées comme des secrets. Ces secrets doivent être stockés dans Azure Key Vault, et votre code doit s’authentifier auprès de Key Vault pour les récupérer.

Une meilleure option consiste à utiliser la fonctionnalité d’identités managées dans Azure Active Directory. Avec les identités managées, vous avez seulement besoin de l’URL de point de terminaison App Configuration pour amorcer l’accès à votre magasin App Configuration. Vous pouvez incorporer l’URL dans votre code d’application (par exemple, dans le fichier *appsettings.json*). Pour plus d’informations, consultez [S’intégrer avec des identités managées Azure](howto-integrate-azure-managed-service-identity.md).

## <a name="app-or-function-access-to-app-configuration"></a>Accès des applications ou des fonctions à App Configuration

Vous pouvez fournir un accès à App Configuration pour Web Apps ou Azure Functions en utilisant une des méthodes suivantes :

* À partir du portail Azure, entrez la chaîne de connexion à votre magasin App Configuration dans les paramètres Application d’App Service.
* Stockez la chaîne de connexion à votre magasin App Configuration dans Key Vault et [faites-y référence dans App Service](../app-service/app-service-key-vault-references.md).
* Utilisez les identités managées Azure pour accéder au magasin App Configuration. Pour plus d’informations, consultez [S’intégrer avec des identités managées Azure](howto-integrate-azure-managed-service-identity.md).
* Envoyer (push) la configuration d’App Configuration vers App Service. App Configuration propose une fonction d’exportation (sur le portail Azure et dans Azure CLI) qui envoie directement les données dans App Service. Avec cette méthode, vous n’avez pas du tout besoin de modifier le code d’application.

## <a name="reduce-requests-made-to-app-configuration"></a>Réduire le nombre de requêtes adressées à App Configuration

Un nombre excessif de requêtes envoyées à App Configuration peut entraîner des frais de limitation de requêtes ou de dépassement. Pour réduire le nombre de requêtes effectuées :

* Augmentez le délai d’actualisation, surtout si vos valeurs de configuration ne changent pas fréquemment. Spécifiez un nouveau délai d’actualisation à l’aide de la [méthode `SetCacheExpiration`](/dotnet/api/microsoft.extensions.configuration.azureappconfiguration.azureappconfigurationrefreshoptions.setcacheexpiration).

* Surveillez une seule *clé Sentinel*, plutôt que de surveiller des clés individuelles. Actualisez toute la configuration uniquement si la clé Sentinel est modifiée. Pour obtenir un exemple, consultez [Utiliser la configuration dynamique dans une application ASP.NET Core](enable-dynamic-configuration-aspnet-core.md).

* Utilisez Azure Event Grid pour recevoir des notifications en cas de modification de la configuration, au lieu de l’interroger constamment sur d’éventuelles modifications. Pour plus d’informations, consultez [Utiliser Event Grid pour les notifications de changement de données d’App Configuration](./howto-app-configuration-event.md).

* Répartissez vos demandes dans plusieurs magasins App Configuration. Par exemple, utilisez un magasin différent de chaque région géographique pour une application déployée au niveau mondial. Chaque magasin App Configuration a son propre quota de demandes. Cette configuration vous donne un modèle pour la scalabilité et évite le point de défaillance unique.

## <a name="importing-configuration-data-into-app-configuration"></a>Importation des données de configuration dans App Configuration

App Configuration vous offre la possibilité d’[importer](./howto-import-export-data.md) en bloc vos paramètres de configuration à partir de vos fichiers config actuels à l’aide du Portail Azure ou de l’interface CLI. Vous pouvez également utiliser les mêmes options pour exporter des valeurs clés depuis App Configuration, par exemple entre des magasins associés. Si vous souhaitez configurer une synchronisation continu avec votre référentiel dans GitHub ou Azure DevOps, vous pouvez utiliser notre [action GitHub](./concept-github-action.md) ou [Azure Pipeline Push Task](./push-kv-devops-pipeline.md) afin de pouvoir continuer à utiliser vos pratiques existantes de contrôle de code source tout en bénéficiant des avantages d’App Configuration.

## <a name="multi-region-deployment-in-app-configuration"></a>Déploiement multirégional dans App Configuration

App Configuration est service régional. Pour les applications ayant des configurations différentes par région, le stockage de ces configurations dans une seule instance peut créer un point de défaillance unique. Le déploiement multirégional d’App Configuration, à raison d’une instance par région, peut être une meilleure option. Il peut contribuer à la récupération d’urgence, aux performances et au cloisonnement de la sécurité au niveau régional. La configuration par région améliore également la latence et utilise des quotas de limitation distincts, puisque chaque instance a sa propre limitation. Pour appliquer l’atténuation de la récupération d’urgence, vous pouvez utiliser [plusieurs magasins de configuration](./concept-disaster-recovery.md). 

## <a name="client-applications-in-app-configuration"></a>Applications clientes dans App Configuration 

Lorsque vous utilisez App Configuration dans des applications clientes, veillez à prendre en compte deux facteurs principaux. Tout d’abord, si vous utilisez la chaîne de connexion dans une application cliente, vous risquez d’exposer au public la clé d’accès de votre magasin App Configuration. Deuxièmement, l’échelle classique d’une application cliente est susceptible de donner lieu à des demandes excessives à votre magasin App Configuration, ce qui peut entraîner des frais de dépassement ou une limitation. Pour plus d’informations sur la limitation, consultez la [FAQ](./faq.yml#are-there-any-limits-on-the-number-of-requests-made-to-app-configuration).

Pour résoudre ces problèmes, nous vous recommandons d’utiliser un service proxy entre vos applications clientes et votre magasin App Configuration. Le service proxy peut s’authentifier en toute sécurité auprès de votre magasin App Configuration sans problème de sécurité de type fuite d’informations d’authentification. Vous pouvez créer un service proxy à l’aide d’une des bibliothèques du fournisseur App Configuration. Ainsi, vous tirerez parti des fonctionnalités intégrées de mise en cache et d’actualisation pour optimiser le volume de demandes envoyées à App Configuration. Pour plus d’informations sur les fournisseurs App Configuration, consultez les articles des guides de démarrage rapide et des tutoriels. Le service proxy traite la configuration de vos applications clientes à partir de son cache. De votre côté, vous évitez les deux problèmes potentiels mentionnés dans cette section.

## <a name="next-steps"></a>Étapes suivantes

* [Clés et valeurs](./concept-key-value.md) 
