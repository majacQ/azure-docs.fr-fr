---
title: Alertes de sécurité pour les rôles Azure AD dans PIM - Azure AD | Microsoft Docs
description: Configurer des alertes de sécurité pour les rôles Azure AD Privileged Identity Management (PIM) dans Azure Active Directory.
services: active-directory
documentationcenter: ''
author: curtand
manager: KarenH444
editor: ''
ms.service: active-directory
ms.topic: how-to
ms.workload: identity
ms.subservice: pim
ms.date: 06/30/2021
ms.author: curtand
ms.reviewer: shaunliu
ms.custom: pim
ms.collection: M365-identity-device-management
ms.openlocfilehash: 87a31a810b82824f1d25d79e581e6dbd638bf5bb
ms.sourcegitcommit: bee590555f671df96179665ecf9380c624c3a072
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/07/2021
ms.locfileid: "129668862"
---
# <a name="configure-security-alerts-for-azure-ad-roles-in-privileged-identity-management"></a>Configurer les alertes de sécurité pour les rôles Azure AD dans Privileged Identity Management

Privileged Identity Management (PIM) génère des alertes en cas d’activité suspecte ou non fiable dans votre organisation Azure Active Directory (Azure AD). Lorsqu’une alerte est déclenchée, elle s’affiche dans le tableau de bord Privileged Identity Management. Sélectionnez l’alerte pour obtenir un rapport qui répertorie les utilisateurs ou les rôles à l’origine de son déclenchement.

![Capture d’écran montrant la page « alertes » avec une liste d’alertes et leur gravité.](./media/pim-how-to-configure-security-alerts/view-alerts.png)

## <a name="security-alerts"></a>Alertes de sécurité

Cette section répertorie toutes les alertes de sécurité pour les rôles Azure AD, et explique comment les corriger et les éviter. Le terme Gravité revêt les significations suivantes :

- **Haute** : nécessite une action immédiate, car il s’agit d’une violation de stratégie.
- **Moyenne** : ne nécessite pas d’action immédiate, mais signale une violation potentielle de la stratégie.
- **Faible** : ne nécessite pas d’action immédiate, mais suggère de préférence une modification de la stratégie.

### <a name="administrators-arent-using-their-privileged-roles"></a>Les administrateurs n’utilisent pas leurs rôles privilégiés

Gravité : **Faible**

| | Description |
| --- | --- |
| **Pourquoi reçois-je cette alerte ?** | L’affectation de rôles privilégiés à des utilisateurs qui n’en ont pas besoin augmente la probabilité d’une attaque. Il est également plus facile pour les attaquants de passer inaperçus dans des comptes qui ne sont pas activement utilisés. |
| **Procédure de résolution** | Passez en revue les utilisateurs de la liste et retirez-les des rôles privilégiés dont ils n’ont pas besoin. |
| **Prévention** | Affectez des rôles privilégiés uniquement aux utilisateurs qui en ont besoin sur le plan professionnel. </br>Planifiez des [révisions d’accès](./pim-create-azure-ad-roles-and-resource-roles-review.md) régulières pour vérifier que les utilisateurs ont toujours besoin de leur accès. |
| **Action d´atténuation dans le portail** | Supprime le compte de leur rôle privilégié. |
| **Déclencheur** | Déclenché si un utilisateur dépasse un nombre spécifié de jours sans activer un rôle. |
| **Nombre de jours** | Ce paramètre spécifie le nombre maximal de jours, de 0 à 100, pendant lesquels un utilisateur peut rester sans activer un rôle.|

### <a name="roles-dont-require-multi-factor-authentication-for-activation"></a>Les rôles ne nécessitent pas l’authentification multifacteur pour l’activation

Gravité : **Faible**

| | Description |
| --- | --- |
| **Pourquoi reçois-je cette alerte ?** | Sans authentification multifacteur, les utilisateurs compromis peuvent activer des rôles privilégiés. |
| **Procédure de résolution** | Passez en revue la liste des rôles et [exigez l’authentification multifacteur](pim-how-to-change-default-settings.md) pour chaque rôle. |
| **Prévention** | [Demander l’authentification multifacteur](pim-how-to-change-default-settings.md) pour chaque rôle.  |
| **Action d´atténuation dans le portail** | Rend l’authentification multifacteur obligatoire pour l’activation du rôle privilégié. |

