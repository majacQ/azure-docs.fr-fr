---
title: Fichier include
description: Fichier include
services: service-bus-messaging
author: spelluru
ms.service: service-bus-messaging
ms.topic: include
ms.date: 10/08/2021
ms.author: spelluru
ms.custom: include file
ms.openlocfilehash: 978e703285317c7a644f6d0c459a1f1d572b9179
ms.sourcegitcommit: af303268d0396c0887a21ec34c9f49106bb0c9c2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/11/2021
ms.locfileid: "129801176"
---
## <a name="create-a-namespace-in-the-azure-portal"></a>Créer un espace de noms dans le Portail Azure
Pour commencer à utiliser des entités de messagerie Service Bus dans Azure, vous devez d’abord créer un espace de noms avec un nom unique dans Azure. Ce dernier fournit un conteneur d’étendue pour l’adressage des ressources Service Bus au sein de votre application.

Pour créer un espace de noms :

1. Connectez-vous au [portail Azure](https://portal.azure.com)
2. Dans le volet de navigation de gauche du portail, sélectionnez **+ Créer une ressource**, **Intégration**, puis **Service Bus**.

    :::image type="content" source="./media/service-bus-create-namespace-portal/create-resource-service-bus-menu.png" alt-text="Image illustrant la sélection de Créer une ressource, puis celles de Intégration et de Service Bus, dans le menu.":::
3. Dans l’étiquette **De base** de la page **Créer un espace de noms**, suivez ces étapes : 
    1. Pour l’option **Abonnement**, choisissez un abonnement Azure dans lequel créer l’espace de noms.
    1. Pour l’option **Groupe de ressources**, choisissez un groupe de ressources existant dans lequel l’espace de noms sera utilisé, ou créez-en un nouveau.      
    1. Entrez un **nom pour l’espace de noms**. Le système vérifie immédiatement si le nom est disponible. Pour obtenir la liste des règles de nommage des espaces de noms, consultez [API REST de création des espaces de noms](/rest/api/servicebus/create-namespace).
    1. Pour l’option **Emplacement**, choisissez la région dans laquelle héberger votre espace de noms.
    1. Pour le **Niveau tarifaire**, sélectionnez le SKU (De base, Standard ou Premium) destiné à l’espace de noms. Pour ce guide de démarrage rapide, sélectionnez **Standard**. 
    
        Si vous voulez utiliser des [rubriques et des abonnements](../service-bus-queues-topics-subscriptions.md#topics-and-subscriptions), choisissez Standard ou Premium. Les rubriques/abonnements ne sont pas pris en charge dans le niveau tarifaire De base. 

        Si vous avez sélectionné le SKU **Premium**, précisez le nombre d’**unité de messagerie**. Le niveau Premium isole les ressources au niveau du processeur et de la mémoire, ce qui permet d’exécuter chaque charge de travail de manière isolée. Ce conteneur de ressources est appelé unité de messagerie. Un espace de noms Premium a au moins une unité de messagerie. Vous pouvez sélectionner une, deux ou quatre unités de messagerie pour chaque espace de noms Service Bus Premium. Pour plus d’informations, consultez [Messagerie Service Bus Premium](../service-bus-premium-messaging.md).
    1. Sélectionnez **Revoir + créer**. Le système crée l’espace de noms de service et l’active. Vous devrez peut-être attendre plusieurs minutes afin que le système approvisionne des ressources pour votre compte.
   
        :::image type="content" source="./media/service-bus-create-namespace-portal/create-namespace.png" alt-text="Image représentant la page Créer un espace de noms":::
    1. Dans la page **Vérifier + créer**, passez en revue les paramètres, puis sélectionnez **Créer**. 
4. Sélectionnez **Accéder à la ressource** dans la page de déploiement. 

    :::image type="content" source="./media/service-bus-create-namespace-portal/deployment-alert.png" alt-text="Image représentant la page du déploiement réussi avec le lien Atteindre la ressource.":::
6. Vous voyez la page d’accueil de votre espace de noms Service Bus. 

    :::image type="content" source="./media/service-bus-create-namespace-portal/service-bus-namespace-home-page.png" alt-text="Image représentant la page d’accueil de l’espace de noms Service Bus créé." :::

## <a name="get-the-connection-string"></a>Obtenir la chaîne de connexion 
La création d’un espace de noms génère automatiquement une première stratégie de signature d’accès partagé (SAS) avec des clés primaires et secondaires, ainsi que des chaînes de connexion primaires et secondaires, qui donnent chacune un contrôle total sur tous les aspects de l’espace de noms. Consultez [Authentification et autorisation Service Bus](../service-bus-authentication-and-authorization.md) pour plus d’information sur la façon de créer des règles avec des droits plus restreints pour les expéditeurs et destinataires réguliers. 

Pour copier la chaîne de connexion primaire pour votre espace de noms, suivez ces étapes : 

1. Dans la page **Espace de noms Service Bus**, sélectionnez **Stratégies d’accès partagé** dans le menu de gauche.
3. Dans la page **Stratégies d’accès partagé**, sélectionnez **RootManageSharedAccessKey**.
   
    :::image type="content" source="./media/service-bus-create-namespace-portal/connection-info.png" alt-text="La capture d’écran montre la fenêtre Stratégies d’accès partagé avec une stratégie mise en évidence.":::
4. Dans la fenêtre **Stratégie : RootManageSharedAccessKey**, cliquez sur le bouton Copier situé en regard de **Chaîne de connexion primaire**, pour copier la chaîne de connexion dans le presse-papiers pour une utilisation ultérieure. Copiez cette valeur dans le Bloc-notes ou un autre emplacement temporaire.
   
    :::image type="content" source="./media/service-bus-create-namespace-portal/connection-string.png" alt-text="La capture d’écran montre une stratégie S A S appelée RootManageSharedAccessKey, qui comprend des clés et des chaînes de connexion.":::

    Vous pouvez utiliser cette page pour copier la clé primaire, la clé secondaire et la chaîne de connexion secondaire. 
