---
title: Azure Relay - Migrer vers l’autorisation de signature d’accès partagé
description: Explique comment faire passer les applications Azure Relay d’une utilisation d’Azure Active Directory Access Control Service à une utilisation de l’autorisation de signature d’accès partagé.
ms.topic: article
ms.date: 06/23/2021
ms.openlocfilehash: 30e656b3e4960cb7e2a8bdb23a2d8ce1aa91edbf
ms.sourcegitcommit: d9a2b122a6fb7c406e19e2af30a47643122c04da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/24/2021
ms.locfileid: "114668713"
---
# <a name="azure-relay---migrate-from-azure-active-directory-access-control-service-to-shared-access-signature-authorization"></a>Azure Relay - Migrer depuis le service Access Control Service d’Azure Active Directory vers l’autorisation de signature d’accès partagé

Les applications Azure Relay avaient historiquement la possibilité d’utiliser deux modèles d’autorisation différents : le modèle de jeton de la [Signature d’accès partagé (SAP)](../service-bus-messaging/service-bus-sas.md) fourni directement par le service Relay et un modèle fédéré où la gestion des règles d’autorisation se fait en interne par le service Access Control Service (ACS) d’[Azure Active Directory](../active-directory/index.yml) et où les jetons obtenus à partir d’Access Control Service sont passés à Relay pour autoriser l’accès aux fonctionnalités souhaitées.

Le modèle d’autorisation d’Access Control Service a longtemps été remplacé par l’[autorisation de signature d’accès partagé](../service-bus-messaging/service-bus-authentication-and-authorization.md) en tant que modèle par défaut et toute la documentation, les conseils et les exemples utilisent exclusivement la signature d’accès partagé. En outre, il n’est plus possible de créer de nouveaux espaces de noms Relay associés à Access Control Service.

La signature d’accès partagé présente l’avantage de ne pas être immédiatement dépendante d’un autre service, mais de pouvoir être utilisée directement à partir d’un client sans intermédiaire en accordant au client l’accès au nom et à la clé de la règle de la signature d’accès partagé. De plus, la signature d’accès partagé peut être facilement intégrée à une approche dans laquelle un client doit tout d’abord passer une vérification d’autorisation avec un autre service avant l’émission d’un jeton. Cette dernière approche est similaire au modèle d’utilisation d’Access Control Service, mais permet l’émission des jetons d’accès en fonction des conditions spécifiques à l’application, difficiles à exprimer dans Access Control Service.

Pour toutes les applications existantes qui dépendent d’Access Control Service, nous encourageons plutôt les clients à migrer leurs applications pour s’appuyer sur la signature d’accès partagé.

## <a name="migration-scenarios"></a>Scénarios de migration

Access Control Service et Relay sont intégrés à travers les connaissances partagées d’une *clé de signature*. La clé de signature est utilisée par un espace de noms Access Control Service pour signer des jetons d’autorisation et est utilisée par Azure Relay pour vérifier que le jeton a été émis par l’espace de noms Access Control Service associé. L’espace de noms d’Access Control Service conserve les identités de service et les règles d’autorisation. Les règles d’autorisation définissent quel type d’accès à une partie du graphique de l’espace de noms Relay sera obtenu par quelle identité de service ou quel jeton émis par un fournisseur d’identité externe, sous la forme de la correspondance de préfixe la plus longue.

Par exemple, une règle d’Access Control Service peut accorder la revendication **Envoyer** sur le préfixe de chemin `/` à une identité de service, ce qui signifie qu’un jeton émis par Access Control Service basé sur cette règle autorise le client à envoyer à toutes les entités dans l’espace de noms. Si le préfixe de chemin est `/abc`, l’identité est restreinte à l’envoi à des entités nommées `abc` ou organisées sous ce préfixe. On suppose que les lecteurs de ce guide de migration connaissent déjà ces concepts.

Les scénarios de migration se répartissent en trois grandes catégories :

1.  **Valeurs par défaut non modifiées**. Certains clients utilisent un objet [SharedSecretTokenProvider](/dotnet/api/microsoft.servicebus.sharedsecrettokenprovider) , en passant l’identité du service du **propriétaire** automatiquement généré et sa clé secrète pour l’espace de noms Access Control Service associé à l’espace de noms Relay et n’ajoute pas de nouvelles règles.