### <a name="the-organization-doesnt-have-azure-ad-premium-p2"></a>L’organisation ne dispose pas d’Azure AD Premium P2

Gravité : **Faible**

| | Description |
| --- | --- |
| **Pourquoi reçois-je cette alerte ?** | L’organisation Azure AD actuelle ne dispose pas d’Azure AD Premium P2. |
| **Procédure de résolution** | Passez en revue les informations sur les [éditions d’Azure AD](../fundamentals/active-directory-whatis.md). Effectuez une mise à niveau vers Azure AD Premium P2. |

### <a name="potential-stale-accounts-in-a-privileged-role"></a>Comptes périmés potentiels dans un rôle privilégié

Gravité : **Moyenne**

| | Description |
| --- | --- |
| **Pourquoi reçois-je cette alerte ?** | Comptes dans un rôle privilégié qui n’ont pas changé leur mot de passe au cours des 90 derniers jours. Ces comptes peuvent être des comptes partagés ou de service, qui ne sont pas gérés et qui sont vulnérables aux attaques. |
| **Procédure de résolution** | Passez en revue les comptes de la liste. S’ils n’ont plus besoin d’un accès, supprimez-les de leurs rôles privilégiés. |
| **Prévention** | Assurez-vous que les comptes partagés modifient régulièrement leur mot de passe fort en cas de changement des utilisateurs qui connaissent le mot de passe. </br>Passez régulièrement en revue les comptes avec des rôles privilégiés en utilisant des [révisions d’accès](./pim-create-azure-ad-roles-and-resource-roles-review.md) et supprimez les attributions de rôles qui ne sont plus nécessaires. |
| **Action d´atténuation dans le portail** | Supprime le compte de leur rôle privilégié. |
| **Bonnes pratiques** | Les mots de passe des comptes d’accès partagés, de service et d’urgence qui s’authentifient avec un mot de passe et qui sont attribués à des rôles d’administrateur disposant de privilèges élevés, comme Administrateur général ou Administrateur de la sécurité, doivent faire l’objet d’une rotation dans les cas suivants :<ul><li>Après un incident de sécurité impliquant une mauvaise utilisation ou une compromission de droits d’accès d’administration</li><li>Après la modification des privilèges d’un utilisateur afin qu’il ne soit plus administrateur (par exemple si un employé qui était administrateur quitte le département informatique ou l’organisation)</li><li>À intervalles réguliers (par exemple trimestriels ou annuels), même s’il n’y a pas eu de violation connue ou de modification du personnel du département informatique</li></ul>Comme plusieurs personnes ont accès aux informations d’identification de ces comptes, les informations d’identification doivent faire l’objet d’une rotation pour que les personnes ayant quitté leurs rôles ne puissent plus accéder aux comptes. [En savoir plus sur la sécurisation des comptes](../roles/security-planning.md) |

### <a name="roles-are-being-assigned-outside-of-privileged-identity-management"></a>Les rôles sont affectés en dehors de Privileged Identity Management

Gravité : **Élevée**

| | Description |
| --- | --- |
| **Pourquoi reçois-je cette alerte ?** | Les affectations de rôle privilégié effectuées en dehors de Privileged Identity Management ne sont pas analysées correctement et peuvent indiquer une attaque active. |
| **Procédure de résolution** | Passez en revue les utilisateurs de la liste et retirez-les des rôles privilégiés qui leur ont été attribués en dehors de Privileged Identity Management. Vous pouvez également activer ou désactiver l’alerte et la notification par e-mail qui l’accompagne dans les paramètres de l’alerte. |
| **Prévention** | Recherchez les endroits où des rôles privilégiés ont été assignés aux utilisateurs en dehors de Privileged Identity Management et interdisez toute future affectation à partir de là. |
| **Action d´atténuation dans le portail** | Supprime l’utilisateur de leur rôle privilégié. |

### <a name="there-are-too-many-global-administrators"></a>Trop d'administrateurs généraux

