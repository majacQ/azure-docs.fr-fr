---
title: Utiliser Azure IoT Tools pour VSCode pour la messagerie du hub informatique
description: Découvrez comment utiliser Azure IoT Tools pour Visual Studio Code afin de superviser les messages appareil-à-cloud et d’envoyer des messages cloud-à-appareil dans Azure IoT Hub.
author: formulahendry
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 01/18/2019
ms.author: junhan
ms.custom:
- 'Role: Cloud Development'
ms.openlocfilehash: 8c311ead11ed41494bf1cff82809ba8f06843226
ms.sourcegitcommit: cc099517b76bf4b5421944bd1bfdaa54153458a0
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/09/2021
ms.locfileid: "113551215"
---
# <a name="use-azure-iot-tools-for-visual-studio-code-to-send-and-receive-messages-between-your-device-and-iot-hub"></a>Utiliser Azure IoT Tools pour Visual Studio Code afin d’envoyer et de recevoir des messages entre votre appareil et un hub IoT

![Diagramme de bout en bout](./media/iot-hub-vscode-iot-toolkit-cloud-device-messaging/e-to-e-diagram.png)

Dans cet article, vous allez découvrir comment utiliser Azure IoT Tools pour Visual Studio Code afin de superviser les messages appareil-à-cloud et d’envoyer des messages cloud-à-appareil. Il peut s’agir de données de capteur collectées par votre appareil, puis envoyées vers votre instance IoT Hub. Il peut s’agir de commandes envoyées par votre IoT Hub à votre appareil pour faire clignoter un voyant connecté à votre appareil.

[Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-toolkit) est une extension Visual Studio Code qui facilite la gestion d’IoT Hub et le développement des applications IoT. Cet article explique comment utiliser Azure IoT Tools pour Visual Studio Code afin d’envoyer et de recevoir des messages entre votre appareil et votre hub IoT.

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-partial.md)]

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure actif.

* Une instance Azure IoT Hub associée à votre abonnement.

* [Visual Studio Code](https://code.visualstudio.com/)

* [Azure IoT Tools pour VS Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools) ou copiez et collez cette URL dans une fenêtre de navigateur : `vscode:extension/vsciot-vscode.azure-iot-tools`.

## <a name="sign-in-to-access-your-iot-hub"></a>Se connecter pour accéder à votre hub ioT

1. En mode **Explorateur** dans Visual Studio Code, développez la section **Appareils Azure IoT Hub** dans le coin inférieur gauche.

2. Cliquez sur **Sélectionner IoT Hub** dans le menu contextuel.

3. Une fenêtre contextuelle s’affiche en bas à droite pour vous permettre de vous connecter à Azure pour la première fois.

4. Une fois que vous êtes connecté, votre liste d’abonnements Azure s’affiche. Sélectionnez Abonnement Azure et IoT Hub.

5. La liste des appareils apparaît quelques secondes plus tard sous l’onglet **Appareils Azure IoT Hub**.

   > [!Note]
   > Vous pouvez également effectuer la configuration en choisissant **Définir la chaîne de connexion IoT Hub**. Entrez la chaîne de connexion de stratégie **iothubowner** IoT Hub à laquelle se connecte votre appareil IoT dans la fenêtre contextuelle.

## <a name="monitor-device-to-cloud-messages"></a>Analyse de messages appareil vers cloud

Pour analyser les messages envoyés à partir de votre appareil à votre instance IoT H+ub, procédez comme suit :

1. Cliquez avec le bouton droit sur votre appareil et sélectionnez **Start Monitoring Built-in Event Endpoint** (Démarrer la surveillance du point de terminaison d’événements intégré).

2. Les messages supervisés seront affichés dans la vue **SORTIE** > **Azure IoT Hub**.

3. Pour arrêter la surveillance, cliquez avec le bouton droit sur la vue **SORTIE**, puis sélectionnez **Stop Monitoring Built-in Event Endpoint** (Arrêter la surveillance du point de terminaison d’événements intégré).

## <a name="send-cloud-to-device-messages"></a>Envoi de messages cloud vers appareil

Pour envoyer un message à partir de votre instance IoT Hub sur votre appareil, procédez comme suit :

1. Cliquez avec le bouton droit sur votre appareil et sélectionnez **Envoyer un message cloud-à-appareil**.

2. Entrez le message dans la zone de saisie.

3. Les résultats seront affichés dans la vue **SORTIE** > **Azure IoT Hub**.

## <a name="next-steps"></a>Étapes suivantes

Vous avez appris à analyser des messages appareil vers cloud et à envoyer des messages cloud vers appareil entre votre appareil IoT et l’instance IoT Hub.

[!INCLUDE [iot-hub-get-started-next-steps](../../includes/iot-hub-get-started-next-steps.md)]