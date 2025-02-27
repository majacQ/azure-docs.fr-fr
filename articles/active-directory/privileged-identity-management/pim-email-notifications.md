---
title: Notifications par e-mail dans Privileged Identity Management (PIM) - Azure Active Directory | Microsoft Docs
description: Explique ce que sont les notifications par e-mail dans Azure AD Privileged Identity Management (PIM).
services: active-directory
documentationcenter: ''
author: curtand
manager: KarenH444
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.subservice: pim
ms.date: 10/07/2021
ms.author: curtand
ms.reviewer: shaunliu
ms.custom: pim
ms.collection: M365-identity-device-management
ms.openlocfilehash: f9cd2b72b58899ce37843d6117f975f74ef97493
ms.sourcegitcommit: bee590555f671df96179665ecf9380c624c3a072
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/07/2021
ms.locfileid: "129667323"
---
# <a name="email-notifications-in-pim"></a>Notifications par e-mail dans PIM

Privileged Identity Management (PIM) vous informe quand des événements importants se produisent dans votre organisation Azure Active Directory (Azure AD), par exemple l’attribution ou l’activation d’un rôle. Pour vous tenir informé, Privileged Identity Management envoie des notifications par e-mail qui sont adressées à vous et à d’autres participants. Ces e-mails peuvent également inclure des liens vers des tâches appropriées, comme l’activation ou le renouvellement d’un rôle. Cet article décrit à quoi ressemblent ces e-mails, quand ils sont envoyés et qui les reçoit.

## <a name="sender-email-address-and-subject-line"></a>Adresse e-mail de l’expéditeur et ligne Objet

L’adresse de l’expéditeur des e-mails envoyés par Privileged Identity Management pour les rôles de ressources Azure AD et Azure est la suivante :

- Adresse de messagerie : **azure-noreply\@microsoft.com**
- Nom complet : Microsoft Azure

Ces e-mails incluent un préfixe **PIM** dans la ligne Objet. Voici un exemple :

- PIM : Le rôle Lecteur de sauvegarde a été définitivement attribué à Alain Charon

## <a name="email-timing-for-activation-approvals"></a>Chronologie des e-mails pour les approbations d’activation

Lorsque des utilisateurs activent leur rôle et que le paramètre de rôle nécessite une approbation, les approbateurs reçoivent deux e-mails pour chaque approbation :

- Demande d’approbation ou de refus de la demande d’activation de l’utilisateur (envoyée par le moteur d’approbation des demandes)
- La demande de l’utilisateur est approuvée (envoyée par le moteur d’approbation des demandes)

En outre, les Administrateurs généraux et les Administrateurs de rôle privilégié reçoivent un e-mail pour chaque approbation :

- Le rôle de l’utilisateur est activé (envoyé par Privileged Identity Management)

Les deux premiers e-mails envoyés par le moteur d’approbation des demandes peuvent être retardés. Actuellement, 90 % des e-mails prennent trois à dix minutes, mais pour 1 % des clients, ce délai peut s’avérer plus long, jusqu’à quinze minutes.

Si une demande d’approbation est approuvée dans le portail Azure avant l’envoi du premier e-mail, le premier e-mail n’est plus déclenché et les autres approbateurs ne sont pas avertis par e-mail de la demande d’approbation. Cela peut donner l’impression qu’ils n’ont pas reçu d’e-mail, mais c’est le comportement attendu.

## <a name="notifications-for-azure-ad-roles"></a>Notifications pour les rôles Azure AD

Privileged Identity Management envoie des e-mails quand les événements suivants se produisent pour des rôles Azure AD :

- Quand l’activation d’un rôle privilégié est en attente d’approbation
- Quand une demande d’activation de rôle privilégié est approuvée
- Quand Azure AD Privileged Identity Management est activé

Les destinataires de ces e-mails pour les rôles Azure AD varient selon votre rôle, l’événement et le paramètre de notification.

| Utilisateur | Une activation d’un rôle est en attente d’approbation | Une demande d’activation de rôle est terminée | PIM est activé |
| --- | --- | --- | --- |
| Administrateur de rôle privilégié</br>(Activé/éligible) | Oui</br>(uniquement si aucun approbateur explicite n’est spécifié) | Oui* | Oui |
| Security Administrator</br>(Activé/éligible) | Non | Oui* | Oui |
| Administrateur général</br>(Activé/éligible) | Non | Oui* | Oui |

\* Si le paramètre [**Notifications**](pim-how-to-change-default-settings.md) a la valeur **Activer**.

L’exemple suivant montre un exemple d’e-mail envoyé quand un utilisateur active un rôle Azure AD pour l’organisation fictive Contoso.

![Nouvel e-mail Privileged Identity Management pour les rôles Azure AD](./media/pim-email-notifications/email-directory-new.png)

