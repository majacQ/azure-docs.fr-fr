---
title: 'Tutoriel : Configurer Envoy pour l’attribution automatique d’utilisateurs avec Azure Active Directory | Microsoft Docs'
description: Découvrez comment provisionner et déprovisionner automatiquement des comptes d’utilisateur d’Azure AD vers Envoy.
services: active-directory
author: twimmers
writer: twimmers
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 06/28/2021
ms.author: thwimmer
ms.openlocfilehash: f33f9e04781344cd0bb926cecd81f31a2b645bba
ms.sourcegitcommit: 9339c4d47a4c7eb3621b5a31384bb0f504951712
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/14/2021
ms.locfileid: "113763947"
---
# <a name="tutorial-configure-envoy-for-automatic-user-provisioning"></a>Tutoriel : Configurer Envoy pour l’attribution automatique d’utilisateurs

Ce tutoriel décrit les étapes à effectuer dans Envoy et Azure Active Directory (Azure AD) pour configurer le provisionnement automatique d’utilisateurs. Une fois configuré, Azure AD provisionne et déprovisionne automatiquement les utilisateurs et les groupes pour [Envoy](https://envoy.com/pricing/) à l’aide du service Provisionnement Azure AD. Pour découvrir les informations importantes sur ce que fait ce service, comment il fonctionne et consulter le forum aux questions, reportez-vous à l’article [Automatiser l’attribution et l’annulation de l’attribution des utilisateurs dans les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md). 


## <a name="capabilities-supported"></a>Fonctionnalités prises en charge
> [!div class="checklist"]
> * Créer des utilisateurs dans Envoy
> * Supprimer des utilisateurs dans Envoy quand ils ne nécessitent plus d’accès
> * Conserver les attributs utilisateur synchronisés entre Azure AD et Envoy
> * Provisionner des groupes et des appartenances aux groupes dans Envoy
> * [Authentification unique](./envoy-tutorial.md) auprès d’Envoy (recommandé)

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce tutoriel part du principe que vous disposez des prérequis suivants :

* [Un locataire Azure AD](../develop/quickstart-create-new-tenant.md) 
* Un compte d’utilisateur dans Azure AD avec l’[autorisation](../roles/permissions-reference.md) de configurer l’approvisionnement (par exemple, Administrateur d’application, Administrateur d’application cloud, Propriétaire d’application ou Administrateur général). 
* [Un locataire Envoy](https://envoy.com/pricing/).
* un compte d’utilisateur dans Envoy avec des autorisations d’administrateur.

> [!NOTE]
> Cette intégration peut également être utilisée à partir de l’environnement cloud US Government Azure AD. Cette application est disponible dans la Galerie d’applications cloud US Government Azure AD et peut être configurée de la même façon que dans le cloud public.

## <a name="step-1-plan-your-provisioning-deployment"></a>Étape 1. Planifier votre déploiement de l’approvisionnement
1. En savoir plus sur le [fonctionnement du service d’approvisionnement](../app-provisioning/user-provisioning.md).
2. Déterminez qui sera dans l’[étendue pour l’approvisionnement](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).
3. Déterminez les données à [mapper entre Azure AD et Envoy](../app-provisioning/customize-application-attributes.md). 

## <a name="step-2-configure-envoy-to-support-provisioning-with-azure-ad"></a>Étape 2. Configurer Envoy pour prendre en charge le provisionnement avec Azure AD

1. Connectez-vous à la [Console d’administration Envoy](https://dashboard.envoy.com/login). Cliquez sur **Integrations** (Intégrations).

    ![Intégrations d’Envoy](media/envoy-provisioning-tutorial/envoy-01.png)

2. Cliquez sur l’option **Installer** correspondant à **Intégration de Microsoft Azure SCIM**.

    ![Installation d’Envoy](media/envoy-provisioning-tutorial/integrations.png)

3. Cliquez sur **Enregistrer** pour **synchroniser tous les utilisateurs**. 

    ![Enregistrement d’Envoy](media/envoy-provisioning-tutorial/microsoft-azure.png)

4. Copiez le **JETON DU PORTEUR OAUTH**. Vous devrez entrer cette valeur dans le champ **Jeton secret** sous l’onglet Provisionnement de votre application Envoy dans le portail Azure.
    
    ![OAUTH d’Envoy](media/envoy-provisioning-tutorial/token.png)

## <a name="step-3-add-envoy-from-the-azure-ad-application-gallery"></a>Étape 3. Ajouter Envoy à partir de la galerie d’applications Azure AD

Ajoutez Envoy à partir de la galerie d’applications Azure AD pour commencer à gérer le provisionnement pour Envoy. Si vous avez déjà configuré Envoy pour l’authentification unique, vous pouvez utiliser la même application. Toutefois, il est recommandé de créer une application distincte lors du test initial de l’intégration. En savoir plus sur l’ajout d’une application à partir de la galerie [ici](../manage-apps/add-application-portal.md). 

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Étape 4. Définir qui sera dans l’étendue pour l’approvisionnement 

Le service d’approvisionnement Azure AD vous permet de définir l’étendue des utilisateurs approvisionnés en fonction de l’affectation à l’application et/ou en fonction des attributs de l’utilisateur/groupe. Si vous choisissez de définir l’étendue de l’approvisionnement pour votre application en fonction de l’attribution, vous pouvez utiliser les étapes de [suivantes](../manage-apps/assign-user-or-group-access-portal.md) pour affecter des utilisateurs et des groupes à l’application. Si vous choisissez de définir l’étendue de l’approvisionnement en fonction uniquement des attributs de l’utilisateur ou du groupe, vous pouvez utiliser un filtre d’étendue comme décrit [ici](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 

* Quand vous affectez des utilisateurs et des groupes à Envoy, vous devez sélectionner un rôle différent du rôle **Accès par défaut**. Les utilisateurs disposant du rôle Accès par défaut sont exclus de l’approvisionnement et sont marqués comme non autorisés dans les journaux de configuration. Si le seul rôle disponible dans l’application est le rôle d’accès par défaut, vous pouvez [mettre à jour le manifeste de l’application](../develop/howto-add-app-roles-in-azure-ad-apps.md) pour ajouter des rôles supplémentaires. 

* Commencez progressivement. Testez avec un petit ensemble d’utilisateurs et de groupes avant d’effectuer un déploiement général. Lorsque l’étendue de l’approvisionnement est définie sur les utilisateurs et les groupes attribués, vous pouvez contrôler cela en affectant un ou deux utilisateurs ou groupes à l’application. Lorsque l’étendue est définie sur tous les utilisateurs et groupes, vous pouvez spécifier un [filtre d’étendue basé sur l’attribut](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 


## <a name="step-5-configure-automatic-user-provisioning-to-envoy"></a>Étape 5. Configurer le provisionnement d’utilisateurs automatique sur Envoy 

Cette section vous guide tout au long des étapes de configuration du service d’approvisionnement d’Azure AD pour créer, mettre à jour et désactiver des utilisateurs et/ou des groupes dans TestApp en fonction des assignations d’utilisateurs et/ou de groupes dans Azure AD.

### <a name="to-configure-automatic-user-provisioning-for-envoy-in-azure-ad"></a>Pour configurer l’attribution automatique d’utilisateurs pour Envoy dans Azure AD :

1. Connectez-vous au [portail Azure](https://portal.azure.com). Sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Envoy**.

    ![Lien Envoy dans la liste des applications](common/all-applications.png)

3. Sélectionnez l’onglet **Approvisionnement**.

    ![Capture d’écran des options Gérer avec l’option Provisionnement en évidence.](common/provisioning.png)

4. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Capture d’écran de la liste déroulante Mode de provisionnement avec l’option Automatique en évidence.](common/provisioning-automatic.png)

5. Dans la section **Informations d’identification de l’administrateur**, entrez `https://app.envoy.com/scim/v2` dans **URL de locataire**. Entrez la valeur du **JETON DU PORTEUR OAUTH** récupérée précédemment dans **Jeton secret**. Cliquez sur **Tester la connexion** pour vérifier qu’Azure AD peut se connecter à Envoy. Si la connexion échoue, vérifiez que votre compte Envoy dispose des autorisations d’administrateur, puis réessayez.

   ![Capture d’écran de la boîte de dialogue Informations d’identification de l’administrateur permettant d’entrer l’URL du locataire et le jeton secret](./media/envoy-tutorial/provisioning.png)

6. Dans le champ **E-mail de notification**, entrez l’adresse e-mail de la personne ou du groupe qui doit recevoir les notifications d’erreur de provisionnement et sélectionnez la case à cocher **Envoyer une notification par e-mail en cas de défaillance**.

    ![E-mail de notification](common/provisioning-notification-email.png)

7. Sélectionnez **Enregistrer**.

8. Dans la section **Mappages**, sélectionnez **Synchroniser les utilisateurs Azure Active Directory avec Envoy**.

9. Dans la section **Mappage d’attributs**, passez en revue les attributs utilisateur qui sont synchronisés entre Azure AD et Envoy. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les comptes d’utilisateurs dans Envoy pour les opérations de mise à jour. Si vous choisissez de modifier l’[attribut cible correspondant](../app-provisioning/customize-application-attributes.md), vous devez vérifier que l’API Envoy prend en charge le filtrage des utilisateurs en fonction de cet attribut. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

   |Attribut|Type|
   |---|---|
   |userName|String|
   |externalId|String|
   |displayName|String|
   |title|String|
   |emails[type eq "work"].value|String|
   |preferredLanguage|String|
   |department|String|
   |addresses[type eq "work"].country|String|
   |addresses[type eq "work"].locality|String|
   |addresses[type eq "work"].region|String|
   |addresses[type eq "work"].postalCode|String|
   |addresses[type eq "work"].formatted|String|
   |addresses[type eq "work"].streetAddress|String|
   |name.givenName|String|
   |name.familyName|String|
   |name.formatted|String|
   |phoneNumbers[type eq "mobile"].value|String|
   |phoneNumbers[type eq "work"].value|String|
   |locale|String|

10. Dans la section **Mappages**, sélectionnez **Synchroniser les groupes Azure Active Directory avec Envoy**.

11. Dans la section **Mappage d’attributs**, passez en revue les attributs de groupe qui sont synchronisés entre Azure AD et Envoy. Les attributs sélectionnés comme propriétés de **Correspondance** sont utilisés pour faire correspondre les groupes dans Envoy dans le cadre des opérations de mise à jour. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

      |Attribut|Type|
      |---|---|
      |displayName|String|
      |externalId|String|
      |membres|Informations de référence|

12. Pour configurer des filtres d’étendue, reportez-vous aux instructions suivantes fournies dans [Approvisionnement d’applications basé sur les attributs avec filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Pour activer le service d’approvisionnement d’Azure AD pour Envoy, définissez le paramètre **État d’approvisionnement** sur **Activé** dans la section **Paramètres**.

    ![État d’approvisionnement activé](common/provisioning-toggle-on.png)

14. Définissez les utilisateurs ou groupes que vous souhaitez approvisionner dans Envoy en choisissant les valeurs souhaitées dans **Étendue**, dans la section **Paramètres**.

    ![Étendue de l’approvisionnement](common/provisioning-scope.png)

15. Lorsque vous êtes prêt à effectuer l’approvisionnement, cliquez sur **Enregistrer**.

    ![Enregistrement de la configuration de l’approvisionnement](common/provisioning-configuration-save.png)

Cette opération démarre le cycle de synchronisation initiale de tous les utilisateurs et groupes définis dans **Étendue** dans la section **Paramètres**. Le cycle de synchronisation initiale prend plus de temps que les cycles de synchronisation suivants, qui se produisent toutes les 40 minutes environ tant que le service de provisionnement Azure AD est en cours d’exécution. 

## <a name="step-6-monitor-your-deployment"></a>Étape 6. Surveiller votre déploiement
Une fois que vous avez configuré l’approvisionnement, utilisez les ressources suivantes pour surveiller votre déploiement :

* Utilisez les [journaux d’approvisionnement](../reports-monitoring/concept-provisioning-logs.md) pour déterminer quels utilisateurs ont été configurés avec succès ou échoué.
* Consultez la [barre de progression](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md) pour afficher l’état du cycle d’approvisionnement et quand il se termine
* Si la configuration de l’approvisionnement semble se trouver dans un état non sain, l’application passe en quarantaine. Pour en savoir plus sur les états de quarantaine, cliquez [ici](../app-provisioning/application-provisioning-quarantine-status.md).

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)