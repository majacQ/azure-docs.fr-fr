---
title: 'Didacticiel : Configurer Klaxoon SAML pour l’approvisionnement automatique d’utilisateurs avec Azure Active Directory | Microsoft Docs'
description: Découvrez comment approvisionner et désapprovisionner automatiquement des comptes d’utilisateur d’Azure AD dans Klaxoon SAML.
services: active-directory
documentationcenter: ''
author: twimmer
writer: twimmer
manager: beatrizd
ms.assetid: 5aaacb86-4fb0-49f3-9f7d-e9ea94829b2b
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/22/2021
ms.author: thwimmer
ms.openlocfilehash: c032dcd6cfe41e2893b53029254a96f54c9fc595
ms.sourcegitcommit: 1d56a3ff255f1f72c6315a0588422842dbcbe502
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/06/2021
ms.locfileid: "129621058"
---
# <a name="tutorial-configure-klaxoon-saml-for-automatic-user-provisioning"></a>Didacticiel : Configurer Klaxoon SAML pour l’approvisionnement automatique d’utilisateurs

Ce didacticiel décrit les étapes à suivre dans Klaxoon SAML et Azure Active Directory (Azure AD) pour configurer l’approvisionnement automatique d’utilisateurs. Une fois configuré, Azure AD approvisionne et désapprovisionne automatiquement des utilisateurs et des groupes dans [Klaxoon SAML](https://www.klaxoon.com/) à l’aide du service d’approvisionnement Azure AD. Pour découvrir les informations importantes sur ce que fait ce service, comment il fonctionne et consulter le forum aux questions, reportez-vous à l’article [Automatiser l’attribution et l’annulation de l’attribution des utilisateurs dans les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md). 


## <a name="capabilities-supported"></a>Fonctionnalités prises en charge
> [!div class="checklist"]
> * Créer des utilisateurs dans Klaxoon.
> * Désactiver des utilisateurs dans Klaxoon lorsqu’ils n’ont plus besoin d’accès.
> * Conserver les attributs utilisateur synchronisés entre Azure AD et Klaxoon.
> * Fournir des licences aux utilisateurs dans Klaxoon en fonction des groupes Azure AD.
> * [Authentification unique](klaxoon-saml-tutorial.md) auprès de Klaxoon à l’aide de SAML (recommandée).

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce tutoriel part du principe que vous disposez des prérequis suivants :

* [Un locataire Azure AD](../develop/quickstart-create-new-tenant.md). 
* Un compte d’utilisateur dans Azure AD avec l’[autorisation](../roles/permissions-reference.md) de configurer l’approvisionnement (par exemple, administrateur d’application, administrateur d’application Cloud, propriétaire d’application ou administrateur général). 
* Un [contrat Klaxoon](https://klaxoon.com/enterprise).

## <a name="step-1-plan-your-provisioning-deployment"></a>Étape 1. Planifier votre déploiement de l’approvisionnement
1. En savoir plus sur le [fonctionnement du service d’approvisionnement](../app-provisioning/user-provisioning.md).
1. Déterminez qui sera dans l’[étendue pour l’approvisionnement](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).
1. Déterminez les données à [mapper entre Azure AD et Klaxoon SAML](../app-provisioning/customize-application-attributes.md). 

## <a name="step-2-configure-klaxoon-saml-to-support-provisioning-with-azure-ad"></a>Étape 2. Configurer Klaxoon pour prendre en charge l’approvisionnement avec Azure AD

* Contactez [Klaxoon](https://klaxoon.com/) pour recevoir une **URL de locataire** unique et un **jeton secret**.

## <a name="step-3-add-klaxoon-saml-from-the-azure-ad-application-gallery"></a>Étape 3. Ajouter Klaxoon SAML à partir de la galerie d’applications Azure AD

Ajoutez Klaxoon SAML à partir de la galerie d’applications Azure AD pour commencer à gérer l’approvisionnement dans Klaxoon. Si vous avez déjà configuré Klaxoon SAML pour l’authentification unique, vous pouvez utiliser la même application. Toutefois, il est recommandé de créer une application distincte lors du test initial de l’intégration. En savoir plus sur l’ajout d’une application à partir de la galerie [ici](../manage-apps/add-application-portal.md). 

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Étape 4. Définir qui sera dans l’étendue pour l’approvisionnement 

Le service d’approvisionnement Azure AD vous permet de définir l’étendue des utilisateurs approvisionnés en fonction de l’affectation à l’application et/ou en fonction des attributs de l’utilisateur/groupe. Si vous choisissez de définir l’étendue de l’approvisionnement pour votre application en fonction de l’attribution, vous pouvez utiliser les étapes de [suivantes](../manage-apps/assign-user-or-group-access-portal.md) pour affecter des utilisateurs et des groupes à l’application. Si vous choisissez de définir l’étendue de l’approvisionnement en fonction uniquement des attributs de l’utilisateur ou du groupe, vous pouvez utiliser un filtre d’étendue comme décrit [ici](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

* Quand vous attribuez des utilisateurs et des groupes à Klaxoon SAML, vous devez sélectionner un rôle différent du rôle **Accès par défaut**. Les utilisateurs disposant du rôle Accès par défaut sont exclus de l’approvisionnement et sont marqués comme non autorisés dans les journaux de configuration. Si le seul rôle disponible dans l’application est le rôle d’accès par défaut, vous pouvez [mettre à jour le manifeste de l’application](../develop/howto-add-app-roles-in-azure-ad-apps.md) pour ajouter des rôles supplémentaires. 

* Commencez progressivement. Testez avec un petit ensemble d’utilisateurs et de groupes avant d’effectuer un déploiement général. Lorsque l’étendue de l’approvisionnement est définie sur les utilisateurs et les groupes attribués, vous pouvez contrôler cela en affectant un ou deux utilisateurs ou groupes à l’application. Lorsque l’étendue est définie sur tous les utilisateurs et groupes, vous pouvez spécifier un [filtre d’étendue basé sur l’attribut](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 


## <a name="step-5-configure-automatic-user-provisioning-to-klaxoon"></a>Étape 5. Configurer l’attribution automatique d’utilisateurs à Klaxoon 

Cette section vous guide tout au long des étapes de configuration du service d’approvisionnement Azure AD pour créer, mettre à jour et désactiver des utilisateurs et/ou des groupes dans Klaxoon SAML en fonction des affectations d’utilisateurs et/ou de groupes dans Azure AD.

### <a name="to-configure-automatic-user-provisioning-for-klaxoon-saml-in-azure-ad"></a>Pour configurer l’approvisionnement automatique d’utilisateurs pour Klaxoon SAML dans Azure AD :

1. Connectez-vous au [portail Azure](https://portal.azure.com). Sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

1. Dans la liste des applications, sélectionnez **Klaxoon SAML**.

    ![Lien Klaxoon SAML dans la liste Applications](common/all-applications.png)

1. Sélectionnez l’onglet **Approvisionnement**.

    ![Onglet Approvisionnement](common/provisioning.png)

1. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Onglet Provisionnement automatique](common/provisioning-automatic.png)

1. Dans la section **Informations d’identification de l’administrateur**, saisissez votre URL de locataire Klaxoon et le jeton secret fournis par Klaxoon. Cliquez sur **Tester la connexion** pour vérifier qu’Azure AD peut se connecter à Klaxoon. Si la connexion échoue, veuillez contacter Klaxoon pour vérifier la configuration de votre compte.

    ![par jeton](common/provisioning-testconnection-tenanturltoken.png)

1. Dans le champ **E-mail de notification**, entrez l’adresse e-mail de la personne ou du groupe qui doit recevoir les notifications d’erreur de provisionnement et sélectionnez la case à cocher **Envoyer une notification par e-mail en cas de défaillance**.

    ![E-mail de notification](common/provisioning-notification-email.png)

1. Sélectionnez **Enregistrer**.

1. Dans la section **Mappages**, sélectionnez **Synchroniser les utilisateurs Azure Active Directory avec Klaxoon SAML**.

1. Dans la section **Mappages des attributs**, passez en revue les attributs utilisateur qui sont synchronisés entre Azure AD et Klaxoon. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les comptes d’utilisateurs dans Klaxoon pour les opérations de mise à jour. Si vous choisissez de modifier l’[attribut cible correspondant](../app-provisioning/customize-application-attributes.md), vous devez vérifier que l’API Klaxoon prend en charge le filtrage des utilisateurs en fonction de cet attribut. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

   |Attribut|Type|Pris en charge pour le filtrage|Requis par Klaxoon|
   |---|---|---|---|
   |userName|String|&check;|&check;|
   |emails[type eq "work"].value|String|&check;|&check;|
   |externalId|String||&check;|
   |name.givenName|String||&check;|
   |name.familyName|String||&check;|
   |active|Boolean||&check;|

1. Dans la section **Mappages**, sélectionnez **Synchroniser les groupes Azure Active Directory avec Klaxoon SAML**.

1. Dans la section **Mappage des attributs**, passez en revue les attributs de groupe qui sont synchronisés entre Azure AD et Klaxoon. Les attributs sélectionnés comme propriétés de **Correspondance** sont utilisés afin de faire correspondre les groupes dans Klaxoon pour les opérations de mise à jour. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

      |Attribut|Type|Pris en charge pour le filtrage|Requis par Klaxoon|
      |---|---|---|---|
      |displayName|String|&check;|&check;
      |externalId|String||&check;
      |membres|Informations de référence||
      |urn:ietf:params:scim:schemas:extension:klaxoon:2.0:Group:license|String||


1. Définissez l’attribut **urn:ietf:params:scim:schemas:extension:klaxoon:2.0:Group:license** pour fournir des licences Klaxoon PRO aux utilisateurs liés à un groupe.

      |Valeur|Groupe avec licence Klaxoon PRO|
      |---|---|
      |true|&check;|
      |false|No|
      |non spécifié (valeur par défaut)|&check;|

1. Pour configurer des filtres d’étendue, reportez-vous aux instructions suivantes fournies dans [Approvisionnement d’applications basé sur les attributs avec filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

1. Pour activer le service d’approvisionnement Azure AD pour Klaxoon SAML, attribuez la valeur **Activé** au paramètre **État de l’approvisionnement** dans la section **Paramètres**.

    ![État d’approvisionnement activé](common/provisioning-toggle-on.png)

1. Définissez les utilisateurs et/ou les groupes que vous souhaitez approvisionner sur Klaxoon SAML en choisissant les valeurs souhaitées sous **Étendue** dans la section **Paramètres**.

    ![Étendue de l’approvisionnement](common/provisioning-scope.png)

1. Lorsque vous êtes prêt à effectuer l’approvisionnement, cliquez sur **Enregistrer**.

    ![Enregistrement de la configuration de l’approvisionnement](common/provisioning-configuration-save.png)

Cette opération démarre le cycle de synchronisation initiale de tous les utilisateurs et groupes définis dans **Étendue** dans la section **Paramètres**. Le cycle de synchronisation initiale prend plus de temps que les cycles de synchronisation suivants, qui se produisent toutes les 40 minutes environ tant que le service de provisionnement Azure AD est en cours d’exécution. 

## <a name="step-6-monitor-your-deployment"></a>Étape 6. Surveiller votre déploiement
Une fois que vous avez configuré l’approvisionnement, utilisez les ressources suivantes pour surveiller votre déploiement :

* Utilisez les [journaux d’approvisionnement](../reports-monitoring/concept-provisioning-logs.md) pour déterminer quels utilisateurs ont été configurés avec succès ou échoué.
* Consultez la [barre de progression](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md) pour afficher l’état du cycle d’approvisionnement et quand il se termine
* Si la configuration de l’approvisionnement semble se trouver dans un état non sain, l’application passe en quarantaine. Pour en savoir plus sur les états de quarantaine, cliquez [ici](../app-provisioning/application-provisioning-quarantine-status.md).  

## <a name="more-resources"></a>Plus de ressources

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)