### <a name="weekly-privileged-identity-management-digest-email-for-azure-ad-roles"></a>E-mail Privileged Identity Management de synthèse hebdomadaire pour les rôles Azure AD

Un e-mail Privileged Identity Management récapitulatif pour les rôles Azure AD est envoyé chaque semaine aux administrateurs de rôles privilégiés, administrateurs de sécurité et administrateurs généraux qui ont activé Privileged Identity Management. Cet e-mail hebdomadaire fournit un instantané des activités Privileged Identity Management pour la semaine, ainsi que les attributions de rôles privilégiés. Il est uniquement disponible pour les organisations Azure AD sur le cloud public. Voici un exemple d’e-mail :

![E-mail Privileged Identity Management de synthèse hebdomadaire pour les rôles Azure AD](./media/pim-email-notifications/email-directory-weekly.png)

Cet e-mail comprend :

| Vignette | Description |
| --- | --- |
| **Utilisateurs activés** | Nombre de fois que les utilisateurs ont activé leur rôle éligible au sein de l’organisation. |
| **Utilisateurs devenus permanents** | Nombre de fois que les utilisateurs avec une attribution éligible sont devenus permanents. |
| **Attributions de rôle dans Privileged Identity Management** | Nombre de fois que les utilisateurs se voient attribuer un rôle éligible dans Privileged Identity Management. |
| **Attributions de rôles en dehors de PIM** | Nombre de fois que les utilisateurs se voient attribuer un rôle permanent en dehors de Privileged Identity Management (dans Azure AD). Cette alerte et l’e-mail qui l’accompagne peuvent être activés ou désactivés en ouvrant les paramètres d’alerte. |

La **vue d’ensemble des rôles principaux** liste les cinq rôles principaux de votre organisation en fonction du nombre total d’administrateurs permanents et éligibles pour chaque rôle. Le lien **Entreprendre une action** ouvre [Discovery & Insights](pim-security-wizard.md) où vous pouvez convertir des administrateurs permanents en administrateurs éligibles par lots.

## <a name="notifications-for-azure-resource-roles"></a>Notifications pour les rôles de ressources Azure

Privileged Identity Management envoie des e-mails aux propriétaires et administrateurs de l’accès utilisateur quand les événements suivants se produisent pour les rôles de ressources Azure :

- Quand une attribution de rôle est en attente d’approbation
- Quand un rôle est attribué
- Quand un rôle va bientôt expirer
- Quand un rôle peut être étendu
- Quand un rôle est renouvelé par un utilisateur final
- Quand une demande d’activation de rôle est terminée

Privileged Identity Management envoie des e-mails aux utilisateurs finals quand les événements suivants se produisent pour les rôles de ressources Azure :

- Quand un rôle est attribué à l’utilisateur
- Quand le rôle d’un utilisateur a expiré
- Quand le rôle d’un utilisateur est étendu
- Quand la demande d’activation de rôle d’un utilisateur est terminée

L’exemple suivant montre un exemple d’e-mail envoyé quand un utilisateur se voit attribuer un rôle de ressource Azure pour l’organisation fictive Contoso.

![Nouvel e-mail Privileged Identity Management pour les rôles de ressources Azure](./media/pim-email-notifications/email-resources-new.png)

## <a name="notifications-for-privileged-access-groups"></a>Notifications pour les groupes d’accès privilégiés

Privileged Identity Management envoie des e-mails aux propriétaires uniquement lorsque les événements suivants se produisent pour les attributions de groupe d’accès privilégié :

- lorsqu’une attribution de rôle de propriétaire ou de membre est en attente d’approbation
- lorsqu’un rôle de propriétaire ou de membre est attribué
- lorsqu’un rôle de propriétaire ou de membre est sur le point d’expirer
- lorsqu’un rôle de propriétaire ou de membre est éligible pour une extension
- lorsqu’un rôle de propriétaire ou de membre est renouvelé par l’utilisateur final
- lorsqu’une requête d’activation de rôle de propriétaire ou de membre est terminée

Privileged Identity Management envoie des e-mails aux utilisateurs finaux uniquement lorsque les événements suivants se produisent pour les attributions de groupe d’accès privilégié :

- lorsqu’un rôle de propriétaire ou de membre est attribué à un utilisateur
- lorsque le rôle de propriétaire ou de membre d’un utilisateur a expiré
- lorsque le rôle de propriétaire ou de membre d’un utilisateur a été étendu
- lorsque la requête d’activation de rôle de propriétaire ou de membre d’un utilisateur est terminée


## <a name="next-steps"></a>Étapes suivantes

- [Configurer les paramètres des rôles Azure AD dans Privileged Identity Management](pim-how-to-change-default-settings.md)
- [Approuver ou rejeter des requêtes de rôles Azure AD dans Privileged Identity Management](azure-ad-pim-approval-workflow.md)
