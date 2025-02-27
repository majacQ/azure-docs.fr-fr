---
title: Afficher les données de télémétrie de l’inférence du modèle du DK Azure Percept
description: Découvrez comment afficher les données de télémétrie de l’inférence du modèle de vision du DK Azure Percept dans l’explorateur Azure IoT
author: NabilaBabar
ms.author: amiyouss
ms.service: azure-percept
ms.topic: how-to
ms.date: 02/17/2021
ms.custom: template-how-to, ignite-fall-2021
ms.openlocfilehash: 0f8b56aef3a3e141c80b09e97101f1d78a2c761b
ms.sourcegitcommit: 106f5c9fa5c6d3498dd1cfe63181a7ed4125ae6d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/02/2021
ms.locfileid: "131006257"
---
# <a name="view-your-azure-percept-dks-model-inference-telemetry"></a>Afficher les données de télémétrie de l’inférence du modèle du DK Azure Percept

Suivez ce guide pour afficher les données de télémétrie de l’inférence du modèle de vision du DK Azure Percept dans l’[explorateur Azure IoT](https://github.com/Azure/azure-iot-explorer/releases).

## <a name="prerequisites"></a>Prérequis

- DK (devkit) Azure Percept
- [Abonnement Azure](https://azure.microsoft.com/free/)
- [Expérience d’installation du DK Azure Percept](./quickstart-percept-dk-set-up.md) : vous avez connecté votre DevKit à un réseau Wi-Fi, créé un hub IoT et connecté votre DevKit au hub IoT.
- [Le modèle IA de vision a été déployé sur votre DK Azure Percept](./how-to-deploy-model.md)

## <a name="view-telemetry"></a>Afficher les données de télémétrie

1. Mettez votre kit de développement sous tension.

1. Téléchargez et installez l’[explorateur Azure IoT](https://github.com/Azure/azure-iot-explorer/releases). Si vous êtes un utilisateur Windows, sélectionnez le fichier .msi.

    :::image type="content" source="./media/how-to-view-telemetry/azure-iot-explorer-download.png" alt-text="Écran de téléchargement pour l’explorateur Azure IoT.":::

1. Connectez votre hub IoT à l’explorateur Azure IoT :

    1. Accédez au [portail Azure](https://portal.azure.com).

    1. Sélectionnez **Toutes les ressources**.

        :::image type="content" source="./media/how-to-view-telemetry/azure-portal.png" alt-text="Page d’accueil du portail Azure.":::

    1. Sélectionnez le hub IoT auquel votre DK Azure Percept est connecté.

        :::image type="content" source="./media/how-to-view-telemetry/iot-hub.png" alt-text="Liste des hubs IoT dans le portail Azure.":::

    1. Sur le côté gauche de votre page IoT Hub, sélectionnez **Stratégies d’accès partagé**.

        :::image type="content" source="./media/how-to-view-telemetry/shared-access-policies.png" alt-text="Page IoT Hub présentant les stratégies d’accès partagé.":::

    1. Cliquez sur **iothubowner**.

        :::image type="content" source="./media/how-to-view-telemetry/iothubowner.png" alt-text="Écran des stratégies d’accès partagé avec iothubowner mis en surbrillance.":::

    1. Cliquez sur l’icône de copie bleue en regard de **Chaîne de connexion - Clé primaire**.

        :::image type="content" source="./media/how-to-view-telemetry/connection-string.png" alt-text="Fenêtre iothubowner avec le bouton de copie en regard de Chaîne de connexion mis en surbrillance.":::

    1. Ouvrez l’explorateur Azure IoT, puis cliquez sur **+ Ajouter une connexion**.

    1. Collez la chaîne de connexion dans la zone **Chaîne de connexion** de la fenêtre **Ajouter une chaîne de connexion**, puis cliquez sur **Enregistrer**.

        :::image type="content" source="./media/how-to-view-telemetry/add-connection-string.png" alt-text="Fenêtre de l’explorateur Azure IoT avec une zone dans laquelle coller la chaîne de connexion.":::

    1. Pointez le SoM de vision sur un objet pour l’inférence de modèle.

    1. Sélectionnez **Télémétrie**.

    1. Cliquez sur **Démarrer** pour afficher les événements de télémétrie à partir de l’appareil.

## <a name="next-steps"></a>Étapes suivantes
Découvrez comment afficher votre [flux vidéo de DK Azure Percept](./how-to-view-video-stream.md).
