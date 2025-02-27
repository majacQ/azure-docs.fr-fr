---
title: Accès conditionnel - Bloquer l’accès par emplacement - Azure Active Directory
description: Créer une stratégie d’accès conditionnel personnalisée pour bloquer l’accès aux ressources par emplacement IP
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: how-to
ms.date: 05/26/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: karenhoran
ms.reviewer: calebb, rogoya
ms.collection: M365-identity-device-management
ms.openlocfilehash: f20eb91d18c85b21bd9623fec2129fbbde787764
ms.sourcegitcommit: f6e2ea5571e35b9ed3a79a22485eba4d20ae36cc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/24/2021
ms.locfileid: "128568285"
---
# <a name="conditional-access-block-access-by-location"></a>Accès conditionnel : Bloquer l’accès par emplacement

Avec la condition d’emplacement dans l’accès conditionnel, vous pouvez contrôler l’accès à vos applications cloud basées sur l’emplacement réseau d’un utilisateur. La condition d’emplacement est couramment utilisée pour bloquer l’accès à partir des pays/régions d’où votre organisation sait que le trafic ne doit pas provenir.

> [!NOTE]
> Des stratégies d'accès conditionnel sont appliquées au terme de l'authentification premier facteur. L'accès conditionnel n'est pas destiné à être la première ligne de défense d'une organisation pour des scénarios comme les attaques par déni de service (DoS), mais il peut utiliser les signaux de ces événements pour déterminer l'accès.

## <a name="define-locations"></a>Définir des emplacements

1. Connectez-vous au **portail Microsoft Azure** en tant qu’administrateur général, administrateur de sécurité ou administrateur de l’accès conditionnel.
1. Accédez à **Azure Active Directory** > **Sécurité** > **Accès conditionnel** > **Emplacements nommés**.
1. Choisissez **Nouvel emplacement**.
1. Donnez un nom à votre emplacement.
1. Choisissez **Plages d’adresses IP** si vous connaissez les plages d’adresses IPv4 accessibles de l’extérieur qui composent cet emplacement ou ces **pays/régions**.
   1. Fournissez les **plages d’adresses IP** ou sélectionnez **Pays/régions** pour l’emplacement que vous spécifiez.
      * Si vous choisissez Pays/régions, vous pouvez éventuellement choisir d’inclure les zones inconnues.
1. Choisir **Enregistrer**

Pour plus d’informations sur la condition d’emplacement dans l’accès conditionnel, consultez l’article [Qu’est-ce que la condition d’emplacement de l’accès conditionnel Azure Active Directory ?](location-condition.md)

## <a name="create-a-conditional-access-policy"></a>Créer une stratégie d’accès conditionnel

1. Connectez-vous au **portail Microsoft Azure** en tant qu’administrateur général, administrateur de sécurité ou administrateur de l’accès conditionnel.
1. Accédez à **Azure Active Directory** > **Sécurité** > **Accès conditionnel.**
1. Sélectionnez **Nouvelle stratégie**.
1. Donnez un nom à votre stratégie. Nous recommandons aux organisations de créer une norme explicite pour les noms de leurs stratégies.
1. Sous **Affectations**, sélectionnez **Utilisateurs et groupes**
   1. Sous **Inclure**, sélectionnez **Tous les utilisateurs**.
   1. Sous **Exclure**, sélectionnez **Utilisateurs et groupes**, puis choisissez les comptes d’accès d’urgence ou de secours de votre organisation. 
   1. Sélectionnez **Terminé**.
1. Sous **Applications ou actions cloud** > **Inclure**, sélectionnez **Toutes les applications cloud**.
1. Sous **Conditions** > **Emplacement**.
   1. Définissez **Configurer** sur **Oui**
   1. Sous **Inclure**, sélectionnez **Emplacements sélectionnés**
   1. Sélectionnez l’emplacement bloqué que vous avez créé pour votre organisation.
   1. Cliquez sur **Sélectionner**.
1. Sous **Contrôles d’accès**, sélectionnez **Bloquer l’accès**, puis **Sélectionner**.
1. Confirmez vos paramètres et réglez **Activer la stratégie** sur **Activé**.
1. Sélectionnez **Créer** pour appliquer la stratégie d’accès conditionnel.

## <a name="next-steps"></a>Étapes suivantes

[Stratégies d’accès conditionnel courantes](concept-conditional-access-policy-common.md)

[Déterminer l'impact à l'aide du mode Rapport seul de l'Accès conditionnel](howto-conditional-access-insights-reporting.md)

[Simuler le comportement de connexion à l’aide de l’outil What If pour l’accès conditionnel](troubleshoot-conditional-access-what-if.md)