2.  **Identités de service personnalisé avec des règles simples**. Certains clients ajoutent de nouvelles identités de service et accordent à chaque nouvelle identité de service les autorisations **Envoyer**, **Écouter** et **Gérer** pour une entité spécifique.

3.  **Identités de service personnalisé avec des règles complexes**. Très peu de clients possèdent des ensembles de règles complexes dans lesquels les jetons émis en externe sont mappés sur des droits sur Relay, ou dans lesquels une identité de service unique se voit attribuer des droits différenciés sur plusieurs chemins d’accès d’espace de noms via plusieurs règles.

Pour obtenir de l’aide sur la migration d’ensembles de règles complexes, vous pouvez contacter le [support technique Azure](https://azure.microsoft.com/support/options/). Les deux autres scénarios activent la migration directe.

### <a name="unchanged-defaults"></a>Valeurs par défaut non modifiées

Si votre application n’a pas modifié les valeurs par défaut d’Access Control Service, vous pouvez remplacer toutes les utilisations de [SharedSecretTokenProvider](/dotnet/api/microsoft.servicebus.sharedsecrettokenprovider) avec un objet [SharedAccessSignatureTokenProvider](/dotnet/api/microsoft.servicebus.sharedaccesssignaturetokenprovider) et utiliser l’espace de noms **RootManageSharedAccessKey** déjà configuré à la place du compte **propriétaire** d’Access Control Service. Note : même avec le compte **propriétaire** d’Access Control Service, cette configuration était et est toujours déconseillée, étant donné que ce compte ou cette règle fournit une autorité de gestion complète sur l’espace de noms, y compris l’autorisation de supprimer n’importe quelle entité.

### <a name="simple-rules"></a>Règles simples

Si l’application utilise des identités de service personnalisé avec des règles simples, la migration est directe dans le cas où une identité de service Access Control Service a été créée pour fournir un contrôle d’accès sur un relais spécifique. Ce scénario est souvent le cas dans les solutions de type SaaS où chaque relais est utilisé comme un pont vers un site abonné. L’identité du service est créée pour ce site en particulier. Dans ce cas, l’identité du service respectif peut être migrée vers une règle de la signature d’accès partagé, directement sur le relais. Le nom d’identité du service peut devenir le nom de la règle de signature d’accès partagé et la clé d’identité de service peut devenir la clé de la règle de signature d’accès partagé. Les droits de la règle de signature d’accès partagé sont ensuite configurés d’une manière équivalente à la règle d’Access Control Service respectivement applicable pour l’entité.

Vous pouvez effectuer cette nouvelle configuration supplémentaire de la signature d’accès partagé sur place sur n’importe quel espace de noms existant fédéré avec Access Control Service. La migration en dehors d’Access Control Service est effectuée par la suite en utilisant l’objet [SharedAccessSignatureTokenProvider](/dotnet/api/microsoft.servicebus.sharedaccesssignaturetokenprovider) et non l’objet [SharedSecretTokenProvider](/dotnet/api/microsoft.servicebus.sharedsecrettokenprovider). L’espace de noms n’a pas besoin d’être dissocié d’Access Control Service.

### <a name="complex-rules"></a>Règles complexes

Les règles de signature d’accès partagé ne sont pas destinées à être des comptes; elles sont des clés de signatures nommées associées à des droits. Par conséquent, les scénarios dans lesquels l’application crée plusieurs identités de service et leur accorde des droits d’accès pour plusieurs entités ou pour l’espace de noms entier requièrent toujours un intermédiaire pour émettre des jetons. Vous pouvez obtenir des conseils pour ce type d’intermédiaire en [contactant le support technique](https://azure.microsoft.com/support/options/).

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur l’authentification Azure Relay, consultez les sujets suivants :

* [Authentification et autorisation Azure Relay](relay-authentication-and-authorization.md)
* [Authentification de Service Bus avec les signatures d’accès partagé](../service-bus-messaging/service-bus-sas.md)
