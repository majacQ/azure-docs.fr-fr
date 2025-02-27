---
title: Créer une ressource Personalizer
description: Dans cet article, vous allez apprendre à créer une ressource personnalisable dans le Portail Azure pour chaque boucle de rétroaction.
author: jeffmend
ms.author: jeffme
ms.manager: nitinme
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: how-to
ms.date: 03/26/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: b901153653b292a77ff19a25f065cf225e54f76e
ms.sourcegitcommit: 16e25fb3a5fa8fc054e16f30dc925a7276f2a4cb
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/25/2021
ms.locfileid: "122831556"
---
# <a name="create-a-personalizer-resource"></a>Création d’une ressource Personalizer

Une ressource Personalizer est la même chose qu’une boucle d’apprentissage learning loop. Une seule ressource, ou boucle d’apprentissage, est créée pour chacun de vos domaines de sujet et chacune de vos zones de contenu. N’utilisez pas plusieurs zones de contenu dans la même boucle d’apprentissage, car celle-ci s’en trouverait perturbée et produirait de mauvaises prédictions.

Si vous souhaitez que Personalizer sélectionne le meilleur contenu pour plusieurs zones de contenu d’une page web, utilisez une boucle d’apprentissage différente pour chacune.


## <a name="create-a-resource-in-the-azure-portal"></a>Créer une ressource dans le portail Azure

Créez une ressource Personalizer pour chaque boucle de rétroaction.

1. Connectez-vous au [portail Azure](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesPersonalizer). Le lien précédent vous permet d'accéder à la page **Créer** du service Personalizer.
1. Entrez le nom de votre service, puis sélectionnez un abonnement, un emplacement, un niveau tarifaire et un groupe de ressources.

    > [!div class="mx-imgBorder"]
    > ![Utiliser le Portail Azure pour créer une ressource Personalizer, également appelée « boucle d’apprentissage »](./media/how-to-create-resource/how-to-create-personalizer-resource-learning-loop.png)

1. Sélectionnez **Créer** pour créer la ressource.

1. Une fois votre ressource Personalizer déployée, sélectionnez le bouton **Accéder à la ressource** pour y accéder.

1. Sélectionnez la page **Démarrage rapide** de votre ressource, puis copiez les valeurs de votre point de terminaison et de votre clé. Le point de terminaison et la clé de la ressource sont tous deux nécessaires pour utiliser les API de classement et de récompense.

1. Sélectionnez la page **Configuration** de la nouvelle ressource pour [configurer la boucle d'apprentissage](how-to-settings.md).

## <a name="create-a-resource-with-the-azure-cli"></a>Création d’une ressource avec Azure CLI

1. Connectez-vous à Azure CLI avec la commande suivante :

    ```azurecli-interactive
    az login
    ```

1. Créez un groupe de ressources, regroupement logique permettant de gérer toutes les ressources Azure que vous avez l’intention d’utiliser avec la ressource Personalizer.


    ```azurecli-interactive
    az group create \
        --name your-personalizer-resource-group \
        --location westus2
    ```

1. Créez une ressource Personalizer, la _boucle d’apprentissage_, avec la commande suivante pour un groupe de ressources existant.

    ```azurecli-interactive
    az cognitiveservices account create \
        --name your-personalizer-learning-loop \
        --resource-group your-personalizer-resource-group \
        --kind Personalizer \
        --sku F0 \
        --location westus2 \
        --yes
    ```

    Cette opération renvoie un objet JSON qui comprend le **point de terminaison de votre ressource**.

1. Pour obtenir la **clé de votre ressource**, utilisez la commande Azure CLI suivante.

    ```azurecli-interactive
        az cognitiveservices account keys list \
        --name your-personalizer-learning-loop \
        --resource-group your-personalizer-resource-group
    ```

    Le point de terminaison et la clé de la ressource sont tous deux nécessaires pour utiliser les API de classement et de récompense.

## <a name="next-steps"></a>Étapes suivantes

* [Configuration](how-to-settings.md) de la boucle d’apprentissage Personalizer
