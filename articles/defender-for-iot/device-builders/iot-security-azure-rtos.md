---
title: Présentation du micro-agent Defender-IoT pour Azure RTOS
description: En savoir plus sur le micro-agent Defender-IoT pour la prise en charge et l’implémentation d’Azure RTOS dans le cadre de Microsoft Defender pour IoT.
ms.topic: conceptual
ms.date: 11/14/2021
ms.openlocfilehash: 138c08e510bbfe66cb52460c96ba70764cd3ce20
ms.sourcegitcommit: 362359c2a00a6827353395416aae9db492005613
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/15/2021
ms.locfileid: "132487120"
---
# <a name="overview-defender-for-iot-defender-iot-micro-agent-for-azure-rtos"></a>Présentation : Defender pour IoT Defender-IoT-micro-agent pour Azure RTOS

Le micromodule Microsoft Defender pour IoT constitue une solution de sécurité complète pour les appareils qui utilisent Azure RTOS. Il permet de couvrir les menaces courantes et les activités malveillantes potentielles sur les appareils de système d’exploitation en temps réel (RTOS). Azure RTOS est désormais équipé du micro-agent Defender-IoT d’Azure IoT.

:::image type="content" source="./media/architecture/azure-rtos-security-monitoring.png" alt-text="Visualisation de Azure Defender pour IoT Azure RTOS.":::

Le micromodule pour Azure RTOS offre les fonctionnalités suivantes :

- Détection des activités réseau malveillantes
- Établissement d’une ligne de base du comportement de l’appareil basé sur des alertes personnalisées
- Amélioration de l’hygiène de sécurité des appareils

## <a name="detect-malicious-network-activities"></a>Détection des activités malveillantes sur les réseaux

L’activité réseau entrante et sortante de chaque appareil est analysée. Sont pris en charge les protocoles TCP, UDP et ICMP sur IPv4 et IPv6. Defender pour IoT inspecte chacune de ces activités réseau par rapport au flux de renseignement sur les menaces de Microsoft. Le flux est mis à jour en temps réel par des millions d’indicateurs de menace uniques collectés dans le monde entier.

## <a name="device-behavior-baselining-based-on-custom-alerts"></a>Établissement d’une ligne de base du comportement de l’appareil basé sur des alertes personnalisées

L’établissement d’une ligne de base permet le clustering d’appareils dans des groupes de sécurité et la définition du comportement attendu de chaque groupe. Étant donné que les appareils IoT sont généralement conçus pour fonctionner dans des scénarios limités et bien définis, il est facile de créer une ligne de base qui définit le comportement attendu à l’aide d’un ensemble de paramètres. Tout écart par rapport à la ligne de base déclenche une alerte.

## <a name="improve-your-device-security-hygiene"></a>Amélioration de l’hygiène de sécurité de vos appareils

En utilisant l’infrastructure recommandée par Defender pour IoT, vous pouvez obtenir des connaissances et des informations sur les problèmes de votre environnement qui ont un impact sur la sécurité de vos appareils. Une sécurité faible de l’appareil IoT peut permettre à des attaques potentielles de réussir si elle n’est pas renforcée. La sécurité est toujours mesurée selon le maillon le plus faible au sein d’une organisation.

## <a name="get-started-protecting-azure-rtos-devices"></a>Prise en main de la protection des appareils Azure RTOS

Le micro-agent Defender-IoT pour Azure RTOS est fourni en téléchargement gratuit pour vos appareils. Le service Defender pour IoT est disponible avec un essai gratuit de 30 jours par abonnement Azure. Pour commencer, téléchargez le [micro-agent Defender-IoT pour Azure RTOS](https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/defender-for-iot/device-builders/iot-security-azure-rtos.md).

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez découvert le micro-agent Defender-IoT pour Azure RTOS. Pour en savoir plus sur le micro-agent Defender-IoT et le prendre en main, consultez les articles suivants :

- [Concepts relatifs au micro-agent Defender-IoT pour Azure RTOS](concept-rtos-security-module.md)
- [Démarrage rapide : Micro-agent Defender-IoT d’IoT pour Azure RTOS](quickstart-azure-rtos-security-module.md)