Gravité : **Faible**

| | Description |
| --- | --- |
| **Pourquoi reçois-je cette alerte ?** | Administrateur général est le rôle privilégié le plus élevé. Si un administrateur général est victime d’une attaque, la personne malveillante a accès à toutes ses autorisations, ce qui met en danger l’intégralité de votre système. |
| **Procédure de résolution** | Passez en revue les utilisateurs de la liste et supprimez ceux qui n’ont absolument pas besoin du rôle Administrateur général. </br>Affectez des rôles privilégiés inférieurs à ces utilisateurs. |
| **Prévention** | Affectez aux utilisateurs le rôle le moins privilégié dont ils ont besoin. |
| **Action d´atténuation dans le portail** | Supprime le compte de leur rôle privilégié. |
| **Déclencheur** | Cette alerte se déclenche si deux critères correspondent, et vous pouvez configurer ces deux valeurs. Vous devez tout d’abord atteindre un certain seuil d’attributions de rôle Administrateur général. Puis vous devez réserver un certain pourcentage du total de vos affectations à des rôles d’administrateur général. L’alerte ne s’affiche pas si vous ne remplissez qu’un seul de ces critères. |
| **Nombre minimal d’administrateurs généraux** | Ce paramètre spécifie le nombre d’attributions de rôle Administrateurs général, de 2 à 100, que vous jugez trop peu nombreux pour votre organisation Azure AD. |
| **Pourcentage d'administrateurs généraux** | Ce paramètre spécifie, de 0 % à 100 %, le pourcentage d'administrateurs généraux par rapport au nombre total d’administrateurs en dessous duquel vous ne souhaitez pas que votre organisation Azure AD descende. |

### <a name="roles-are-being-activated-too-frequently"></a>Les rôles sont activés trop fréquemment

Gravité : **Faible**

| | Description |
| --- | --- |
| **Pourquoi reçois-je cette alerte ?** | Plusieurs activations au même rôle privilégié par le même utilisateur sont le signe d’une attaque. |
| **Procédure de résolution** | Passez en revue les utilisateurs de la liste et vérifiez que la [durée d’activation](pim-how-to-change-default-settings.md) de leur rôle privilégié est suffisante pour qu’ils puissent effectuer leurs tâches. |
| **Prévention** | Vérifiez que la [durée d’activation](pim-how-to-change-default-settings.md) des rôles privilégiés est suffisante pour que les utilisateurs puissent effectuer leurs tâches.</br>[Exigez l’authentification multifacteur](pim-how-to-change-default-settings.md) pour les rôles privilégiés qui ont des comptes partagés par plusieurs administrateurs. |
| **Action d´atténuation dans le portail** | N/A |
| **Déclencheur** | Se déclenche si un utilisateur active le même rôle privilégié plusieurs fois pendant une période spécifiée. Vous pouvez configurer la période et le nombre d’activations. |
| **Période de renouvellement d'activation** | Ce paramètre spécifie la période, en jours, heures, minutes et secondes, que vous souhaitez utiliser pour effectuer le suivi des renouvellements suspects. |
| **Nombre de renouvellements de l’activation** | Ce paramètre spécifie le nombre d’activations, de 2 à 100, pour lesquelles vous souhaitez recevoir des notifications, dans l’intervalle de temps que vous avez choisi. Vous pouvez modifier ce paramètre en déplaçant le curseur ou en tapant un nombre dans la zone de texte. |

## <a name="customize-security-alert-settings"></a>Personnaliser les paramètres d’alerte de sécurité

Sur la page **Alertes**, sélectionnez **Paramètre**.

![Page des alertes avec paramètres mis en surbrillance](media/pim-how-to-configure-security-alerts/alert-settings.png)

Personnalisez les paramètres des différentes alertes pour travailler avec votre environnement et les objectifs de sécurité.

![Page de configuration d’une alerte permettant d’activer et de configurer des paramètres](media/pim-how-to-configure-security-alerts/security-alert-settings.png)

## <a name="next-steps"></a>Étapes suivantes

- [Configurer les paramètres des rôles Azure AD dans Privileged Identity Management](pim-how-to-change-default-settings.md)