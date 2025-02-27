---
title: Importer une API SOAP et la convertir pour REST à l’aide du portail Azure | Microsoft Docs
description: Découvrez comment importer une API SOAP, la convertir en REST avec le service Gestion des API, puis tester l’API dans les portails des développeurs et Azure.
services: api-management
documentationcenter: ''
author: dlepow
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: tutorial
ms.date: 11/22/2017
ms.author: danlep
ms.openlocfilehash: 711eb5ed6e92a50ba43565b393c25f270566a36b
ms.sourcegitcommit: 91915e57ee9b42a76659f6ab78916ccba517e0a5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2021
ms.locfileid: "130047415"
---
# <a name="import-a-soap-api-and-convert-to-rest"></a>Importer une API SOAP et la convertir pour REST

Cet article explique comment importer une API SOAP et la convertir pour REST. L’article explique également comment tester l’API APIM.

Dans cet article, vous apprendrez comment :

> [!div class="checklist"]
> * Importer une API SOAP et la convertir pour REST
> * Tester l’API dans le portail Azure
> * Tester l’API dans le portail des développeurs

## <a name="prerequisites"></a>Prérequis

Suivez le guide de démarrage rapide suivant : [Créer une instance du service Gestion des API Azure](get-started-create-service-instance.md).

[!INCLUDE [api-management-navigate-to-instance.md](../../includes/api-management-navigate-to-instance.md)]

## <a name="import-and-publish-a-back-end-api"></a><a name="create-api"> </a>Importer et publier une API back-end

1. Sélectionnez **API** sous **Gestion des API**.
2. Sélectionnez **WSDL** dans la liste **Ajouter une nouvelle API**.

    ![API SOAP](./media/restify-soap-api/wsdl-api.png)
3. Dans **Spécification WSDL**, entrez l’URL correspondant à votre API SOAP.
4. Cliquez sur la case d’option **SOAP à REST**. Lorsque cette option est activée, APIM tente d’effectuer une transformation automatique entre XML et JSON. Ici, les consommateurs doivent appeler l’API en tant qu’API RESTful, qui retourne JSON. APIM convertit chaque requête en un appel SOAP.

    ![SOAP à REST](./media/restify-soap-api/soap-to-rest.png)

5. Appuyez sur la touche de tabulation.

    Les champs suivants sont automatiquement remplis avec les informations de l’interface de programmation d’applications SOAP : Nom d’affichage, nom et description.
6. Ajoutez un suffixe d’URL d’API. Le suffixe est un nom qui identifie cette API spécifique dans cette instance APIM. Il doit être unique dans cette instance APIM.
9. Publiez l’API en l’associant à un produit. Dans ce cas, le produit « *Illimité* » est utilisé.  Si vous souhaitez que l’API soit publiée et mise à la disposition des développeurs, ajoutez-la à un produit. Vous pouvez effectuer cette opération durant la création de l’API ou ultérieurement.

    Les produits sont des associations d’une ou de plusieurs API. Vous pouvez inclure un certain nombre d’API et les proposer aux développeurs dans le portail des développeurs. Les développeurs doivent s’abonner à un produit pour obtenir l’accès à l’API. Quand ils s’abonnent à un produit, ils obtiennent une clé d’abonnement qui est valable pour toutes les API de ce produit. Si vous avez créé l’instance APIM, vous êtes abonné à chaque produit par défaut, car vous êtes déjà administrateur.

    Par défaut, chaque instance Gestion des API est fournie avec deux exemples de produits :

    * **Starter**
    * **Illimité**   
10. Sélectionnez **Create** (Créer).

## <a name="test-the-new-api-in-the-azure-portal"></a>Tester la nouvelle API dans le Portail Azure

Les opérations peuvent être directement appelées depuis le portail Azure, qui permet d’afficher et de tester les opérations d’une API.  

1. Sélectionnez l’API que vous avez créée à l’étape précédente.
2. Appuyez sur l’onglet **Test**.
3. Sélectionnez une opération.

    La page affiche des champs pour les paramètres de requête et des champs pour les en-têtes. L’un des en-têtes est « Ocp-Apim-Subscription-Key », pour la clé d’abonnement du produit qui est associé à cette API. Si vous avez créé l’instance APIM, la clé est renseignée automatiquement, car vous êtes déjà administrateur. 
1. Appuyez sur **Envoyer**.

    Le serveur principal répond avec **200 OK** et certaines données.

[!INCLUDE [api-management-navigate-to-instance.md](../../includes/api-management-append-apis.md)]

[!INCLUDE [api-management-define-api-topics.md](../../includes/api-management-define-api-topics.md)]

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Transform and protect your API](transform-api.md) (Transformer et protéger votre API)